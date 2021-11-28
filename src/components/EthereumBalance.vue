<template>
  <div>
    <div id="balance" />

    <hr />
    <h2>Wallet test</h2>
    <p>
      Write a function that allow user to connect his metamask wallet on the
      ethereum network to our app when he click on connect button.<br />
      If the user has already connected his wallet you should hide the connect
      button and display the list of his accounts with the account balance in
      ETH.<br />
      You must get the account list only once on connection and save them on
      your state (store).<br />
      Finally, the user should be able to see the amount of each account in
      USDT. You can use
      <a href="https://coinmarketcap.com/">coinmarketcap</a> API to get the
      conversion rate.<br />
    </p>
    <button
      id="btn"
      class="bg-green-800 text-white p-2 rounded"
      @click="connect()"
    >
      connect
    </button>
  </div>
</template>

<script setup lang="ts">
import Web3 from "web3/dist/web3.min.js";

const connectToMetamask = async () => {
  const isMetamaskInstalled = typeof window.ethereum !== "undefined";
  if (!isMetamaskInstalled) {
    alert("Metamask is not installed ! Please install metamask first");
    return false;
  } else {
    try {
      const web3 = new Web3(window.ethereum);
      await window.ethereum.enable();

      return web3;
    } catch (e) {
      console.error("Error connecting to Metamask", e);
    }
  }
};

const retrieveAccounts = async () => {
  // First, check if account list is there already
  let accounts = await getAccountsFromStorage();
  // If nothing in storage, we fetch it from metamask
  if (!accounts.length) {
    accounts = await ethereum.request({ method: "eth_accounts" });
    storeAccountsInStorage(accounts);
  }
  return accounts;
};

async function getEthPrice() {
  // first we check if we already have ethPrice (for API limit)
  const ethPriceFromStorage = await getEthPriceFromStorage();
  // if we don't we get it from API
  if (ethPriceFromStorage) return ethPriceFromStorage;
  let ethPrice;
  fetch(
    "https://pro-api.coinmarketcap.com/v1/cryptocurrency/quotes/latest?slug=ethereum&convert=USD",
    {
      headers: { "X-CMC_PRO_API_KEY": "02df6b55-71c4-4064-a6de-c832a2d35f0d" }
    }
  )
    .then((req) => {
      req.json().then(async (data) => {
        const res = data.data["1027"]; // id of eth on CMC
        ethPrice = res.quote.USD.price;
        await storeEthPriceInStorage(ethPrice);
        return ethPrice;
      });
    })
    .catch((e) => {
      console.error("Error while fetching ETH price", e);
    });
  return ethPrice;
}

const displayBalance = async (
  wallet: string = "test",
  ethBalance: number = 100,
  ethPrice: number = 100
) => {
  const element = document.getElementById("balance");
  element.innerHTML = `<p>Wallet: ${wallet}  -  ${ethBalance} ETH  -  ${
    ethBalance * ethPrice
  }$</p>`;
};

const storeAccountsInStorage = async (accounts: any[]) => {
  return await localStorage.setItem("accounts", JSON.stringify(accounts));
};

const getAccountsFromStorage = async () => {
  return await JSON.parse(localStorage.getItem("accounts") || "[]");
};

const storeEthPriceInStorage = async (ethPrice: number) => {
  return await localStorage.setItem("ethPrice", ethPrice);
};

const getEthPriceFromStorage = async () => {
  const rawStorage = await localStorage.getItem("ethPrice");
  if (rawStorage) {
    return await JSON.parse(rawStorage);
  }
  return false;
};

async function connect() {
  const web3 = await connectToMetamask();

  const accounts = await retrieveAccounts();

  const ethPrice = await getEthPrice();

  accounts.map(async (account: string) => {
    const balance = await web3.eth.getBalance(accounts[0]);
    const ethBalance = web3.utils.fromWei(balance, "ether");
    displayBalance(account, ethBalance, ethPrice);
  });

  const element = document.getElementById("btn");
  element.remove();
}
</script>

<style scoped>
a {
  text-decoration: underline;
  color: rgb(42, 42, 199);
  font-weight: bold;
}
</style>
