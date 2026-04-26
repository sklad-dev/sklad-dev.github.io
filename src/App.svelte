<script>
  import Header from './lib/Header.svelte';
  import Landing from './pages/Landing.svelte';
  import Posts from './pages/Posts.svelte';
  import NotFound from './pages/posts/NotFound.svelte';

  import Release_0_1_0 from './pages/posts/Release_0_1_0.svelte';
  import Wal from './pages/posts/Wal.svelte';

  let hash = $state(window.location.hash);
  let segments = $derived(hash.replace(/^#\//, '').split('/'));

  window.addEventListener('hashchange', () => {
    hash = window.location.hash;
    window.scrollTo(0, 0);
  });
</script>

<div class="w-full text-slate-900 font-mono antialiased">
  <Header />
  {#if segments[0] === 'posts' && segments[1]}
    {#if segments[1] === '0.1.0'}
      <Release_0_1_0 />
    {:else if segments[1] === 'wal'}
      <Wal />
    {:else}
      <NotFound />
    {/if}
  {:else if segments[0] === 'posts'}
    <Posts />
  {:else}
    <Landing />
  {/if}
</div>
