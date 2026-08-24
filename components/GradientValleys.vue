<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from 'vue'

const canvas = ref<HTMLCanvasElement | null>(null)
let frame = 0

type Point = { x: number; y: number }

function energy(x: number, y: number) {
  const basinA = Math.exp(-(((x + 0.68) / 0.48) ** 2 + ((y + 0.12) / 0.42) ** 2))
  const basinB = Math.exp(-(((x - 0.62) / 0.38) ** 2 + ((y - 0.34) / 0.5) ** 2))
  const ridge = Math.exp(-(((x - 0.02) / 0.24) ** 2 + ((y + 0.02) / 0.92) ** 2))
  return 0.72 + 0.2 * (x * x + y * y) - 1.5 * basinA - 1.08 * basinB + 0.32 * ridge
}

function gradient(x: number, y: number) {
  const d = 0.002
  return {
    x: (energy(x + d, y) - energy(x - d, y)) / (2 * d),
    y: (energy(x, y + d) - energy(x, y - d)) / (2 * d),
  }
}

function descentPath(start: Point) {
  const path = [start]
  let { x, y } = start
  for (let i = 0; i < 72; i++) {
    const g = gradient(x, y)
    const scale = 0.038 / Math.max(1, Math.hypot(g.x, g.y))
    x -= g.x * scale
    y -= g.y * scale
    path.push({ x, y })
  }
  return path
}

const paths = [
  { x: -1.42, y: -0.82 },
  { x: 1.36, y: -0.78 },
  { x: 1.38, y: 0.88 },
  { x: -1.34, y: 0.9 },
  { x: 0.12, y: -1.02 },
  { x: 0.02, y: 1.02 },
].map(descentPath)

onMounted(() => {
  const c = canvas.value!
  const ctx = c.getContext('2d')!
  const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  let width = 0
  let height = 0
  let dpr = 1

  const fit = () => {
    width = c.clientWidth
    height = c.clientHeight
    dpr = Math.min(2, window.devicePixelRatio || 1)
    c.width = Math.round(width * dpr)
    c.height = Math.round(height * dpr)
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
  }

  const project = (x: number, y: number, z = energy(x, y)) => ({
    x: width * 0.5 + x * width * 0.235 + y * width * 0.145,
    y: height * 0.59 + y * height * 0.17 - (z - 0.38) * height * 0.19,
  })

  const draw = (now: number) => {
    ctx.clearRect(0, 0, width, height)

    const glow = ctx.createRadialGradient(width * 0.51, height * 0.62, 0, width * 0.51, height * 0.62, width * 0.52)
    glow.addColorStop(0, 'rgba(56,189,248,0.10)')
    glow.addColorStop(0.45, 'rgba(167,139,250,0.045)')
    glow.addColorStop(1, 'rgba(5,6,10,0)')
    ctx.fillStyle = glow
    ctx.fillRect(0, 0, width, height)

    const nx = 42
    const ny = 30
    const xmin = -1.65
    const xmax = 1.65
    const ymin = -1.12
    const ymax = 1.12
    const dx = (xmax - xmin) / nx
    const dy = (ymax - ymin) / ny

    for (let j = 0; j < ny; j++) {
      const y0 = ymin + j * dy
      const y1 = y0 + dy
      for (let i = 0; i < nx; i++) {
        const x0 = xmin + i * dx
        const x1 = x0 + dx
        const z = energy((x0 + x1) / 2, (y0 + y1) / 2)
        const t = Math.max(0, Math.min(1, (z + 0.52) / 2.15))
        const p0 = project(x0, y0)
        const p1 = project(x1, y0)
        const p2 = project(x1, y1)
        const p3 = project(x0, y1)
        ctx.beginPath()
        ctx.moveTo(p0.x, p0.y)
        ctx.lineTo(p1.x, p1.y)
        ctx.lineTo(p2.x, p2.y)
        ctx.lineTo(p3.x, p3.y)
        ctx.closePath()
        const r = Math.round(44 + 198 * t)
        const g = Math.round(150 - 48 * t)
        const b = Math.round(220 - 160 * t)
        ctx.fillStyle = `rgba(${r},${g},${b},${0.055 + t * 0.055})`
        ctx.fill()
      }
    }

    ctx.lineWidth = 0.75
    for (let j = 0; j <= ny; j += 2) {
      const y = ymin + j * dy
      ctx.beginPath()
      for (let i = 0; i <= nx; i++) {
        const x = xmin + i * dx
        const p = project(x, y)
        i ? ctx.lineTo(p.x, p.y) : ctx.moveTo(p.x, p.y)
      }
      ctx.strokeStyle = j > ny * 0.55 ? 'rgba(125,211,252,0.21)' : 'rgba(255,138,76,0.15)'
      ctx.stroke()
    }
    for (let i = 0; i <= nx; i += 2) {
      const x = xmin + i * dx
      ctx.beginPath()
      for (let j = 0; j <= ny; j++) {
        const y = ymin + j * dy
        const p = project(x, y)
        j ? ctx.lineTo(p.x, p.y) : ctx.moveTo(p.x, p.y)
      }
      ctx.strokeStyle = 'rgba(160,178,214,0.12)'
      ctx.stroke()
    }

    paths.forEach((path, pathIndex) => {
      const start = project(path[0].x, path[0].y)
      const endPoint = path[path.length - 1]
      const end = project(endPoint.x, endPoint.y)
      const lineGradient = ctx.createLinearGradient(start.x, start.y, end.x, end.y)
      lineGradient.addColorStop(0, 'rgba(255,107,53,0.08)')
      lineGradient.addColorStop(0.45, 'rgba(255,179,71,0.45)')
      lineGradient.addColorStop(1, 'rgba(125,211,252,0.75)')
      ctx.beginPath()
      path.forEach((point, i) => {
        const p = project(point.x, point.y)
        i ? ctx.lineTo(p.x, p.y) : ctx.moveTo(p.x, p.y)
      })
      ctx.strokeStyle = lineGradient
      ctx.lineWidth = 1.5
      ctx.setLineDash([3, 4])
      ctx.stroke()
      ctx.setLineDash([])

      const cycle = reducedMotion ? 1 : ((now / 3800 + pathIndex * 0.137) % 1)
      const eased = 1 - (1 - cycle) ** 2.4
      const point = path[Math.min(path.length - 1, Math.floor(eased * path.length))]
      const p = project(point.x, point.y)
      const halo = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, 17)
      halo.addColorStop(0, 'rgba(255,245,230,0.9)')
      halo.addColorStop(0.2, 'rgba(255,155,82,0.65)')
      halo.addColorStop(1, 'rgba(255,107,53,0)')
      ctx.fillStyle = halo
      ctx.beginPath()
      ctx.arc(p.x, p.y, 17, 0, Math.PI * 2)
      ctx.fill()
      ctx.fillStyle = '#fff7ed'
      ctx.beginPath()
      ctx.arc(p.x, p.y, 2.6, 0, Math.PI * 2)
      ctx.fill()
    })

    if (!reducedMotion) frame = requestAnimationFrame(draw)
  }

  fit()
  const resizeObserver = new ResizeObserver(() => {
    fit()
    if (reducedMotion) draw(performance.now())
  })
  resizeObserver.observe(c)
  draw(performance.now())

  onBeforeUnmount(() => {
    cancelAnimationFrame(frame)
    resizeObserver.disconnect()
  })
})
</script>

<template>
  <canvas ref="canvas" />
</template>

<style scoped>
canvas {
  display: block;
  width: 100%;
  height: 100%;
}
</style>
