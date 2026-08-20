<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

const props = withDefaults(defineProps<{
  intensity?: number
  embers?: number
}>(), { intensity: 1, embers: 34 })

const cvs = ref<HTMLCanvasElement | null>(null)
let raf = 0

onMounted(() => {
  const c = cvs.value!
  const ctx = c.getContext('2d', { alpha: true })!

  // --- low-res drifting "heat field" -------------------------------------
  const GW = 30, GH = 18
  const off = document.createElement('canvas')
  off.width = GW; off.height = GH
  const octx = off.getContext('2d')!
  const img = octx.createImageData(GW, GH)

  // each cell gets its own frequency + phase -> smooth pseudo-noise, no lib
  const ph = new Float32Array(GW * GH * 3)
  for (let i = 0; i < ph.length; i++) ph[i] = Math.random() * Math.PI * 2
  const fr = new Float32Array(GW * GH * 3)
  for (let i = 0; i < fr.length; i++) fr[i] = 0.10 + Math.random() * 0.34

  // --- slow drifting embers ---------------------------------------------
  type Ember = { x: number, y: number, vx: number, vy: number, r: number, h: number, a: number }
  const embers: Ember[] = []
  const mkEmber = (): Ember => ({
    x: Math.random(), y: Math.random(),
    vx: (Math.random() - 0.5) * 0.00022,
    vy: -0.00008 - Math.random() * 0.00022,
    r: 20 + Math.random() * 95,
    h: Math.random(),
    a: 0.03 + Math.random() * 0.07,
  })
  for (let i = 0; i < props.embers; i++) embers.push(mkEmber())

  let W = 0, H = 0
  const fit = () => {
    W = c.clientWidth; H = c.clientHeight
    c.width = Math.max(1, Math.floor(W)); c.height = Math.max(1, Math.floor(H))
  }
  fit()
  const ro = new ResizeObserver(fit)
  ro.observe(c)

  let t = 0
  const draw = () => {
    t += 0.016
    if (!W || !H) { raf = requestAnimationFrame(draw); return }
    ctx.clearRect(0, 0, W, H)

    // 1. heat field
    let k = 0
    for (let y = 0; y < GH; y++) {
      for (let x = 0; x < GW; x++) {
        const i = (y * GW + x) * 3
        const v =
          Math.sin(t * fr[i] + ph[i]) * 0.5 +
          Math.sin(t * fr[i + 1] * 1.7 + ph[i + 1] + x * 0.35) * 0.3 +
          Math.sin(t * fr[i + 2] * 0.6 + ph[i + 2] + y * 0.5) * 0.2
        const n = Math.max(0, Math.min(1, v * 0.5 + 0.5))
        const heat = Math.pow(n, 2.3)
        img.data[k++] = 255 * (0.15 + 0.85 * heat)
        img.data[k++] = 110 * heat + 60 * (1 - heat)
        img.data[k++] = 255 * (1 - heat) * 0.9 + 40
        img.data[k++] = 255 * (0.05 + 0.16 * heat) * props.intensity
      }
    }
    octx.putImageData(img, 0, 0)
    ctx.imageSmoothingEnabled = true
    ctx.globalCompositeOperation = 'lighter'
    ctx.drawImage(off, 0, 0, W, H)

    // 2. embers
    for (const e of embers) {
      e.x += e.vx; e.y += e.vy
      e.x += (Math.random() - 0.5) * 0.0012
      e.y += (Math.random() - 0.5) * 0.0012
      if (e.y < -0.15) { Object.assign(e, mkEmber(), { y: 1.12 }) }
      if (e.x < -0.2) e.x = 1.2; if (e.x > 1.2) e.x = -0.2
      const px = e.x * W, py = e.y * H
      const g = ctx.createRadialGradient(px, py, 0, px, py, e.r)
      const hot = e.h > 0.45
      g.addColorStop(0, hot ? `rgba(255,150,70,${e.a * props.intensity})` : `rgba(80,180,255,${e.a * 0.85 * props.intensity})`)
      g.addColorStop(1, 'rgba(0,0,0,0)')
      ctx.fillStyle = g
      ctx.beginPath(); ctx.arc(px, py, e.r, 0, 6.284); ctx.fill()
    }

    // 3. Johnson–Nyquist grain
    ctx.globalCompositeOperation = 'source-over'
    const n = Math.floor((W * H) / 5200)
    for (let i = 0; i < n; i++) {
      const x = Math.random() * W, y = Math.random() * H
      const a = Math.random() * 0.16 * props.intensity
      ctx.fillStyle = Math.random() > 0.72 ? `rgba(255,170,110,${a})` : `rgba(180,215,255,${a})`
      ctx.fillRect(x, y, 1.15, 1.15)
    }

    raf = requestAnimationFrame(draw)
  }
  draw()

  onBeforeUnmount(() => { cancelAnimationFrame(raf); ro.disconnect() })
})
</script>

<template>
  <canvas ref="cvs" class="noise-field" />
</template>

<style scoped>
.noise-field {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}
</style>
