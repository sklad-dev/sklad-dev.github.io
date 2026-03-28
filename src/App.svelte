<script>
  import Header from './lib/Header.svelte';
  import Paragraph from './lib/Paragraph.svelte';
</script>

<div class="w-screen text-slate-900 font-sans antialiased">
  <Header />
  <div class="w-full pb-24 flex flex-col items-center mx-auto max-w-3xl">
    <Paragraph class="text-2xl md:text-4xl font-extrabold mt-4 md:mt-12">
      A Lightweight Key-Value Database Built in Zig
    </Paragraph>
    <Paragraph class="text-base md:text-xl">
      Inspired by event-driven systems, Sklad uses an asynchronous task queue and lock-free data structures to achieve predictable performance and low tail latencies.
    </Paragraph>
    <Paragraph class="text-base md:text-xl">
      Currently a prototype. The long-term direction is to explore use in IoT and edge environments.
    </Paragraph>
    <Paragraph class="text-xl md:text-2xl font-extrabold mt-2 md:mt-4">
      Building & running
    </Paragraph>
    <div class="px-8 mb-4 w-full">
      <div class="rounded-2xl bg-slate-900 overflow-hidden shadow-2xl shadow-slate-900/20 ring-1 ring-white/10">
        <div class="flex items-center px-4 py-3 bg-slate-800/80 border-b border-slate-700/50">
          <div class="flex space-x-2">
            <div class="w-3 h-3 rounded-full bg-red-500/90 shadow-sm"></div>
            <div class="w-3 h-3 rounded-full bg-yellow-500/90 shadow-sm"></div>
            <div class="w-3 h-3 rounded-full bg-green-500/90 shadow-sm"></div>
          </div>
        </div>
        <div class="overflow-x-auto text-left">
          <pre class="font-mono text-sm text-slate-300"><code>
  <span class="text-pink-500 select-none">~ %</span> git clone https://github.com/sklad-db/Sklad.git
  <span class="text-pink-500 select-none">~ %</span> cd Sklad
  <span class="text-pink-500 select-none">~ %</span> zig build --release=safe
  <span class="text-pink-500 select-none">~ %</span> ./zig-out/bin/sklad
          </code></pre>
        </div>
      </div>
    </div>
  </div>
</div>
