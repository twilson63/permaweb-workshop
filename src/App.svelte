<script>
  import { ArweaveWebWallet } from "arweave-wallet-connector";
  import Arweave from "arweave";
  import { take, takeLast, prop } from "ramda";

  const slug = (a) => `${take(5, a)}...${takeLast(5, a)}`;
  const sleep = (n) => new Promise((r) => setTimeout(r, n));
  const options =
    import.meta.env.MODE === "development"
      ? {
          host: "arweave.net",
          port: 443,
          protocol: "https",
        }
      : {};
  const arweave = Arweave.init(options);
  const wallet = new ArweaveWebWallet({ name: "Permaweb Social" });
  wallet.setUrl("arweave.app");

  const query = `
query {
  transactions(first: 100, tags: {name: "App-Name", values: ["PublicSquare"]}) {
    edges {
      node {
        id
        owner { address }
        tags {
          name
          value
        }
      }
    }
  }
}
  `;

  function doQuery() {
    return arweave.api.post("graphql", { query }).then(prop("data"));
  }

  async function handleSubmit(e) {
    // find a wallet
    if (window.arweaveWallet) {
      await window.arweaveWallet.connect(["DISPATCH"]);
    } else {
      await wallet.connect();
    }
    const tx = await arweave.createTransaction({ data: e.target.status.value });
    tx.addTag("Content-Type", "text/plain");
    tx.addTag("App-Name", "PublicSquare");
    tx.addTag("Version", "1.0.1");
    tx.addTag("Type", "post");

    // dispatch a message
    await window.arweaveWallet.dispatch(tx);

    e.target.reset();
    sleep(500);
    data = doQuery();
  }

  let data = doQuery();
</script>

<div class="md:w-[600px] md:mx-auto">
  <h1 class="my-4 text-4xl text-center">🐘 Permaweb Workshop 🕸</h1>
  <form class="flex flex-col space-y-2" on:submit|preventDefault={handleSubmit}>
    <textarea class="textarea" name="status" placeholder="whats happening?" />
    <button class="btn">POST</button>
  </form>

  {#await data}
    <div class="bg-[#f2f3f4] min-h-screen  grid items-center justify-center">
      <img
        alt="loading"
        src="https://arweave.net/IkMJRqi_0Xx_QhstK4WE3rsQqQxC07n84UagPgqGXfc"
      />
    </div>
  {:then result}
    <div class="mx-2">
      {#each result.data.transactions.edges as edge}
        <div class="card border-2 border-[#ff8500] my-2">
          <div class="card-body">
            <div class="flex space-x-2 font-bold">
              <a
                class="link"
                href="https://viewblock.io/arweave/tx/{edge.node.id}"
              >
                {slug(edge.node.id)}
              </a>
              &nbsp; by
              <a
                class="link"
                href="https://viewblock.io/arweave/address/{edge.node.owner
                  .address}"
              >
                {slug(edge.node.owner.address)}
              </a>
            </div>
            {#await arweave.api.get(edge.node.id) then response}
              <div class="py-2">
                {response.data}
              </div>
            {/await}
          </div>
        </div>
      {/each}
    </div>
  {/await}
</div>
