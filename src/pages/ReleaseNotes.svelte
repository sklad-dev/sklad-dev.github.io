<script>
  import Paragraph from '../lib/Paragraph.svelte';
</script>

<div class="w-full flex flex-col items-start mx-auto max-w-3xl mt-4 mb-20">
  <h1 class="text-4xl md:text-5xl font-semibold tracking-tight px-8 mb-10">Release Notes</h1>
  <Paragraph>
    <a href="#/release-notes/0.1.0" class="group w-full border border-gray-200 hover:border-gray-900 transition-colors p-6 flex flex-col gap-2">
      <div class="flex items-center justify-between">
        <span class="text-xs font-medium uppercase tracking-widest">April 12, 2026</span>
      </div>
      <p class="text-xl font-semibold tracking-tight bg-[#0022ff] text-white px-2 py-0.5 w-fit">0.1.0</p>
      <p class="text-sm leading-relaxed">
        First prototype release. Implements set/get/delete, TTL, range queries, and built-in metrics.
      </p>
      <p class="text-sm font-semibold mt-1 group-hover:underline text-gray-500 group-hover:text-black">Read more →</p>
    </a>
  </Paragraph>
</div>