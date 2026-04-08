<script>
  let { children, interval = 5000, class: className = '' } = $props();

  let count = $state(0);
  let current = $state(0);
  let paused = $state(false);

  let track;

  $effect(() => {
    if (!track) return;
    count = track.children.length;
  });

  $effect(() => {
    if (paused || count <= 1) return;
    const id = setInterval(() => {
      current = (current + 1) % count;
    }, interval);
    return () => clearInterval(id);
  });
</script>

<div role="region"
     aria-roledescription="carousel"
     aria-label="Carousel"
     onmouseenter={() => (paused = true)}
     onmouseleave={() => (paused = false)}
     onfocusin={() => (paused = true)}
     onfocusout={() => (paused = false)}
     class="flex flex-col items-center">
  <div class="w-full overflow-hidden {className}">
    <div bind:this={track} class="track flex transition-transform duration-500 ease-in-out"
                           style="transform: translateX(-{current * 100}%)">
      {@render children()}
    </div>
  </div>

  {#if count > 1}
    <div class="z-50 relative mt-4 flex gap-3" role="tablist" aria-label="Slides">
      {#each Array(count) as _, i}
        <button
          role="tab"
          aria-selected={i === current}
          aria-label={`Slide ${i + 1}`}
          onclick={() => (current = i)}
          class="size-2.5 rounded-full cursor-pointer transition-colors duration-300 {i === current
            ? 'bg-slate-300'
            : 'bg-slate-800 hover:bg-slate-600'}"
        ></button>
      {/each}
    </div>
  {/if}
</div>

<style>
  .track > :global(*) {
    flex-shrink: 0;
    width: 100%;
  }
</style>
