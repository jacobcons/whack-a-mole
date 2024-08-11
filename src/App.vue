<script setup lang="ts">
import {onMounted, reactive, ref, watch} from 'vue';

const ROUND_LENGTH_SECONDS = 5
const timeLeftSeconds = ref(ROUND_LENGTH_SECONDS)
const started = ref(false)
const ROWS = 3
const COLS = 3
const moles = ref(generateNoMolesArray())
const score = ref(0)
const highScores = ref<number[]>(JSON.parse(localStorage.getItem('highScores') || '[]'))
const MAX_HIGH_SCORES = 5
const isAudioMuted = ref(true)

function generateNoMolesArray() {
  const noMolesArray = []
  let moleRow = []
  for (let i = 0; i < ROWS; i++) {
    for (let j = 0; j < COLS; j++) {
      moleRow.push(false)
    }
    noMolesArray.push([...moleRow])
    moleRow = []
  }
  return noMolesArray
}

let timerIntervalId: number
function startGame() {
  // reset values
  stopGame()
  started.value = true
  score.value = 0
  timeLeftSeconds.value = ROUND_LENGTH_SECONDS

  // start timer
  timerIntervalId = setInterval(() => {
    timeLeftSeconds.value -= 1
    if (timeLeftSeconds.value === 0) {
      started.value = false
      stopGame()
      insertScoreIntoHighScores()
    }
  }, 1000)

  // start game loop
  gameLoop()
}

function stopGame() {
  // stop timer ticking down
  if (timerIntervalId) {
    clearInterval(timerIntervalId)
  }
  // stop moles popping in and out
  if (moleTimeoutId) {
    clearInterval(moleTimeoutId)
  }
  // reset moles to be empty
  moles.value.forEach(row => row.fill(false))
}

function insertScoreIntoHighScores() {
  let i = 0
  while (highScores.value[i] && score.value < highScores.value[i]) {
    i += 1
  }
  highScores.value.splice(i, 0, score.value)
  highScores.value = highScores.value.slice(0, MAX_HIGH_SCORES)
  localStorage.setItem('highScores', JSON.stringify(highScores.value))
}

let moleTimeoutId: number
function gameLoop() {
  const randomRow = getRandomInt(ROWS)
  const randomCol = getRandomInt(COLS)

  // show random mole for a bit
  moles.value[randomRow][randomCol] = true
  moleTimeoutId = setTimeout(() => {
    // hide that mole for a bit
    moles.value[randomRow][randomCol] = false
    moleTimeoutId = setTimeout(gameLoop, getRandomInt(500, 250))
  }, getRandomInt(1000, 250))
}

function getRandomInt(max: number, min = 0) {
  return Math.floor(Math.random() * (max - min) + min);
}

function moleBg(mole: boolean) {
  return mole ? '/mole.webp' : '/empty.webp';
}

const bonkAudio = new Audio('/bonk.m4a')
bonkAudio.volume = 0
const buzzAudio = new Audio('/buzz.m4a')
buzzAudio.volume = 0
function handleMoleClick(mole: boolean, row: number, col: number) {
  if (started.value) {
    setTimeout(() => moles.value[row][col] = false, 50)
    if (mole) {
      score.value += 1
      bonkAudio.currentTime = 0
      bonkAudio.play()
    } else {
      score.value -= 1
      buzzAudio.currentTime = 0
      buzzAudio.play()
    }
  }
}

let marioAudio: HTMLAudioElement
onMounted(() => {
  marioAudio = new Audio('/mario.m4a')
  marioAudio.loop = true
})

function setAudioVolume(volume: number) {
  marioAudio.volume = volume
  buzzAudio.volume = volume
  bonkAudio.volume = volume
}

function flipAudio() {
  isAudioMuted.value = !isAudioMuted.value
  if (isAudioMuted.value) {
    setAudioVolume(0)
  } else {
    if (marioAudio.paused) {
      marioAudio.play()
    }
    setAudioVolume(1)
  }
}
</script>

<template>
  <div class="flex flex-col items-center gap-y-8">
    <h1 class="text-5xl font-bold mt-16">Whack a mole</h1>

    <div class="flex justify-center items-center gap-x-16">
      <span class="text-2xl font-bold">Score: {{ score }}</span>
      <span class="text-2xl font-bold">{{ timeLeftSeconds }}s</span>
      <div class="cursor-pointer" @click="flipAudio">
        <svg v-if="isAudioMuted" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round" d="M17.25 9.75 19.5 12m0 0 2.25 2.25M19.5 12l2.25-2.25M19.5 12l-2.25 2.25m-10.5-6 4.72-4.72a.75.75 0 0 1 1.28.53v15.88a.75.75 0 0 1-1.28.53l-4.72-4.72H4.51c-.88 0-1.704-.507-1.938-1.354A9.009 9.009 0 0 1 2.25 12c0-.83.112-1.633.322-2.396C2.806 8.756 3.63 8.25 4.51 8.25H6.75Z" />
        </svg>
        <svg v-if="!isAudioMuted" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round" d="M19.114 5.636a9 9 0 0 1 0 12.728M16.463 8.288a5.25 5.25 0 0 1 0 7.424M6.75 8.25l4.72-4.72a.75.75 0 0 1 1.28.53v15.88a.75.75 0 0 1-1.28.53l-4.72-4.72H4.51c-.88 0-1.704-.507-1.938-1.354A9.009 9.009 0 0 1 2.25 12c0-.83.112-1.633.322-2.396C2.806 8.756 3.63 8.25 4.51 8.25H6.75Z" />
        </svg>
      </div>
    </div>

    <button class="mx-auto block text-gray-900 bg-white border border-gray-300 focus:outline-none hover:bg-gray-100 focus:ring-4 focus:ring-gray-100 font-medium rounded-lg text-sm px-5 py-2.5 dark:bg-gray-800 dark:text-white dark:border-gray-600 dark:hover:bg-gray-700 dark:hover:border-gray-600 dark:focus:ring-gray-700" @click="startGame">{{ started ? 'Reset' : 'Start'}}</button>

    <div class="relative flex flex-col items-center gap-y-8 xl:block">
      <div class="grid grid-cols-3 gap-4 max-w-xl mx-auto">
        <template v-for="(moleRow, rowIndex) in moles">
          <img v-for="(mole, colIndex) in moleRow" :src="moleBg(mole)" @mousedown="handleMoleClick(mole, rowIndex, colIndex)" class="cursor-pointer" draggable="false">
        </template>
      </div>
      <a href="#" class="static translate-y-0 translate-x-0 xl:absolute top-1/2 xl:-translate-y-2/4 right-[-24px] xl:translate-x-full block max-w-sm p-6 bg-white border border-gray-200 rounded-lg shadow hover:bg-gray-100 dark:bg-gray-800 dark:border-gray-700 dark:hover:bg-gray-700">
        <h5 class="mb-2 text-2xl font-bold tracking-tight text-gray-900 dark:text-white text-center">High Scores</h5>
        <ol class="max-w-md space-y-1 text-gray-500 list-decimal list-inside dark:text-gray-400">
          <li v-for="i in MAX_HIGH_SCORES" class="font-semibold text-gray-900 dark:text-white marker:font-normal">{{ highScores[i-1] }}</li>
        </ol>
      </a>
    </div>
  </div>
</template>

