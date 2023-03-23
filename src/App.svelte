<script>
  import Arweave from "arweave";

  const options =
    import.meta.env.MODE === "development"
      ? { host: "arweave.net", port: 443, protocol: "https" }
      : {};
  const arweave = Arweave.init(options);

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
}
`;

</script>

<h1 class="my-4 text-4xl text-center">🐘 Permaweb Workshop 🕸</h1>
{#await arweave.api.post("graphql", { query })}
  <div
    class="bg-[#f2f3f4] min-h-screen min-w-screen grid items-center justify-center"
  >
    <img
      alt="loading"
      src="https://arweave.net/IkMJRqi_0Xx_QhstK4WE3rsQqQxC07n84UagPgqGXfc"
    />
  </div>
{:then result}
  <div class="mx-2">
    {#each result.data.data.transactions.edges as edge}
      <div class="border-2 border-[#ff8500] py-4 my-2 mx-auto text-center">
        <a href="https://viewblock.io/arweave/tx/{edge.node.id}">
          {edge.node.id}
        </a>
        -
        <a
          href="https://viewblock.io/arweave/address/{edge.node.owner.address}"
        >
          {edge.node.owner.address}
        </a>
      </div>
    {/each}
  </div>
{/await}
