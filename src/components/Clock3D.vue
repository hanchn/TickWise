<script setup lang="ts">
import { Html } from '@tresjs/cientos'

const props = defineProps<{
  hour: number
  minute: number
  mode: 'hour' | 'minute'
}>()

// 12 numbers for the clock face
const numbers = Array.from({ length: 12 }, (_, i) => {
  const num = i === 0 ? 12 : i
  const angle = (Math.PI / 2) - (i * 30 * Math.PI) / 180
  
  // Radius for numbers
  const radius = 3.6
  const x = Math.cos(angle) * radius
  const y = Math.sin(angle) * radius
  
  return { num, position: [x, y, 0.2] as [number, number, number] }
})

// Minute tick marks
const ticks = Array.from({ length: 60 }, (_, i) => {
  const angle = (Math.PI / 2) - (i * 6 * Math.PI) / 180
  const isHour = i % 5 === 0
  const radius = 4.4 
  const x = Math.cos(angle) * radius
  const y = Math.sin(angle) * radius
  return { id: i, position: [x, y, 0.15] as [number, number, number], isHour }
})

// Minute numbers (only for minute mode)
const minuteNumbers = Array.from({ length: 12 }, (_, i) => {
  const num = i === 0 ? 0 : i * 5
  const angle = (Math.PI / 2) - (i * 30 * Math.PI) / 180
  const radius = 5.4 // Put them just outside the main clock
  const x = Math.cos(angle) * radius
  const y = Math.sin(angle) * radius
  
  return { num, position: [x, y, -0.1] as [number, number, number] }
})

// Calculate rotation for hands (in radians)
const getHourRotation = (hour: number, minute: number) => {
  const h = hour % 12
  const degrees = h * 30 + minute * 0.5
  return -degrees * (Math.PI / 180)
}

const getMinuteRotation = (minute: number) => {
  const degrees = minute * 6
  return -degrees * (Math.PI / 180)
}
</script>

<template>
  <TresGroup>
    
    <!-- Premium Wood/Macaron Rim -->
    <TresMesh :position="[0, 0, 0]" cast-shadow receive-shadow>
      <TresTorusGeometry :args="[4.8, 0.35, 64, 128]" />
      <TresMeshStandardMaterial color="#FFB7B2" :roughness="0.6" :metalness="0.1" />
    </TresMesh>

    <!-- Clean White Clock Face -->
    <TresMesh :rotation="[Math.PI / 2, 0, 0]" :position="[0, 0, -0.15]" receive-shadow>
      <TresCylinderGeometry :args="[4.8, 4.8, 0.3, 128]" />
      <TresMeshStandardMaterial color="#FAFAFA" :roughness="0.9" />
    </TresMesh>

    <!-- Subtle Inner Ring -->
    <TresMesh :position="[0, 0, 0.05]">
      <TresRingGeometry :args="[3.0, 3.02, 128]" />
      <TresMeshBasicMaterial color="#E2E8F0" transparent :opacity="0.4" />
    </TresMesh>

    <!-- Elegant Ticks -->
    <TresMesh v-for="tick in ticks" :key="`tick-${tick.id}`" :position="tick.position">
      <!-- Make hour ticks slightly larger and pill-shaped -->
      <TresBoxGeometry v-if="tick.isHour" :args="[0.08, 0.25, 0.05]" />
      <TresCircleGeometry v-else :args="[0.03, 32]" />
      <TresMeshStandardMaterial 
        :color="tick.isHour ? '#FF9F1C' : '#CBD5E1'" 
        :rotation="[0, 0, -tick.id * 6 * Math.PI / 180]" 
      />
    </TresMesh>

    <!-- Center Pin -->
    <TresMesh :position="[0, 0, 0.45]" cast-shadow>
      <TresSphereGeometry :args="[0.25, 64, 64]" />
      <TresMeshStandardMaterial color="#FF595E" :roughness="0.3" :metalness="0.2" />
    </TresMesh>
    <TresMesh :position="[0, 0, 0.35]" cast-shadow>
      <TresCylinderGeometry :args="[0.25, 0.35, 0.2, 64]" />
      <TresMeshStandardMaterial color="#FF595E" :roughness="0.3" />
    </TresMesh>

    <!-- Clean, Modern Numbers -->
    <TresGroup v-for="item in numbers" :key="item.num" :position="item.position">
      <Html
        transform
        center
        :distance-factor="14"
        :zIndexRange="[0, 0]"
      >
        <div 
          class="font-black select-none pointer-events-none text-slate-700"
          :style="{
            fontSize: '48px',
            textShadow: '0px 2px 4px rgba(0,0,0,0.05)'
          }"
        >
          {{ item.num }}
        </div>
      </Html>
    </TresGroup>

    <!-- Minimalist Minute Numbers (Only in minute mode) -->
    <TresGroup v-if="props.mode === 'minute'">
      <TresGroup v-for="item in minuteNumbers" :key="`min-${item.num}`" :position="item.position">
        <Html
          transform
          center
          :distance-factor="14"
          :zIndexRange="[0, 0]"
        >
          <div 
            class="flex items-center justify-center rounded-full bg-blue-50 text-blue-500 font-bold select-none pointer-events-none shadow-sm"
            :style="{ width: '40px', height: '40px', fontSize: '18px' }"
          >
            {{ item.num }}
          </div>
        </Html>
      </TresGroup>
    </TresGroup>

    <!-- Minute Hand Group (Sleek Blue) -->
    <TresGroup :rotation="[0, 0, getMinuteRotation(props.minute)]" :position="[0, 0, 0.25]">
      <TresMesh :position="[0, 1.8, 0]" cast-shadow>
        <TresCapsuleGeometry :args="[0.08, 3.4, 32, 32]" />
        <TresMeshStandardMaterial color="#1982C4" :roughness="0.5" />
      </TresMesh>
      <TresMesh :position="[0, -0.5, 0]" cast-shadow>
        <TresCapsuleGeometry :args="[0.08, 1.0, 32, 32]" />
        <TresMeshStandardMaterial color="#1982C4" :roughness="0.5" />
      </TresMesh>
    </TresGroup>

    <!-- Hour Hand Group (Sleek Green) -->
    <TresGroup :rotation="[0, 0, getHourRotation(props.hour, props.minute)]" :position="[0, 0, 0.35]">
      <TresMesh :position="[0, 1.1, 0]" cast-shadow>
        <TresCapsuleGeometry :args="[0.12, 2.0, 32, 32]" />
        <TresMeshStandardMaterial color="#8AC926" :roughness="0.5" />
      </TresMesh>
      <TresMesh :position="[0, -0.4, 0]" cast-shadow>
        <TresCapsuleGeometry :args="[0.12, 0.8, 32, 32]" />
        <TresMeshStandardMaterial color="#8AC926" :roughness="0.5" />
      </TresMesh>
    </TresGroup>

  </TresGroup>
</template>
