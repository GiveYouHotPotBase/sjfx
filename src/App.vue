<template>
  <div class="app">
    <section class="hero">
      <h1>全球人均温室气体排放探索</h1>
      <p class="subtitle">
        基于OECD数据，交互式探索2014-2023年间全球各国人均温室气体排放的变化趋势与差异
      </p>
      <div class="stats-row">
        <div class="stat-item">
          <div class="stat-value">90</div>
          <div class="stat-label">国家与地区</div>
        </div>
        <div class="stat-item">
          <div class="stat-value">10</div>
          <div class="stat-label">年份跨度</div>
        </div>
        <div class="stat-item">
          <div class="stat-value">{{ latestAvg }}</div>
          <div class="stat-label">2023年OECD均值 (吨)</div>
        </div>
      </div>
      <div class="scroll-hint">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M12 5v14M5 12l7 7 7-7"/>
        </svg>
        <span>向下滚动探索</span>
      </div>
    </section>

    <div class="divider"></div>

    <section class="section" id="ranking">
      <div class="section-header">
        <div class="section-number">Part 01</div>
        <h2 class="section-title">谁的排放最多？</h2>
        <p class="section-desc">
          通过排名柱状图观察各国人均温室气体排放量。拖动滑块切换年份，点击播放按钮观看排名动态变化。
          柱状图按排放量从高到低排列，让高排放国家一目了然。
        </p>
      </div>
      <div class="chart-container">
        <div class="controls">
          <button class="btn btn-icon" @click="togglePlay" :title="playing ? '暂停' : '播放'">
            <svg v-if="!playing" width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
              <polygon points="5,3 19,12 5,21"/>
            </svg>
            <svg v-else width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
              <rect x="6" y="4" width="4" height="16"/><rect x="14" y="4" width="4" height="16"/>
            </svg>
          </button>
          <div class="year-display">{{ currentYear }}</div>
          <div class="slider-wrapper">
            <input type="range" v-model.number="currentYear" :min="2014" :max="2023" step="1" />
          </div>
          <div class="filter-group">
            <button class="btn" :class="{ active: barFilter === 'all' }" @click="barFilter = 'all'">全部</button>
            <button class="btn" :class="{ active: barFilter === 'top20' }" @click="barFilter = 'top20'">前20</button>
            <button class="btn" :class="{ active: barFilter === 'oecd' }" @click="barFilter = 'oecd'">OECD</button>
          </div>
        </div>
        <div ref="barChartRef" class="bar-chart-wrapper"></div>
      </div>
    </section>

    <div class="divider"></div>

    <section class="section" id="trends">
      <div class="section-header">
        <div class="section-number">Part 02</div>
        <h2 class="section-title">排放趋势如何变化？</h2>
        <p class="section-desc">
          选择感兴趣的国家，观察其人均排放量在十年间的变化趋势。
          点击下方的国家标签添加或移除追踪。将鼠标悬停在折线上查看具体数值。
        </p>
      </div>
      <div class="chart-container">
        <div class="controls">
          <input class="search-input" type="text" v-model="searchQuery" placeholder="搜索国家..." />
          <div class="filter-group">
            <button class="btn" @click="selectPreset('major')">主要经济体</button>
            <button class="btn" @click="selectPreset('europe')">欧洲</button>
            <button class="btn" @click="selectPreset('asia')">亚太</button>
            <button class="btn" @click="clearSelection">清除全部</button>
          </div>
        </div>
        <div class="country-picker">
          <span
            v-for="c in filteredCountries"
            :key="c.code"
            class="country-chip"
            :class="{ selected: selectedCountries.has(c.code) }"
            :style="selectedCountries.has(c.code) ? chipStyle(c.code) : {}"
            @click="toggleCountry(c.code)"
          >{{ c.name }}</span>
        </div>
        <div ref="lineChartRef" class="line-chart-wrapper"></div>
      </div>
    </section>

    <div class="divider"></div>

    <section class="section" id="change">
      <div class="section-header">
        <div class="section-number">Part 03</div>
        <h2 class="section-title">谁在进步，谁在退步？</h2>
        <p class="section-desc">
          对比各国在2014年与最新数据年份之间人均排放的变化幅度。
          绿色代表减排成功，红色代表排放增加。悬停查看详细变化数据。
        </p>
      </div>
      <div class="chart-container">
        <div class="controls">
          <div class="filter-group">
            <button class="btn" :class="{ active: changeFilter === 'all' }" @click="changeFilter = 'all'">全部</button>
            <button class="btn" :class="{ active: changeFilter === 'improved' }" @click="changeFilter = 'improved'">减排国家</button>
            <button class="btn" :class="{ active: changeFilter === 'worsened' }" @click="changeFilter = 'worsened'">增排国家</button>
          </div>
          <div class="filter-group">
            <button class="btn" :class="{ active: changeSortBy === 'absolute' }" @click="changeSortBy = 'absolute'">绝对变化</button>
            <button class="btn" :class="{ active: changeSortBy === 'percent' }" @click="changeSortBy = 'percent'">百分比变化</button>
          </div>
        </div>
        <div ref="changeChartRef" class="change-chart-wrapper"></div>
      </div>
    </section>

    <div class="divider"></div>

    <section class="doc-section" id="docs">
      <h2>项目说明文档</h2>

      <h3>研究问题</h3>
      <p>
        本可视化旨在回答以下核心问题：在全球应对气候变化的进程中，各国人均温室气体排放呈现怎样的格局与演变趋势？
        哪些国家在减排方面取得了显著成效？哪些国家的人均排放仍在增长？不同区域之间存在怎样的差异？
      </p>

      <h3>设计决策</h3>
      <ul>
        <li><strong>排名柱状图（Part 01）</strong>：采用水平柱状图展示排名，支持年份动画切换。水平布局使国家名称更易阅读，颜色编码映射排放量高低，年份滑块与播放按钮支持动态探索。</li>
        <li><strong>趋势折线图（Part 02）</strong>：采用多线折线图展示时间序列变化，支持用户自由选择对比国家。折线图是展示连续时间趋势的最佳选择，交互式国家选择器避免信息过载。</li>
        <li><strong>变化哑铃图（Part 03）</strong>：采用水平哑铃/棒棒糖图展示首尾年份差异，颜色区分减排（绿色）与增排（红色），支持排序与过滤切换。</li>
        <li><strong>替代方案考量</strong>：曾考虑使用地图可视化，但由于数据覆盖国家有限且地理面积易造成视觉误导，最终选择柱状图+折线图的组合方案。也考虑过Bump Chart展示排名变化，但折线图能更直观地展示绝对数值趋势。</li>
      </ul>

      <h3>数据来源</h3>
      <p>
        数据来自OECD环境指标数据库（OECD.Stat），数据集为"Air emissions - Greenhouse gas emissions Inventories"。
        指标为人均温室气体排放量（千克CO2当量/人），不含土地利用变化与林业（LULUCF）。
        数据覆盖2014-2023年，涵盖约90个国家与地区。
      </p>
      <p>数据集链接：<a href="https://data-explorer.oecd.org/" target="_blank">OECD Data Explorer</a></p>

      <h3>开发流程</h3>
      <ul>
        <li><strong>数据预处理</strong>（~1小时）：解析OECD原始CSV格式，提取关键字段并转换为JSON</li>
        <li><strong>架构搭建</strong>（~2小时）：Vite + Vue 3 + D3.js项目初始化，响应式布局设计</li>
        <li><strong>排名柱状图</strong>（~4小时）：D3动画过渡、年份控制、播放功能实现</li>
        <li><strong>趋势折线图</strong>（~3小时）：多线绑定、交互式国家选择、Tooltip联动</li>
        <li><strong>变化分析图</strong>（~3小时）：数据计算、双向排序、颜色映射</li>
        <li><strong>样式与交互优化</strong>（~3小时）：亮色主题设计、响应式适配、过渡动画</li>
        <li><strong>总计约16工时</strong>，其中排名柱状图的D3动画过渡与年份切换耗时最多</li>
      </ul>

      <h3>团队成员</h3>
      <p>本项目由个人独立完成，负责数据处理、可视化设计、前端开发与部署全流程。</p>
    </section>

    <footer>
      <p>数据来源: <a href="https://data-explorer.oecd.org/" target="_blank">OECD Environment Statistics</a> | 技术栈: Vue 3 + D3.js + Vite</p>
    </footer>

    <div
      class="tooltip"
      :class="{ visible: tooltip.show }"
      :style="{ left: tooltip.x + 'px', top: tooltip.y + 'px' }"
    >
      <div class="tt-title">{{ tooltip.title }}</div>
      <div class="tt-value">{{ tooltip.value }}</div>
      <div class="tt-sub">{{ tooltip.sub }}</div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, watch, nextTick, onUnmounted } from 'vue'
import * as d3 from 'd3'

const AGG_CODES = new Set(['EU27_2020', 'OECD', 'OECDA', 'OECDSO', 'OECDE'])
const OECD_CODES = new Set([
  'AUS','AUT','BEL','CAN','CHL','COL','CRI','CZE','DNK','EST','FIN','FRA',
  'DEU','GRC','HUN','ISL','IRL','ISR','ITA','JPN','KOR','LVA','LTU','LUX',
  'MEX','NLD','NZL','NOR','POL','PRT','SVK','SVN','ESP','SWE','CHE','TUR',
  'GBR','USA'
])
const MAJOR_CODES = ['USA','CHN','IND','JPN','DEU','GBR','FRA','RUS','BRA','CAN','AUS','KOR']
const EUROPE_CODES = ['DEU','FRA','GBR','ITA','ESP','POL','NLD','BEL','SWE','NOR','DNK','FIN','CHE','AUT','CZE','GRC','PRT']
const ASIA_CODES = ['JPN','KOR','CHN','IND','IDN','SGP','THA','MYS','AUS','NZL']

const COLORS = [
  '#6366f1','#e11d48','#0891b2','#ca8a04','#16a34a','#9333ea',
  '#dc2626','#0d9488','#c026d3','#2563eb','#ea580c','#4f46e5',
  '#059669','#d946ef','#0284c7','#65a30d'
]

const allData = ref([])
const countryList = ref([])
const currentYear = ref(2023)
const playing = ref(false)
const barFilter = ref('top20')
const searchQuery = ref('')
const selectedCountries = reactive(new Set(['USA', 'CHN', 'DEU', 'JPN', 'GBR', 'IND']))
const changeFilter = ref('all')
const changeSortBy = ref('percent')

const barChartRef = ref(null)
const lineChartRef = ref(null)
const changeChartRef = ref(null)

let playInterval = null
let barSvg = null
let lineSvg = null
let changeSvg = null

const tooltip = reactive({ show: false, x: 0, y: 0, title: '', value: '', sub: '' })

const latestAvg = computed(() => {
  const oecdRow = allData.value.find(d => d.code === 'OECD' && d.year === 2023)
  return oecdRow ? oecdRow.value.toFixed(1) : '—'
})

const countryData = computed(() => allData.value.filter(d => !AGG_CODES.has(d.code)))

const filteredCountries = computed(() => {
  if (!searchQuery.value) return countryList.value
  const q = searchQuery.value.toLowerCase()
  return countryList.value.filter(c =>
    c.name.toLowerCase().includes(q) || c.code.toLowerCase().includes(q)
  )
})

function getCountryColor(code) {
  const arr = Array.from(selectedCountries)
  const idx = arr.indexOf(code)
  return idx >= 0 ? COLORS[idx % COLORS.length] : COLORS[0]
}

function chipStyle(code) {
  const color = getCountryColor(code)
  return {
    '--chip-color': color,
    '--chip-bg': color + '22',
    borderColor: color,
    color: color
  }
}

function toggleCountry(code) {
  if (selectedCountries.has(code)) {
    selectedCountries.delete(code)
  } else {
    selectedCountries.add(code)
  }
}

function selectPreset(preset) {
  selectedCountries.clear()
  const codes = preset === 'major' ? MAJOR_CODES : preset === 'europe' ? EUROPE_CODES : ASIA_CODES
  codes.forEach(c => {
    if (countryList.value.find(x => x.code === c)) selectedCountries.add(c)
  })
}

function clearSelection() {
  selectedCountries.clear()
}

function togglePlay() {
  playing.value = !playing.value
  if (playing.value) {
    if (currentYear.value >= 2023) currentYear.value = 2014
    playInterval = setInterval(() => {
      if (currentYear.value < 2023) {
        currentYear.value++
      } else {
        playing.value = false
        clearInterval(playInterval)
      }
    }, 800)
  } else {
    clearInterval(playInterval)
  }
}

function showTooltip(event, title, value, sub) {
  tooltip.show = true
  tooltip.title = title
  tooltip.value = value
  tooltip.sub = sub
  tooltip.x = event.clientX + 15
  tooltip.y = event.clientY - 10
}

function hideTooltip() {
  tooltip.show = false
}

async function loadData() {
  const resp = await fetch('./data.json')
  const raw = await resp.json()
  allData.value = raw

  const cMap = new Map()
  raw.forEach(d => {
    if (!AGG_CODES.has(d.code)) {
      if (!cMap.has(d.code)) cMap.set(d.code, d.name)
    }
  })
  countryList.value = Array.from(cMap.entries())
    .map(([code, name]) => ({ code, name }))
    .sort((a, b) => a.name.localeCompare(b.name))
}

function drawBarChart() {
  if (!barChartRef.value || !countryData.value.length) return

  const container = barChartRef.value
  const width = container.clientWidth
  let yearData = countryData.value.filter(d => d.year === currentYear.value)
  yearData.sort((a, b) => b.value - a.value)

  if (barFilter.value === 'top20') {
    yearData = yearData.slice(0, 20)
  } else if (barFilter.value === 'oecd') {
    yearData = yearData.filter(d => OECD_CODES.has(d.code))
  }

  const margin = { top: 10, right: 60, bottom: 10, left: 130 }
  const barHeight = 26
  const gap = 4
  const height = yearData.length * (barHeight + gap) + margin.top + margin.bottom

  const maxVal = d3.max(yearData, d => d.value) || 30
  const x = d3.scaleLinear().domain([0, maxVal * 1.05]).range([margin.left, width - margin.right])
  const y = d3.scaleBand()
    .domain(yearData.map(d => d.code))
    .range([margin.top, height - margin.bottom])
    .padding(0.12)

  const colorScale = d3.scaleSequential(d3.interpolateRgbBasis(['#6366f1', '#8b5cf6', '#a855f7', '#d946ef', '#e11d48']))
    .domain([0, maxVal])

  if (!barSvg) {
    barSvg = d3.select(container).append('svg')
      .attr('width', width)
      .attr('height', height)
  } else {
    barSvg.attr('width', width).attr('height', height)
  }

  const bars = barSvg.selectAll('.bar-group').data(yearData, d => d.code)

  const enter = bars.enter().append('g').attr('class', 'bar-group')

  enter.append('text')
    .attr('class', 'bar-label')
    .attr('x', margin.left - 8)
    .attr('text-anchor', 'end')
    .attr('fill', '#57534e')
    .attr('font-size', '12px')
    .attr('dominant-baseline', 'central')

  enter.append('rect')
    .attr('class', 'bar-rect')
    .attr('rx', 4)
    .attr('ry', 4)
    .attr('height', y.bandwidth())
    .attr('x', margin.left)
    .attr('width', 0)

  enter.append('text')
    .attr('class', 'bar-value')
    .attr('fill', '#1c1917')
    .attr('font-size', '11px')
    .attr('font-weight', '600')
    .attr('dominant-baseline', 'central')

  const merged = enter.merge(bars)

  merged.transition().duration(500).ease(d3.easeCubicOut)
    .attr('transform', d => `translate(0, ${y(d.code) || 0})`)

  merged.select('.bar-label')
    .text(d => d.name.length > 16 ? d.name.slice(0, 16) + '...' : d.name)
    .transition().duration(500)
    .attr('y', y.bandwidth() / 2)

  merged.select('.bar-rect')
    .on('mouseenter', function(event, d) {
      d3.select(this).attr('opacity', 0.85)
      showTooltip(event, d.name, `${d.value.toFixed(2)} 吨CO2e/人`, `${currentYear.value}年`)
    })
    .on('mousemove', function(event) {
      tooltip.x = event.clientX + 15
      tooltip.y = event.clientY - 10
    })
    .on('mouseleave', function() {
      d3.select(this).attr('opacity', 1)
      hideTooltip()
    })
    .transition().duration(500).ease(d3.easeCubicOut)
    .attr('y', 0)
    .attr('height', y.bandwidth())
    .attr('width', d => Math.max(0, x(d.value) - margin.left))
    .attr('fill', d => colorScale(d.value))

  merged.select('.bar-value')
    .text(d => d.value.toFixed(1))
    .transition().duration(500)
    .attr('x', d => x(d.value) + 6)
    .attr('y', y.bandwidth() / 2)

  bars.exit().transition().duration(300).attr('opacity', 0).remove()
}

function drawLineChart() {
  if (!lineChartRef.value || !countryData.value.length) return

  const container = lineChartRef.value
  const width = container.clientWidth
  const height = 420
  const margin = { top: 20, right: 100, bottom: 40, left: 60 }

  if (!lineSvg) {
    lineSvg = d3.select(container).append('svg')
      .attr('width', width)
      .attr('height', height)
  } else {
    lineSvg.attr('width', width).attr('height', height)
  }

  lineSvg.selectAll('*').remove()

  const codes = Array.from(selectedCountries)
  if (codes.length === 0) {
    lineSvg.append('text')
      .attr('x', width / 2)
      .attr('y', height / 2)
      .attr('text-anchor', 'middle')
      .attr('fill', '#a8a29e')
      .attr('font-size', '14px')
      .text('请在上方选择要对比的国家')
    return
  }

  const seriesData = codes.map(code => {
    const points = countryData.value
      .filter(d => d.code === code)
      .sort((a, b) => a.year - b.year)
    return { code, name: points[0]?.name || code, points }
  }).filter(s => s.points.length > 0)

  const allPoints = seriesData.flatMap(s => s.points)
  const x = d3.scaleLinear()
    .domain([2014, 2023])
    .range([margin.left, width - margin.right])
  const y = d3.scaleLinear()
    .domain([0, d3.max(allPoints, d => d.value) * 1.1])
    .range([height - margin.bottom, margin.top])

  const xAxis = d3.axisBottom(x).ticks(10).tickFormat(d3.format('d'))
  const yAxis = d3.axisLeft(y).ticks(6)

  lineSvg.append('g')
    .attr('transform', `translate(0,${height - margin.bottom})`)
    .call(xAxis)
    .selectAll('text').attr('fill', '#78716c').attr('font-size', '11px')

  lineSvg.selectAll('.domain, .tick line').attr('stroke', '#e7e5e4')

  lineSvg.append('g')
    .attr('transform', `translate(${margin.left},0)`)
    .call(yAxis)
    .selectAll('text').attr('fill', '#78716c').attr('font-size', '11px')

  lineSvg.append('text')
    .attr('x', margin.left)
    .attr('y', margin.top - 6)
    .attr('fill', '#a8a29e')
    .attr('font-size', '11px')
    .text('吨 CO2e / 人')

  const gridLines = y.ticks(6)
  lineSvg.selectAll('.grid-line')
    .data(gridLines)
    .enter().append('line')
    .attr('x1', margin.left)
    .attr('x2', width - margin.right)
    .attr('y1', d => y(d))
    .attr('y2', d => y(d))
    .attr('stroke', '#f5f5f4')
    .attr('stroke-dasharray', '3,3')

  const line = d3.line()
    .x(d => x(d.year))
    .y(d => y(d.value))
    .curve(d3.curveMonotoneX)

  seriesData.forEach((series, i) => {
    const color = COLORS[codes.indexOf(series.code) % COLORS.length]

    lineSvg.append('path')
      .datum(series.points)
      .attr('fill', 'none')
      .attr('stroke', color)
      .attr('stroke-width', 2.5)
      .attr('d', line)
      .attr('stroke-dasharray', function() { return this.getTotalLength() })
      .attr('stroke-dashoffset', function() { return this.getTotalLength() })
      .transition().duration(800).delay(i * 100)
      .attr('stroke-dashoffset', 0)

    lineSvg.selectAll(`.dot-${series.code}`)
      .data(series.points)
      .enter().append('circle')
      .attr('cx', d => x(d.year))
      .attr('cy', d => y(d.value))
      .attr('r', 3.5)
      .attr('fill', color)
      .attr('stroke', '#ffffff')
      .attr('stroke-width', 2)
      .on('mouseenter', function(event, d) {
        d3.select(this).attr('r', 6)
        showTooltip(event, series.name, `${d.value.toFixed(2)} 吨CO2e/人`, `${d.year}年`)
      })
      .on('mousemove', function(event) {
        tooltip.x = event.clientX + 15
        tooltip.y = event.clientY - 10
      })
      .on('mouseleave', function() {
        d3.select(this).attr('r', 3.5)
        hideTooltip()
      })

    const lastPoint = series.points[series.points.length - 1]
    if (lastPoint) {
      lineSvg.append('text')
        .attr('x', x(lastPoint.year) + 8)
        .attr('y', y(lastPoint.value))
        .attr('fill', color)
        .attr('font-size', '11px')
        .attr('font-weight', '600')
        .attr('dominant-baseline', 'central')
        .text(series.name.length > 12 ? series.code : series.name)
    }
  })
}

function drawChangeChart() {
  if (!changeChartRef.value || !countryData.value.length) return

  const container = changeChartRef.value
  const width = container.clientWidth
  const margin = { top: 10, right: 80, bottom: 10, left: 150 }

  const codeYears = new Map()
  countryData.value.forEach(d => {
    if (!codeYears.has(d.code)) codeYears.set(d.code, new Map())
    codeYears.get(d.code).set(d.year, d)
  })

  let changeData = []
  codeYears.forEach((yearMap, code) => {
    const years = Array.from(yearMap.keys()).sort()
    const earliest = yearMap.get(years[0])
    const latest = yearMap.get(years[years.length - 1])
    if (earliest && latest && earliest.year !== latest.year) {
      const absChange = latest.value - earliest.value
      const pctChange = ((latest.value - earliest.value) / earliest.value) * 100
      changeData.push({
        code,
        name: latest.name,
        earliest: earliest.value,
        latest: latest.value,
        earliestYear: earliest.year,
        latestYear: latest.year,
        absChange,
        pctChange
      })
    }
  })

  if (changeFilter.value === 'improved') {
    changeData = changeData.filter(d => d.absChange < 0)
  } else if (changeFilter.value === 'worsened') {
    changeData = changeData.filter(d => d.absChange > 0)
  }

  if (changeSortBy.value === 'absolute') {
    changeData.sort((a, b) => a.absChange - b.absChange)
  } else {
    changeData.sort((a, b) => a.pctChange - b.pctChange)
  }

  changeData = changeData.slice(0, 35)

  const barHeight = 24
  const gap = 4
  const height = changeData.length * (barHeight + gap) + margin.top + margin.bottom

  if (!changeSvg) {
    changeSvg = d3.select(container).append('svg')
      .attr('width', width)
  }
  changeSvg.attr('width', width).attr('height', height)
  changeSvg.selectAll('*').remove()

  const sortVal = changeSortBy.value === 'absolute' ? 'absChange' : 'pctChange'
  const extent = d3.extent(changeData, d => d[sortVal])
  const maxAbs = Math.max(Math.abs(extent[0] || 0), Math.abs(extent[1] || 0))

  const x = d3.scaleLinear()
    .domain([-maxAbs * 1.1, maxAbs * 1.1])
    .range([margin.left, width - margin.right])

  const y = d3.scaleBand()
    .domain(changeData.map(d => d.code))
    .range([margin.top, height - margin.bottom])
    .padding(0.15)

  const zeroX = x(0)

  changeSvg.append('line')
    .attr('x1', zeroX).attr('x2', zeroX)
    .attr('y1', margin.top).attr('y2', height - margin.bottom)
    .attr('stroke', '#d6d3d1').attr('stroke-width', 1).attr('stroke-dasharray', '4,3')

  const groups = changeSvg.selectAll('.change-group')
    .data(changeData, d => d.code)
    .enter().append('g')
    .attr('class', 'change-group')
    .attr('transform', d => `translate(0,${y(d.code)})`)

  groups.append('text')
    .attr('x', margin.left - 8)
    .attr('y', y.bandwidth() / 2)
    .attr('text-anchor', 'end')
    .attr('dominant-baseline', 'central')
    .attr('fill', '#57534e')
    .attr('font-size', '11px')
    .text(d => d.name.length > 18 ? d.name.slice(0, 18) + '...' : d.name)

  groups.append('rect')
    .attr('x', d => {
      const val = d[sortVal]
      return val < 0 ? x(val) : zeroX
    })
    .attr('y', 0)
    .attr('width', d => Math.abs(x(d[sortVal]) - zeroX))
    .attr('height', y.bandwidth())
    .attr('rx', 3)
    .attr('fill', d => d[sortVal] < 0 ? '#16a34a' : '#dc2626')
    .attr('opacity', 0.75)
    .on('mouseenter', function(event, d) {
      d3.select(this).attr('opacity', 1)
      showTooltip(
        event,
        d.name,
        `${d.absChange > 0 ? '+' : ''}${d.absChange.toFixed(2)} 吨CO2e/人`,
        `${d.earliestYear}: ${d.earliest.toFixed(2)} → ${d.latestYear}: ${d.latest.toFixed(2)} (${d.pctChange > 0 ? '+' : ''}${d.pctChange.toFixed(1)}%)`
      )
    })
    .on('mousemove', function(event) {
      tooltip.x = event.clientX + 15
      tooltip.y = event.clientY - 10
    })
    .on('mouseleave', function() {
      d3.select(this).attr('opacity', 0.75)
      hideTooltip()
    })

  groups.append('text')
    .attr('x', d => {
      const val = d[sortVal]
      return val < 0 ? x(val) - 5 : x(val) + 5
    })
    .attr('y', y.bandwidth() / 2)
    .attr('text-anchor', d => d[sortVal] < 0 ? 'end' : 'start')
    .attr('dominant-baseline', 'central')
    .attr('fill', d => d[sortVal] < 0 ? '#16a34a' : '#dc2626')
    .attr('font-size', '10px')
    .attr('font-weight', '600')
    .text(d => {
      if (changeSortBy.value === 'percent') return `${d.pctChange > 0 ? '+' : ''}${d.pctChange.toFixed(1)}%`
      return `${d.absChange > 0 ? '+' : ''}${d.absChange.toFixed(1)}`
    })
}

let resizeTimer = null
function handleResize() {
  clearTimeout(resizeTimer)
  resizeTimer = setTimeout(() => {
    if (barSvg) { barSvg.remove(); barSvg = null }
    if (lineSvg) { lineSvg.remove(); lineSvg = null }
    if (changeSvg) { changeSvg.remove(); changeSvg = null }
    drawBarChart()
    drawLineChart()
    drawChangeChart()
  }, 200)
}

onMounted(async () => {
  await loadData()
  await nextTick()
  drawBarChart()
  drawLineChart()
  drawChangeChart()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  if (playInterval) clearInterval(playInterval)
  window.removeEventListener('resize', handleResize)
})

watch(currentYear, () => {
  drawBarChart()
})

watch(barFilter, () => {
  if (barSvg) { barSvg.remove(); barSvg = null }
  drawBarChart()
})

watch(selectedCountries, () => {
  drawLineChart()
}, { deep: true })

watch([changeFilter, changeSortBy], () => {
  if (changeSvg) { changeSvg.remove(); changeSvg = null }
  drawChangeChart()
})
</script>
