<template>
  <div class="pixel-art-creator">
    <div class="controls">
      <div class="color-picker">
        <input type="color" v-model="currentColor" />
        <span>Couleur sélectionnée</span>
      </div>
      <div class="tools">
        <button 
          :class="{ active: currentTool === 'brush' }" 
          @click="currentTool = 'brush'"
        >
          🖌️ Pinceau
        </button>
        <button 
          :class="{ active: currentTool === 'eraser' }" 
          @click="currentTool = 'eraser'"
        >
          ❌ Gomme
        </button>
      </div>
      <div class="grid-size">
        <label>
          Taille de la grille:
          <select v-model="gridSize">
            <option value="8">8x8</option>
            <option value="16">16x16</option>
            <option value="32">32x32</option>
          </select>
        </label>
      </div>
    </div>

    <div 
      class="pixel-grid"
      :style="{ 
        'grid-template-columns': `repeat(${gridSize}, 1fr)`,
        'grid-template-rows': `repeat(${gridSize}, 1fr)`
      }"
    >
      <div
        v-for="(pixel, index) in pixels"
        :key="index"
        class="pixel"
        :style="{ backgroundColor: pixel }"
        @mousedown="startDrawing(index)"
        @mouseover="draw(index)"
        @mouseup="stopDrawing"
      ></div>
    </div>

    <div class="actions">
      <button @click="clearCanvas">Effacer tout</button>
      <button @click="exportImage">Exporter en PNG</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const currentColor = ref('#000000')
const currentTool = ref('brush')
const gridSize = ref(16)
const isDrawing = ref(false)

const pixels = ref(Array(gridSize.value * gridSize.value).fill('#ffffff'))

const startDrawing = (index) => {
  isDrawing.value = true
  draw(index)
}

const draw = (index) => {
  if (!isDrawing.value) return
  pixels.value[index] = currentTool.value === 'brush' ? currentColor.value : '#ffffff'
}

const stopDrawing = () => {
  isDrawing.value = false
}

const clearCanvas = () => {
  pixels.value = pixels.value.map(() => '#ffffff')
}

watch(gridSize, (newSize) => {
  pixels.value = Array(newSize * newSize).fill('#ffffff')
})

const exportImage = async () => {
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')
  const pixelSize = 20
  
  canvas.width = gridSize.value * pixelSize
  canvas.height = gridSize.value * pixelSize
  
  pixels.value.forEach((color, index) => {
    const x = (index % gridSize.value) * pixelSize
    const y = Math.floor(index / gridSize.value) * pixelSize
    
    ctx.fillStyle = color
    ctx.fillRect(x, y, pixelSize, pixelSize)
  })
  
  const link = document.createElement('a')
  link.download = 'pixel-art.png'
  link.href = canvas.toDataURL()
  link.click()
}
</script>

<style lang="scss" scoped>
.pixel-art-creator {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
  padding: 2rem;
  max-width: 800px;
  margin: 0 auto;
  font-family: 'Montserrat', sans-serif;
}

.controls {
  display: flex;
  gap: 2rem;
  align-items: center;

  .color-picker {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.5rem;

    input[type="color"] {
      width: 50px;
      height: 50px;
      padding: 0;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
  }

  .tools {
    display: flex;
    gap: 1rem;

    button {
      padding: 0.5rem 1rem;
      border: 2px solid #ddd;
      border-radius: 4px;
      background: white;
      cursor: pointer;
      transition: all 0.2s;

      &.active {
        background: #e0e0e0;
        border-color: #666;
      }
    }
  }
}

.pixel-grid {
  display: grid;
  gap: 1px;
  background: #ddd;
  border: 1px solid #999;
  width: 100%;
  max-width: 600px;
  aspect-ratio: 1;
}

.pixel {
  background: #fff;
  aspect-ratio: 1;
  cursor: pointer;
  transition: background-color 0.1s;

  &:hover {
    opacity: 0.8;
  }
}

.actions {
  display: flex;
  gap: 1rem;

  button {
    padding: 0.75rem 1.5rem;
    border: none;
    border-radius: 4px;
    background: #2196f3;
    color: white;
    cursor: pointer;
    transition: background-color 0.2s;

    &:hover {
      background: #1976d2;
    }
  }
}
</style>