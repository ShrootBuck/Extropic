---
theme: default
colorSchema: dark
title: Noise Is Not the Enemy
info: |
  ## Noise Is Not the Enemy
  Thermodynamic computing, p-bits, and Extropic's Z1.
  Zayd Krunz, physics capstone, 5 minutes.
class: text-left
drawings:
  persist: false
transition: thermo
mdc: true
duration: 5min
fonts:
  sans: Inter
  serif: Inter
  mono: JetBrains Mono
  weights: '300,400,500,600,700,800'
  webfonts:
    - Space Grotesk
  provider: google
---

<div class="h-full flex flex-col justify-center">

<h1 class="!text-[4.4rem] !leading-[0.92] !mb-0 tracking-tighter">
  <NoiseTitle class="w-[16.5rem] h-[5.25rem] -ml-2 -mb-2 align-bottom" /><br>
  <span class="text-white">IS NOT THE</span><br>
  <span class="text-white">ENEMY</span>
</h1>

<div class="hair w-72 my-7" />

<div class="text-[0.95rem] text-[#98a2b8] max-w-[30rem] leading-relaxed">
  Every chip ever built spends energy <em class="text-white not-italic font-semibold">fighting</em> thermal noise.<br>
  What if we compute with it instead?
</div>

</div>

<!--

Right now, inside every chip in this room, there is a war going on.

Electrons are rattling around at random, because the chip is warm. That's thermal noise. And essentially all of modern computer engineering is dedicated to crushing it.

I want to tell you about a company that decided to stop fighting it and use it instead.
-->

---
layout: default
---

<div class="h-full flex flex-col">

<h1 class="mt-3">Your laptop is at war with heat</h1>
<p class="!mt-1 text-[0.86rem] max-w-[46rem]">A voltage inside a real chip does not look like a 1 or a 0. It looks like this.</p>

<div class="panel mt-4 p-3 flex-1 min-h-0">
  <BitCrusher class="h-full" :noise="0.17" />
</div>

<div class="grid grid-cols-3 gap-3 mt-4">
  <div v-click class="panel p-3">
    <div class="mono text-[0.55rem] tracking-[0.2em] uppercase text-[#ff8a4c]">the rule</div>
    <div class="text-[0.75rem] mt-1 text-[#c8d0e0]">Above the line it's a <b class="text-white">1</b>. Below, a <b class="text-white">0</b>. Cross it and the chip <b class="text-white">lies</b>.</div>
  </div>
  <div v-click class="panel p-3">
    <div class="mono text-[0.55rem] tracking-[0.2em] uppercase text-[#ff8a4c]">the price</div>
    <div class="text-[0.75rem] mt-1 text-[#c8d0e0]">Voltage headroom, bigger transistors, more heat, all just to keep the orange <b class="text-white">away from the line</b>.</div>
  </div>
  <div v-click class="panel p-3">
    <div class="mono text-[0.55rem] tracking-[0.2em] uppercase text-[#ff8a4c]">the physics</div>
    <div class="text-[0.75rem] mt-1 text-[#c8d0e0]">Erasing randomness has a <b class="text-white">thermodynamic cost</b>. It's a law, not a bug.</div>
  </div>
</div>

</div>

<!--

This is what a voltage inside a chip actually looks like: the orange trace. Fuzzy. Analog. Undecided.

[click]

A digital computer's entire job is to look at that and declare, with total confidence: above the line, one; below the line, zero. That's the blue trace, the fiction the chip insists on.

And mostly the fiction holds. But keep an eye on it. Every so often the noise spikes far enough to cross the line, and the chip confidently reports a bit that never happened. *(red dot + the counter goes red)*

[click]

So the whole game is keeping that orange trace away from that line. You buy that margin with voltage headroom, with bigger transistors, and with waste heat. *(drag the noise slider up; the blue line falls apart)* That's what you're paying to prevent.

[click]

And it isn't sloppy engineering. Erasing randomness has a thermodynamic price. It's a law, and you can't design around it.

*[If time permits, I can talk briefly about Maxwell's demon.]*
-->

---
layout: default
---

<div class="h-full flex flex-col">

<h1 class="mt-3">…and then we ask it to roll dice</h1>
<p class="!mt-1 text-[0.86rem] max-w-[52rem]">
  Generative AI doesn't <em>calculate</em> an answer. It <b class="text-white">draws a sample</b> from a probability distribution millions of times.
</p>

<div class="flex-1 min-h-0 flex items-center mt-4 pb-1">
  <PrngVsPbit />
</div>

<div v-click class="mt-3 text-center text-[0.8rem] text-[#ffb347]">
  We built a machine that can't be uncertain, then handed it the one job that's made of uncertainty.
</div>

</div>

<!--

Here's the irony.

Generative AI, the biggest workload we've built these hyper-precise machines for, is fundamentally a dice-rolling machine. Image models, language models: they don't compute an answer. They draw a random sample from a probability distribution, over and over and over.

So a GPU burns thousands of logic operations, and real power, running a pseudo-random number generator, an elaborate deterministic imitation of a coin flip, precisely because it has spent its whole existence guaranteeing that it can't actually flip a coin.

[click]

We built a machine that can't be uncertain, then handed it the one job that's made of uncertainty. Meanwhile there's a perfectly good source of true randomness sitting right there in the silicon, for free. It's the noise we've been trying to kill.
-->

---
layout: default
---

<div class="h-full flex flex-col">

<h1 class="mt-3">Stop suppressing the noise. <span class="grad-hot">Aim it.</span></h1>
<p class="!mt-1 text-[0.86rem] max-w-[50rem]">
  A <b class="text-white">p&#8209;bit</b> is an ordinary transistor circuit biased right at the tipping point, so thermal jitter knocks it back and forth on its own.
</p>

<div class="panel mt-6 px-6 py-6">
  <PBit />
</div>

<div class="grid grid-cols-3 gap-4 mt-5 text-[0.72rem]">
  <div v-click class="aster">One control voltage sets <b class="text-white">how loaded the coin is</b>: 0.1, 0.5, 0.9, anything.</div>
  <div v-click class="aster">Tens of millions of flips per second, from <b class="text-white">one tiny circuit</b>.</div>
  <div v-click class="aster">The randomness costs nothing to make. <b class="text-white">Heat was already doing it.</b></div>
</div>

</div>

<!--

So here's the move: a p-bit, short for probabilistic bit.

It's a small circuit made of completely ordinary transistors, biased right at the edge where thermal noise pushes it back and forth between zero and one, all by itself. That's the flickering square.

And the trick is this control-voltage knob. *(drag the slider)* Turn it down, and it spends most of its time at zero. Turn it up, most of its time at one. Watch the bar underneath: the measured fraction converges to exactly the probability I dialed in.

[click]

So it's not vaguely random. It's a *tunable* coin.

[click]

It runs at tens of megahertz.

[click]

And you're not generating the randomness; you're harvesting it. The heat was doing it anyway.
-->

---
layout: default
---

<div class="h-full flex flex-col">

<h1 class="mt-3">Nature has always been a sampler</h1>
<p class="!mt-1 text-[0.86rem] max-w-[54rem]">
  Statistical mechanics: a warm system doesn't freeze and doesn't go everywhere. <b class="text-white">The lower the energy, the exponentially more time it spends there.</b>
</p>

<div class="panel mt-3 p-3 flex-1 min-h-0">
  <EnergyLandscape class="h-full" />
</div>

<div class="flex gap-6 mt-3 text-[0.7rem]">
  <div v-click><span class="mono text-[#7dd3fc]">COLD →</span> each walker freezes in its nearest valley</div>
  <div v-click><span class="mono text-[#ff8a4c]">HOT →</span> so much energy it stops caring about the landscape</div>
  <div v-click><span class="mono text-[#ffb347]">JUST RIGHT →</span> finds the deep valleys, still escapes to look around</div>
</div>

</div>

<!--

Now the physics, and this is the part I actually love.

Statistical mechanics says a system at some temperature doesn't sit still, and it doesn't wander everywhere either. It explores, spending exponentially more time in lower-energy configurations. Deeper valley, far more likely. That's it. That's the whole law.

Watch.

[click]

Cold: each walker freezes in the valley nearest where it started. That is why the orange histogram has three peaks instead of matching the dashed equilibrium curve. *(drag cold)*

[click]

Hot: it's got so much energy it wanders anywhere and stops caring about the landscape at all. *(drag hot)*

[click]

In between is where it gets interesting. It finds the deep valleys, but it can still climb out and look around. *(drag to the middle)*

And look at the bottom: the orange histogram is where the ball actually spent its time. The dashed blue line is what thermodynamics predicts it should be. They're the same curve.

Nature is running the sampling algorithm for us. For free.
-->

---
layout: two-cols
layoutClass: gap-8
---

<div class="h-full flex flex-col justify-center pr-2">

<h1 class="mt-3 !text-[2.1rem]">Now wire a few hundred thousand of them together</h1>

<div class="hair w-40 my-5" />

<div class="space-y-3 text-[0.78rem]">
  <div v-click>Put p&#8209;bits on a grid. Let each one <b class="text-white">see only its neighbours</b>. That gives you an Ising model.</div>
  <div v-click>Every tick, each p&#8209;bit asks its neighbours what they're doing and re&#8209;flips itself accordingly. <b class="text-white">No CPU. No instructions.</b></div>
  <div v-click>Memory and computation are <b class="text-white">the same transistors</b>. Nothing has to travel across the chip, which is where GPUs burn most of their power.</div>
  <div v-click class="pt-1 text-[#ffb347]">Cool it down and structure appears out of static. Nobody computed those domains. <b class="text-white">The chip relaxed into them.</b></div>
</div>

</div>

::right::

<div class="h-full flex items-center py-6">
  <div class="w-full aspect-square max-h-full">
    <IsingLattice class="h-full" :n="112" :t-hot="3.6" :t-cold="1.15" :cycle="14" label="live · gibbs sampling" />
  </div>
</div>

<!--

So take a few hundred thousand of these.

[click]

Put them on a grid where each p-bit can only see its immediate neighbours. That is literally the Ising model: the same lattice of interacting spins from statistical mechanics.

[click]

Every tick, each p-bit asks its neighbours what they're doing and re-flips itself accordingly. There's no CPU driving this and no instruction stream.

[click]

Memory and computation are the same transistors, so nothing has to be shuttled across the chip. Moving data is where a GPU burns most of its energy.

[click]

Watch what happens as it cools. At high temperature it's pure static. Cool it, and structure appears: domains and order, out of noise. Nobody computed those patterns. The chip just relaxed into them.
-->

---
layout: two-cols
layoutClass: gap-8
---

<div class="h-full flex items-center py-6">
  <div class="w-full aspect-square max-h-full">
    <IsingLattice class="h-full" :n="160" pattern="ORDER" :h0="1.9" :j="0.45" :t-hot="4.2" :t-cold="0.75" :cycle="13" label="same chip · different energy landscape" />
  </div>
</div>

::right::

<div class="h-full flex flex-col justify-center pl-2">

<h1 class="mt-3 !text-[2.1rem]">Order, pulled out of noise</h1>

<div class="hair w-40 my-5" />

<div class="space-y-3 text-[0.78rem]">
  <div v-click>Those neighbour connections are <b class="text-white">programmable</b>. Change them and you change the energy landscape. You choose <em class="text-white not-italic">which patterns are the deep valleys</em>.</div>
  <div v-click>Train it so the deep valleys are your <b class="text-white">data</b>. Then just let the physics fall in.</div>
  <div v-click>Same silicon. Same thermal noise. I only told it what <b class="text-white">"low energy"</b> means, and a word crystallised out of static.</div>
  <div v-click class="pt-1 text-[#ffb347]">That's a generative model. It uses the same idea as an image diffusion model, except nothing computed its way there. <b class="text-white">It settled.</b></div>
</div>

</div>

<!--

And here's the payoff.

[click] Those neighbour connections are programmable. Change them, and you change the energy landscape. You're choosing which patterns get to be the deep valleys.

[click] So you train it so that the deep valleys are your data. And then you just let the physics fall into them.

[click] Same silicon, same thermal noise as the last slide. The only thing I changed is what "low energy" means, and a word crystallises out of static.

[click] That is a generative model. It's the same idea as an image diffusion model pulling a picture out of noise, except nothing here computed its way to the answer. It settled into it. Extropic calls the chip a Thermodynamic Sampling Unit.
-->

---
layout: two-cols
layoutClass: gap-6
---

<div class="h-full flex flex-col justify-center">
  <div class="h-[62%]">
    <Z1Die class="h-full" />
  </div>
</div>

::right::

<div class="h-full flex flex-col justify-center pl-1">

<h1 class="mt-3 !text-[2.15rem]">This is not a thought experiment</h1>

<div class="mt-5 space-y-2.5">
  <div v-click class="flex items-baseline gap-3">
    <span class="mono text-[0.58rem] text-[#5d6780] w-16 shrink-0">2025</span>
    <span class="text-[0.76rem]"><b class="text-white">X0:</b> proof-of-concept chip + the <span class="mono text-[#ffb347]">XTR&#8209;0</span> dev board</span>
  </div>
  <div v-click class="flex items-baseline gap-3">
    <span class="mono text-[0.58rem] text-[#5d6780] w-16 shrink-0">JUL 2026</span>
    <span class="text-[0.76rem]">US Dept. of Commerce signs a <b class="text-white">$75M</b> letter of intent to scale it in American fabs</span>
  </div>
  <div v-click class="flex items-baseline gap-3">
    <span class="mono text-[0.58rem] text-[#ff8a4c] w-16 shrink-0">NOW</span>
    <span class="text-[0.76rem]"><b class="text-white">Z1</b> announced as the first full-scale sampling chip</span>
  </div>
</div>

<div v-click class="panel mt-5 p-4 grid grid-cols-2 gap-y-3 gap-x-4 stat">
  <div><div class="n text-[1.15rem]">269,568</div><div class="u text-[0.55rem] tracking-[0.14em] uppercase">p&#8209;bits on one die</div></div>
  <div><div class="n text-[1.15rem]">&gt; 50 MHz</div><div class="u text-[0.55rem] tracking-[0.14em] uppercase">sampling rate</div></div>
  <div><div class="n text-[1.15rem]">&lt; 1 W</div><div class="u text-[0.55rem] tracking-[0.14em] uppercase">total power draw</div></div>
  <div><div class="n text-[1.15rem]">&lt; 12 mm</div><div class="u text-[0.55rem] tracking-[0.14em] uppercase">per side, room temperature</div></div>
</div>

<div v-click class="mt-3 text-[0.68rem] text-[#7d879c]">
  Ordinary CMOS. <b class="text-white">Not cryogenic, not quantum.</b> Roadmap runs to a billion p&#8209;bits in a rack.
</div>

</div>

<!--

And this is real, and it's recent.

[click]

Extropic was founded by people out of Google's quantum lab, and last year it shipped X0, a working proof-of-concept chip.

[click]

In July, the Commerce Department signed a seventy-five-million-dollar letter of intent to help scale it in American fabs.

[click]

And this summer they announced Z1, the first full-scale one.

[click]

About two hundred seventy thousand p-bits on a single die. Each one talking to sixteen neighbours. Sampling above fifty megahertz. Under twelve millimetres a side, drawing less than one watt.

[click]

Most importantly, this is ordinary CMOS at room temperature. It is not cryogenic and it is not quantum. Their roadmap goes to a billion p-bits in a rack.
-->

---
layout: default
---

<div class="h-full flex flex-col">

<h1 class="mt-3">Ten thousand times. <span class="text-[#5d6780]">Allegedly.</span></h1>

<div class="panel mt-5 px-8 py-7 flex-1 min-h-0 flex items-center">
  <EnergyBars />
</div>

<div class="grid grid-cols-4 gap-3 mt-4 text-[0.68rem]">
  <div v-click class="aster">That number is from a <b class="text-white">simulation</b> of a chip that isn't shipping yet, not a measurement.</div>
  <div v-click class="aster">"The right workloads" is doing heavy lifting. It <b class="text-white">can't run today's AI models</b> at all.</div>
  <div v-click class="aster">You'd have to <b class="text-white">rewrite everything</b> as energy-based models. The software is months old.</div>
  <div v-click class="aster">An independent tester found <b class="text-white">plain laptop code beat their own simulator</b> by 70×.</div>
</div>

<div v-click class="mt-4 text-[0.85rem] text-center text-[#ffb347]">
  Be skeptical of the number. The physics underneath it is the part that isn't sketchy.
</div>

</div>

<!--

Their headline number is up to ten thousand times more energy-efficient than a GPU. And I want to be honest about that one.

[click] It comes from a simulation, of a chip that isn't shipping yet. It's not a measurement.

[click] "For the right workloads" is doing an enormous amount of work in that sentence because a TSU can't run today's AI models at all.

[click] You'd have to rewrite them as energy-based models, and that software ecosystem is a few months old.

[click] And when an independent blogger benchmarked their library, naive laptop code beat it by about seventy times because the hardware it's written for doesn't exist yet.

[click] So: don't buy the ten thousand. But the physics underneath it is not the sketchy part.
-->

---
layout: default
---

<div class="h-full flex flex-col justify-center">


<h1 class="mt-6 !text-[2.9rem] !leading-[1.06] max-w-[44rem]">
  For eighty years we've built computers that spend energy
  <span class="text-[#7dd3fc]">fighting thermodynamics.</span>
</h1>

<h1 v-click class="mt-5 !text-[2.9rem] !leading-[1.06] max-w-[44rem]">
  This is a machine that lets thermodynamics
  <span class="grad-hot">do the computing.</span>
</h1>

<div class="hair w-80 my-8" />

<div v-click class="text-[0.88rem] text-[#98a2b8] max-w-[40rem]">
  Extropic may well not be the company that wins. But reframing noise as a <b class="text-white">resource</b> rather than a defect gives a genuinely different answer to the question <em class="text-white not-italic">"what is a computer?"</em>
</div>

</div>

<!--

So here's why I think it's worth your attention, whether or not it works.

For about eighty years we have built computers that spend energy fighting thermodynamics by pinning every bit down, erasing every trace of randomness, and paying for it in heat.

[click] This is a machine that lets thermodynamics do the computing instead.

[click] Extropic may well not be the company that wins this. But reframing noise as a resource instead of a defect gives a genuinely different answer to the question "what is a computer."
-->

---
layout: default
class: thank-you-slide
---

<div class="absolute inset-0 overflow-hidden">
  <GradientValleys class="h-full w-full" />
</div>

<div class="relative z-10 h-full flex flex-col justify-center items-center text-center">
  <h1 class="!text-[5.2rem] !leading-none !mb-0 tracking-tighter">THANK YOU</h1>
  <div class="hair w-56 my-7" />
  <div class="text-[0.9rem] tracking-[0.04em] text-[#c8d0e0]">Questions?</div>
</div>

<!--

Thank you. Questions?
-->
