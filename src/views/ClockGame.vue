<script setup lang="ts">
import { ref, computed } from 'vue'
import { TresCanvas } from '@tresjs/core'
import { OrbitControls } from '@tresjs/cientos'
import Clock3D from '../components/Clock3D.vue'

const score = ref(0)
const feedback = ref('')

// Game modes
type GameMode = 'hour' | 'minute'
const currentMode = ref<GameMode>('hour')

// State for controlling the manual target time override
const isManualTarget = ref(false)

// State for showing/hiding the manual time control panel to prevent cheating
const showTimeControls = ref(false)

// Generate a random time based on mode
const generateRandomTime = () => {
  const h = Math.floor(Math.random() * 12) + 1
  let m = 0
  
  if (currentMode.value === 'hour') {
    // Hour mode: strictly full hours (00 minutes)
    m = 0
  } else {
    // Minute mode: 00, 15, 30, 45 minutes
    const mOptions = [0, 15, 30, 45]
    m = mOptions[Math.floor(Math.random() * mOptions.length)]
  }
  
  return { h, m }
}

const targetTime = ref(generateRandomTime())

const switchMode = (mode: GameMode) => {
  currentMode.value = mode
  isManualTarget.value = false
  targetTime.value = generateRandomTime()
  feedback.value = ''
}

// Time control functions
const adjustTime = (type: 'hour' | 'minute', amount: number) => {
  isManualTarget.value = true
  feedback.value = ''
  
  let newH = targetTime.value.h
  let newM = targetTime.value.m

  if (type === 'hour') {
    newH += amount
    if (newH > 12) newH -= 12
    if (newH < 1) newH += 12
  } else if (type === 'minute') {
    newM += amount
    if (newM >= 60) {
      newM -= 60
      newH += 1
      if (newH > 12) newH -= 12
    }
    if (newM < 0) {
      newM += 60
      newH -= 1
      if (newH < 1) newH += 12
    }
  }
  
  targetTime.value = { h: newH, m: newM }
}

// Generate options for the multiple-choice buttons
const options = computed(() => {
  const opts = new Set<string>()
  const correctStr = `${targetTime.value.h}:${targetTime.value.m.toString().padStart(2, '0')}`
  opts.add(correctStr)

  while (opts.size < 3) {
    const fakeH = Math.floor(Math.random() * 12) + 1
    let fakeM = 0
    
    if (currentMode.value === 'minute') {
      const mOptions = [0, 15, 30, 45]
      fakeM = mOptions[Math.floor(Math.random() * mOptions.length)]
    }
    
    opts.add(`${fakeH}:${fakeM.toString().padStart(2, '0')}`)
  }

  // Shuffle options
  return Array.from(opts).sort(() => Math.random() - 0.5)
})

const checkAnswer = (opt: string) => {
  const correctStr = `${targetTime.value.h}:${targetTime.value.m.toString().padStart(2, '0')}`
  if (opt === correctStr) {
    feedback.value = '🎉 答对啦！太棒了！'
    score.value++
    setTimeout(() => {
      isManualTarget.value = false
      targetTime.value = generateRandomTime()
      feedback.value = ''
    }, 1500)
  } else {
    feedback.value = '🤔 不太对哦，再想想？'
  }
}
</script>

<template>
  <div class="flex-1 flex flex-col relative h-full w-full">
    <!-- Header -->
    <header class="p-6 text-center shadow-sm bg-white rounded-b-3xl z-10 relative">
      <h1 class="text-4xl font-extrabold text-blue-600 mb-4">认识时间 3D 游戏</h1>
      
      <!-- Mode Switcher -->
      <div class="flex justify-center gap-4 mb-2">
        <button 
          @click="switchMode('hour')"
          class="px-6 py-2 rounded-full font-bold transition-all duration-200"
          :class="currentMode === 'hour' ? 'bg-blue-500 text-white shadow-md' : 'bg-gray-100 text-gray-500 hover:bg-gray-200'"
        >
          整点模式
        </button>
        <button 
          @click="switchMode('minute')"
          class="px-6 py-2 rounded-full font-bold transition-all duration-200"
          :class="currentMode === 'minute' ? 'bg-green-500 text-white shadow-md' : 'bg-gray-100 text-gray-500 hover:bg-gray-200'"
        >
          分钟模式
        </button>
      </div>

      <div class="absolute top-6 right-6 bg-yellow-400 text-yellow-900 font-bold px-4 py-2 rounded-full shadow-md text-xl">
        得分: {{ score }}
      </div>
    </header>

    <!-- Main Game Area -->
    <main class="flex-1 flex flex-col lg:flex-row relative">
      <!-- 3D Canvas Container -->
      <div class="flex-1 relative min-h-[50vh] w-full h-full bg-slate-50">
        <TresCanvas alpha shadows>
          <TresPerspectiveCamera :position="[0, 0, 14]" :look-at="[0, 0, 0]" />
          <OrbitControls 
            :enable-pan="false" 
            :enable-zoom="false"
            :min-polar-angle="Math.PI / 3.5"
            :max-polar-angle="Math.PI / 1.5"
            :min-azimuth-angle="-Math.PI / 3"
            :max-azimuth-angle="Math.PI / 3"
          />
          
          <TresAmbientLight :intensity="1.5" color="#ffffff" />
          <TresDirectionalLight :position="[5, 10, 8]" :intensity="1.8" cast-shadow />
          <TresDirectionalLight :position="[-5, 5, -5]" :intensity="0.8" color="#e2e8f0" />

          <!-- The 3D Clock -->
          <Clock3D :hour="targetTime.h" :minute="targetTime.m" :mode="currentMode" />
        </TresCanvas>
      </div>

      <!-- Controls Panel -->
      <div class="lg:w-1/3 p-4 lg:p-8 flex flex-col justify-start items-center bg-white/50 backdrop-blur-sm border-l border-white/50 z-10 shadow-[0_-10px_40px_-15px_rgba(0,0,0,0.1)] lg:shadow-none overflow-y-auto">
        <div class="h-12 mb-2 flex items-center justify-center">
          <p v-if="feedback" class="text-xl font-bold text-center animate-bounce" :class="feedback.includes('对') ? 'text-green-500' : 'text-orange-500'">
            {{ feedback }}
          </p>
        </div>

        <!-- Manual Time Controls -->
        <div class="w-full max-w-sm mb-4">
          <button 
            @click="showTimeControls = !showTimeControls" 
            class="w-full text-center text-sm font-bold text-gray-400 mb-2 hover:text-gray-600 transition-colors flex items-center justify-center gap-1"
          >
            <span v-if="!showTimeControls">⚙️ 开启手动调时</span>
            <span v-else>🔒 关闭调时 (防作弊)</span>
          </button>
          
          <div v-if="showTimeControls" class="bg-white/90 rounded-3xl shadow-sm border border-gray-100 p-4 transition-all animate-fade-in">
            <div class="flex flex-col gap-2">
              <div class="flex items-center gap-3 justify-center">
                <span class="text-sm font-bold text-gray-400 w-8 text-right">小时</span>
                <button @click="adjustTime('hour', -1)" class="w-10 h-10 flex items-center justify-center rounded-full bg-blue-100 text-blue-600 hover:bg-blue-200 hover:scale-110 active:scale-95 transition-all font-bold text-xl">-</button>
                <span class="text-2xl font-black text-slate-700 w-8 text-center">{{ targetTime.h }}</span>
                <button @click="adjustTime('hour', 1)" class="w-10 h-10 flex items-center justify-center rounded-full bg-blue-100 text-blue-600 hover:bg-blue-200 hover:scale-110 active:scale-95 transition-all font-bold text-xl">+</button>
              </div>
              
              <div class="flex items-center gap-3 justify-center" :class="{'opacity-40 grayscale pointer-events-none': currentMode === 'hour'}">
                <span class="text-sm font-bold text-gray-400 w-8 text-right">分钟</span>
                <button @click="adjustTime('minute', -15)" class="w-10 h-10 flex items-center justify-center rounded-full bg-green-100 text-green-600 hover:bg-green-200 hover:scale-110 active:scale-95 transition-all font-bold text-xl">-</button>
                <span class="text-2xl font-black text-slate-700 w-8 text-center">{{ targetTime.m.toString().padStart(2, '0') }}</span>
                <button @click="adjustTime('minute', 15)" class="w-10 h-10 flex items-center justify-center rounded-full bg-green-100 text-green-600 hover:bg-green-200 hover:scale-110 active:scale-95 transition-all font-bold text-xl">+</button>
              </div>
            </div>
          </div>
        </div>

        <div class="w-full max-w-sm space-y-3">
          <p class="text-center text-sm font-bold text-gray-400 mb-1">猜猜现在是几点？</p>
          <button
            v-for="opt in options"
            :key="opt"
            @click="checkAnswer(opt)"
            class="w-full py-3 text-2xl font-bold text-white bg-blue-500 rounded-2xl shadow-[0_6px_0_#2563eb] hover:bg-blue-400 hover:translate-y-[2px] hover:shadow-[0_4px_0_#2563eb] active:translate-y-[6px] active:shadow-none transition-all duration-150"
          >
            {{ opt }}
          </button>
        </div>
      </div>
    </main>
  </div>
</template>
