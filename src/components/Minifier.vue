<script setup lang="ts">
defineOptions({
  name: 'CodeMinifier'
})
import { ref, computed } from 'vue'
import DropZone from './DropZone.vue'
// Imports for minification (requires vite-plugin-node-polyfills for Node APIs)
import { minify as terserMinify } from 'terser'
import CleanCSS from 'clean-css'
import { minify as htmlMinify } from 'html-minifier-terser'

const sourceCode = ref('')
const minifiedCode = ref('')
const selectedFormat = ref('js')
const isLoading = ref(false)
const errorMessage = ref('')

const originalSize = computed(() => new Blob([sourceCode.value]).size)
const finalSize = computed(() => new Blob([minifiedCode.value]).size)
const reductionPercentage = computed(() => {
  if (originalSize.value === 0) return 0
  const reduction = ((originalSize.value - finalSize.value) / originalSize.value) * 100
  return reduction.toFixed(2)
})

const formatBytes = (bytes: number) => {
  if (bytes === 0) return '0 B'
  const k = 1024
  const sizes = ['B', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
}

const handleFileLoaded = (content: string, extension: string) => {
  sourceCode.value = content
  if (['js', 'css', 'html'].includes(extension)) {
    selectedFormat.value = extension
  }
}

const performMinification = async () => {
  if (!sourceCode.value.trim()) return

  isLoading.value = true
  errorMessage.value = ''

  try {
    if (selectedFormat.value === 'js') {
      const result = await terserMinify(sourceCode.value, { sourceMap: false })
      minifiedCode.value = result.code || ''
    } else if (selectedFormat.value === 'css') {
      const result = new CleanCSS().minify(sourceCode.value)
      if (result.errors.length) throw new Error(result.errors[0])
      minifiedCode.value = result.styles
    } else if (selectedFormat.value === 'html') {
      const result = await htmlMinify(sourceCode.value, {
        collapseWhitespace: true,
        removeComments: true,
        minifyJS: true,
        minifyCSS: true
      })
      minifiedCode.value = result
    }
  } catch (error: any) {
    errorMessage.value = error.message || 'Erreur lors de la minification'
  } finally {
    isLoading.value = false
  }
}

const copyToClipboard = async () => {
  try {
    await navigator.clipboard.writeText(minifiedCode.value)
    alert('Code copié dans le presse-papiers !')
  } catch (err) {
    console.error('Erreur de copie: ', err)
  }
}

const downloadMinifiedFile = () => {
  const element = document.createElement('a')
  const file = new Blob([minifiedCode.value], { type: 'text/plain;charset=utf-8' })
  element.href = URL.createObjectURL(file)
  const ext = selectedFormat.value
  element.download = `minified.${ext}`
  document.body.appendChild(element)
  element.click()
  document.body.removeChild(element)
}
</script>

<template>
  <div class="container py-5">
    <div class="row justify-content-center">
      <div class="col-lg-10">
        <h2 class="text-center mb-4 text-primary">SwiftShrink <i class="bi bi-lightning-charge"></i></h2>

        <DropZone @file-loaded="handleFileLoaded" />

        <div class="card shadow-sm mb-4">
          <div class="card-body">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h5 class="card-title mb-0">Code Source</h5>
              <select v-model="selectedFormat" class="form-select w-auto">
                <option value="js">JavaScript (.js)</option>
                <option value="css">CSS (.css)</option>
                <option value="html">HTML (.html)</option>
              </select>
            </div>

            <textarea v-model="sourceCode" class="form-control font-monospace text-bg-light" rows="8"
              placeholder="Collez votre code ici ou via Drag & Drop..."></textarea>

            <div v-if="errorMessage" class="alert alert-danger mt-3 mb-0" role="alert">
              {{ errorMessage }}
            </div>

            <div class="text-center mt-4">
              <button @click="performMinification" class="btn btn-primary px-5 py-2 fw-bold"
                :disabled="isLoading || !sourceCode">
                <span v-if="isLoading" class="spinner-border spinner-border-sm me-2" role="status"
                  aria-hidden="true"></span>
                Minifier le code
              </button>
            </div>
          </div>
        </div>

        <div v-if="minifiedCode" class="card shadow-sm border-success">
          <div class="card-body">
            <div class="d-flex justify-content-between align-items-center mb-3">
              <h5 class="card-title mb-0 text-success">Résultat</h5>
              <div class="badge bg-success fs-6">
                Gain : {{ reductionPercentage }}%
              </div>
            </div>

            <textarea :value="minifiedCode" class="form-control font-monospace mb-3" rows="6" readonly></textarea>

            <div class="d-flex justify-content-between align-items-center">
              <div class="text-muted small">
                Taille originelle: <strong>{{ formatBytes(originalSize) }}</strong> &nbsp;|&nbsp;
                Taille finale: <strong>{{ formatBytes(finalSize) }}</strong>
              </div>
              <div class="d-flex gap-2">
                <button @click="copyToClipboard" class="btn btn-outline-secondary">
                  <i class="bi bi-clipboard"></i> Copier
                </button>
                <button @click="downloadMinifiedFile" class="btn btn-success">
                  <i class="bi bi-download"></i> Télécharger .min
                </button>
              </div>
            </div>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<style scoped>
textarea {
  resize: vertical;
}

.font-monospace {
  font-family: 'Courier New', Courier, monospace;
  font-size: 0.9rem;
}
</style>
