<script>
  import Header from './lib/Header.svelte';
  import Paragraph from './lib/Paragraph.svelte';
  import LatencyChart from './lib/LatencyChart.svelte';
  import ImageContainer from './lib/ImageContainer.svelte';
  import FeatureCard from './lib/FeatureCard.svelte';
  import Carousel from './lib/Carousel.svelte';
  import TerminalWindowFrame from './lib/TerminalWindowFrame.svelte';
  import GithubButton from './lib/GithubButton.svelte';
</script>

<div class="w-full text-slate-900 font-mono antialiased">
  <Header />
  <div class="w-full flex flex-col items-center pt-8 pb-16 bg-linear-to-br from-[#ccff00] to-[#2200cc]/80 from-10%">
    <div class="w-full md:max-w-3xl">
      <Paragraph class="text-4xl md:text-5xl font-semibold tracking-tight">
        A Lightweight Key-Value Database Built in Zig
      </Paragraph>
      <Paragraph class="text-base" leading='relaxed'>
        Inspired by event-driven systems, Sklad uses an <span class="font-extrabold">asynchronous</span> task queue and <span class="font-extrabold">lock-free</span> data structures to achieve predictable performance and low tail latencies.
      </Paragraph>
      <Paragraph class="text-base" leading='relaxed'>
        Currently a prototype. The long-term direction is to explore use in <span class="font-extrabold">IoT</span> and <span class="font-extrabold">edge</span> environments.
      </Paragraph>
      <Paragraph class="text-2xl md:text-3xl font-semibold tracking-tight mt-2 md:mt-4">
        Installation
      </Paragraph>
      <div class="px-8 mb-4 w-full">
        <div class="overflow-x-auto text-left border border-black py-2 bg-slate-50">
  <pre class="font-mono text-base"><code>  <span class="text-[#ee0066] select-none">$</span> git clone https://github.com/sklad-dev/Sklad.git
  <span class="text-[#ee0066] select-none">$</span> cd Sklad
  <span class="text-[#ee0066] select-none">$</span> zig build --release=safe
  <span class="text-[#ee0066] select-none">$</span> ./zig-out/bin/sklad</code></pre>
          </div>
      </div>
    </div>
  </div>
  <div class="relative z-10 w-full px-8 -mt-10 flex flex-row flex-wrap sm:flex-nowrap items-center justify-center gap-6 mx-auto max-w-3xl text-white">
    <FeatureCard>
      <div class="font-bold uppercase tracking-wide">Asynchronous</div>
    </FeatureCard>
    <FeatureCard>
      <div class="font-extrabold uppercase">Built-in</div>
      <div class="font-extrabold uppercase">observability</div>
    </FeatureCard>
    <FeatureCard>
      <div class="text-2xl md:text-3xl font-bold">&lt;2ms</div>
      <div class="text-sm font-semibold uppercase tracking-wider">P99<sup>*</sup></div>
    </FeatureCard>
    <FeatureCard>
      <div class="text-2xl md:text-3xl font-bold">&lt;80%</div>
      <div class="text-sm font-semibold uppercase tracking-wider">P50/P99<sup>*</sup></div>
    </FeatureCard>
  </div>
  <div class="w-full flex flex-col items-center mx-auto max-w-3xl mt-4">
    <Paragraph class="text-xs text-gray-700" leading='relaxed'>
      <sup>*</sup>Client-side measurements for 1 million requests on a consumer laptop, using 32 parallel connections and a 70/30 write-to-read ratio
    </Paragraph>
  </div>
  <div class="w-full flex flex-col items-center mx-auto max-w-3xl">
    <Paragraph class="text-3xl md:text-4xl font-semibold tracking-tight mt-4 md:mt-6">
      Performance
    </Paragraph>
    <Paragraph class="text-base" leading='relaxed'>
      Preliminary benchmark was done on a consumer laptop using synthetic workload simulating the sensors data.
      The latencies were measured on the client side. Results are indicative only and not representative of production or edge environments.
    </Paragraph>
    <LatencyChart/>
    <Paragraph class="text-3xl md:text-4xl font-semibold tracking-tight mt-4 md:mt-8">
      Built-in observability
    </Paragraph>
    <Paragraph class="text-base" leading='relaxed'>
      Metrics collection is a core feature. The prototype exposes internal statistics that can be visualized through a web dashboard.
    </Paragraph>
    <ImageContainer imageLink="./src/assets/metrics-dashboard.png" />
  </div>
  <div class="w-full flex flex-col items-center pt-8 pb-12 bg-linear-to-br from-[#2200cc] to-[#ee0066] text-white">
    <div class="w-full flex flex-col items-center md:max-w-3xl">
      <Paragraph class="text-3xl md:text-4xl font-semibold tracking-tight">
        <span class="text-[#ccff00]">sklient</span>
      </Paragraph>
      <Paragraph class="text-base" leading='relaxed'>
        A terminal UI client for interacting with Sklad.
      </Paragraph>
      <Paragraph class="text-base" leading='relaxed'>
        Two modes: <span class="font-extrabold">TUI mode</span> guides you through queries with structured input elements, while <span class="font-extrabold">Raw mode</span> lets you freely type queries directly.
      </Paragraph>
      <div class="px-8 py-4 w-full">
        <Carousel class="rounded-lg shadow-2xl shadow-black/50">
          <TerminalWindowFrame>
            <img src="./src/assets/sklient-tui.png" alt="sklient terminal UI - TUI mode" class="w-full block">
          </TerminalWindowFrame>
          <TerminalWindowFrame>
            <img src="./src/assets/sklient-raw.png" alt="sklient terminal UI - raw mode" class="w-full block">
          </TerminalWindowFrame>
        </Carousel>
      </div>
      <GithubButton class="px-8 mt-6 mb-2" />
    </div>
  </div>
  <footer class="w-full py-4 flex flex-col items-center gap-2 bg-black text-white">
    <p class="text-sm mx-4">Licensed under the Apache License, Version 2.0</p>
  </footer>
</div>
