<script>
  const LINUX_OS = 'linux';
  const MAC_OS = 'macos';
  const AARCH64 = 'aarch64';
  const X86_64 = 'x86_64';

  const OS_LABELS = {
    [MAC_OS]: 'macOS',
    [LINUX_OS]: 'Linux'
  };

  const ARCH_LABELS = {
    [AARCH64]: 'ARM64',
    [X86_64]: 'x86-64'
  };

  const OS_ARCH_PAIRS = {
    [MAC_OS]: [AARCH64],
    [LINUX_OS]: [AARCH64, X86_64],
  };

  let os = MAC_OS;
  let arch = AARCH64;
  let copied = false;

  function setOS(newOS) {
    os = newOS;
    arch = OS_ARCH_PAIRS[newOS][0];
  }

  $: downloadUrl = `https://github.com/sklad-dev/Sklad/releases/download/0.1.0/sklad-0.1.0-${arch}-${os}.tar.gz`;
  $: command = `curl -L "${downloadUrl}" | tar -xz`;

  function copyToClipboard() {
    navigator.clipboard.writeText(command).then(() => {
      copied = true;
      setTimeout(() => {
        copied = false;
      }, 2000);
    });
  }
</script>

<div class="flex flex-col gap-2">
  <div class="flex gap-4 text-sm font-medium font-sans">

    <div class="flex bg-slate-200/60 p-1 border border-slate-300">
      {#each Object.keys(OS_ARCH_PAIRS) as osOption}
        <button class="px-2 py-1 transition-colors cursor-pointer {os === osOption ? 'bg-white shadow-sm text-black' : 'text-gray-600 hover:text-black'}"
                on:click={() => setOS(osOption)}>
          {OS_LABELS[osOption]}
        </button>
      {/each}
    </div>

    <div class="flex bg-slate-200/60 p-1 border border-slate-300 transition-opacity {os === LINUX_OS ? 'opacity-100' : 'opacity-70'}">
      {#each OS_ARCH_PAIRS[os] as archOption}
        <button class="px-2 py-1 transition-colors cursor-pointer {arch === archOption ? 'bg-white shadow-sm text-black' : 'text-gray-600 hover:text-black'}"
                on:click={() => arch = archOption}>
          {ARCH_LABELS[archOption]}
        </button>
      {/each}
    </div>

  </div>

  <div class="relative group flex flex-row">
    <div class="overflow-x-auto text-left border border-black py-4 px-4 bg-slate-50">
      <pre class="font-mono text-base"><code><span class="text-[#ee0066] select-none">$</span> {command}</code></pre>
    </div>
    <button on:click={copyToClipboard}
            class="flex items-center justify-center cursor-pointer min-w-16 border-r border-y border-black bg-slate-200 hover:bg-slate-300 px-1.5 py-1.5 font-sans font-medium transition-colors">
      {#if copied}
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="size-5">
          <path fill-rule="evenodd" d="M10 18a8 8 0 1 0 0-16 8 8 0 0 0 0 16Zm3.857-9.809a.75.75 0 0 0-1.214-.882l-3.483 4.79-1.88-1.88a.75.75 0 1 0-1.06 1.061l2.5 2.5a.75.75 0 0 0 1.137-.089l4-5.5Z" clip-rule="evenodd" />
        </svg>
      {:else}
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="size-5">
          <path d="M7 3.5A1.5 1.5 0 0 1 8.5 2h3.879a1.5 1.5 0 0 1 1.06.44l3.122 3.12A1.5 1.5 0 0 1 17 6.622V12.5a1.5 1.5 0 0 1-1.5 1.5h-1v-3.379a3 3 0 0 0-.879-2.121L10.5 5.379A3 3 0 0 0 8.379 4.5H7v-1Z" />
          <path d="M4.5 6A1.5 1.5 0 0 0 3 7.5v9A1.5 1.5 0 0 0 4.5 18h7a1.5 1.5 0 0 0 1.5-1.5v-5.879a1.5 1.5 0 0 0-.44-1.06L9.44 6.439A1.5 1.5 0 0 0 8.378 6H4.5Z" />
        </svg>
      {/if}
    </button>
  </div>
</div>
