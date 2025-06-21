# Life App - Cellular Automata

A modern implementation of Conway's Game of Life built with Vue.js 3, TypeScript, and Vuetify.

## Features

- **Interactive Variable Grid**: Adjustable grid size from 10×10 to 128×128 cells
- **Dynamic Grid Controls**: Separate sliders for width and height adjustment
- **Interactive Cell Editing**: Click cells to toggle their state
- **Real-time Simulation**: Watch the cellular automata evolve
- **Population Chart**: Live visualization of cell population over time
- **Adjustable Speed**: Control simulation speed from 50ms to 1000ms intervals
- **Step-by-Step Mode**: Manual step-through for detailed observation
- **Random Generation**: Generate random starting patterns
- **Live Statistics**: Track generation count and live cell count
- **Responsive Design**: Adapts to different screen sizes
- **Modern UI**: Beautiful dark theme with Vuetify components

## Game Rules (Conway's Game of Life)

1. **Survival**: A live cell with 2 or 3 live neighbors survives
2. **Birth**: A dead cell with exactly 3 live neighbors becomes alive
3. **Death**: All other cells die or remain dead

## Getting Started

### Prerequisites

- Node.js (version 18 or higher)
- npm or yarn

### Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd lifeapp-zeta
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Start the development server:
   ```bash
   npm run dev
   ```

4. Open your browser and navigate to `http://localhost:5173`

### Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build

## How to Use

1. **Start/Pause**: Click the green "Start" button to begin the simulation
2. **Reset**: Clear the grid and stop the simulation
3. **Random**: Generate a random starting pattern (30% cell density)
4. **Step**: Manually advance one generation (only when paused)
5. **Speed Control**: Adjust the simulation speed using the slider
6. **Grid Size**: Adjust the width and height of the grid independently (10-128 cells)
7. **Interactive Editing**: Click any cell to toggle its state
8. **Grid Info**: Monitor the current grid dimensions and live cell count
9. **Population Chart**: Watch the real-time population evolution graph below the grid
10. **Clear Chart**: Reset the population tracking history

## Technologies Used

- **Vue.js 3** - Progressive JavaScript framework
- **TypeScript** - Static type checking
- **Vuetify 3** - Material Design component library
- **Chart.js** - Interactive charting library for population visualization
- **Vue Chart.js** - Vue.js wrapper for Chart.js
- **Vite** - Fast build tool and development server
- **HTML5 Canvas** - High-performance grid rendering

## Grid Specifications

- **Variable Size**: Adjustable from 10×10 to 128×128 cells
- **Cell Size**: 8×8 pixels per cell
- **Canvas Size**: Dynamic based on grid dimensions
- **Rendering**: Optimized canvas-based rendering for smooth performance
- **Responsive**: Grid scales appropriately on different screen sizes

Enjoy exploring the fascinating world of cellular automata!
