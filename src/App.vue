<script setup lang="ts">
import GameFigure from './components/GameFigure.vue'
import GameHeader from './components/GameHeader.vue'
import GamePopup from '@/components/GamePopup.vue'
import GameWord from './components/GameWord.vue'
import GameWrongLetters from './components/GameWrongLetters.vue'
import GameNotification from '@/components/GameNotification.vue'
import { ref, computed, onMounted, watch } from 'vue'
import axios from 'axios'
// import { watch } from 'fs'
const word = ref('')
const getRandomName = async () => {
  try {
    const { data } = await axios<{ FirstName: string }>(
      'https://api.randomdatatools.ru/?unescaped=false&params=FirstName'
    )
    word.value = data.FirstName.toLowerCase()
  } catch (err) {
    console.log(err)
    word.value = ''
  }
}
getRandomName()
const letters = ref<string[]>([])
const correctLetters = computed(() => letters.value.filter((x) => word.value.includes(x)))
const wrongLetters = computed(() => letters.value.filter((x) => !word.value.includes(x)))
const notification = ref<InstanceType<typeof GameNotification> | null>(null)
const popup = ref<InstanceType<typeof GamePopup> | null>(null)
const isSatusLose = computed(() => wrongLetters.value.length === 6)
const isStatusWin = computed(() => [...word.value].every((x) => correctLetters.value.includes(x)))
watch(wrongLetters, () => {
  if (isSatusLose.value) {
    popup.value?.open('lose')
  }
})
watch(correctLetters, () => {
  if (isStatusWin.value) {
    popup.value?.open('win')
  }
})

window.addEventListener('keydown', ({ key }) => {
  if (isSatusLose.value || isStatusWin.value) {
    return
  }
  if (letters.value.includes(key)) {
    notification.value?.open()
    setTimeout(() => notification.value?.close(), 2000)
    return
  }
  if (/[а-яА-ЯёЁ]/.test(key)) {
    letters.value.push(key.toLowerCase())
  }
})
const restart = async () => {
  await getRandomName()
  letters.value = []
  popup.value?.close()
}
</script>

<template>
  <div>
    <GameHeader />
    <div class="game-container">
      <GameFigure :wrong-letters-count="wrongLetters.length" />

      <GameWrongLetters :wrong-letters="wrongLetters" />

      <GameWord :word="word" :correct-letters="correctLetters" />
    </div>

    <!-- Container for final message -->
    <GamePopup ref="popup" :word="word" @restart="restart" />
    <!-- Notification -->
    <GameNotification ref="notification" />
  </div>
</template>
