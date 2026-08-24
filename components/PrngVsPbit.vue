<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

const hex = ref('0000000000000000')
const stage = ref(0)
const bit = ref(0)
const ops = ref(0)
const tape = ref<number[]>(Array.from({ length: 46 }, () => Math.random() < 0.5 ? 1 : 0))
const stages = ['seed', 'multiply', 'xor-shift', 'round', 'output']
let a: any, b: any

const rnd = () => Array.from({ length: 16 }, () => '0123456789abcdef'[(Math.random() * 16) | 0]).join('')

onMounted(() => {
  a = setInterval(() => { hex.value = rnd(); stage.value = (stage.value + 1) % stages.length }, 120)
  b = setInterval(() => {
    bit.value = Math.random() < 0.5 ? 1 : 0
    tape.value.push(bit.value); if (tape.value.length > 46) tape.value.shift()
    ops.value += 231
  }, 60)
})
onBeforeUnmount(() => { clearInterval(a); clearInterval(b) })
</script>

<template>
  <div class="cmp">
    <!-- GPU -->
    <div class="card">
      <div class="hd mono">how a GPU rolls a die</div>
      <div class="pipe">
        <div v-for="(s, i) in stages" :key="s" class="st mono" :class="{ on: i === stage }">{{ s }}</div>
      </div>
      <div class="hexline mono">0x{{ hex }}</div>
      <div class="meter">
        <div class="mfill" :style="{ width: (30 + 60 * Math.abs(Math.sin(ops / 900))) + '%' }" />
      </div>
      <div class="ft">Thousands of logic operations, burning real power,<br>to <em>imitate</em> a coin flip it can never actually make.</div>
    </div>

    <div class="vs mono">vs</div>

    <!-- p-bit -->
    <div class="card hot">
      <div class="hd mono">how a p&#8209;bit rolls a die</div>
      <div class="cellrow">
        <div class="cell" :class="{ hi: bit === 1 }"><span class="mono">{{ bit }}</span></div>
        <div class="side mono">
          <div><b>1</b> circuit</div>
          <div><b>1</b> tick</div>
          <div class="dim">thermal noise, already there</div>
        </div>
      </div>
      <div class="tape">
        <i v-for="(b, i) in tape" :key="i" class="tb" :class="{ on: b === 1 }" />
      </div>
      <div class="ft">It doesn't calculate a random number. It <em>is</em> one.</div>
    </div>
  </div>
</template>

<style scoped>
.cmp { display: flex; align-items: stretch; gap: 0.9rem; width: 100%; }
.card {
  flex: 1; padding: 1.1rem 1.25rem 1.15rem;
  border-radius: 14px;
  border: 1px solid rgba(255,255,255,0.09);
  background: rgba(255,255,255,0.03);
  display: flex; flex-direction: column; gap: 0.6rem;
}
.card.hot {
  border-color: rgba(255,140,80,0.38);
  background: linear-gradient(160deg, rgba(255,107,53,0.10), rgba(255,107,53,0.02));
  box-shadow: 0 0 44px -12px rgba(255,107,53,0.7);
}
.hd { font-size: 0.6rem; letter-spacing: 0.2em; text-transform: uppercase; color: #8b96ae; }
.card.hot .hd { color: #ffb347; }
.pipe { display: flex; gap: 4px; }
.st {
  flex: 1; text-align: center; font-size: 0.55rem; padding: 6px 0; border-radius: 5px;
  color: #4d566d; background: rgba(255,255,255,0.04); transition: all 140ms;
}
.st.on { color: #05060a; background: #94a3b8; }
.hexline { font-size: 1.05rem; color: #9fb0cc; letter-spacing: 0.04em; }
.ft { font-size: 0.7rem; line-height: 1.5; color: #7d879c; margin-top: auto; }
.ft em { color: #fff; font-style: italic; }
.vs { align-self: center; font-size: 0.6rem; color: #46506a; letter-spacing: 0.2em; }

.cellrow { display: flex; align-items: center; gap: 0.8rem; }
.cell {
  width: 66px; height: 66px; border-radius: 14px; display: grid; place-items: center;
  font-size: 1.9rem; font-weight: 700; color: #4a5468;
  background: rgba(255,255,255,0.04); border: 1px solid rgba(255,255,255,0.10);
  transition: all 60ms linear;
}
.cell.hi {
  color: #fff; background: linear-gradient(160deg,#ff7a3d,#ff4d1c);
  border-color: rgba(255,170,120,0.9);
  box-shadow: 0 0 26px 4px rgba(255,107,53,0.6);
}
.meter { height: 6px; border-radius: 4px; background: rgba(255,255,255,0.05); overflow: hidden; }
.mfill { height: 100%; background: linear-gradient(90deg, rgba(148,163,184,0.35), #94a3b8); border-radius: 4px; }
.readout { font-size: 0.55rem; letter-spacing: 0.08em; color: #5d6780; }
.readout.hotr { color: #b07a4e; }
.tape { display: flex; gap: 2px; height: 20px; }
.tb { flex: 1; border-radius: 2px; background: rgba(255,255,255,0.06); transition: background 260ms ease; }
.tb.on { background: linear-gradient(180deg,#ffb347,#ff5b1f); box-shadow: 0 0 6px rgba(255,110,50,0.7); }
.side { font-size: 0.7rem; color: #98a2b8; line-height: 1.5; }
.side b { color: #fff; }
.side .dim { color: #5d6780; font-size: 0.6rem; }
</style>
