<script>
  import Header from './lib/Header.svelte';
  import Paragraph from './lib/Paragraph.svelte';
  import LatencyChart from './lib/LatencyChart.svelte';
</script>

<div class="w-screen text-slate-900 font-sans antialiased">
  <Header />
  <div class="w-full pb-24 flex flex-col items-center mx-auto max-w-3xl">
    <Paragraph class="text-4xl font-extrabold mt-4 md:mt-12">
      A Lightweight Key-Value Database Built in Zig
    </Paragraph>
    <Paragraph class="text-base">
      Inspired by event-driven systems, Sklad uses an asynchronous task queue and lock-free data structures to achieve predictable performance and low tail latencies.
    </Paragraph>
    <Paragraph class="text-base">
      Currently a prototype. The long-term direction is to explore use in <span class="font-bold">IoT</span> and <span class="font-bold">edge</span> environments.
    </Paragraph>
    <Paragraph class="text-2xl font-extrabold mt-2 md:mt-4">
      Building & running
    </Paragraph>
    <div class="px-8 mb-4 w-full">
      <div class="overflow-x-auto text-left border border-black py-2">
<pre class="font-mono text-base"><code>  <span class="text-[#ee0066] select-none">$</span> git clone https://github.com/sklad-db/Sklad.git
  <span class="text-[#ee0066] select-none">$</span> cd Sklad
  <span class="text-[#ee0066] select-none">$</span> zig build --release=safe
  <span class="text-[#ee0066] select-none">$</span> ./zig-out/bin/sklad</code></pre>
        </div>
    </div>
    <Paragraph class="text-2xl font-extrabold mt-2 md:mt-4">
      Performance
    </Paragraph>
    <Paragraph class="text-base">
      Preliminary local benchmark was done on a laptop using synthetic workload simulating the sensors data.
      The latencies were measured on the client side. Results are indicative only and not representative of production or edge environments.
    </Paragraph>
    <LatencyChart/>
    <Paragraph class="text-2xl font-extrabold mt-2 md:mt-4">
      Built-in observability
    </Paragraph>
    <Paragraph class="text-base">
      Sklad has metrics collection as a core feature. The prototype exposes internal statistics that can be visualized through a web dashboard.
    </Paragraph>
    <div class="px-8 my-4 w-full">
      <img src="./src/assets/metrics-dashboard.png" alt="Metrics Dashboard screenshot" class="border-8">
    </div>
    <Paragraph class="text-2xl font-extrabold mt-2 md:mt-4">
      <span class="text-[#ee0066]">sklient</span> - a CLI tool
    </Paragraph>
    <div class="px-8 mb-4 w-full">
      <div class="overflow-x-auto text-left border border-black py-2">
<pre class="font-mono text-base"><code>  <span class="text-[#ee0066] select-none">$</span> git clone https://github.com/sklad-db/sklient.git
  <span class="text-[#ee0066] select-none">$</span> cd sklient
  <span class="text-[#ee0066] select-none">$</span> zig build --release=safe
  <span class="text-[#ee0066] select-none">$</span> ./zig-out/bin/sklient</code></pre>
        </div>
    </div>
  </div>
</div>
