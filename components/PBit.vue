<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from 'vue'

const bias = ref(0.5)           // "control voltage" 0..1
const bit = ref(0)
const tape = ref<number[]>(Array.from({ length: 78 }, () => 0))
const ones = ref(0)
const total = ref(0)
let timer: any = null

const p = computed(() => bias.value)
const frac = computed(() => (total.value ? ones.value / total.value : 0))

const tick = () => {
  const b = Math.random() < p.value ? 1 : 0
  bit.value = b
  ones.value += b
  total.value++
  tape.value.push(b)
  if (tape.value.length > 78) tape.value.shift()
}

const reset = () => { ones.value = 0; total.value = 0 }

onMounted(() => { timer = setInterval(tick, 55) })
onBeforeUnmount(() => clearInterval(timer))
</script>

<template>
  <div class="pbit-wrap">
    <!-- the bit itself -->
    <div class="cellcol">
      <div class="cell" :class="{ hi: bit === 1 }">
        <span class="mono">{{ bit }}</span>
      </div>
      <div class="cap mono">one p&#8209;bit</div>
    </div>

    <div class="right">
      <!-- control voltage -->
      <div class="row">
        <span class="lbl mono">control voltage</span>
        <input class="thermo grow" type="range" min="0.02" max="0.98" step="0.01"
               v-model.number="bias" @input="reset" />
        <span class="val mono">P(1) = {{ p.toFixed(2) }}</span>
      </div>

      <!-- the bitstream tape -->
      <div class="tape">
        <i v-for="(b, i) in tape" :key="i" class="tb" :class="{ on: b === 1 }" />
      </div>

      <!-- convergence -->
      <div class="conv">
        <div class="bar">
          <div class="fill" :style="{ width: (frac * 100).toFixed(1) + '%' }" />
          <div class="target" :style="{ left: (p * 100).toFixed(1) + '%' }" />
        </div>
        <div class="row2 mono">
          <span>measured&nbsp;<b>{{ frac.toFixed(3) }}</b></span>
          <span class="dim">{{ total.toLocaleString() }} samples</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.pbit-wrap { display: flex; gap: 1.5rem; align-items: center; width: 100%; }
.cellcol { display: flex; flex-direction: column; align-items: center; gap: 0.45rem; }
.cell {
  width: 82px; height: 82px; border-radius: 16px;
  display: grid; place-items: center;
  font-size: 2.4rem; font-weight: 700; color: #4a5468;
  background: rgba(255,255,255,0.035);
  border: 1px solid rgba(255,255,255,0.10);
  transition: background 70ms linear, box-shadow 70ms linear, color 70ms linear, border-color 70ms;
}
.cell.hi {
  color: #fff;
  background: linear-gradient(160deg, #ff7a3d, #ff4d1c);
  border-color: rgba(255,160,110,0.8);
  box-shadow: 0 0 32px 6px rgba(255,107,53,0.55), inset 0 0 22px rgba(255,220,190,0.35);
}
.cap { font-size: 0.58rem; letter-spacing: 0.22em; text-transform: uppercase; color: #5d6780; }

.right { flex: 1; display: flex; flex-direction: column; gap: 0.85rem; }
.row { display: flex; align-items: center; gap: 0.8rem; }
.grow { flex: 1; }
.lbl { font-size: 0.6rem; letter-spacing: 0.16em; text-transform: uppercase; color: #7d879c; white-space: nowrap; }
.val { font-size: 0.72rem; color: #ffb347; white-space: nowrap; }

.tape { display: flex; gap: 2px; height: 26px; align-items: stretch; }
.tb {
  flex: 1; border-radius: 2px;
  background: rgba(255,255,255,0.055);
  transition: background 240ms ease, box-shadow 240ms ease;
}
.tb.on { background: linear-gradient(180deg, #ffb347, #ff5b1f); box-shadow: 0 0 7px rgba(255,110,50,0.75); }

.conv { display: flex; flex-direction: column; gap: 0.32rem; }
.bar { position: relative; height: 8px; border-radius: 5px; background: rgba(255,255,255,0.06); overflow: visible; }
.fill { position: absolute; inset: 0 auto 0 0; border-radius: 5px; background: linear-gradient(90deg, #38bdf8, #a78bfa 60%, #ff6b35); transition: width 160ms linear; }
.target { position: absolute; top: -5px; width: 2px; height: 18px; background: #fff; box-shadow: 0 0 8px #fff; transform: translateX(-1px); }
.row2 { display: flex; justify-content: space-between; font-size: 0.62rem; color: #98a2b8; }
.row2 b { color: #fff; }
.dim { color: #5d6780; }
</style>
