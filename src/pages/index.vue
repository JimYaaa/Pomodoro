
<template>
  <div class="w-full h-full bg-white flex items-center justify-center">
    <button class="menu-btn cursor-pointer absolute top-10px left-10px">
      <carbon-menu style="font-size: 20px; color: #4b5563" />
    </button>
    <div class="timer absolute top-10px right-10px text-xl font-ubuntu">
      {{ formatMinute }}:{{ formatSecond }}
    </div>
    <div class="content text-gray-600 flex flex-col items-center justify-center ">
      <div class="desc text-lg">
        Some Descriptioin.....
      </div>
      <div class="image-wrap w-120px my-80px">
        <img v-if="isFocus" src="https://camo.githubusercontent.com/3c553beb641d154ec09f3f1cce78f434eb72a9b2843dc45e5aa191cc6234b383/687474703a2f2f7374617469632e76656c76657463616368652e6f72672f70616765732f323031382f30362f31332f70617274792d676f706865722f64616e63696e672d676f706865722e676966" alt="">
        <img v-else src="https://camo.githubusercontent.com/e59bf7afa2dc165bb4b0a6fb84299ccbf12a5f2fb2e2e2e54df51db98c73614a/68747470733a2f2f7261772e6769746875622e636f6d2f676f6c616e672d73616d706c65732f676f706865722d766563746f722f6d61737465722f676f706865722d736964655f636f6c6f722e706e67" alt="">
      </div>
      <button class="focus-btn w-120px h-120px rounded-1/2 bg-primary text-primary font-bold focus:(outline-none)" @click="toggleFocus">
        <span v-if="isFocus">Finish</span>
        <span v-else>Focus</span>
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useStorage } from '@vueuse/core'

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
    state.value[formatDate.value].push('TEST')
    second.value = 0
    minute.value = 0
  }
}
</script>

<route lang="yaml">
meta:
  layout: home
</route>
