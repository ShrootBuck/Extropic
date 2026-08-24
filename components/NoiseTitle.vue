<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from 'vue'

const canvas = ref<HTMLCanvasElement | null>(null)
let frame = 0

onMounted(async () => {
  const c = canvas.value!
  const ctx = c.getContext('2d')!
  const source = document.createElement('canvas')
  const sourceCtx = source.getContext('2d')!
  const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  let width = 0
  let height = 0
  let rowHeight = 1

  await document.fonts.ready

  const fit = () => {
    const cssWidth = Math.max(1, c.clientWidth)
    const cssHeight = Math.max(1, c.clientHeight)
    const dpr = Math.min(2, window.devicePixelRatio || 1)
    width = Math.round(cssWidth * dpr)
    height = Math.round(cssHeight * dpr)
    rowHeight = Math.max(1, Math.round(dpr))
    c.width = width
    c.height = height
    source.width = width
    source.height = height

    let fontSize = height * 0.83
    sourceCtx.font = `700 ${fontSize}px "Space Grotesk", sans-serif`
    let metrics = sourceCtx.measureText('NOISE')
    if (metrics.width > width - 30 * dpr) {
      fontSize *= (width - 30 * dpr) / metrics.width
      sourceCtx.font = `700 ${fontSize}px "Space Grotesk", sans-serif`
      metrics = sourceCtx.measureText('NOISE')
    }

    const x = 8 * dpr
    const y = (height + metrics.actualBoundingBoxAscent - metrics.actualBoundingBoxDescent) / 2
    sourceCtx.clearRect(0, 0, width, height)
    sourceCtx.fillStyle = '#e9ecf4'
    sourceCtx.fillText('NOISE', x, y)
  }

  const draw = () => {
    ctx.clearRect(0, 0, width, height)
    for (let y = 0; y < height; y += rowHeight) {
      const offset = Math.floor(0.34 * (Math.random() - 0.5) * 30 * rowHeight)
      ctx.drawImage(source, 0, y, width, rowHeight, offset, y, width, rowHeight)
    }
    if (!reducedMotion) frame = requestAnimationFrame(draw)
  }

  fit()
  const resizeObserver = new ResizeObserver(() => {
    fit()
    if (reducedMotion) draw()
  })
  resizeObserver.observe(c)
  draw()

  onBeforeUnmount(() => {
    cancelAnimationFrame(frame)
    resizeObserver.disconnect()
  })
})
</script>

<template>
  <span class="noise-title"><canvas ref="canvas" /></span>
</template>

<style scoped>
.noise-title {
  display: inline-block;
  overflow: visible;
}
canvas {
  display: block;
  width: 100%;
  height: 100%;
}
</style>
