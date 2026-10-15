<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import * as d3 from 'd3'

const props = defineProps({
  countryCode: { type: String, default: 'WLD' },
  year:        { type: Number, default: 1960 },
  ghostYear:   { type: Number, default: null }
})

const container = ref<HTMLDivElement | null>(null)
let svg: any = null
let xScale: any = null
let yScale: any = null
let data: any = null

const ageGroups = [
  '00-04','05-09','10-14','15-19','20-24','25-29',
  '30-34','35-39','40-44','45-49','50-54','55-59',
  '60-64','65-69','70-74','75-79','80+'
]

async function loadData() {
  const res = await fetch(import.meta.env.BASE_URL + 'data/population.json')
  data = await res.json()
  console.log('Data loaded!', Object.keys(data.countries).length, 'countries')
  console.log('Sample AFG 1960:', data.countries['AFG']?.years[1960])
}

function getYearData(countryCode: string, year: number) {
  const country = data.countries[countryCode]
  if (!country) return null
  const y = country.years[year]
  if (!y) return null

  const total = y.male.reduce((a: number, b: number) => a + (b || 0), 0)
              + y.female.reduce((a: number, b: number) => a + (b || 0), 0)

  return ageGroups.map((ag, i) => ({
    age:    ag,
    male:   -((y.male[i]   || 0) / total) * 100,
    female:  ((y.female[i] || 0) / total) * 100,
  }))
}

function initChart() {
  const el = container.value
  if (!el) return

  const margin = { top: 20, right: 30, bottom: 40, left: 50 }
  const width  = el.clientWidth  - margin.left - margin.right
  const height = el.clientHeight - margin.top  - margin.bottom

  svg = d3.select(el)
    .append('svg')
    .attr('width',  el.clientWidth)
    .attr('height', el.clientHeight)
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`)

  xScale = d3.scaleLinear()
    .domain([-10, 10])
    .range([0, width])

  yScale = d3.scaleBand()
    .domain(ageGroups)
    .range([height, 0])
    .padding(0.1)

  const currentData = getYearData(props.countryCode, props.year)
  if (!currentData) return

  // Ghost bars
  svg.selectAll('.ghost-male')
    .data(currentData)
    .enter().append('rect')
    .attr('class', 'ghost-male')
    .attr('y',      (d: any) => yScale(d.age))
    .attr('height', yScale.bandwidth())
    .attr('fill', '#808080')
    .attr('opacity', 0)

  svg.selectAll('.ghost-female')
    .data(currentData)
    .enter().append('rect')
    .attr('class', 'ghost-female')
    .attr('y',      (d: any) => yScale(d.age))
    .attr('height', yScale.bandwidth())
    .attr('fill', '#a0a0a0')
    .attr('opacity', 0)

  // Main bars
  svg.selectAll('.bar-male')
    .data(currentData)
    .enter().append('rect')
    .attr('class', 'bar-male')
    .attr('y',      (d: any) => yScale(d.age))
    .attr('height', yScale.bandwidth())
    .attr('fill', '#808080')
    .attr('opacity', 0.85)

  svg.selectAll('.bar-female')
    .data(currentData)
    .enter().append('rect')
    .attr('class', 'bar-female')
    .attr('y',      (d: any) => yScale(d.age))
    .attr('height', yScale.bandwidth())
    .attr('fill', '#a0a0a0')
    .attr('opacity', 0.85)

  // Y axis
  svg.append('g')
    .attr('class', 'y-axis')
    .call(d3.axisLeft(yScale).tickSize(0))
    .select('.domain').remove()

  svg.selectAll('.y-axis text')
    .attr('fill', '#ffffff')

  // X axis
  svg.append('g')
    .attr('class', 'x-axis')
    .attr('transform', `translate(0,${height})`)
    .call(
      d3.axisBottom(xScale)
        .tickFormat((d: any) => `${Math.abs(+d).toFixed(1)}%`)
        .ticks(6)
    )

  svg.selectAll('.x-axis text')
    .attr('fill', '#ffffff')
  
  svg.selectAll('.x-axis .domain, .x-axis .tick line')
  .attr('stroke', '#808080')

  // Male label
  svg.append('text')
    .attr('x', xScale(-5))
    .attr('y', height + 35)
    .attr('text-anchor', 'middle')
    .attr('fill', '#808080')
    .attr('font-size', '13px')
    .text('← Male')

  // Female label
  svg.append('text')
    .attr('x', xScale(5))
    .attr('y', height + 35)
    .attr('text-anchor', 'middle')
    .attr('fill', '#a0a0a0')
    .attr('font-size', '13px')
    .text('Female →')

  // Center divider line
  svg.append('line')
    .attr('x1', xScale(0))
    .attr('x2', xScale(0))
    .attr('y1', 0)
    .attr('y2', height)
    .attr('stroke', '#808080')
    .attr('stroke-width', 1)

  updateChart()
}

function updateChart() {
  if (!svg || !data) return

  const currentData = getYearData(props.countryCode, props.year)
  if (!currentData) return

  const t = d3.transition().duration(80).ease(d3.easeLinear)

  // Update main bars
  svg.selectAll('.bar-male')
    .data(currentData)
    .transition(t)
    .attr('x',     (d: any) => xScale(d.male))
    .attr('width', (d: any) => xScale(0) - xScale(d.male))

  svg.selectAll('.bar-female')
    .data(currentData)
    .transition(t)
    .attr('x',     xScale(0))
    .attr('width', (d: any) => xScale(d.female) - xScale(0))

  // Update ghost bars
  if (props.ghostYear !== null) {
    const ghostData = getYearData(props.countryCode, props.ghostYear)
    if (ghostData) {
      svg.selectAll('.ghost-male')
        .data(ghostData)
        .transition(t)
        .attr('x',      (d: any) => xScale(d.male))
        .attr('width',  (d: any) => xScale(0) - xScale(d.male))
        .attr('opacity', 0.2)

      svg.selectAll('.ghost-female')
        .data(ghostData)
        .transition(t)
        .attr('x',      xScale(0))
        .attr('width',  (d: any) => xScale(d.female) - xScale(0))
        .attr('opacity', 0.2)
    }
  } else {
    svg.selectAll('.ghost-male, .ghost-female')
      .transition(t)
      .attr('opacity', 0)
  }
}

onMounted(async () => {
  await loadData()
  initChart()
})

watch(() => props.countryCode, () => {
  const el = container.value
  if (!el) return
  el.innerHTML = ''
  svg = null
  initChart()
})

watch(() => [props.year, props.ghostYear], updateChart)
</script>

<template>
  <div ref="container" class="pyramid-container"></div>
</template>

<style scoped>
.pyramid-container {
  width: 100%;
  height: 500px;
}
</style>