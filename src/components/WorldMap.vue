<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import * as d3 from 'd3'
import * as topojson from 'topojson-client'

const props = defineProps({
  year: { type: Number, default: 1960 }
})

const container = ref<HTMLDivElement | null>(null)
let svg: any = null
let path: any = null
let countryData: any = null
let isoMapping: Record<number, string> = {}

const WIDTH  = window.innerWidth * 0.9
const HEIGHT = WIDTH * 0.5

async function loadData() {
  const [topoRes, changeRes, isoRes] = await Promise.all([
    fetch('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json'),
    fetch('/data/child_population_change.json'),
    fetch('https://raw.githubusercontent.com/lukes/ISO-3166-Countries-with-Regional-Codes/master/all/all.json')
  ])

  const worldData = await topoRes.json()
  countryData     = await changeRes.json()
  const isoData   = await isoRes.json()

  isoData.forEach((c: any) => {
    const num = parseInt(c['country-code'])
    if (!isNaN(num)) isoMapping[num] = c['alpha-3']
  })

  initMap(worldData)
}

function getColor(numericId: number, year: number): string {
  const alpha3 = isoMapping[numericId]
  if (!alpha3) return '#4a4a4a'

  const countryYears = countryData[alpha3]
  if (!countryYears) return '#4a4a4a'

  const val = countryYears[year]
  if (val === null || val === undefined) return '#4a4a4a'

  if (val > 0) return '#f0f0f0'
  else return '#1a1a1a'
}

function initMap(worldData: any) {
  const el = container.value
  if (!el) return

  svg = d3.select(el)
    .append('svg')
    .attr('width',  WIDTH)
    .attr('height', HEIGHT)
    .style('background', '#2d2d2d')

  const projection = d3.geoNaturalEarth1()
    .scale(WIDTH / 6.5)
    .translate([WIDTH / 2, HEIGHT / 2])

  path = d3.geoPath().projection(projection)

  const countries = topojson.feature(worldData, worldData.objects.countries) as any

  svg.selectAll('.country')
    .data(countries.features)
    .enter()
    .append('path')
    .attr('class', 'country')
    .attr('d', path as any)
    .attr('stroke', '#808080')
    .attr('stroke-width', 0.5)
    .attr('fill', (d: any) => getColor(d.id, props.year))

  const legend = svg.append('g')
    .attr('transform', `translate(20, ${HEIGHT - 90})`)

  const legendItems = [
    { color: '#f0f0f0', label: 'Increasing' },
    { color: '#1a1a1a', label: 'Decreasing' },
    { color: '#4a4a4a', label: 'No data'    },
  ]

  legendItems.forEach((item, i) => {
    legend.append('rect')
      .attr('width', 20)
      .attr('height', 20)
      .attr('y', i * 28)
      .attr('fill', item.color)

    legend.append('text')
      .attr('x', 28)
      .attr('y', i * 28 + 15)
      .attr('font-size', '12px')
      .attr('fill', '#d0d0d0')
      .text(item.label)
  })
}

function updateMap() {
  if (!svg) return
  svg.selectAll('.country')
    .attr('fill', (d: any) => getColor(d.id, props.year))
}

onMounted(loadData)
watch(() => props.year, updateMap)
</script>

<template>
  <div ref="container" class="map-container"></div>
</template>

<style scoped>
.map-container {
  width: 100%;
  margin-top: 24px;
}
</style>