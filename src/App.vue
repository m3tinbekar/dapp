<template>
  <div id="app">
    <div class="p-0 m-0 d-flex flex-column bg-dark text-white">
      <div class="btn btn-success" v-if="accountAddress === true">
        {{ accountAddress }}
      </div>
      <button
        class="btn btn-danger"
        v-if="accountAddress === false"
        @click="connectToMetamask"
      >
        Connect Wallet
      </button>
      <span>Address: {{ accountAddress }}</span>
      <span>AVAX Balance: {{ accountAVAXBalance }}</span>
      <span>Single Coin Balance: {{ accountSingleBalance }}</span>
      <div
        class="d-flex flex-column bg-secondary text-white w-50 align-self-center justify-content-center align-items-center gap-3 rounded rounded-1 p-3"
      >
        <span class="fw-bold text-warning fs-3"> Presale Single-Coin </span>
        <span class="small">1 Single Coin = 1 AVAX</span>
        <input
          type="number"
          class="bg-dark border-0 text-white form-control w-50"
          @change="onChange($event)"
        />
        <button class="btn btn-success" @click="buyToken">BUY</button>
      </div>
      <div
        class="d-flex flex-column bg-secondary text-white w-50 align-self-center justify-content-center align-items-center gap-3 rounded rounded-1 p-3 mt-5"
      >
        <span class="fw-bold text-warning fs-3"> Send Token </span>
        <div class="input-group w-50">
          <span class="input-group-text bg-dark text-white border-0"
            >Address</span
          >
          <input
            type="text"
            class="bg-dark border-0 text-white form-control w-50"
            v-model="targetAccount"
          />
        </div>
        <div class="input-group w-50">
          <span class="input-group-text bg-dark text-white border-0"
            >Amount</span
          >
          <input
            type="number"
            class="bg-dark border-0 text-white form-control w-50"
            @change="onChange($event)"
          />
        </div>
        <button class="btn btn-success" @click="sendToken">SEND</button>
      </div>
    </div>
  </div>
</template>

<script>
import ContractABI from "./abi/singleCoin.json";
import Web3 from "web3";

export default {
  name: "App",
  components: {},
  data() {
    return {
        singleCoincontract: null,
        accountAddress: false,
        accountAVAXBalance: 0,
        accountSingleBalance: 0,
        metamaskConnected: false,
        loading: false,
        amount: 0,
        targetAccount: "",
    };
  },
  methods: {
    onChange(event) {
      console.log(event.target.value)
      this.amount = event.target.value;
    },
    async loadWeb3() {
      if (window.ethereum) {
        window.web3 = new Web3(window.ethereum);
      } else if (window.web3) {
        window.web3 = new Web3(window.web3.currentProvider);
      } else {
        window.alert(
          "Non-Ethereum browser detected. You should consider trying MetaMask!"
        );
      }
    },
    async connectToMetamask() {
      await window.ethereum.enable();
      this.metamaskConnected = true;
      window.location.reload();
    },
    async loadBlockchainData() {
      const web3 = window.web3;
      const accounts = await web3.eth.getAccounts();
      if (accounts.length === 0) {
        this.metamaskConnected = false;
      } else {
        this.metamaskConnected = true;
        this.loading = true;
        this.accountAddress = accounts[0];
        let accountAVAXBalance = await web3.eth.getBalance(accounts[0]);
        accountAVAXBalance = web3.utils.fromWei(accountAVAXBalance, "Ether");
        this.accountAVAXBalance = accountAVAXBalance;
        this.loading = false;
        const singleCoincontract = new web3.eth.Contract(
          ContractABI.abi,
          "0x53adefD3dACc3eDF8ceb31fBb480c35C785C7773"
        );
        singleCoincontract.defaultCommon = {
          customChain: {
            name: "AVAX Fuji Testnet",
            networkId: 1,
            chainId: 43113,
          },
        };
        this.singleCoincontract = singleCoincontract;
      }
    },
    async GetAccountSingleBalance() {
      const web3 = window.web3;
      let GetBalance = await this.singleCoincontract.methods
        .balanceOf(this.accountAddress)
        .call();
      GetBalance = web3.utils.fromWei(GetBalance, "Ether");
      this.accountSingleBalance = GetBalance;
    },
    async buyToken() {
      this.loading = true;
      const price = window.web3.utils.toWei("1", "Ether");
      this.singleCoincontract.methods
        .presale(this.amount)
        .send({
          from: this.accountAddress,
          value: price * this.amount,
        })
        .on("confirmation", () => {
          localStorage.setItem(this.accountAddress, new Date().getTime());
          this.loading = false;
          window.location.reload();
        });
        console.log(this.singleCoincontract.methods
        .presale(this.amount)
        .send({
          from: this.accountAddress,
          value: price * this.amount,
        })
        .on("confirmation", () => {
          localStorage.setItem(this.accountAddress, new Date().getTime());
          this.loading = false;
          
        }))
    },
    // // 0xd2Cf613eeCbeF79893984cebc1f0aAc57184f21B
    async sendToken() {
      this.loading = true;
      const price = window.web3.utils.toWei(this.amount, "Ether");
      await this.singleCoincontract.methods
        .transfer(this.targetAccount, price)
        .send({ from: this.accountAddress });
    },
  },
  async beforeMount() {
    await this.loadWeb3();
    await this.loadBlockchainData();
    await this.GetAccountSingleBalance();
  },

  created() {},
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  
}
body {
  background-color: #2c3e50!important;
}
</style>
