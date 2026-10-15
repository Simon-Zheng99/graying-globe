<script setup lang="ts">
import { ref, onMounted } from 'vue'
import PopulationPyramid from './components/PopulationPyramid.vue'
import LineGraph from './components/LineGraph.vue'
import WorldMap from './components/WorldMap.vue'
import StoryBoard from './components/StoryBoard.vue'



const year = ref(1960)
const ghostYear = ref<number | null>(null)
const isPlaying = ref(false)
const selectedCountry = ref('WLD')
const countries = ref<{ code: string, name: string }[]>([])
let interval: number | null = null

async function loadCountries() {
  const res = await fetch(import.meta.env.BASE_URL + 'data/population.json')
  const data = await res.json()
  countries.value = Object.entries(data.countries)
    .map(([code, val]: [string, any]) => ({ code, name: (val as any).name, years: (val as any).years }))
    .filter((item: any) => {
      const years = Object.values(item.years) as any[]
      return years.some((y: any) => y.male && y.male.some((v: any) => v !== null))
    })
    .map((item: any) => ({ code: item.code, name: item.name }))
    .sort((a, b) => a.name.localeCompare(b.name))
}

function onYearChange(e: Event) {
  const newYear = parseInt((e.target as HTMLInputElement).value)
  ghostYear.value = newYear - 10 >= 1960 ? newYear - 10 : null
  year.value = newYear
}

function togglePlay() {
  if (isPlaying.value) pause()
  else play()
}

function play() {
  isPlaying.value = true
  if (year.value >= 2050) year.value = 1960
  interval = setInterval(() => {
    if (year.value >= 2050) { pause(); return }
    ghostYear.value = year.value - 10 >= 1960 ? year.value - 10 : null
    year.value++
  }, 100)
}

function pause() {
  isPlaying.value = false
  if (interval) clearInterval(interval)
  interval = null
}

function onYearSelected(y: number) {
  ghostYear.value = y - 10 >= 1960 ? y - 10 : null
  year.value = y
}

onMounted(loadCountries)
</script>

<template>
  <!-- Title Page -->
  <div class="title-page">
    <div class="title-panel">
      <h1>A Graying Globe</h1>
      <p>How the world's population is changing from 1960 to 2050</p>
      <div class="scroll-hint">↓ Scroll to explore</div>
    </div>
  </div>


  <div style="padding: 24px">
    <StoryBoard />
    <h2 style="color: #f0f0f0;">World Population Pyramid — {{ year }}</h2>

    <!-- Country Dropdown -->
    <div style="margin-bottom: 16px">
      <label for="country-select" style="color: #d0d0d0;">Country: </label>
      <select 
        id="country-select" 
        v-model="selectedCountry"
        style="
          padding: 8px 12px;
          font-size: 14px;
          border: 2px solid #808080;
          border-radius: 6px;
          background-color: #2d2d2d;
          color: #d0d0d0;
          cursor: pointer;
          outline: none;
          width: 150px;
        "
    >
        <option v-for="c in countries" :key="c.code" :value="c.code">
          {{ c.name }}
        </option>
      </select>
    </div>

    <!-- Pyramid and Line Graph Layout -->
    <div style="display: flex; gap: 24px; align-items: flex-start">
      <div style="flex: 1">
        <PopulationPyramid :country-code="selectedCountry" :year="year" :ghost-year="ghostYear ?? undefined" />
      </div>
      <div style="flex: 1">
        <LineGraph
          :current-year="year"
          :country-code="selectedCountry"
          @year-selected="onYearSelected"
        />
      </div>
    </div>

   <!-- Slider -->
  <div style="margin-top: 16px">
    <input
      type="range"
      min="1960"
      max="2050"
      :value="year"
      @input="onYearChange"
      style="width: 100%; accent-color: #808080;"
    />
    <div style="display: flex; justify-content: space-between">
      <span style="color: #d0d0d0;">1960</span>
      <span style="color: #d0d0d0;">2050</span>
    </div>
  </div>

  <!-- Play button -->
  <div style="margin-top: 12px; text-align: center">
    <button
      @click="togglePlay"
      style="
        padding: 8px 24px;
        font-size: 16px;
        cursor: pointer;
        background-color: #2d2d2d;
        color: #d0d0d0;
        border: 2px solid #808080;
        border-radius: 6px;
      "
    >
      {{ isPlaying ? '⏸ Pause' : '▶ Play' }}
    </button>
  </div>

    <!-- World Map -->
    <div style="display: flex; justify-content: center; margin-top: 24px;">
      <WorldMap :year="year" style="width: 100%;" />
    </div>
  </div>
</template>

<style scoped>
#main-container {
  min-height: 100vh;
  background-color: #fafafa;
  padding: 24px;
}
</style>