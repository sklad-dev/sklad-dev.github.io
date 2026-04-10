<script>
  let { class: className = '' } = $props();
</script>

<div class="{className}">
  <a href="https://github.com/sklad-dev/Sklad"
     target="_blank"
     rel="noopener noreferrer"
     class="inline-flex items-center gap-2 bg-black text-white px-5 py-2.5 font-bold uppercase tracking-wide text-sm hover:bg-gray-800 transition-colors w-fit border-2 border-black">
    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" viewBox="0 0 24 24">
      <path d="M12 .587l3.668 7.568 8.332 1.151-6.064 5.828 1.48 8.279-7.416-3.967-7.417 3.967 1.481-8.279-6.064-5.828 8.332-1.151z"/>
    </svg>
    Star on GitHub
  </a>
</div>
