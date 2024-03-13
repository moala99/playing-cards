<script setup>
import { ref, onMounted } from 'vue'

const deckId = ref(null)
const remainingCards = ref(null)
const currentCard = ref(null)
const currentCardValue = ref(null)
const nextCardValue = ref(null)

const didPlayerWin = ref(null)
const isDraw = ref(false)

const previousPile = ref(null)

const faceCardValues = {
  JACK: 12,
  QUEEN: 13,
  KING: 14,
  ACE: 15
}

async function getDeck() {
  return new Promise((resolve, reject) => {
    fetch('https://deckofcardsapi.com/api/deck/new/shuffle/?deck_count=1')
      .then((response) => response.json())
      .then((data) => {
        resolve(data)
      })
      .catch((error) => {
        reject(error)
      })
  })
}

async function getNextCard(deckId) {
  return new Promise((resolve, reject) => {
    fetch(`https://deckofcardsapi.com/api/deck/${deckId}/draw/?count=1`)
      .then((response) => response.json())
      .then((data) => {
        resolve(data)
      })
      .catch((error) => {
        reject(error)
      })
  })
}

async function handlePlayerPrediction(isNextCardHigherPrediction) {
  const nextCardData = await getNextCard(deckId.value)
  nextCardValue.value = nextCardData.cards[0].value

  if (nextCardValue.value in faceCardValues) {
    nextCardValue.value = faceCardValues[nextCardValue.value]
  }

  console.log('nextCard Value:', nextCardValue.value)
  console.log('previousCard:', currentCardValue.value)

  let isNextDrawnCardHigher

  isNextDrawnCardHigher = Number(nextCardValue.value) > Number(currentCardValue.value)

  didPlayerWin.value = isNextCardHigherPrediction === isNextDrawnCardHigher
  isDraw.value = Number(nextCardValue.value) === Number(currentCardValue.value)

  currentCard.value = nextCardData.cards[0]
  currentCardValue.value = nextCardData.cards[0].value

  if (currentCardValue.value in faceCardValues) {
    currentCardValue.value = faceCardValues[currentCardValue.value]
  }

  remainingCards.value = nextCardData.remaining

  if (remainingCards.value === 0) {
    const deckData = await getDeck()
    deckId.value = deckData.deck_id
    remainingCards.value = deckData.remaining
  }

  await addPreviousCardToPile(currentCard.value.code)
  const pileData = await getPile()
  previousPile.value = pileData.piles.previousCardPile.cards
}

async function addPreviousCardToPile(cards) {
  return new Promise((resolve, reject) => {
    fetch(
      `https://deckofcardsapi.com/api/deck/${deckId.value}/pile/previousCardPile/add/?cards=${cards}`
    )
      .then((response) => response.json())
      .then((data) => {
        resolve(data)
      })
      .catch((error) => {
        reject(error)
      })
  })
}

async function getPile() {
  return new Promise((resolve, reject) => {
    fetch(`https://deckofcardsapi.com/api/deck/${deckId.value}/pile/previousCardPile/list`)
      .then((response) => response.json())
      .then((data) => {
        resolve(data)
      })
      .catch((error) => {
        reject(error)
      })
  })
}

onMounted(async () => {
  try {
    const deckData = await getDeck()
    deckId.value = deckData.deck_id
    remainingCards.value = deckData.remaining
    const currentCardData = await getNextCard(deckData.deck_id)
    currentCard.value = currentCardData.cards[0]
    currentCardValue.value = currentCardData.cards[0].value

    if (currentCardValue.value in faceCardValues) {
      currentCardValue.value = faceCardValues[currentCardValue.value]
    }
  } catch (error) {
    console.error('Error fetching deck:', error)
  }
})
</script>

<template>
  <div class="container mx-auto flex flex-col items-center justify-center space-y-10">
    <div v-if="!isDraw" class="mt-8">
      <h2 v-if="didPlayerWin == null" class="text-2xl font-bold">
        Will the next card be higher or lower? Take a guess
      </h2>
      <h2 v-else-if="didPlayerWin" class="text-2xl font-bold text-success">You won! 🎉</h2>
      <h2 v-else class="text-2xl font-bold text-error">You lost! 😢</h2>
    </div>
    <div v-else class="mt-8">
      <h2 class="text-2xl font-bold">It's a draw! 🤷‍♂️</h2>
    </div>

    <div v-if="currentCard" class="card bg-base-100 shadow-xl">
      <figure class="px-10 pt-10">
        <img :src="currentCard.image" :alt="currentCard.code" class="rounded-xl" />
        <img
          src="https://deckofcardsapi.com/static/img/back.png"
          :alt="currentCard.code"
          class="rounded-xl"
        />
      </figure>
      <div class="card-body items-center text-center">
        <h2 class="card-title">{{ currentCard.value }} {{ currentCard.suit }}</h2>
        <p>Will the next be higher o lower? 🤔</p>
        <div class="card-actions">
          <button class="btn btn-primary btn-success" @click="handlePlayerPrediction(true)">
            Higher
          </button>
          <button class="btn btn-primary btn-error" @click="handlePlayerPrediction(false)">
            Lower
          </button>
        </div>
      </div>
    </div>
  </div>
  <ul>
    <li v-for="card in previousPile" :key="card.code">
      <img :src="card.image" :alt="card.code" />
    </li>
  </ul>
</template>
