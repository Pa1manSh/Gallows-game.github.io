<template>
  <div v-show="isVisible" class="popup-container">
    <div class="popup">
      <h2 v-if="gameStatus === 'win'">Поздравляю, вы победили! 😃</h2>
      <template v-else>
        <h2>Вы проиграли. 😕</h2>

        <h3>...имя: {{ word }}</h3>
      </template>
      <button @click="emit('restart')">Сыграть еще раз</button>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'
interface Props {
  word: string
}
defineProps<Props>()
type Status = 'win' | 'lose'
const gameStatus = ref<Status | null>(null)
const isVisible = ref(false)
const open = (status: Status) => {
  isVisible.value = true
  gameStatus.value = status
}
const close = () => {
  isVisible.value = false
}
defineExpose({
  open,
  close
})
const emit = defineEmits<{
  (e: 'restart'): void
}>()
</script>
