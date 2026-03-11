<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import * as d3 from 'd3'

const props = defineProps({
  currentYear: { type: Number, default: 1960 },
  countryCode: { type: String, default: 'WLD' }
})

const emit = defineEmits(['year-selected'])
const container = ref<HTMLDivElement | null>(null)
let svg: any = null
let data: { year: number, population: number }[] = []
let xScale: any = null
let yScale: any = null
let marker: any = null
let tooltip: any = null

const TRANSITION_YEAR = 2020
const milestones = [
  { year: 1960, label: 'Youth Boom' },
  { year: 2017, label: 'Peak Child' },
  { year: 2040, label: 'The Future' },
]

async function loadData() {
  const res = await fetch('/data/population.json')
  const json = await res.json()
  const country = json.countries[props.countryCode]
  if (!country) return

  data = Object.entries(country.years)
    .map(([year, vals]: [string, any]) => ({
      year: parseInt(year),
      population: vals.total
    }))
    .filter(d => d.population !== null)
    .sort((a, b) => a.year - b.year)
}

function initChart() {
  const el = container.value
  if (!el) return

  el.innerHTML = ''

  const margin = { top: 20, right: 30, bottom: 40, left: 70 }
  const width  = el.clientWidth  - margin.left - margin.right
  const height = el.clientHeight - margin.top  - margin.bottom

  svg = d3.select(el)
    .append('svg')
    .attr('width',  el.clientWidth)
    .attr('height', el.clientHeight)
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`)

  xScale = d3.scaleLinear()
    .domain([1960, 2050])
    .range([0, width])

  yScale = d3.scaleLinear()
    .domain([0, 12e9])
    .range([height, 0])

  // X axis
  svg.append('g')
    .attr('transform', `translate(0,${height})`)
    .call(d3.axisBottom(xScale).tickFormat((d: any) => `${d}`).ticks(10))
    .call((g: any) => g.select('.domain').attr('stroke', '#ffffff'))
    .call((g: any) => g.selectAll('.tick line').attr('stroke', '#ffffff'))
    .selectAll('text')
    .attr('fill', '#ffffff')

  // Y axis
  svg.append('g')
    .call(d3.axisLeft(yScale).tickFormat((d: any) => `${(+d / 1e9).toFixed(1)}B`).ticks(6))
    .call((g: any) => g.select('.domain').attr('stroke', '#ffffff'))
    .call((g: any) => g.selectAll('.tick line').attr('stroke', '#ffffff'))
    .selectAll('text')
    .attr('fill', '#ffffff')

  // Solid line (1960-2020)
  const solidData = data.filter(d => d.year <= TRANSITION_YEAR)
  svg.append('path')
    .datum(solidData)
    .attr('fill', 'none')
    .attr('stroke', '#a0a0a0')
    .attr('stroke-width', 2.5)
    .attr('d', d3.line<{ year: number, population: number }>()
      .x(d => xScale(d.year))
      .y(d => yScale(d.population))
    )

  // Dashed line (2020-2050)
  const dashedData = data.filter(d => d.year >= TRANSITION_YEAR)
  svg.append('path')
    .datum(dashedData)
    .attr('fill', 'none')
    .attr('stroke', '#a0a0a0')
    .attr('stroke-width', 2.5)
    .attr('stroke-dasharray', '6 4')
    .attr('d', d3.line<{ year: number, population: number }>()
      .x(d => xScale(d.year))
      .y(d => yScale(d.population))
    )

  // Milestone annotations
  milestones.forEach(m => {
    const d = data.find(d => d.year === m.year)
    if (!d) return

    svg.append('circle')
      .attr('cx', xScale(d.year))
      .attr('cy', yScale(d.population))
      .attr('r', 5)
      .attr('fill', '#f59e0b')

    svg.append('text')
      .attr('x', xScale(d.year) + 8)
      .attr('y', yScale(d.population) - 8)
      .attr('font-size', '12px')
      .attr('fill', '#ffffff')
      .text(m.label)
  })

  // Tooltip
  tooltip = d3.select(container.value)
    .append('div')
    .style('position', 'absolute')
    .style('background', 'rgba(255,255,255,0.15)')
    .style('color', '#ffffff')
    .style('border', '1px solid #808080')
    .style('padding', '6px 10px')
    .style('border-radius', '4px')
    .style('font-size', '13px')
    .style('pointer-events', 'none')
    .style('opacity', 0)

  // Clickable overlay
  svg.append('rect')
    .attr('width', width)
    .attr('height', height)
    .attr('fill', 'transparent')
    .on('click', (event: MouseEvent) => {
      const [mx] = d3.pointer(event)
      const clickedYear = Math.round(xScale.invert(mx))
      const clampedYear = Math.max(1960, Math.min(2050, clickedYear))

      const point = data.find(d => d.year === clampedYear)
      if (point) {
        tooltip
          .style('opacity', 1)
          .style('left', `${event.offsetX + 12}px`)
          .style('top',  `${event.offsetY - 28}px`)
          .html(`<strong>${clampedYear}</strong>: ${(point.population / 1e9).toFixed(2)}B`)
      }

      emit('year-selected', clampedYear)
    })

  // Year marker line
  marker = svg.append('line')
    .attr('y1', 0)
    .attr('y2', height)
    .attr('stroke', '#ffffff')
    .attr('stroke-width', 1.5)
    .attr('stroke-dasharray', '4 2')

  updateMarker()
}

function updateMarker() {
  if (!marker) return
  marker
    .attr('x1', xScale(props.currentYear))
    .attr('x2', xScale(props.currentYear))
}

onMounted(async () => {
  await loadData()
  initChart()
})

watch(() => props.currentYear, updateMarker)
</script>

<template>
  <div ref="container" class="linegraph-container"></div>
</template>

<style scoped>
.linegraph-container {
  width: 100%;
  height: 450px;
  cursor: pointer;
  position: relative;
}
</style>