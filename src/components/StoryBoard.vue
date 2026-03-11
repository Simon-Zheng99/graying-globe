<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import PopulationPyramid from './PopulationPyramid.vue'

const stages = [
  {
    year: 1960,
    title: '1960: The Youth Boom',
    description: 'The world in 1960 was young and growing fast. Advances in modern medicine dramatically reduced child mortality, while birth rates remained high. The result was a classic pyramid shape — a wide base of children supporting a narrow top of elderly. The global population stood at just over 3 billion.'
  },
  {
    year: 2017,
    title: '2017: Peak Child',
    description: 'In 2017, researcher Hans Rosling identified a critical turning point — the number of young children in the world stopped growing, stabilizing at around 2 billion. Birth rates had been falling across the globe as countries developed. The base of the pyramid began to narrow, signaling that infinite population growth was never the destiny of humanity.'
  },
  {
    year: 2050,
    title: '2050: The Silver Tsunami',
    description: 'By 2050, the pyramid no longer looks like a triangle. It has transformed into a vase — wide in the middle and top, narrow at the base. People aged 65 and older are the fastest growing group on Earth. Financial systems built on the assumption of an ever-growing young workforce — pensions, social security — now face an uncertain future.'
  }
]

const currentYear = ref(1960)
let observers: IntersectionObserver[] = []

onMounted(() => {
  stages.forEach((stage, index) => {
    const el = document.getElementById(`stage-${index}`)
    if (!el) return

    const observer = new IntersectionObserver(
        (entries) => {
            entries.forEach(entry => {
            if (entry.isIntersecting) {
                currentYear.value = stage.year
            }
            })
        },
        { 
            rootMargin: '-20% 0px -20% 0px'
        }
    )
    observer.observe(el)
    observers.push(observer)
  })
})

onUnmounted(() => {
  observers.forEach(o => o.disconnect())
})
</script>

<template>
  <div class="storyboard">

    <div class="sticky-pyramid">
      <PopulationPyramid
        country-code="WLD"
        :year="currentYear"
        :ghost-year="undefined"
      />
    </div>

    <div class="scroll-sections">
      <div
        v-for="(stage, index) in stages"
        :key="stage.year"
        :id="`stage-${index}`"
        class="stage"
      >
        <div class="text-panel">
          <h2>{{ stage.title }}</h2>
          <p>{{ stage.description }}</p>
        </div>
      </div>

      <div class="explore-prompt">
        <h2>Now it's your turn to explore</h2>
        <p>Use the controls below to navigate through time and across countries.</p>
      </div>
    </div>

  </div>
</template>

<style scoped>
.storyboard {
  position: relative;
  width: 100%;
}

.sticky-pyramid {
  position: sticky;
  top: 0;
  height: 100vh;
  width: 100%;
  z-index: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}

.scroll-sections {
  position: relative;
  z-index: 2;
  margin-top: -100vh;
}

.stage {
  height: 100vh;
  display: flex;
  align-items: center;
}

.text-panel {
  position: relative;
  left: 80px;
  width: 320px;
  background: rgba(0, 0, 0, 0.65);
  color: white;
  padding: 28px;
  border-radius: 8px;
}

.text-panel h2 {
  font-size: 22px;
  font-weight: bold;
  margin-bottom: 12px;
  color: #f0f0f0;
}

.text-panel p {
  font-size: 15px;
  line-height: 1.7;
  color: #d0d0d0;
}

.explore-prompt {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.65);
  color: white;
}

.explore-prompt h2 {
  font-size: 28px;
  margin-bottom: 12px;
}

.explore-prompt p {
  font-size: 16px;
  color: #d0d0d0;
}
</style>