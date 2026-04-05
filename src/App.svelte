<script>
  import Header from './lib/Header.svelte';
  import Paragraph from './lib/Paragraph.svelte';
  import LatencyChart from './lib/LatencyChart.svelte';
  import ImageContainer from './lib/ImageContainer.svelte';
  import FeatureCard from './lib/FeatureCard.svelte';
</script>

<div class="w-screen text-slate-900 font-mono antialiased">
  <Header />
  <div class="w-full flex flex-col items-center pt-8 pb-16 bg-linear-to-br from-[#ee0066]/80 to-[#ccff00]/80 to-90%">
    <div class="w-full md:max-w-3xl">
      <Paragraph class="text-4xl font-extrabold">
        A Lightweight Key-Value Database Built in Zig
      </Paragraph>
      <Paragraph class="text-base">
        Inspired by event-driven systems, Sklad uses an <span class="font-extrabold">asynchronous</span> task queue and <span class="font-extrabold">lock-free</span> data structures to achieve predictable performance and low tail latencies.
      </Paragraph>
      <Paragraph class="text-base">
        Currently a prototype. The long-term direction is to explore use in <span class="font-extrabold">IoT</span> and <span class="font-extrabold">edge</span> environments.
      </Paragraph>
      <Paragraph class="text-2xl font-extrabold mt-2 md:mt-4">
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
  <div class="relative z-10 w-full px-8 -mt-8 flex flex-row flex-wrap sm:flex-nowrap items-center justify-center gap-6 mx-auto max-w-3xl text-white">
    <FeatureCard>
      <div class="font-extrabold uppercase">Asynchronous</div>
    </FeatureCard>
    <FeatureCard>
      <div class="font-extrabold uppercase">Built-in</div>
      <div class="font-extrabold uppercase">observability</div>
    </FeatureCard>
    <FeatureCard>
      <div class="text-2xl md:text-3xl">P99<sup class="">*</sup></div>
      <div class="font-extrabold uppercase">&lt; 2ms&nbsp;</div>
    </FeatureCard>
    <FeatureCard>
      <div class="text-2xl md:text-3xl">P50/P99<sup>*</sup></div>
      <div class="font-extrabold uppercase">&lt; 80%</div>
    </FeatureCard>
  </div>
  <div class="w-full pt-8 pb-24 flex flex-col items-center mx-auto max-w-3xl">
    <Paragraph class="text-4xl font-extrabold mt-2 md:mt-4">
      Performance
    </Paragraph>
    <Paragraph class="text-base">
      Preliminary benchmark was done on a laptop using synthetic workload simulating the sensors data.
      The latencies were measured on the client side. Results are indicative only and not representative of production or edge environments.
    </Paragraph>
    <LatencyChart/>
    <Paragraph class="text-4xl font-extrabold mt-2 md:mt-4">
      Built-in observability
    </Paragraph>
    <Paragraph class="text-base">
      Metrics collection is a core feature. The prototype exposes internal statistics that can be visualized through a web dashboard.
    </Paragraph>
    <ImageContainer imageLink="./src/assets/metrics-dashboard.png" />
    <Paragraph class="text-4xl font-extrabold mt-2 md:mt-4">
      <span class="text-[#ee0066]">sklient</span>
    </Paragraph>
    <Paragraph class="text-base">
      A terminal UI client for querying Sklad.
    </Paragraph>
    <ImageContainer imageLink="./src/assets/sklient.png" />
  </div>
</div>
