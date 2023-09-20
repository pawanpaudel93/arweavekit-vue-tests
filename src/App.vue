<script setup lang="ts">
import { Othent } from "arweavekit/auth";
import {
  createTransaction,
  signTransaction,
  postTransaction,
} from "arweavekit/transaction";
import { queryAllTransactionsGQL } from "arweavekit/graphql";
import { ref } from "vue";
import Transaction from "arweave/node/lib/transaction";
import { createContract, writeContract } from "arweavekit/contract";

const files = ref<File[]>([]);

function handleFileChange(e: Event) {
  const target = e.target as HTMLInputElement;
  const droppedFiles = target.files;
  if (droppedFiles && droppedFiles.length > 0) {
    files.value.push(droppedFiles[0]);
  }
}

async function logIn() {
  const userDetails = await Othent.logIn({
    apiId: import.meta.env.VITE_OTHENT_API_ID,
  });

  console.log("Othent Login details", userDetails);
}

async function logOut() {
  await Othent.logOut({
    apiId: import.meta.env.VITE_OTHENT_API_ID,
  });

  console.log("Logged out of Othent");
}

const toArrayBuffer = (file: Blob) =>
  new Promise((resolve) => {
    const fr = new FileReader();
    fr.readAsArrayBuffer(file);
    fr.addEventListener("loadend", (evt) => {
      resolve(evt.target!.result);
    });
  });

async function createArweaveTransaction() {
  try {
    console.log(files.value[0]);
    const data = (await toArrayBuffer(files.value[0])) as ArrayBuffer;
    let creator = "Anon";
    try {
      creator = await window.arweaveWallet.getActiveAddress();
    } catch (e) {}
    const metaData = [
      { name: "Creator", value: creator },
      { name: "Title", value: "Bundlr PNG" },
      { name: "Content-Type", value: "image/png" },
    ];
    const transaction = await createTransaction({
      data,
      type: "data",
      environment: "mainnet",
      options: {
        tags: metaData,
      },
    });
    console.log(transaction);
    const signedTransaction = await signTransaction({
      environment: "mainnet",
      createdTransaction: transaction,
    });
    console.log("Signed", signedTransaction);
    const postedTransaction = await postTransaction({
      environment: "mainnet",
      transaction: signedTransaction as Transaction,
    });
    console.log("Posted", postedTransaction);
    console.log("Posted", transaction.id);
    // Example successful id: https://a6gp5qkf2e6mpljgb5ahyk2nobkgtp2zkdqvh2jc6u7j4occw6ha.arweave.net/B4z-wUXRPMetJg9AfCtNcFRpv1lQ4VPpIvU-njhCt44
  } catch (error: any) {
    console.log("Transaction failed due to:", error);
  }
}

async function createBundlrTransaction() {
  try {
    console.log(files.value[0]);
    const data = (await toArrayBuffer(files.value[0])) as ArrayBuffer;
    let creator = "Anon";
    try {
      creator = await window.arweaveWallet.getActiveAddress();
    } catch (e) {}
    const metaData = [
      { name: "Creator", value: creator },
      { name: "Title", value: "Bundlr PNG" },
      { name: "Content-Type", value: "image/png" },
    ];
    const transaction = await createTransaction({
      data: Buffer.from(data),
      // data: data,
      // data: window.Buffer.from(data),
      type: "data",
      environment: "mainnet",
      options: {
        tags: metaData,
        useBundlr: true,
        signAndPost: true,
      },
    });
    console.log("Posted", transaction.postedTransaction.id);
    console.log("Posted txn", transaction);
    // Example successful id: https://a6gp5qkf2e6mpljgb5ahyk2nobkgtp2zkdqvh2jc6u7j4occw6ha.arweave.net/B4z-wUXRPMetJg9AfCtNcFRpv1lQ4VPpIvU-njhCt44
  } catch (error: any) {
    console.log("Transaction failed due to:", error);
  }
}

async function queryGQLTxn() {
  const query = `
query{
  transactions(tags: [
  { name: "Contract-Src", values: ["DG22I8pR_5_7EJGvj5FbZIeEOgfm2o26xwAW5y4Dd14"] }
  ] first: 100) {
edges {
  node {
    id
    owner {
      address
    }
    tags {
      name
      value
    }
    block {
      timestamp
    }
  }
}
}
}

`;

  const res = await queryAllTransactionsGQL(query, {
    gateway: "arweave.net",
    filters: {},
  });

  console.log("This is the result of the query", res);
}

async function createContractTestNet() {
  const { contract, result } = await createContract({
    environment: "testnet",
    initialState: JSON.stringify({ counter: 0 }),
    // strategy: "arweave",
    contractSource: `
    export function handle(state, action) {
      if (action.input.function === 'decrement') {
        state.counter -= 1
      }
      if (action.input.function === 'increment') {
        state.counter += 1
      }
      return { state }
    }`,
  });
  console.log(result, contract);
}

async function writeContractTestNet() {
  const response = await writeContract({
    environment: "testnet",
    contractTxId: "CO7NkmEVj4wEwPySxYUY0_ElrEqU4IeTglU2IilCnLA",
    options: {
      function: "increment",
    },
    // strategy: "arweave",
  });
  console.log(response);
}
</script>

<template>
  <div class="container mx-auto flex justify-center h-screen pt-12">
    <div className="flex flex-col gap-4 w-1/2">
      <button className="border-4" @click="logIn">LogIn with Othent</button>
      <button className="border-4" @click="logOut">LogOut with Othent</button>
      <input className="border-4" type="file" @change="handleFileChange" />
      <button className="border-4" @click="createArweaveTransaction">
        Create and Post Arweave Transaction
      </button>
      <button className="border-4" @click="createBundlrTransaction">
        Create and Post Bundlr Transaction
      </button>
      <button className="border-4" @click="queryGQLTxn">
        Query All GQL Transactions
      </button>
      <button className="border-4" @click="createContractTestNet">
        Create Contract
      </button>
      <button className="border-4" @click="writeContractTestNet">
        Write Contract
      </button>
    </div>
  </div>
</template>
