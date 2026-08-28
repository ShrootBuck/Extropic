<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { onSlideEnter, onSlideLeave } from '@slidev/client'

const cvs = ref<HTMLCanvasElement | null>(null)
let raf = 0
let active = false
let startAnimation = () => {}
let stopAnimation = () => {}

onSlideEnter(() => { active = true; startAnimation() })
onSlideLeave(() => { active = false; stopAnimation() })
onBeforeUnmount(() => stopAnimation())

onMounted(() => {
  const c = cvs.value!
  const ctx = c.getContext('2d')!
  const G = 112
  const off = document.createElement('canvas'); off.width = G; off.height = G
  const octx = off.getContext('2d')!
  const img = octx.createImageData(G, G)

  // mask: which cells sit inside one of the 8 cores
  const mask = new Uint8Array(G * G)
  const cores: [number, number, number, number][] = []
  const M = 9, GAP = 3
  const cw = Math.floor((G - M * 2 - GAP * 3) / 4)
  const ch = Math.floor((G - M * 2 - GAP * 1) / 2)
  for (let r = 0; r < 2; r++) for (let q = 0; q < 4; q++) {
    const x0 = M + q * (cw + GAP), y0 = M + r * (ch + GAP)
    cores.push([x0, y0, cw, ch])
    for (let y = y0; y < y0 + ch; y++) for (let x = x0; x < x0 + cw; x++) mask[y * G + x] = 1
  }
  const state = new Uint8Array(G * G)
  for (let i = 0; i < state.length; i++) state[i] = Math.random() < 0.45 ? 1 : 0

  let W = 0, H = 0
  const dpr = Math.min(2, window.devicePixelRatio || 1)
  const fit = () => {
    W = c.clientWidth; H = c.clientHeight
    c.width = W * dpr; c.height = H * dpr
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
  }
  fit()
  const ro = new ResizeObserver(fit); ro.observe(c)

  let t = 0
  const resetSimulation = () => {
    t = 0
    for (let i = 0; i < state.length; i++) state[i] = Math.random() < 0.45 ? 1 : 0
  }

  let running = false
  const draw = () => {
    if (!running) return
    if (!W) { raf = requestAnimationFrame(draw); return }
    t += 0.016

    // flip a slice of the p-bits every frame
    const flips = (G * G) >> 2
    for (let k = 0; k < flips; k++) {
      const i = (Math.random() * G * G) | 0
      if (mask[i]) state[i] = Math.random() < 0.44 ? 1 : 0
    }

    const d = img.data
    for (let i = 0; i < G * G; i++) {
      const j = i * 4
      if (!mask[i]) { d[j] = 8; d[j + 1] = 12; d[j + 2] = 22; d[j + 3] = 255; continue }
      if (state[i]) { d[j] = 255; d[j + 1] = 140; d[j + 2] = 62; d[j + 3] = 255 }
      else { d[j] = 24; d[j + 1] = 36; d[j + 2] = 58; d[j + 3] = 255 }
    }
    octx.putImageData(img, 0, 0)

    const S = Math.min(W, H) * 0.86
    const ox = (W - S) / 2, oy = (H - S) / 2
    ctx.clearRect(0, 0, W, H)

    // package glow
    const g = ctx.createRadialGradient(W / 2, H / 2, S * 0.2, W / 2, H / 2, S * 0.85)
    g.addColorStop(0, `rgba(255,110,50,${0.20 + 0.05 * Math.sin(t * 1.6)})`)
    g.addColorStop(1, 'rgba(255,110,50,0)')
    ctx.fillStyle = g; ctx.fillRect(0, 0, W, H)

    // substrate
    ctx.save()
    const r = 10
    ctx.beginPath()
    ctx.roundRect(ox, oy, S, S, r)
    ctx.clip()
    ctx.imageSmoothingEnabled = false
    ctx.drawImage(off, ox, oy, S, S)
    ctx.restore()

    // core outlines + bus traces
    ctx.strokeStyle = 'rgba(255,255,255,0.16)'; ctx.lineWidth = 1
    const sc = S / G
    for (const [x0, y0, w, hh] of cores) {
      ctx.strokeRect(ox + x0 * sc, oy + y0 * sc, w * sc, hh * sc)
    }
    // die edge
    ctx.strokeStyle = 'rgba(255,180,120,0.55)'; ctx.lineWidth = 1.6
    ctx.beginPath(); ctx.roundRect(ox, oy, S, S, r); ctx.stroke()

    // scanning read-out line
    const sy = oy + ((t * 0.16) % 1) * S
    const lg = ctx.createLinearGradient(ox, sy - 14, ox, sy + 14)
    lg.addColorStop(0, 'rgba(120,215,255,0)')
    lg.addColorStop(0.5, 'rgba(150,225,255,0.30)')
    lg.addColorStop(1, 'rgba(120,215,255,0)')
    ctx.fillStyle = lg; ctx.fillRect(ox, sy - 14, S, 28)

    if (running) raf = requestAnimationFrame(draw)
  }
  startAnimation = () => {
    stopAnimation()
    resetSimulation()
    running = true
    raf = requestAnimationFrame(draw)
  }
  stopAnimation = () => {
    running = false
    cancelAnimationFrame(raf)
  }
  if (active) startAnimation()
  onBeforeUnmount(() => ro.disconnect())
})
</script>

<template>
  <div class="die">
    <canvas ref="cvs" />
    <div class="cap mono">Z1 &middot; 8 cores &middot; live p&#8209;bit activity</div>
  </div>
</template>

<style scoped>
.die { position: relative; width: 100%; height: 100%; }
canvas { width: 100%; height: calc(100% - 16px); min-height: 0; display: block; }
.cap { text-align: center; font-size: 0.53rem; letter-spacing: 0.2em; text-transform: uppercase; color: #5d6780; }
</style>
