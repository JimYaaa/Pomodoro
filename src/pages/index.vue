
<template>
  <div class="w-full h-full bg-white flex items-end justify-center bg-center bg-no-repeat bg-cover" :style="{ 'background-image': `url(${pomodoroBackground})` }">
    <ul class="pomodoro-scoreboard absolute top-10px left-10px flex items-center justify-start flex-wrap">
      <li
        v-for="score in currentScore"
        :key="score"
        class="score w-30px mx-5px my-5px"
      >
        <img :src="pomodoroScore" alt="">
      </li>
    </ul>
    <div class="timer absolute top-10px right-10px text-2xl font-mono text-primary">
      {{ formatMinute }}:{{ formatSecond }}
    </div>
    <div class="pomodoro-gopher absolute" :class="{ buoyant: isFocus }">
      <img :src="PomodoroGopher" alt="">
    </div>
    <div class="pomodoro-pooh relative w-full h-200px bg-center bg-no-repeat bg-cover" :style="{ 'background-image': `url(${pomodoroPooh})` }">
      <button class="pomodoro-pooh__start-btn focus:(outline-none) font-ubuntu" @click="toggleFocus">
        <span v-if="isFocus">Stop</span>
        <span v-else>Start</span>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useStorage } from '@vueuse/core'
import PomodoroBackground from '~/assets/pomodoro-background.png'
import PomodoroPooh from '~/assets/pomodoro-pooh.png'
import PomodoroGopher from '~/assets/gopher-alien.png'
import PomodoroScore from '~/assets/pomodoro-ufo.png'

const pomodoroBackground: string = PomodoroBackground
const pomodoroPooh: string = PomodoroPooh
const pomodoroGopher: string = PomodoroGopher
const pomodoroScore: string = PomodoroScore
const isFocus = ref(false)
const start = ref(0)
const second = ref<number>(0)
const minute = ref<number>(0)
const state = ref<any>(useStorage('focusData', {}))
const month = ref([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12])

const formatSecond = computed(() => {
  if (second.value <= 9)
    return `0${second.value}`
  else
    return `${second.value}`
})

const formatMinute = computed(() => {
  if (minute.value <= 9)
    return `0${minute.value}`
  else
    return `${minute.value}`
})

const formatDate = computed(() => {
  const date = new Date()

  return `${month.value[date.getMonth()]}-${date.getDate()}`
})

const currentScore = computed(() => {
  return state.value[formatDate.value] || []
})

const step = (timestamp: number) => {
  if (!isFocus.value) return
  const progress = timestamp - start.value

  if (progress >= 1000) {
    start.value = timestamp

    if (second.value < 59) {
      second.value++
    }
    else {
      second.value = 0
      minute.value++
    }
  }
  window.requestAnimationFrame(step)
}

const toggleFocus = () => {
  isFocus.value = !isFocus.value

  if (isFocus.value) { window.requestAnimationFrame(step) }
  else {
    if (!state.value[formatDate.value]) state.value[formatDate.value] = []
    state.value[formatDate.value].push(`${formatMinute.value}:${formatSecond.value}`)
    second.value = 0
    minute.value = 0
  }
}
</script>

<style lang="scss" scoped>
@keyframes buoyant {
  0% {
    transform: translate(-50%, 0px);
  }
  50% {
    transform: translate(-50%, -10px);
  }
  100% {
    transform: translate(-50%, 0px);
  }
}

.pomodoro-pooh {
  &__start-btn {
    position: absolute;
    top: 30px;
    left: 50%;
    font-size: 50px;
    transform: translateX(-50%);
  }
}

.pomodoro-gopher {
  position: absolute;
  bottom: 220px;
  left: 50%;
  width: 210px;
  transform: translateX(-50%);

  &.buoyant {
    animation: buoyant 3s ease-in-out infinite;
  }
}

.pomodoro-scoreboard {
  width: calc(40px * 5);
}
</style>

<route lang="yaml">
meta:
  layout: home
</route>
