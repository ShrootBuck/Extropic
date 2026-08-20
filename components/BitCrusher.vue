<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

const props = withDefaults(defineProps<{ noise?: number, showDigital?: boolean }>(),
  { noise: 0.17, showDigital: true })

const level = ref(props.noise)   // "thermal noise" knob
const errors = ref(0)
const cvs = ref<HTMLCanvasElement | null>(null)
let raf = 0

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

  const N = 260
  const buf: number[] = []      // what the voltage really is
  const truth: number[] = []    // what it was supposed to be
  let phase = 0

  // Johnson-Nyquist noise is Gaussian -- and the Gaussian tail is the whole point,
  // so use a real normal deviate rather than something conveniently bounded.
  let spare: number | null = null
  const gauss = () => {
    if (spare !== null) { const g = spare; spare = null; return g }
    let u = 0, v = 0, q = 0
    do { u = Math.random() * 2 - 1; v = Math.random() * 2 - 1; q = u * u + v * v } while (q === 0 || q >= 1)
    const m = Math.sqrt(-2 * Math.log(q) / q)
    spare = v * m
    return u * m
  }

  const push = () => {
    phase += 0.028
    const drive = Math.sin(phase) > 0 ? 1 : 0
    buf.push(drive * 0.86 + 0.07 + gauss() * level.value)
    truth.push(drive)
    if (buf.length > N) { buf.shift(); truth.shift() }
  }
  for (let i = 0; i < N; i++) push()

  const draw = () => {
    if (!W) { raf = requestAnimationFrame(draw); return }
    push()
    ctx.clearRect(0, 0, W, H)

    const pad = 10
    const LO = -0.38, HI = 1.38
    const y = (v: number) => H - pad - ((v - LO) / (HI - LO)) * (H - pad * 2)
    const x = (i: number) => (i / (N - 1)) * W

    // the decision band -- everything in here is a coin toss the chip has to win
    ctx.fillStyle = 'rgba(255,255,255,0.04)'
    ctx.fillRect(0, y(0.60), W, y(0.40) - y(0.60))
    ctx.strokeStyle = 'rgba(255,255,255,0.28)'
    ctx.setLineDash([3, 4]); ctx.lineWidth = 1
    ctx.beginPath(); ctx.moveTo(0, y(0.5)); ctx.lineTo(W, y(0.5)); ctx.stroke()
    ctx.setLineDash([])
    ctx.font = '9px "JetBrains Mono", monospace'
    ctx.fillStyle = 'rgba(255,255,255,0.30)'
    ctx.fillText('THRESHOLD', W - 74, y(0.5) - 5)

    // --- mark every sample where the noise pushed it over the line ---
    let errs = 0
    for (let i = 0; i < N; i++) {
      const b = buf[i] > 0.5 ? 1 : 0
      if (b !== truth[i]) {
        errs++
        ctx.fillStyle = 'rgba(255,60,60,0.16)'
        ctx.fillRect(x(i) - 1.6, pad * 0.4, 3.2, H - pad)
      }
    }
    errors.value = errs

    // analog noisy trace
    ctx.beginPath()
    for (let i = 0; i < N; i++) { const px = x(i), py = y(buf[i]); i ? ctx.lineTo(px, py) : ctx.moveTo(px, py) }
    ctx.strokeStyle = 'rgba(255,150,90,0.8)'
    ctx.lineWidth = 1.25
    ctx.shadowColor = 'rgba(255,120,50,0.6)'; ctx.shadowBlur = 7
    ctx.stroke()
    ctx.shadowBlur = 0

    // digital reconstruction -- glitches wherever the noise won
    if (props.showDigital) {
      ctx.beginPath()
      let prev = buf[0] > 0.5 ? 1 : 0
      ctx.moveTo(x(0), y(prev * 0.86 + 0.07))
      for (let i = 1; i < N; i++) {
        const b = buf[i] > 0.5 ? 1 : 0
        const px = x(i)
        if (b !== prev) { ctx.lineTo(px, y(prev * 0.86 + 0.07)); ctx.lineTo(px, y(b * 0.86 + 0.07)); prev = b }
        else ctx.lineTo(px, y(b * 0.86 + 0.07))
      }
      ctx.strokeStyle = 'rgba(120,210,255,0.95)'
      ctx.lineWidth = 2
      ctx.shadowColor = 'rgba(56,189,248,0.8)'; ctx.shadowBlur = 9
      ctx.stroke()
      ctx.shadowBlur = 0

      // red dot on each lie
      for (let i = 0; i < N; i++) {
        const b = buf[i] > 0.5 ? 1 : 0
        if (b !== truth[i]) {
          ctx.fillStyle = '#ff4d4d'
          ctx.beginPath(); ctx.arc(x(i), y(b * 0.86 + 0.07), 2.6, 0, 6.284); ctx.fill()
        }
      }
    }

    raf = requestAnimationFrame(draw)
  }
  draw()
  onBeforeUnmount(() => { cancelAnimationFrame(raf); ro.disconnect() })
})
</script>

<template>
  <div class="bc">
    <canvas ref="cvs" />
    <div class="ctl">
      <span class="lbl mono">thermal noise</span>
      <input class="thermo" type="range" min="0.04" max="0.42" step="0.005" v-model.number="level" />
      <span class="legend mono">
        <span><i class="sw" style="background:#ff965a" /> real voltage</span>
        <span><i class="sw" style="background:#78d2ff" /> what the chip insists it is</span>
      </span>
      <span class="errs mono" :class="{ bad: errors > 0 }">
        {{ errors }} wrong bit{{ errors === 1 ? '' : 's' }} on screen
      </span>
    </div>
  </div>
</template>

<style scoped>
.bc { position: relative; width: 100%; height: 100%; display: flex; flex-direction: column; gap: 0.35rem; }
canvas { width: 100%; flex: 1; min-height: 0; display: block; }
.ctl { display: flex; align-items: center; gap: 0.9rem; }
.ctl input { width: 8rem; flex: none; }
.lbl { font-size: 0.55rem; letter-spacing: 0.2em; text-transform: uppercase; color: #7d879c; white-space: nowrap; }
.legend { display: flex; gap: 1.1rem; font-size: 0.58rem; color: #7d879c; letter-spacing: 0.04em; margin-left: auto; }
.sw { display: inline-block; width: 9px; height: 2px; vertical-align: middle; margin-right: 5px; border-radius: 2px; }
.errs {
  font-size: 0.58rem; letter-spacing: 0.04em; color: #5d6780;
  border: 1px solid rgba(255,255,255,0.10); border-radius: 999px; padding: 2px 9px; white-space: nowrap;
}
.errs.bad { color: #ff6b6b; border-color: rgba(255,77,77,0.45); background: rgba(255,60,60,0.10); }
</style>
