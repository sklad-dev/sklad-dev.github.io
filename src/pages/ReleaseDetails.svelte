<script>
  import GithubButton from '../lib/GithubButton.svelte';
  import Paragraph from '../lib/Paragraph.svelte';
</script>

<div class="w-full flex flex-col items-start mx-auto max-w-3xl mt-2 mb-20">
  <Paragraph leading='relaxed' class="mb-6">
    <a href="#/release-notes" class="hover:underline hover:text-gray-700 transition-colors">
      &#8592; All release notes
    </a>
  </Paragraph>
  <Paragraph class="text-sm text-gray-700 font-medium" leading='relaxed'>
    April 12, 2026
  </Paragraph>
  <Paragraph class="text-2xl md:text-3xl font-semibold tracking-tight">
    0.1.0 Release notes
  </Paragraph>
  <Paragraph leading='relaxed'>
    <a href="https://github.com/sklad-dev/Sklad/releases/tag/0.1.0"
       target="_blank" rel="noopener noreferrer" class="underline text-blue-700 hover:text-blue-500 transition-colors">
      View release on GitHub
    </a>
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    <span class="font-bold">The first Sklad release is out!</span> But beware, the project is still in its prototype stage, meaning that the core architecture is in place and the database is usable,
    however, it is not yet production-ready. There are an unknown number of bugs and untested edge cases, all of which need to be surfaced through rigorous stress testing.
  </Paragraph>
  <Paragraph class="text-lg md:text-xl font-semibold tracking-tight border-l-8 border-[#ccff00]">
    What's included today?
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    Sklad 0.1.0 implements fundamental CRUD operations (set, get, delete), TTL support, range queries and internal metrics collection.
    Under the hood, it uses an LSM-tree with a lock-free skip-list memtable, simple tiered compaction, and an asynchronous non-blocking I/O model
    backed by a dynamic worker pool.
  </Paragraph>
  <Paragraph class="text-lg md:text-xl font-semibold tracking-tight border-l-8 border-[#ccff00]">
    What's still missing?
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    There are known gaps and likely undiscovered bugs. Stress testing has not been done at scale, and several features considered essential for a stable 1.0
    release are still on the roadmap.
  </Paragraph>
  <Paragraph class="text-base italic" leading='relaxed'>
    Speaking of which…
  </Paragraph>
  <Paragraph class="text-lg md:text-xl font-semibold tracking-tight border-l-8 border-[#ccff00]">
    The roadmap to 1.0
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    The vision for Sklad is to grow into a database suited for IoT and edge environments. Thus, the primary goal before any stable release is stress testing
    on resource-constrained hardware.
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    Beyond testing, the following features are considered must-haves for 1.0:
  </Paragraph>
  <ul class="leading-relaxed text-base px-8 mb-4 list-disc ml-4">
    <li>Unix domain sockets support</li>
    <li>
      Data integrity checks
      <ul class="list-disc ml-4">
        <li>Per-record xxHash-3 checksums for WAL entries</li>
        <li>File-level xxHash-3 checksums for SSTable files</li>
        <li>SSTable completion marker to detect interrupted writes</li>
      </ul>
    </li>
    <li>Finalized metrics aggregation</li>
    <li>More advanced compaction strategies</li>
    <li>Client libraries for (but not limited to) C/C++, Rust, Go, Java and Python</li>
  </ul>
  <Paragraph class="text-lg md:text-xl font-semibold tracking-tight border-l-8 border-[#ccff00]">
    Future work
  </Paragraph>
  <Paragraph class="text-base" leading='relaxed'>
    There is a list of features I have in mind, but I am not sure if they are necessary for the 1.0 release.
  </Paragraph>
  <ul class="leading-relaxed text-base px-8 mb-4 list-disc ml-4">
    <li>Key-value separation</li>
    <li>io_uring support for Linux version of Sklad</li>
  </ul>
  <GithubButton class="w-full flex justify-center mt-8 mb-2" />
</div>
