<script>
  let className = '';
  export let leading = 'tight';
  export { className as class };
</script>

<p class="leading-{leading} {className} px-8 mb-4 w-full">
  <slot></slot>
</p>
