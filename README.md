# Permaweb Workshop Part 1

Querying Arweave with Arweave SDK

# Getting started with ArweaveJS and GraphQL

Requirements:

* NodeJS v16+
* VSCode or equivalent
* Fullstack Javascript/Typescript experience
* JAM Stack experience with GraphQL
* Some experience with Svelte is a plus but not necessary

## Setup

```sh
npx degit twilson63/permaweb-workshop
cd permaweb-workshop
yarn
yarn dev
```

## Install arweave js

```sh
npm install arweave
```

## First Query

Open `src/App.svelte`, in the script section lets create a query.

```svelte
<script>
  import Arweave from 'arweave'
  
  const options = import.meta.env.MODE === 'development' ? { host: 'arweave.net', port: 443, protocol: 'https' } : {}
  const arweave = Arweave.init(options)
  
  const query = `
query {
  transactions {
    edges {
      node {
        id
      }
    }
  }
}`

  function load() {
    return arweave.api.post('graphql', { query })
  }

</script>

{#await load() then result}
  {#each result.data.data.transactions.edges as edge}
     <div class="border-1 py-4">{edge.node.id}</div>
  {/each}
{/await}

```

## Get First 100

Change the query to use an input parameter to get the first 100 transactions with owner address

```js
const query = `
query {
  transactions(first: 100) {
    edges {
      node {
        id
        owner { address }
      }
    }
  }
}`
```

## Get transactions by owner address

```js
const query = `
query {
  transactions(owners: ["XXXX"]) {
    edges {
      node {
        id
        owner { address }
      }
    }
  }
}`
```


## Show Tags in Query Result

Tags are key/value pairs of descriptive meta-data on a given transaction.

```js
const query = `
query {
  transactions(first: 100) {
    edges {
      node {
        id,
        tags {
          name 
          value
        }
      }
    }
  }
}`
```

```svelte
{#await load() then result}
  {#each result.data.data.transactions.edges as edge}
     <div class="border-1 py-4">{edge.node.id} - ({edge.node.owner.address})</div>
  {/each}
{/await}
```

## Get transactions by content type

```js
const query = `
query {
  transactions(tags: { name: "Content-Type", values: ["image/png"] }) {
    edges {
      node {
        id,
        tags {
          name 
          value
        }
      }
    }
  }
}`
```

## Get transactions by App-Name Tag

```js
const query = `
query {
  transactions(tags: { name: "App-Name", values: ["PublicSquare"] }) {
    edges {
      node {
        id,
        tags {
          name 
          value
        }
      }
    }
  }
}`
```

## Get transaction data by TX ID

```svelte
{#await arweave.api.get(edge.node.id) then response}
  {response.data}
{/await}
```

## Summary

In this workshop, we demonstrated how to install the arweave library and how to query with graphQL using the arweave library and various graphql queries. Also, we demonstrated how to get the transaction data using the arweave libary. In the next workshop, we will show how to create a wallet and submit a transaction using a wallet.


