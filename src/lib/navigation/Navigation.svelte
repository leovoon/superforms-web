<script lang="ts">
  import { onMount } from 'svelte';
  import { getDrawerStore } from '@skeletonlabs/skeleton';
  import A from './A.svelte';
  import Title from './Title.svelte';
  import { page } from '$app/stores';
  import { concepts } from './sections';
  import skeleton from '$lib/assets/skeleton.svg?raw';
  import mdsvex from '$lib/assets/mdsvex.svg?raw';
  import SearchButton from '../../routes/SearchButton.svelte';

  const drawerStore = getDrawerStore();

  function drawerClose(): void {
    drawerStore.close();
  }

  let nav: HTMLElement;

  onMount(() => {
    nav.addEventListener('click', (e) => {
      const target = e.target as HTMLElement;
      if (target && target.tagName == 'A') {
        drawerClose();
      }
    });
  });

  $: active = $page.url.pathname;
</script>

<nav class="list-nav p-4" bind:this={nav}>
  <SearchButton cls="md:hidden mx-4 mb-3" width="w-40" />
  <ul>
    <li><A href="/">Home</A></li>
    <li><A href="/get-started">Get started</A></li>
    <li><A href="/api">API</A></li>
    <li><A href="/sponsors">Sponsors</A></li>
    <li><A href="/migration-v2" class="glow">Version 2 alpha</A></li>

    <Title id="concepts">Concepts</Title>

    {#each concepts as concept}
      <li><A href={concept[0]}>{concept[1]}</A></li>
    {/each}

    <Title id="concepts">Guides</Title>

    <li><A href="/components">Componentization</A></li>
    <li><A href="/crud">CRUD tutorial</A></li>
    <li><A href="/default-values">Default values</A></li>
    <li><A href="/super-debug">SuperDebug</A></li>

    <Title id="concepts">Other topics</Title>

    <li><A href="/contributing">Contribute</A></li>
    <li><A href="/faq">FAQ</A></li>
    <li><A href="/flash-messages">Flash messages</A></li>
    <li><A href="/playground">Playground</A></li>
    <li><A href="/rate-limiting">Rate limiting</A></li>
    <li><A href="/legacy">Legacy version</A></li>
  </ul>

  <div class="p-4 text-gray-600">
    <hr class="mb-2 mt-5" />
    Built with<br />
    <a
      href="https://www.skeleton.dev/"
      target="_blank"
      class="mt-1 !p-0 text-gray-400 hover:!bg-transparent hover:text-gray-300"
      >{@html skeleton}</a>
    <a
      href="https://mdsvex.com/"
      target="_blank"
      class="mt-0 w-20 !p-0 text-gray-400 hover:!bg-transparent hover:text-gray-300"
      >{@html mdsvex}</a>
  </div>
</nav>

<style lang="scss">
  :global(.glow:not(.bg-primary-active-token)) {
    -webkit-animation: glow 1s ease-in-out infinite alternate;
    -moz-animation: glow 1s ease-in-out infinite alternate;
    animation: glow 1s ease-in-out infinite alternate;
  }

  @keyframes glow {
    from {
      text-shadow: 0 0 10px #eeeeee, 0 0 20px #000000, 0 0 30px #000000,
        0 0 40px #000000, 0 0;
    }
    to {
      text-shadow: 0 0 20px #eeeeee, 0 0 30px #ff4da6, 0 0 40px #ff4da6,
        0 0 50px #ff4da6, 0 0;
    }
  }
</style>
