<script>
  import Header from './lib/Header.svelte';
  import Landing from './pages/Landing.svelte';
  import ReleaseNotes from './pages/ReleaseNotes.svelte';
  import ReleaseDetails from './pages/ReleaseDetails.svelte';

  let hash = $state(window.location.hash);
  let segments = $derived(hash.replace(/^#\//, '').split('/'));

  window.addEventListener('hashchange', () => {
    hash = window.location.hash;
    window.scrollTo(0, 0);
  });
</script>

<div class="w-full text-slate-900 font-mono antialiased">
  <Header />
  {#if segments[0] === 'release-notes' && segments[1]}
    <ReleaseDetails />
  {:else if segments[0] === 'release-notes'}
    <ReleaseNotes />
  {:else}
    <Landing />
  {/if}
</div>
