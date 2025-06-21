<template>
  <v-card class="mt-4">
    <v-card-title class="d-flex align-center">
      <v-icon class="mr-2">mdi-chart-line</v-icon>
      Population Evolution
      <v-spacer />
      <div class="d-flex align-center" style="width: 500px;">
        <v-slider
          v-model="maxDataPoints"
          :min="50"
          :max="5000"
          :step="50"
          color="primary"
          label="Max Generations"
          density="compact"
          hide-details
          class="mr-3"
        >
          <template v-slot:append>
            <v-text-field
              v-model.number="maxDataPoints"
              :min="200"
              :max="1000"
              type="number"
              style="width: 96px"
              density="compact"
              variant="outlined"
              hide-details
            />
          </template>
        </v-slider>
      </div>
    </v-card-title>
    <v-card-text>
      
      <div class="chart-container">
        <Line
          :data="chartData"
          :options="chartOptions"
          :height="400"
        />
      </div>
      
      <v-row class="mt-3" align="center">
        <v-col cols="auto">
          <v-btn
            color="warning"
            size="small"
            prepend-icon="mdi-refresh"
            @click="clearChart"
            variant="outlined"
          >
            Clear Chart
          </v-btn>
        </v-col>
        <v-col cols="auto">
          <v-chip color="info" variant="outlined" size="small">
            Max Population: {{ maxPopulation }}
          </v-chip>
        </v-col>
        <v-col cols="auto">
          <v-chip color="secondary" variant="outlined" size="small">
            Generations Tracked: {{ populationHistory.length }}
          </v-chip>
        </v-col>
      </v-row>
    </v-card-text>
  </v-card>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  Filler
} from 'chart.js'
import { Line } from 'vue-chartjs'

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  Filler
)

// Props
interface Props {
  liveCellCount: number
  generation: number
  isRunning: boolean
}

const props = defineProps<Props>()

// Population tracking - use plain array to avoid reactivity issues
let populationData: number[] = []
const populationHistory = ref<number[]>([])
const maxDataPoints = ref(200)
const lastGeneration = ref(-1)

// Computed properties
const maxPopulation = computed(() => {
  return populationHistory.value.length > 0 
    ? Math.max(...populationHistory.value)
    : 0
})

const chartData = computed(() => ({
  labels: populationHistory.value.map((_, index) => index.toString()),
  datasets: [
    {
      label: 'Live Cells',
      data: populationHistory.value,
      borderColor: '#4CAF50',
      backgroundColor: 'rgba(76, 175, 80, 0.1)',
      borderWidth: 2,
      pointRadius: 1,
      pointHoverRadius: 4,
      tension: 0.2,
      fill: true,
    }
  ]
}))

const chartOptions = computed(() => ({
  responsive: true,
  maintainAspectRatio: false,
  interaction: {
    intersect: false,
    mode: 'index' as const,
  },
  plugins: {
    legend: {
      display: true,
      labels: {
        color: '#fff',
        font: {
          size: 12
        }
      }
    },
    tooltip: {
      backgroundColor: 'rgba(0, 0, 0, 0.8)',
      titleColor: '#fff',
      bodyColor: '#fff',
      borderColor: '#4CAF50',
      borderWidth: 1,
      callbacks: {
        title: (context: any) => `Generation ${context[0].label}`,
        label: (context: any) => `Population: ${context.parsed.y}`,
      }
    }
  },
  scales: {
    x: {
      display: true,
      title: {
        display: true,
        text: 'Generation',
        color: '#fff',
        font: {
          size: 12
        }
      },
      ticks: {
        color: '#ccc',
        maxTicksLimit: 10,
        font: {
          size: 10
        }
      },
      grid: {
        color: 'rgba(255, 255, 255, 0.1)',
        drawBorder: false,
      }
    },
    y: {
      display: true,
      title: {
        display: true,
        text: 'Live Cells',
        color: '#fff',
        font: {
          size: 12
        }
      },
      ticks: {
        color: '#ccc',
        font: {
          size: 10
        }
      },
      grid: {
        color: 'rgba(255, 255, 255, 0.1)',
        drawBorder: false,
      },
      beginAtZero: true,
    }
  },
  elements: {
    point: {
      hoverBackgroundColor: '#4CAF50',
      hoverBorderColor: '#fff',
    }
  }
}))

// Add data point manually
const addDataPoint = (generation: number, population: number) => {
  if (generation === 0 && lastGeneration.value > 0) {
    // Reset case
    populationData = [population]
  } else if (generation > lastGeneration.value || populationData.length === 0) {
    // New generation or first time
    populationData.push(population)
    
    if (populationData.length > maxDataPoints.value) {
      populationData.shift()
    }
  }
  
  // Manually update the reactive reference
  populationHistory.value = [...populationData]
  lastGeneration.value = generation
}

// Watch props and update manually
watch(() => [props.generation, props.liveCellCount], ([gen, pop]) => {
  addDataPoint(gen, pop)
}, { immediate: true })

// Watch maxDataPoints changes to trim existing data if needed
watch(maxDataPoints, (newMax) => {
  if (populationData.length > newMax) {
    const excess = populationData.length - newMax
    populationData.splice(0, excess)
    populationHistory.value = [...populationData]
  }
})

// Clear chart data
const clearChart = () => {
  populationData = [props.liveCellCount]
  populationHistory.value = [...populationData]
  lastGeneration.value = 0
}

// Expose methods for parent component
defineExpose({
  clearChart
})
</script>

<style scoped>
.chart-container {
  position: relative;
  height: 400px;
  width: 100%;
}

.v-card {
  background-color: rgba(255, 255, 255, 0.05);
}

.v-chip {
  font-weight: 500;
}
</style>
