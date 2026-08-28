<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from 'vue'
import { onSlideEnter, onSlideLeave } from '@slidev/client'

const T = ref(0.42)
const cvs = ref<HTMLCanvasElement | null>(null)
let raf = 0
let active = false
let startAnimation = () => {}
let stopAnimation = () => {}

onSlideEnter(() => { active = true; startAnimation() })
onSlideLeave(() => { active = false; stopAnimation() })
onBeforeUnmount(() => stopAnimation())

const BINS = 110
const NGHOST = 48

function E(x: number) {
  const g = (c: number, w: number) => Math.exp(-(((x - c) / w) ** 2))
  return 2.35 - 1.05 * g(0.17, 0.085) - 2.05 * g(0.50, 0.098) - 1.45 * g(0.84, 0.082) + 0.55 * (x - 0.5) ** 2
}

let hist = new Float64Array(BINS)
let walkers = Array.from({ length: NGHOST }, () => Math.random())
let lead = 0.17
let trail: number[] = []
let burnUntil = 0
const resetHist = () => { hist = new Float64Array(BINS); burnUntil = performance.now() + 1100 }
watch(T, resetHist)

onMounted(() => {
  const c = cvs.value!
  const ctx = c.getContext('2d')!
  let W = 0, H = 0
  const dpr = Math.min(2, window.devicePixelRatio || 1)
  const fit = () => {
    W = c.clientWidth; H = c.clientHeight
    c.width = W * dpr; c.height = H * dpr
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
  }
  fit()
  const ro = new ResizeObserver(fit); ro.observe(c)

  const step = (x: number, t: number) => {
    const s = 0.014 + 0.075 * Math.sqrt(t)
    const nx = x + (Math.random() + Math.random() + Math.random() - 1.5) * s * 1.6
    if (nx < 0.01 || nx > 0.99) return x
    const dE = E(nx) - E(x)
    return (dE <= 0 || Math.random() < Math.exp(-dE / t)) ? nx : x
  }

  let last = performance.now()
  const resetSimulation = () => {
    T.value = 0.42
    hist = new Float64Array(BINS)
    walkers = Array.from({ length: NGHOST }, () => Math.random())
    lead = 0.17
    trail = []
    last = performance.now()
    burnUntil = last + 1100
  }
  resetSimulation()

  let running = false
  const draw = () => {
    if (!running) return
    if (!W) { raf = requestAnimationFrame(draw); return }
    const t = T.value
    const now = performance.now()
    const dt = Math.min(120, now - last); last = now
    // --- simulate (frame-rate independent) ---
    const warm = now < burnUntil
    const steps = Math.max(1, Math.min(30, Math.round(dt / 16.7 * 6)))
    if (!warm) {
      const keep = Math.exp(-dt / 6500)          // gently forget, so it tracks the knob
      for (let i = 0; i < BINS; i++) hist[i] *= keep
    }
    for (let k = 0; k < steps; k++) {
      lead = step(lead, t)
      for (let i = 0; i < walkers.length; i++) {
        walkers[i] = step(walkers[i], t)
        if (!warm) hist[Math.min(BINS - 1, Math.floor(walkers[i] * BINS))]++
      }
    }
    trail.push(lead); if (trail.length > 34) trail.shift()

    // --- geometry ---
    const padX = 10
    const eTop = 6, eBot = H * 0.60
    const hTop = H * 0.68, hBot = H - 14
    const px = (x: number) => padX + x * (W - padX * 2)
    const EMIN = -0.05, EMAX = 2.6
    const py = (e: number) => eBot - ((e - EMIN) / (EMAX - EMIN)) * (eBot - eTop)

    ctx.clearRect(0, 0, W, H)

    // --- energy landscape ---
    const grad = ctx.createLinearGradient(0, eTop, 0, eBot)
    grad.addColorStop(0, 'rgba(255,107,53,0.13)')
    grad.addColorStop(1, 'rgba(56,189,248,0.04)')
    ctx.beginPath()
    ctx.moveTo(px(0), eTop)
    for (let i = 0; i <= 240; i++) { const x = i / 240; ctx.lineTo(px(x), py(E(x))) }
    ctx.lineTo(px(1), eTop); ctx.closePath()
    ctx.fillStyle = grad; ctx.fill()
    ctx.beginPath()
    for (let i = 0; i <= 240; i++) { const x = i / 240; const yy = py(E(x)); i ? ctx.lineTo(px(x), yy) : ctx.moveTo(px(x), yy) }
    ctx.strokeStyle = 'rgba(190,205,235,0.55)'; ctx.lineWidth = 1.4; ctx.stroke()

    // ghosts
    ctx.fillStyle = 'rgba(255,150,90,0.22)'
    for (const w of walkers) { ctx.beginPath(); ctx.arc(px(w), py(E(w)) - 4, 2.4, 0, 6.284); ctx.fill() }

    // trail + lead ball
    trail.forEach((x, i) => {
      const k = i / trail.length
      ctx.fillStyle = `rgba(255,190,140,${(0.30 * k * k).toFixed(3)})`
      ctx.beginPath(); ctx.arc(px(x), py(E(x)) - 5, 1.8 + 1.6 * k, 0, 6.284); ctx.fill()
    })
    const bx = px(lead), by = py(E(lead)) - 5.5
    const bg = ctx.createRadialGradient(bx, by, 0, bx, by, 20)
    bg.addColorStop(0, 'rgba(255,190,130,0.85)'); bg.addColorStop(1, 'rgba(255,120,50,0)')
    ctx.fillStyle = bg; ctx.beginPath(); ctx.arc(bx, by, 20, 0, 6.284); ctx.fill()
    ctx.fillStyle = '#fff5ec'; ctx.beginPath(); ctx.arc(bx, by, 4.6, 0, 6.284); ctx.fill()

    // axis label
    ctx.font = '9px "JetBrains Mono", monospace'
    ctx.shadowColor = 'rgba(0,0,0,0.95)'; ctx.shadowBlur = 5
    ctx.fillStyle = 'rgba(255,255,255,0.46)'
    ctx.shadowBlur = 0

    // --- histogram: where it actually spends its time ---
    ctx.shadowBlur = 0
    let mx = 1e-9
    for (let i = 0; i < BINS; i++) mx = Math.max(mx, hist[i])
    const bw = (W - padX * 2) / BINS
    for (let i = 0; i < BINS; i++) {
      const h = (hist[i] / mx) * (hBot - hTop)
      const x0 = padX + i * bw
      const g2 = ctx.createLinearGradient(0, hBot - h, 0, hBot)
      g2.addColorStop(0, 'rgba(255,150,80,0.95)'); g2.addColorStop(1, 'rgba(255,90,40,0.20)')
      ctx.fillStyle = g2
      ctx.fillRect(x0, hBot - h, Math.max(1, bw - 0.6), h)
    }
    // theoretical curve
    let zmax = 0
    const th: number[] = []
    for (let i = 0; i < BINS; i++) { const v = Math.exp(-E((i + 0.5) / BINS) / t); th.push(v); zmax = Math.max(zmax, v) }
    ctx.beginPath()
    for (let i = 0; i < BINS; i++) {
      const X = padX + (i + 0.5) * bw, Y = hBot - (th[i] / zmax) * (hBot - hTop)
      i ? ctx.lineTo(X, Y) : ctx.moveTo(X, Y)
    }
    ctx.strokeStyle = 'rgba(120,215,255,0.9)'; ctx.lineWidth = 1.6
    ctx.setLineDash([4, 3]); ctx.stroke(); ctx.setLineDash([])
    ctx.fillStyle = 'rgba(120,215,255,0.85)'
    ctx.fillText('what thermodynamics predicts', W - 168, hTop - 3)
    ctx.shadowBlur = 0

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
  <div class="el">
    <canvas ref="cvs" />
    <div class="ctl">
      <span class="lbl mono">temperature</span>
      <input class="thermo" type="range" min="0.06" max="1.1" step="0.01" v-model.number="T" />
      <span class="val mono" :style="{ color: T > 0.6 ? '#ff8a4c' : T > 0.28 ? '#ffb347' : '#7dd3fc' }">
        {{ T < 0.2 ? 'cold' : T < 0.55 ? 'warm' : 'hot' }}
      </span>
    </div>
  </div>
</template>

<style scoped>
.el { display: flex; flex-direction: column; gap: 0.4rem; width: 100%; height: 100%; }
canvas { width: 100%; flex: 1; min-height: 0; display: block; }
.ctl { display: flex; align-items: center; gap: 0.8rem; }
.ctl input { flex: 1; }
.lbl { font-size: 0.58rem; letter-spacing: 0.2em; text-transform: uppercase; color: #7d879c; }
.val { font-size: 0.66rem; width: 3rem; }
</style>
