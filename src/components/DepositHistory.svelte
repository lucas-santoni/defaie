<script lang="ts">
  import { request, gql } from "graphql-request"
  import { useQuery } from "@sveltestack/svelte-query"
  import BigNumber from "bignumber.js"
  import _ from "lodash"

  const endpoint = "https://api.thegraph.com/subgraphs/name/aave/protocol-v2"

  export let address: string

  interface UserHistoryEvent {
    id: string
    timestamp: number
    amount: string
    reserve: {
      id: string
      symbol: string
      name: string
      decimals: number
    }
  }

  interface UserHistory {
    depositHistory: Array<UserHistoryEvent>
    redeemUnderlyingHistory: Array<UserHistoryEvent>
  }

  interface GetDepositHistoryResponse {
    user: UserHistory
  }

  interface ParsedUserHistoryEvent {
    direction: "in" | "out"
    amount: BigNumber
    reward?: BigNumber
    balance?: BigNumber
    timestamp: Date
    hash: string
    asset: {
      address: string
      name: string
      symbol: string
      decimals: number
      logo?: string
    }
  }

  const parseUserHistoryEvent = (
    event: UserHistoryEvent,
    direction: "in" | "out"
  ): ParsedUserHistoryEvent => {
    const match = event.id.match(/0x\w+/)
    if (match === null) {
      throw new Error("could not parse transaction hash")
    }

    return {
      direction,
      amount: new BigNumber(event.amount),
      reward: undefined,
      balance: undefined,
      timestamp: new Date(event.timestamp),
      hash: match[0],
      asset: {
        address: event.reserve.id, // TODO: Parse this
        name: event.reserve.name,
        symbol: event.reserve.symbol,
        decimals: event.reserve.decimals,
        logo: undefined
      }
    }
  }

  const getUserHistory = async (address: string) => {
    const resp = await request<GetDepositHistoryResponse>(
      endpoint,
      gql`{
	    user(id: "${address.toLowerCase()}") {
		    depositHistory {
			    id
			    amount
          timestamp
			    reserve {
            id
				    name
            symbol
            decimals
			    }
		    }
        redeemUnderlyingHistory {
			    id
			    amount
          timestamp
			    reserve {
            id
				    name
            symbol
            decimals
			    }
		    }
	    }
    }`
    )

    const rawHistory = resp
    const parsedDeposits = rawHistory.user.depositHistory.map((e) =>
      parseUserHistoryEvent(e, "out")
    )
    const parsedRedeems = rawHistory.user.redeemUnderlyingHistory.map((e) =>
      parseUserHistoryEvent(e, "in")
    )
    const fullHistory: Array<ParsedUserHistoryEvent> = []
      .concat(parsedDeposits)
      .concat(parsedRedeems)

    const groups = _.groupBy(fullHistory, "asset.address")
    for (const groupKey in groups) {
      const group = groups[groupKey]
      const sortedGroup = _.sortBy(group, (x) => x.timestamp.getTime())

      let balance = new BigNumber(0)
      for (const e of sortedGroup) {
        if (e.direction === "out") {
          balance = balance.minus(e.amount)
        } else if (e.direction === "in") {
          balance = balance.plus(e.amount)
          if (balance.isPositive()) {
            e.reward = balance
          }
        }

        e.balance = balance
      }
    }

    // Most recent transactions first
    const sortedHistory = _.sortBy(fullHistory, (x) => x.timestamp.getTime()).reverse()

    return sortedHistory
  }

  const userHistory = useQuery<Array<ParsedUserHistoryEvent>>(["userHistory", address], () =>
    getUserHistory(address)
  )
</script>

<div>
  {#if $userHistory.status === "loading"}
    <p>Loading...</p>
  {:else if $userHistory.data === undefined}
    <p>
      No data! Something went wrong... Are you sure this is a valid Ethereum address? And are you
      sure you have been using AAVE V2?
    </p>
  {:else if $userHistory.status === "error"}
    <p>An error occured. Try again later maybe!</p>
  {:else}
    <ul id="history">
      {#each $userHistory.data as event}
        <li class={event.direction === "in" ? "green" : "red"}>
          <a target="_blank" href={`https://etherscan.io/tx/${event.hash}`}>
            {event.direction === "in" ? "↗" : "↘"}
            <span id="amount">{event.amount.shiftedBy(-event.asset.decimals).toFixed(2)}</span>
            <span id="asset">{event.asset.symbol}</span>
            <span id="balance" class={event.balance?.isPositive() ? "green" : "red"}
              >({event.balance?.shiftedBy(-event.asset.decimals).toFixed(5)})</span
            >
            {#if event.reward !== undefined}
              <span
                >REWARD! {event.reward.shiftedBy(-event.asset.decimals).toFixed(5)}
                {event.asset.symbol}</span
              >
            {/if}
          </a>
        </li>
      {/each}
    </ul>
  {/if}
</div>

<style>
  #history {
    font-family: "Courier New", Courier, monospace;
    list-style: none;
    padding: 0;
    margin-left: 1em;
  }

  #amount {
    min-width: 6em;
    display: inline-block;
  }

  #asset {
    min-width: 4em;
    display: inline-block;
  }

  #balance {
    min-width: 12em;
    display: inline-block;
  }

  #history a {
    color: inherit;
  }

  .red {
    color: red;
  }

  .green {
    color: green;
  }
</style>
