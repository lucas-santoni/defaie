<h1>DeFaie</h1>

<div>
  <!-- {#if !address || $depositHistory.status === 'loading' || $depositHistory.data === undefined}
    Loading...
  {:else if $depositHistory.status === 'error'}
    <span>Error: {$depositHistory.error}</span>
  {:else}
    <div>
      <p>{$depositHistory.data}</p>
    </div>
  {/if} -->
</div>

<script lang="ts">
  import { request, gql } from "graphql-request"
  import { useQuery } from "@sveltestack/svelte-query"

  const endpoint = "https://api.thegraph.com/subgraphs/name/aave/protocol-v2"

  interface DepositHistory {
    depositHistory: Array<{
      id: string
      amount: string
      reserve: {
        name: string
      }
    }>
  }

  interface GetDepositHistoryResponse {
    data: {
      user: {
        depositHistory: DepositHistory
      }
    }
  }

  const getDeposithistory = async (address: string) => {
    const resp = await request<GetDepositHistoryResponse>(endpoint, gql`{
	    user(id: "${address}") {
		    depositHistory {
			    id
			    amount
			    reserve {
				    name
			    }
		    }
	    }
    }`)

    return resp.data.user.depositHistory
  }

  const address = "0xeaa81a8e73db6b8efd34ecee34cbddf98635163b"
  // const depositHistory = useQuery<DepositHistory>(["depositHistory", address], () => getDeposithistory(address))
</script>