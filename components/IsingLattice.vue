<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

const props = withDefaults(defineProps<{
  n?: number
  pattern?: string
  h0?: number
  j?: number
  tHot?: number
  tCold?: number
  cycle?: number      // seconds for a full hot -> cold -> hot cycle
  label?: string
}>(), { n: 108, pattern: '', h0: 0.42, j: 1, tHot: 3.4, tCold: 1.05, cycle: 16, label: '' })

const T = ref(props.tHot)
const auto = ref(true)
const sweeps = ref(0)
const cvs = ref<HTMLCanvasElement | null>(null)
let raf = 0

onMounted(() => {
  const N = props.n
  const c = cvs.value!
  const ctx = c.getContext('2d')!
  ctx.imageSmoothingEnabled = false

  const off = document.createElement('canvas')
  off.width = N; off.height = N
  const octx = off.getContext('2d')!
  const img = octx.createImageData(N, N)

  // spins
  const s = new Int8Array(N * N)
  for (let i = 0; i < s.length; i++) s[i] = Math.random() < 0.5 ? -1 : 1

  // external field from an optional text pattern
  const h = new Float32Array(N * N)
  if (props.pattern) {
    const m = document.createElement('canvas')
    m.width = N; m.height = N
    const mc = m.getContext('2d')!
    mc.fillStyle = '#000'; mc.fillRect(0, 0, N, N)
    mc.fillStyle = '#fff'
    mc.textAlign = 'center'; mc.textBaseline = 'middle'
    let size = N * 0.34
    mc.font = `800 ${size}px "Space Grotesk", Helvetica, sans-serif`
    while (mc.measureText(props.pattern).width > N * 0.86 && size > 6) {
      size -= 1
      mc.font = `800 ${size}px "Space Grotesk", Helvetica, sans-serif`
    }
    mc.fillText(props.pattern, N / 2, N / 2 + size * 0.04)
    const d = mc.getImageData(0, 0, N, N).data
    for (let i = 0; i < N * N; i++) h[i] = d[i * 4] > 110 ? props.h0 : -props.h0
  }

  const idx = (x: number, y: number) => ((y + N) % N) * N + ((x + N) % N)

  const sweep = (parity: number) => {
    const t = T.value
    const ramp = Math.max(0, Math.min(1, (props.tHot - t) / (props.tHot - props.tCold)))
    const hs = ramp * ramp
    const J = props.j
    for (let y = 0; y < N; y++) {
      for (let x = (y + parity) & 1; x < N; x += 2) {
        const i = y * N + x
        const nb = s[idx(x + 1, y)] + s[idx(x - 1, y)] + s[idx(x, y + 1)] + s[idx(x, y - 1)]
        const field = J * nb + h[i] * hs
        // Gibbs: probability this p-bit lands "up"
        const p = 1 / (1 + Math.exp(-2 * field / t))
        s[i] = Math.random() < p ? 1 : -1
      }
    }
  }

  let W = 0, H = 0, t0 = performance.now()
  const fit = () => { W = c.clientWidth; H = c.clientHeight; c.width = W; c.height = H; ctx.imageSmoothingEnabled = false }
  fit()
  const ro = new ResizeObserver(fit); ro.observe(c)

  const draw = () => {
    if (!W) { raf = requestAnimationFrame(draw); return }
    if (auto.value) {
      const el = (performance.now() - t0) / 1000
      const u = 0.5 - 0.5 * Math.cos((el / props.cycle) * Math.PI * 2)   // 0 -> 1 -> 0
      T.value = props.tHot + (props.tCold - props.tHot) * u
    }
    for (let k = 0; k < 3; k++) { sweep(0); sweep(1); sweeps.value++ }

    const d = img.data
    for (let i = 0; i < N * N; i++) {
      const up = s[i] === 1
      const j = i * 4
      if (up) { d[j] = 255; d[j + 1] = 128; d[j + 2] = 54 }
      else { d[j] = 12; d[j + 1] = 22; d[j + 2] = 42 }
      d[j + 3] = 255
    }
    octx.putImageData(img, 0, 0)
    ctx.clearRect(0, 0, W, H)
    ctx.drawImage(off, 0, 0, W, H)

    raf = requestAnimationFrame(draw)
  }
  draw()
  onBeforeUnmount(() => { cancelAnimationFrame(raf); ro.disconnect() })
})
</script>

<template>
  <div class="ising">
    <div class="frame">
      <canvas ref="cvs" />
      <div class="glow" />
      <div v-if="label" class="tag mono">{{ label }}</div>
    </div>
    <div class="ctl">
      <span class="lbl mono">temperature</span>
      <input class="thermo" type="range" min="0.6" max="4" step="0.02"
             v-model.number="T" @input="auto = false" />
      <span class="val mono" :style="{ color: T > 2.6 ? '#ff8a4c' : T > 1.9 ? '#ffb347' : '#7dd3fc' }">
        {{ T.toFixed(2) }}
      </span>
      <button class="auto mono" :class="{ on: auto }" @click="auto = !auto">auto</button>
    </div>
  </div>
</template>

<style scoped>
.ising { display: flex; flex-direction: column; gap: 0.45rem; width: 100%; height: 100%; }
.frame {
  position: relative; flex: 1; min-height: 0; border-radius: 12px; overflow: hidden;
  border: 1px solid rgba(255,255,255,0.10);
  box-shadow: 0 0 46px -8px rgba(255,107,53,0.45), inset 0 0 40px rgba(0,0,0,0.6);
  background: #060910;
}
canvas { width: 100%; height: 100%; min-height: 0; display: block; }
.glow { position: absolute; inset: 0; pointer-events: none; background: radial-gradient(85% 85% at 50% 50%, transparent 55%, rgba(0,0,0,0.55) 100%); }
.tag {
  position: absolute; left: 8px; top: 7px;
  font-size: 0.52rem; letter-spacing: 0.2em; text-transform: uppercase;
  color: rgba(255,255,255,0.55); background: rgba(0,0,0,0.45);
  padding: 2px 7px; border-radius: 5px;
}
.ctl { display: flex; align-items: center; gap: 0.7rem; }
.ctl input { flex: 1; }
.lbl { font-size: 0.55rem; letter-spacing: 0.2em; text-transform: uppercase; color: #7d879c; }
.val { font-size: 0.64rem; width: 2.4rem; }
.auto {
  font-size: 0.53rem; letter-spacing: 0.14em; text-transform: uppercase;
  padding: 2px 8px; border-radius: 20px; cursor: pointer;
  border: 1px solid rgba(255,255,255,0.16); color: #7d879c; background: transparent;
}
.auto.on { color: #05060a; background: #ffb347; border-color: #ffb347; }
</style>
