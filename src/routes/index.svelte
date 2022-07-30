<script lang="ts">
  import { QueryClient, QueryClientProvider } from "@sveltestack/svelte-query"
  import DepositHistory from "../components/DepositHistory.svelte"

  const queryClient = new QueryClient()

  let address: string | undefined
</script>

<div id="main">
  <h1 id="logo"><a href="/">🤕 DeF<span id="aie">aie</span></a></h1>

  <QueryClientProvider client={queryClient}>
    <p>
      Working on address <input
        id="input"
        type="text"
        bind:value={address}
        placeholder="0x..."
        on:blur={() => queryClient.invalidateQueries("userHistory")}
      />.
    </p>
    {#if address !== undefined}
      <DepositHistory {address} />
    {/if}
  </QueryClientProvider>
</div>

<style>
  #main {
    width: fit-content;
    overflow-x: scroll;
    white-space: nowrap;
  }

  #input {
    border-radius: 0;
    border-color: #eee;
    box-shadow: none;
    font-family: "Courier New", Courier, monospace;
  }

  :global(a) {
    text-decoration: none;
  }

  #logo a {
    color: black;
  }

  #aie {
    color: red;
  }
</style>
