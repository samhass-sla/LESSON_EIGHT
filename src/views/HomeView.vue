<script setup lang="ts">
import { computed, ref } from 'vue'
import { Bar, Line } from 'vue-chartjs'
import {
  BarElement,
  CategoryScale,
  Chart as ChartJS,
  Filler,
  Legend,
  LinearScale,
  LineElement,
  PointElement,
  Title,
  Tooltip,
} from 'chart.js'
import monthlyMetrics from '../data/metrics.json'

type MetricEntry = {
  month: string
  label: string
  revenue: number
  visitors: number
  conversions: number
  orders: number
}

const metrics = monthlyMetrics as MetricEntry[]
const monthOptions = [
  { title: 'All months', value: 'all' },
  ...metrics.map((item) => ({
    title: item.label,
    value: item.month,
  })),
]
const selectedMonth = ref('all')

const selectedMonthIndex = computed(() =>
  selectedMonth.value === 'all' ? -1 : metrics.findIndex((item) => item.month === selectedMonth.value),
)

const previousMonthData = computed(() => {
  if (selectedMonth.value === 'all') {
    return metrics[metrics.length - 2] ?? metrics[metrics.length - 1]
  }

  const currentIndex = metrics.findIndex((item) => item.month === selectedMonth.value)
  return currentIndex > 0 ? metrics[currentIndex - 1] : null
})

const totalRevenue = computed(() =>
  metrics.reduce((sum, item) => sum + item.revenue, 0),
)
const totalVisitors = computed(() =>
  metrics.reduce((sum, item) => sum + item.visitors, 0),
)
const averageConversion = computed(() => {
  const total = metrics.reduce((sum, item) => sum + item.conversions, 0)
  return total / metrics.length
})
const totalOrders = computed(() =>
  metrics.reduce((sum, item) => sum + item.orders, 0),
)

const formatCurrency = (value: number) =>
  new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    maximumFractionDigits: 0,
  }).format(value)

const formatNumber = (value: number) =>
  new Intl.NumberFormat('en-US', {
    maximumFractionDigits: 0,
  }).format(value)

const formatPercent = (value: number) => `${value.toFixed(1)}%`

const getChange = (current: number, previous: number | null) => {
  if (!previous || previous === 0) return 0
  return ((current - previous) / previous) * 100
}

const mainCards = computed(() => {
  const currentRevenue = selectedMonth.value === 'all' ? totalRevenue.value : metrics[Number(selectedMonthIndex.value)]?.revenue ?? 0
  const currentVisitors = selectedMonth.value === 'all' ? totalVisitors.value : metrics[Number(selectedMonthIndex.value)]?.visitors ?? 0
  const currentConversions = selectedMonth.value === 'all' ? averageConversion.value : metrics[Number(selectedMonthIndex.value)]?.conversions ?? 0
  const currentOrders = selectedMonth.value === 'all' ? totalOrders.value : metrics[Number(selectedMonthIndex.value)]?.orders ?? 0

  const previousRevenue = previousMonthData.value?.revenue ?? null
  const previousVisitors = previousMonthData.value?.visitors ?? null
  const previousConversions = previousMonthData.value?.conversions ?? null
  const previousOrders = previousMonthData.value?.orders ?? null

  return [
    {
      title: 'Revenue',
      value: formatCurrency(currentRevenue),
      delta: getChange(currentRevenue, previousRevenue),
      accent: getChange(currentRevenue, previousRevenue) >= 0 ? 'success' : 'error',
      icon: getChange(currentRevenue, previousRevenue) >= 0 ? 'mdi-arrow-up' : 'mdi-arrow-down',
    },
    {
      title: 'Visitors',
      value: formatNumber(currentVisitors),
      delta: getChange(currentVisitors, previousVisitors),
      accent: getChange(currentVisitors, previousVisitors) >= 0 ? 'success' : 'error',
      icon: getChange(currentVisitors, previousVisitors) >= 0 ? 'mdi-arrow-up' : 'mdi-arrow-down',
    },
    {
      title: 'Conversions',
      value: formatPercent(currentConversions),
      delta: getChange(currentConversions, previousConversions),
      accent: getChange(currentConversions, previousConversions) >= 0 ? 'success' : 'error',
      icon: getChange(currentConversions, previousConversions) >= 0 ? 'mdi-arrow-up' : 'mdi-arrow-down',
    },
    {
      title: 'Orders',
      value: formatNumber(currentOrders),
      delta: getChange(currentOrders, previousOrders),
      accent: getChange(currentOrders, previousOrders) >= 0 ? 'success' : 'error',
      icon: getChange(currentOrders, previousOrders) >= 0 ? 'mdi-arrow-up' : 'mdi-arrow-down',
    },
  ]
})

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  BarElement,
  Title,
  Tooltip,
  Legend,
  Filler,
)

const hexToRgba = (hex: string, alpha: number) => {
  const normalized = hex.replace('#', '')
  const bigint = Number.parseInt(normalized, 16)
  const r = (bigint >> 16) & 255
  const g = (bigint >> 8) & 255
  const b = bigint & 255
  return `rgba(${r}, ${g}, ${b}, ${alpha})`
}

const selectedColor = '#312e81'
const faintSelectedColor = '#a5b4fc'
const selectedVisitorColor = '#0ea5e9'
const selectedConversionColor = '#10b981'

const revenueChartData = computed(() => ({
  labels: metrics.map((item) => item.label),
  datasets: [
    {
      label: 'Revenue',
      data: metrics.map((item) => item.revenue),
      backgroundColor: metrics.map((_, index) => {
        if (selectedMonthIndex.value === -1) return selectedColor
        return index === selectedMonthIndex.value ? selectedColor : hexToRgba(faintSelectedColor, 0.9)
      }),
      borderColor: metrics.map((_, index) => {
        if (selectedMonthIndex.value === -1) return selectedColor
        return index === selectedMonthIndex.value ? selectedColor : hexToRgba(faintSelectedColor, 0.95)
      }),
      borderRadius: 8,
      borderSkipped: false,
    },
  ],
}))

const visitorsChartData = computed(() => ({
  labels: metrics.map((item) => item.label),
  datasets: [
    {
      label: 'Visitors',
      data: metrics.map((item) => item.visitors),
      borderColor: metrics.map((_, index) =>
        index === selectedMonthIndex.value ? selectedVisitorColor : hexToRgba(selectedVisitorColor, 0.35),
      ),
      backgroundColor: metrics.map((_, index) =>
        index === selectedMonthIndex.value ? hexToRgba(selectedVisitorColor, 0.18) : hexToRgba(selectedVisitorColor, 0.04),
      ),
      pointBackgroundColor: metrics.map((_, index) =>
        index === selectedMonthIndex.value ? selectedVisitorColor : hexToRgba(selectedVisitorColor, 0.5),
      ),
      pointBorderColor: '#ffffff',
      pointBorderWidth: 2,
      pointRadius: metrics.map((_, index) => (index === selectedMonthIndex.value ? 5 : 3)),
      pointHoverRadius: metrics.map((_, index) => (index === selectedMonthIndex.value ? 6 : 4)),
      fill: true,
      tension: 0.35,
    },
  ],
}))

const conversionChartData = computed(() => ({
  labels: metrics.map((item) => item.label),
  datasets: [
    {
      label: 'Conversion rate',
      data: metrics.map((item) => item.conversions),
      borderColor: metrics.map((_, index) =>
        index === selectedMonthIndex.value ? selectedConversionColor : hexToRgba(selectedConversionColor, 0.3),
      ),
      backgroundColor: metrics.map((_, index) =>
        index === selectedMonthIndex.value ? hexToRgba(selectedConversionColor, 0.1) : hexToRgba(selectedConversionColor, 0.02),
      ),
      pointBackgroundColor: metrics.map((_, index) =>
        index === selectedMonthIndex.value ? selectedConversionColor : hexToRgba(selectedConversionColor, 0.5),
      ),
      pointBorderColor: '#ffffff',
      pointBorderWidth: 2,
      pointRadius: metrics.map((_, index) => (index === selectedMonthIndex.value ? 5 : 3)),
      pointHoverRadius: metrics.map((_, index) => (index === selectedMonthIndex.value ? 6 : 4)),
      fill: true,
      tension: 0.35,
    },
  ],
}))

const sharedChartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      display: false,
    },
  },
  scales: {
    x: {
      grid: {
        display: false,
      },
      ticks: {
        color: '#64748b',
      },
    },
    y: {
      grid: {
        color: '#e2e8f0',
      },
      ticks: {
        color: '#64748b',
      },
    },
  },
} as const

const conversionChartOptions = {
  ...sharedChartOptions,
  scales: {
    ...sharedChartOptions.scales,
    y: {
      ...sharedChartOptions.scales.y,
      ticks: {
        callback: (value: string | number) => `${value}%`,
        color: '#64748b',
      },
    },
  },
} as const
</script>

<template>
  <v-app>
    <v-app-bar flat elevation="1" class="dashboard-header">
      <v-container class="py-0">
        <v-row align="center" class="header-row">
          <v-col cols="12" md="6">
            <div class="dashboard-title">Dashboard</div>
          </v-col>
          <v-col cols="12" md="6" class="d-flex justify-end">
            <v-select
              v-model="selectedMonth"
              :items="monthOptions"
              item-title="title"
              item-value="value"
              variant="outlined"
              density="comfortable"
              hide-details
              class="month-picker"
              label="Month"
            />
          </v-col>
        </v-row>
      </v-container>
    </v-app-bar>

    <v-main class="bg-grey-lighten-4 main-shell">
      <v-container class="py-8 main-container">
        <v-row class="mb-8 metrics-row">
          <v-col v-for="card in mainCards" :key="card.title" cols="12" sm="6" md="3">
            <v-card class="pa-4 rounded-xl" flat elevation="0" color="white">
              <div class="d-flex align-center justify-space-between mb-3">
                <span class="metric-label">{{ card.title }}</span>
                <v-icon v-if="selectedMonth !== 'all'" :icon="card.icon" :color="card.accent" />
              </div>

              <div class="metric-value mb-2">{{ card.value }}</div>

              <div
                v-if="selectedMonth !== 'all'"
                class="metric-trend d-flex align-center gap-2"
                :class="card.accent === 'success' ? 'text-success' : 'text-error'"
              >
                <v-icon :icon="card.icon" size="small" />
                {{ Math.abs(card.delta).toFixed(1) }}% vs prev month
              </div>
            </v-card>
          </v-col>
        </v-row>

        <v-row class="mb-6">
          <v-col cols="12" lg="7">
            <v-card flat class="pa-4 rounded-xl" color="white">
              <div class="d-flex justify-space-between align-center mb-4">
                <div>
                  <div class="text-caption text-grey-darken-1">Revenue</div>
                  <div class="text-h6 font-weight-bold text-grey-darken-4">Monthly revenue</div>
                </div>
              </div>
              <div class="chart-panel">
                <Bar :data="revenueChartData" :options="sharedChartOptions" />
              </div>
            </v-card>
          </v-col>

          <v-col cols="12" lg="5">
            <v-card flat class="pa-4 rounded-xl" color="white">
              <div class="mb-4">
                <div class="text-caption text-grey-darken-1">Visitors</div>
                <div class="text-h6 font-weight-bold text-grey-darken-4">Traffic trend</div>
              </div>
              <div class="chart-panel compact">
                <Line :data="visitorsChartData" :options="sharedChartOptions" />
              </div>
            </v-card>
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12">
            <v-card flat class="pa-4 rounded-xl" color="white">
              <div class="mb-4">
                <div class="text-caption text-grey-darken-1">Conversions</div>
                <div class="text-h6 font-weight-bold text-grey-darken-4">Conversion rate trend</div>
              </div>
              <div class="chart-panel wide">
                <Line :data="conversionChartData" :options="conversionChartOptions" />
              </div>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<style scoped>
.main-shell {
  padding-top: 0;
}

.main-container {
  padding-top: 32px !important;
}

.metrics-row {
  margin-top: 14px;
}

.dashboard-header {
  position: relative !important;
  top: auto !important;
  z-index: 10;
  background: linear-gradient(to bottom, #eee9ff 0%, #e3e2e8 100%);
  overflow: hidden;
  padding-top: 10px;
  padding-bottom: 10px;
}

.header-row {
  min-height: 82px;
  position: relative;
  z-index: 1;
}

.dashboard-title {
  color: #29233f;
  font-family: "Avenir Next", "Helvetica Neue", sans-serif;
  font-size: 1.7rem;
  font-weight: 700;
  letter-spacing: 0.02em;
  line-height: 1.1;
}

:deep(.v-toolbar__content) {
  padding-top: 16px;
  padding-bottom: 20px;
  min-height: 118px !important;
  background: transparent;
}

:deep(.dashboard-header.v-app-bar) {
  position: relative !important;
  top: auto !important;
  transform: none !important;
}

:deep(.month-picker) {
  max-width: 180px;
  margin-bottom: 10px;
}

.metric-label {
  font-size: 0.72rem;
  font-weight: 600;
  letter-spacing: 0.02em;
  color: #616161;
}

.metric-value {
  font-size: clamp(1.8rem, 2vw, 2.5rem);
  font-weight: 800;
  line-height: 1.1;
  color: #1f2937;
}

.metric-trend {
  font-size: 0.7rem;
  font-weight: 600;
}

.chart-panel {
  position: relative;
  height: 300px;
}

.chart-panel.compact {
  height: 260px;
}

.chart-panel.wide {
  height: 280px;
}
</style>
