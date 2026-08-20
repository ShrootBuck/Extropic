# Noise Is Not the Enemy

A 5-minute Slidev deck on thermodynamic computing, p-bits, and Extropic's Z1.

## Run it

```bash
pnpm install
pnpm dev
```

Opens <http://localhost:3030>.

- **Presenter mode** (script + 5-min timer): <http://localhost:3030/presenter>
- **Overview** (all slides): <http://localhost:3030/overview>

Press `Space` / `→` to advance, `F` for fullscreen, `O` for overview, `C` to draw on a slide.

## The word-for-word script

Lives in the `<!-- ... -->` block at the end of every slide in `slides.md`, timestamped to hit
5:00. It shows up in the right-hand pane of presenter mode. `[click]` markers in the notes line up
with the click-reveals on the slide.

## The live bits

Everything animated is real computation running in the browser, not a video:

| Component | What it actually does |
|---|---|
| `NoiseField.vue` | Drifting thermal field + grain behind every slide |
| `BitCrusher.vue` | Gaussian (Johnson-Nyquist) noise on a real voltage, thresholded into a digital bit. Tail events cross the line and the digital output glitches — the slider drives it to failure |
| `PrngVsPbit.vue` | PRNG pipeline vs. a single physically-random bit |
| `PBit.vue` | A tunable Bernoulli sampler — drag the bias, the measured fraction converges |
| `EnergyLandscape.vue` | Metropolis walkers on an energy landscape; the histogram converges to the Boltzmann curve |
| `IsingLattice.vue` | Checkerboard Gibbs sampling on a 2D Ising model, annealed live. With a `pattern` prop it pulls a word out of static |
| `Z1Die.vue` | Stylised Z1 die with live p-bit activity |
| `EnergyBars.vue` | Log-scale GPU vs TSU energy comparison |

Sliders on slides 2, 4, 5, 6 and 7 are meant to be dragged mid-talk. Slides 6 and 7 also anneal on
their own (the `auto` toggle) so you don't have to touch anything.

## Build

```bash
pnpm build      # static SPA in dist/
```
