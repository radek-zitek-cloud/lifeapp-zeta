<template>
  <div class="life-game">
    <v-card class="mb-4">
      <v-card-title>
        <span class="text-h5">Conway's Game of Life</span>
      </v-card-title>
      <v-card-text>
        <v-row align="center" class="mb-3">
          <v-col cols="auto">
            <v-btn
              :color="isRunning ? 'error' : 'success'"
              :prepend-icon="isRunning ? 'mdi-pause' : 'mdi-play'"
              @click="toggleSimulation"
            >
              {{ isRunning ? 'Pause' : 'Start' }}
            </v-btn>
          </v-col>
          <v-col cols="auto">
            <v-btn
              color="warning"
              prepend-icon="mdi-refresh"
              @click="resetGrid"
            >
              Reset
            </v-btn>
          </v-col>
          <v-col cols="auto">
            <v-btn
              color="info"
              prepend-icon="mdi-shuffle"
              @click="randomizeGrid"
            >
              Random
            </v-btn>
          </v-col>
          <v-col cols="auto">
            <v-btn
              color="secondary"
              prepend-icon="mdi-step-forward"
              @click="stepForward"
              :disabled="isRunning"
            >
              Step
            </v-btn>
          </v-col>
        </v-row>

        <v-row align="center" class="mb-3">
          <v-col cols="12" sm="6" md="4">
            <v-slider
              v-model="speed"
              :min="SPEED_MIN"
              :max="SPEED_MAX"
              :step="SPEED_STEP"
              label="Speed (ms)"
              thumb-label
              prepend-icon="mdi-speedometer"
            />
          </v-col>
          <v-col cols="auto">
            <v-chip color="primary" variant="outlined">
              Generation: {{ generation }}
            </v-chip>
          </v-col>
          <v-col cols="auto">
            <v-chip color="success" variant="outlined">
              Live Cells: {{ liveCellCount }}
            </v-chip>
          </v-col>
        </v-row>

        <v-row align="center" class="mb-3">
          <v-col cols="12" sm="6" md="4">
            <v-slider
              v-model="gridWidth"
              :min="GRID_MIN"
              :max="GRID_MAX"
              :step="GRID_STEP"
              label="Grid Width"
              thumb-label
              prepend-icon="mdi-resize-horizontal"
              :disabled="isRunning"
              @update:model-value="onGridSizeChange"
            />
          </v-col>
          <v-col cols="12" sm="6" md="4">
            <v-slider
              v-model="gridHeight"
              :min="GRID_MIN"
              :max="GRID_MAX"
              :step="GRID_STEP"
              label="Grid Height"
              thumb-label
              prepend-icon="mdi-resize-vertical"
              :disabled="isRunning"
              @update:model-value="onGridSizeChange"
            />
          </v-col>
          <v-col cols="auto">
            <v-chip color="info" variant="outlined">
              Grid: {{ gridWidth }} × {{ gridHeight }}
            </v-chip>
          </v-col>
        </v-row>

        <v-row align="center" class="mb-3">
          <v-col cols="12" sm="6" md="4">
            <v-text-field
              v-model="birthRule"
              label="Birth Rule (B)"
              prepend-icon="mdi-plus-circle"
              placeholder="3"
              hint="Neighbors needed for birth (e.g., '3' or '24')"
              persistent-hint
              density="compact"
              :disabled="isRunning"
              :rules="[validateRule]"
            />
          </v-col>
          <v-col cols="12" sm="6" md="4">
            <v-text-field
              v-model="survivalRule"
              label="Survival Rule (S)"
              prepend-icon="mdi-heart"
              placeholder="23"
              hint="Neighbors needed for survival (e.g., '23' or '135')"
              persistent-hint
              density="compact"
              :disabled="isRunning"
              :rules="[validateRule]"
            />
          </v-col>
          <v-col cols="auto">
            <v-chip color="purple" variant="outlined">
              Rule: B{{ birthRule }}/S{{ survivalRule }}
            </v-chip>
          </v-col>
          <v-col cols="auto">
            <v-btn
              color="secondary"
              size="small"
              prepend-icon="mdi-restore"
              @click="resetToConwayRules"
              :disabled="isRunning"
              variant="outlined"
            >
              Conway
            </v-btn>
          </v-col>
        </v-row>
      </v-card-text>
    </v-card>

    <v-card>
      <v-card-text>
        <div class="grid-container" ref="gridContainer">
          <canvas
            ref="canvas"
            :width="canvasWidth"
            :height="canvasHeight"
            @click="handleCanvasClick"
            @mousemove="handleCanvasMouseMove"
            class="game-canvas"
          />
        </div>
      </v-card-text>
    </v-card>

    <population-chart
      :live-cell-count="liveCellCount"
      :generation="generation"
      :is-running="isRunning"
      ref="populationChart"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed, watch } from 'vue'
import PopulationChart from './PopulationChart.vue'

// Slider configuration constants
const SPEED_MIN = 50
const SPEED_MAX = 1000
const SPEED_STEP = 50

const GRID_MIN = 10
const GRID_MAX = 128
const GRID_STEP = 1

// Game state
const cellSize = 8

// Reactive state
const grid = ref<boolean[][]>([])
const isRunning = ref(false)
const generation = ref(0)
const speed = ref(200)
const gridWidth = ref(64)
const gridHeight = ref(64)
const birthRule = ref('3')
const survivalRule = ref('23')
const canvas = ref<HTMLCanvasElement>()
const gridContainer = ref<HTMLDivElement>()
const populationChart = ref<InstanceType<typeof PopulationChart>>()

// Animation frame and interval
let intervalId: number | null = null

// Computed properties
const canvasWidth = computed(() => gridWidth.value * cellSize)
const canvasHeight = computed(() => gridHeight.value * cellSize)

const liveCellCount = computed(() => {
  return grid.value.reduce((count, row) => {
    return count + row.reduce((rowCount, cell) => rowCount + (cell ? 1 : 0), 0)
  }, 0)
})

// Initialize empty grid
const initializeGrid = (): boolean[][] => {
  return Array(gridHeight.value).fill(null).map(() => Array(gridWidth.value).fill(false))
}

// Reset grid to empty state
const resetGrid = () => {
  grid.value = initializeGrid()
  generation.value = 0
  if (isRunning.value) {
    toggleSimulation()
  }
  populationChart.value?.clearChart()
  drawGrid()
}

// Randomize grid with ~30% chance of live cells
const randomizeGrid = () => {
  for (let i = 0; i < gridHeight.value; i++) {
    for (let j = 0; j < gridWidth.value; j++) {
      grid.value[i][j] = Math.random() < 0.3
    }
  }
  generation.value = 0
  if (isRunning.value) {
    toggleSimulation()
  }
  populationChart.value?.clearChart()
  drawGrid()
}

// Handle grid size changes
const onGridSizeChange = () => {
  if (isRunning.value) {
    toggleSimulation()
  }
  
  // Preserve existing cells when resizing
  const oldGrid = [...grid.value.map(row => [...row])]
  grid.value = initializeGrid()
  
  // Copy over existing cells that fit in new dimensions
  const minHeight = Math.min(oldGrid.length, gridHeight.value)
  const minWidth = Math.min(oldGrid[0]?.length || 0, gridWidth.value)
  
  for (let i = 0; i < minHeight; i++) {
    for (let j = 0; j < minWidth; j++) {
      grid.value[i][j] = oldGrid[i][j]
    }
  }
  
  generation.value = 0
  populationChart.value?.clearChart()
  drawGrid()
}

// Count live neighbors for a cell
const countNeighbors = (row: number, col: number): number => {
  let count = 0
  for (let i = -1; i <= 1; i++) {
    for (let j = -1; j <= 1; j++) {
      if (i === 0 && j === 0) continue
      const newRow = row + i
      const newCol = col + j
      if (newRow >= 0 && newRow < gridHeight.value && newCol >= 0 && newCol < gridWidth.value) {
        if (grid.value[newRow][newCol]) count++
      }
    }
  }
  return count
}

// Apply Conway's Game of Life rules (configurable)
const updateGrid = () => {
  const newGrid = initializeGrid()
  
  // Parse birth and survival rules
  const birthNumbers = parseBirthSurvivalRule(birthRule.value)
  const survivalNumbers = parseBirthSurvivalRule(survivalRule.value)
  
  for (let i = 0; i < gridHeight.value; i++) {
    for (let j = 0; j < gridWidth.value; j++) {
      const neighbors = countNeighbors(i, j)
      const isAlive = grid.value[i][j]
      
      // Apply configurable rules:
      // 1. Live cell survives if neighbor count is in survival rule
      // 2. Dead cell is born if neighbor count is in birth rule
      if (isAlive && survivalNumbers.includes(neighbors)) {
        newGrid[i][j] = true
      } else if (!isAlive && birthNumbers.includes(neighbors)) {
        newGrid[i][j] = true
      }
    }
  }
  
  grid.value = newGrid
  generation.value++
  drawGrid()
}

// Parse birth/survival rule string into array of numbers
const parseBirthSurvivalRule = (rule: string): number[] => {
  return rule.split('').map(char => parseInt(char, 10)).filter(num => !isNaN(num) && num >= 0 && num <= 8)
}

// Validate rule input
const validateRule = (value: string): boolean | string => {
  if (!value) return 'Rule cannot be empty'
  if (!/^[0-8]*$/.test(value)) return 'Rule must contain only digits 0-8'
  if (value.length === 0) return 'Rule must contain at least one digit'
  return true
}

// Reset to Conway's original rules
const resetToConwayRules = () => {
  birthRule.value = '3'
  survivalRule.value = '23'
}

// Step forward one generation
const stepForward = () => {
  updateGrid()
}

// Toggle simulation running state
const toggleSimulation = () => {
  isRunning.value = !isRunning.value
  
  if (isRunning.value) {
    startAnimation()
  } else {
    stopAnimation()
  }
}

// Start animation loop
const startAnimation = () => {
  if (intervalId) clearInterval(intervalId)
  
  intervalId = window.setInterval(() => {
    updateGrid()
  }, speed.value)
}

// Stop animation loop
const stopAnimation = () => {
  if (intervalId) {
    clearInterval(intervalId)
    intervalId = null
  }
}

// Watch speed changes and restart animation if running
watch(speed, () => {
  if (isRunning.value) {
    stopAnimation()
    startAnimation()
  }
})

// Draw the grid on canvas
const drawGrid = () => {
  if (!canvas.value) return
  
  const ctx = canvas.value.getContext('2d')
  if (!ctx) return
  
  // Update canvas size if needed
  canvas.value.width = canvasWidth.value
  canvas.value.height = canvasHeight.value
  
  // Clear canvas
  ctx.fillStyle = '#121212'
  ctx.fillRect(0, 0, canvasWidth.value, canvasHeight.value)
  
  // Draw grid lines
  ctx.strokeStyle = '#333'
  ctx.lineWidth = 0.5
  
  for (let i = 0; i <= gridWidth.value; i++) {
    ctx.beginPath()
    ctx.moveTo(i * cellSize, 0)
    ctx.lineTo(i * cellSize, canvasHeight.value)
    ctx.stroke()
  }
  
  for (let i = 0; i <= gridHeight.value; i++) {
    ctx.beginPath()
    ctx.moveTo(0, i * cellSize)
    ctx.lineTo(canvasWidth.value, i * cellSize)
    ctx.stroke()
  }
  
  // Draw live cells
  ctx.fillStyle = '#4CAF50'
  for (let i = 0; i < gridHeight.value; i++) {
    for (let j = 0; j < gridWidth.value; j++) {
      if (grid.value[i][j]) {
        ctx.fillRect(j * cellSize + 1, i * cellSize + 1, cellSize - 2, cellSize - 2)
      }
    }
  }
}

// Handle canvas click to toggle cells
const handleCanvasClick = (event: MouseEvent) => {
  if (!canvas.value) return
  
  const rect = canvas.value.getBoundingClientRect()
  const x = event.clientX - rect.left
  const y = event.clientY - rect.top
  
  const col = Math.floor(x / cellSize)
  const row = Math.floor(y / cellSize)
  
  if (row >= 0 && row < gridHeight.value && col >= 0 && col < gridWidth.value) {
    grid.value[row][col] = !grid.value[row][col]
    drawGrid()
  }
}

// Handle mouse move for potential drawing mode (future enhancement)
const handleCanvasMouseMove = (_event: MouseEvent) => {
  // Future: implement drag to draw functionality
}

// Initialize on component mount
onMounted(() => {
  grid.value = initializeGrid()
  randomizeGrid() // Start with a random pattern
  drawGrid()
})

// Cleanup on component unmount
onUnmounted(() => {
  stopAnimation()
})
</script>

<style scoped>
.life-game {
  max-width: 1200px;
  margin: 0 auto;
}

.grid-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 16px;
  overflow-x: auto;
}

.game-canvas {
  border: 2px solid #333;
  border-radius: 4px;
  cursor: crosshair;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  max-width: 100%;
  height: auto;
}

.game-canvas:hover {
  border-color: #1976D2;
}

.v-card {
  background-color: rgba(255, 255, 255, 0.05);
}

.v-chip {
  font-weight: 500;
}

@media (max-width: 768px) {
  .grid-container {
    padding: 8px;
  }
  
  .game-canvas {
    max-width: calc(100vw - 32px);
  }
}
</style>
