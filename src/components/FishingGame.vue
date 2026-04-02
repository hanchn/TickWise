<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'
import { TresCanvas, useRenderLoop } from '@tresjs/core'
import { OrbitControls } from '@tresjs/cientos'

// State
const score = ref(0)
const feedback = ref('')

// Fishes
interface Fish {
  id: number
  x: number
  y: number
  z: number
  speed: number
  color: string
  scale: number
  direction: number // 1 (moving right) or -1 (moving left)
}

const fishes = ref<Fish[]>([])
let fishIdCounter = 0
const colors = ['#ef4444', '#f59e0b', '#10b981', '#3b82f6', '#8b5cf6', '#ec4899', '#f43f5e', '#ffedd5', '#14b8a6']

const spawnFish = () => {
  if (fishes.value.length > 12) return // Max 12 fishes
  
  const direction = Math.random() > 0.5 ? 1 : -1
  const x = direction === 1 ? -25 : 25
  const y = Math.random() * 4 + 1 // Height above water (water is at 0)
  const z = (Math.random() - 0.5) * 20 // Depth
  const speed = (Math.random() * 0.08 + 0.04) * direction
  const color = colors[Math.floor(Math.random() * colors.length)]
  const scale = Math.random() * 0.5 + 0.6

  fishes.value.push({
    id: fishIdCounter++,
    x, y, z, speed, color, scale, direction
  })
}

const catchFish = (id: number) => {
  const index = fishes.value.findIndex(f => f.id === id)
  if (index !== -1) {
    fishes.value.splice(index, 1)
    score.value++
    feedback.value = '🎣 抓到啦！'
    setTimeout(() => { feedback.value = '' }, 1000)
  }
}

// Animation Loop
const { onLoop } = useRenderLoop()

onLoop(() => {
  // Move fishes
  fishes.value.forEach(fish => {
    fish.x += fish.speed
    // small bobbing effect
    fish.y += Math.sin(Date.now() / 200 + fish.id) * 0.01
  })
  
  // Remove out of bounds fishes
  fishes.value = fishes.value.filter(fish => fish.x > -30 && fish.x < 30)
})

let interval: number
onMounted(() => {
  interval = window.setInterval(spawnFish, 1000)
  
  // Spawn a few initial fishes scattered around
  for(let i = 0; i < 8; i++) {
    spawnFish()
    if (fishes.value[i]) {
      fishes.value[i].x = (Math.random() - 0.5) * 30
    }
  }
})

onUnmounted(() => {
  clearInterval(interval)
})
</script>

<template>
  <div class="w-full h-full relative bg-sky-200 overflow-hidden flex flex-col">
    <!-- UI Overlay -->
    <div class="absolute top-4 left-4 md:top-6 md:left-6 z-10 pointer-events-none">
      <h2 class="text-3xl md:text-4xl font-extrabold text-blue-600 drop-shadow-md">3D 钓鱼游戏 🎣</h2>
      <p class="text-blue-800 font-bold mt-2 text-base md:text-xl bg-white/60 px-4 py-2 rounded-xl inline-block backdrop-blur-sm border border-white/50 shadow-sm">
        点击游动的小鱼，把它们抓起来！
      </p>
    </div>
    
    <div class="absolute top-4 right-4 md:top-6 md:right-6 z-10 bg-yellow-400 text-yellow-900 font-black px-6 py-3 rounded-full shadow-lg text-2xl md:text-3xl border-4 border-white pointer-events-none">
      得分: {{ score }}
    </div>

    <!-- Feedback animation -->
    <div class="absolute inset-0 flex items-center justify-center pointer-events-none z-20">
      <Transition name="bounce">
        <div v-if="feedback" class="text-5xl md:text-7xl font-black text-green-500 drop-shadow-[0_4px_4px_rgba(0,0,0,0.3)]">
          {{ feedback }}
        </div>
      </Transition>
    </div>

    <!-- 3D Scene -->
    <div class="flex-1 w-full relative">
      <TresCanvas alpha shadows>
        <TresPerspectiveCamera :position="[0, 12, 25]" :look-at="[0, 0, 0]" />
        
        <!-- Controls constrained so baby doesn't lose the scene -->
        <OrbitControls 
          :enable-pan="false" 
          :min-polar-angle="Math.PI / 6"
          :max-polar-angle="Math.PI / 2.2"
          :min-distance="10"
          :max-distance="40"
        />

        <!-- Lighting -->
        <TresAmbientLight :intensity="1.5" color="#ffffff" />
        <TresDirectionalLight :position="[10, 20, 10]" :intensity="2" cast-shadow />
        <TresDirectionalLight :position="[-10, 10, -10]" :intensity="1" color="#93c5fd" />

        <!-- Water Plane -->
        <TresMesh :position="[0, -1, 0]" :rotation="[-Math.PI / 2, 0, 0]" receive-shadow>
          <TresPlaneGeometry :args="[200, 200]" />
          <TresMeshStandardMaterial color="#0ea5e9" transparent :opacity="0.7" :roughness="0.1" :metalness="0.5" />
        </TresMesh>

        <!-- Fishes -->
        <TresGroup 
          v-for="fish in fishes" 
          :key="fish.id" 
          :position="[fish.x, fish.y, fish.z]" 
          :scale="[fish.scale, fish.scale, fish.scale]" 
        >
          <!-- Invisible clickable hit box (larger than fish to make it easier for baby to click) -->
          <TresMesh @click="catchFish(fish.id)">
             <TresBoxGeometry :args="[5, 4, 3]" />
             <TresMeshBasicMaterial transparent :opacity="0" :depthWrite="false" />
          </TresMesh>

          <!-- Fish Body -->
          <TresMesh cast-shadow :rotation="[0, 0, Math.PI / 2]">
            <TresCapsuleGeometry :args="[0.6, 1.5, 4, 16]" />
            <TresMeshStandardMaterial :color="fish.color" :roughness="0.2" :metalness="0.1" />
          </TresMesh>
          
          <!-- Fish Tail -->
          <TresMesh cast-shadow :position="[fish.direction === 1 ? -1.2 : 1.2, 0, 0]" :rotation="[0, 0, fish.direction === 1 ? -Math.PI / 2 : Math.PI / 2]">
            <TresConeGeometry :args="[0.8, 1.2, 4]" />
            <TresMeshStandardMaterial :color="fish.color" :roughness="0.2" :metalness="0.1" />
          </TresMesh>
          
          <!-- Fish Eye (Front) -->
          <TresMesh :position="[fish.direction === 1 ? 0.8 : -0.8, 0.2, 0.45]">
            <TresSphereGeometry :args="[0.18, 16, 16]" />
            <TresMeshBasicMaterial color="white" />
          </TresMesh>
          <TresMesh :position="[fish.direction === 1 ? 0.85 : -0.85, 0.2, 0.55]">
            <TresSphereGeometry :args="[0.08, 16, 16]" />
            <TresMeshBasicMaterial color="black" />
          </TresMesh>
          
          <!-- Fish Eye (Back) -->
          <TresMesh :position="[fish.direction === 1 ? 0.8 : -0.8, 0.2, -0.45]">
            <TresSphereGeometry :args="[0.18, 16, 16]" />
            <TresMeshBasicMaterial color="white" />
          </TresMesh>
          <TresMesh :position="[fish.direction === 1 ? 0.85 : -0.85, 0.2, -0.55]">
            <TresSphereGeometry :args="[0.08, 16, 16]" />
            <TresMeshBasicMaterial color="black" />
          </TresMesh>
          
          <!-- Pectoral Fin (Side) -->
          <TresMesh cast-shadow :position="[0, -0.3, 0.6]" :rotation="[Math.PI / 4, 0, fish.direction === 1 ? -Math.PI / 6 : Math.PI / 6]">
            <TresConeGeometry :args="[0.3, 0.6, 4]" />
            <TresMeshStandardMaterial :color="fish.color" />
          </TresMesh>
          <TresMesh cast-shadow :position="[0, -0.3, -0.6]" :rotation="[-Math.PI / 4, 0, fish.direction === 1 ? -Math.PI / 6 : Math.PI / 6]">
            <TresConeGeometry :args="[0.3, 0.6, 4]" />
            <TresMeshStandardMaterial :color="fish.color" />
          </TresMesh>

        </TresGroup>

      </TresCanvas>
    </div>
  </div>
</template>

<style scoped>
.bounce-enter-active {
  animation: bounce-in 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.bounce-leave-active {
  animation: bounce-in 0.3s reverse ease-in;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  50% {
    transform: scale(1.1);
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}
</style>
