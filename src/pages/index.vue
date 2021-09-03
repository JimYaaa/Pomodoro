
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
      <span v-if="!isTimeup" class="standard-time">{{ formatMinute }}:{{ formatSecond }}</span>
      <span v-else class="over-time text-waring">{{ formatOverTimeMinute }}:{{ formatOverTimeSecond }}</span>
    </div>
    <div class="pomodoro-gopher absolute" :class="{ buoyant: pomodoroState.mode === FOCUS }">
      <img :src="PomodoroGopher" alt="">
    </div>
    <div class="pomodoro-pooh relative w-full h-200px bg-center bg-no-repeat bg-cover" :style="{ 'background-image': `url(${pomodoroPooh})` }">
      <button class="pomodoro-pooh__start-btn focus:(outline-none) font-ubuntu" @click="startFocus">
        <span v-if="pomodoroState.mode !== STOP && !isTimeup" @click.stop="stopFocus">Stop</span>
        <span v-else-if="isTimeup && nextMode === BREAK" @click.stop="startBreak">Break</span>
        <span v-else @click.stop="startFocus">Start</span>
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
import { FOCUS, BREAK, STOP } from '~/constant'

interface IPomodoroState {
  mode: typeof STOP | typeof FOCUS | typeof BREAK
  goal: number
  [FOCUS]: {
    minute: number
    second: number
    overMinute: number
    overSecond: number
  }
  [BREAK]: {
    minute: number
    second: number
    overMinute: number
    overSecond: number
  }
}

const pomodoroBackground: string = PomodoroBackground
const pomodoroPooh: string = PomodoroPooh
const pomodoroGopher: string = PomodoroGopher
const pomodoroScore: string = PomodoroScore
const pomodoroState = ref<IPomodoroState>({
  mode: STOP,
  goal: 30,
  [FOCUS]: {
    minute: 0,
    second: 0,
    overMinute: 0,
    overSecond: 0,
  },
  [BREAK]: {
    minute: 0,
    second: 0,
    overMinute: 0,
    overSecond: 0,
  },
})
const start = ref(0)
const second = ref<number>(10)
const minute = ref<number>(0)
const overTimeSecond = ref(0)
const overTimeMinute = ref(0)
const state = ref<any>(useStorage('focusData', {}))
const month = ref([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12])
const noticeMessage = ref({
  [BREAK]: 'Your Break Over Time!!!! Is Time To Focus',
  [FOCUS]: 'Your Focus Over Time!!!! Is Time To Break',
})

const formatTime = (time: number) => {
  if (time <= 9)
    return `0${time}`
  else
    return `${time}`
}

const formatSecond = computed(() => {
  return formatTime(second.value)
})
const formatMinute = computed(() => {
  return formatTime(minute.value)
})
const formatOverTimeSecond = computed(() => {
  return formatTime(overTimeSecond.value)
})
const formatOverTimeMinute = computed(() => {
  return formatTime(overTimeMinute.value)
})
const formatDate = computed(() => {
  const date = new Date()

  return `${month.value[date.getMonth()]}-${date.getDate()}`
})
const currentScore = computed(() => {
  return state.value[formatDate.value] || []
})
const nextMode = computed(() => {
  switch (pomodoroState.value.mode) {
    case FOCUS:
      return BREAK
    case BREAK:
      return FOCUS
    default:
      return STOP
  }
})
const isTimeup = computed(() => {
  return !second.value && !minute.value
})

const step = (timestamp: number) => {
  if (pomodoroState.value.mode === STOP) return
  const progress = timestamp - start.value

  if (progress >= 1000) {
    start.value = timestamp

    switch (true) {
      case isTimeup.value:
        if (overTimeSecond.value % 10 === 0) alert(`${noticeMessage.value[pomodoroState.value.mode]}`)
        if (overTimeSecond.value >= 59) {
          overTimeSecond.value = 0
          overTimeMinute.value++
        }
        else {
          overTimeSecond.value++
        }
        break
      default:
        if (second.value <= 0) {
          second.value = 59
          minute.value--
        }
        else {
          second.value--
        }
    }
  }
  window.requestAnimationFrame(step)
}

const startFocus = () => {
  if (pomodoroState.value.focus.minute + pomodoroState.value.focus.second) {
    pomodoroState.value.focus.minute = 0
    pomodoroState.value.focus.second = 5
    pomodoroState.value.focus.overMinute = overTimeMinute.value
    pomodoroState.value.focus.overSecond = overTimeSecond.value

    if (!state.value[formatDate.value]) state.value[formatDate.value] = []
    state.value[formatDate.value].push(pomodoroState.value)
  }

  pomodoroState.value.mode = FOCUS
  second.value = 10
  minute.value = 0
  overTimeSecond.value = 0
  overTimeMinute.value = 0
  window.requestAnimationFrame(step)
}

const stopFocus = () => {
  pomodoroState.value.mode = STOP
  second.value = 10
  minute.value = 0
}

const startBreak = () => {
  pomodoroState.value.focus.minute = 0
  pomodoroState.value.focus.second = 10
  pomodoroState.value.focus.overMinute = overTimeMinute.value
  pomodoroState.value.focus.overSecond = overTimeSecond.value

  pomodoroState.value.mode = BREAK
  second.value = 5
  minute.value = 0
  overTimeSecond.value = 0
  overTimeMinute.value = 0
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
    animation: buoyant 2s ease-in-out infinite;
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
