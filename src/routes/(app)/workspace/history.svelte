<script>
    import { history } from '$lib/stores';
  </script>
  
  <svelte:head>
    <title>History | {$WEBUI_NAME}</title>
  </svelte:head>
  
  <div class="px-4 py-2">
    <h1 class="text-xl font-semibold">History</h1>
    <ul class="mt-2">
      {#each $history as url (url)}
        <li class="py-1">
          <a href={url}>{url}</a>
        </li>
      {/each}
    </ul>
  </div>