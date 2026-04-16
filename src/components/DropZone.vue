<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits<{
  (e: 'file-loaded', content: string, extension: string): void
}>()

const isDragging = ref(false)
const fileInputRef = ref<HTMLInputElement | null>(null)

const onDragOver = (event: DragEvent) => {
  event.preventDefault()
  isDragging.value = true
}

const onDragLeave = () => {
  isDragging.value = false
}

const onDrop = async (event: DragEvent) => {
  event.preventDefault()
  isDragging.value = false
  
  const file = event.dataTransfer?.files[0]
  if (!file) return

  await processFile(file)
}

const onClickZone = () => {
  // Create and trigger file input click
  const input = document.createElement('input')
  input.type = 'file'
  input.accept = '.js,.css,.html'
  input.onchange = (e: Event) => {
    const target = e.target as HTMLInputElement
    const file = target.files?.[0]
    if (file) processFile(file)
  }
  input.click()
}

const processFile = async (file: File) => {
  const extension = file.name.split('.').pop()?.toLowerCase() || ''
  const content = await file.text()
  emit('file-loaded', content, extension)
}
</script>

<template>
  <div 
    class="drop-zone p-5 text-center mb-4 rounded-3 border-2 border-dashed cursor-pointer"
    :class="{ 
      'border-primary bg-light shadow-sm': isDragging, 
      'border-secondary': !isDragging,
      'transition-all': true
    }"
    @dragover="onDragOver"
    @dragleave="onDragLeave"
    @drop="onDrop"
    @click="onClickZone"
  >
    <div class="d-flex flex-column align-items-center justify-content-center py-4">
      <i 
        class="bi bi-cloud-arrow-up fs-1 mb-3 transition-color" 
        :class="isDragging ? 'text-primary animate__animated animate__pulse' : 'text-secondary'"
      ></i>
      <h5 class="fw-bold mb-2">{{ isDragging ? 'Déposez votre fichier' : 'Glissez-déposez votre fichier' }}</h5>
      <p class="mb-0 text-muted">
        ou <span class="text-decoration-underline fw-semibold text-primary">cliquez pour sélectionner</span>
        <br>
        <small class="text-success fw-semibold">Formats supportés: .js, .css, .html</small>
      </p>
    </div>
  </div>
</template>

<style scoped>
.drop-zone {
  transition: all 0.3s ease;
  border-style: dashed !important;
  min-height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.transition-all {
  transition: all 0.3s ease;
}

.cursor-pointer {
  cursor: pointer;
}

.transition-color {
  transition: color 0.2s ease;
}

.drop-zone:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1) !important;
}
</style>
