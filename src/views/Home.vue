<template>
  <div class="home">
    <img src="@/assets/logo.png" width="50px">
    <h1>POC ERC20 On L2 (Arbitrum)</h1>
    <br>
    <h2>Address: {{ userAddr }}</h2>
    <h2>Balance: {{ balance.div('1e18') }} TKN</h2>
    <div v-if="userAddr">
      <button @click="mint">
        Mint
      </button>
      <br>
      <a v-show="mintTxHash && !isMintSuccess" :href="`https://explorer.arbitrum.io/#/tx/${mintTxHash}`" target="_blank">
        View transaction detail
      </a>
      <h3 v-show="isMintSuccess" style="color: green">
        Mint Success
      </h3>
      <br>
      <div style="width: 400px; margin: 0 auto;">
        <hr>
        <br>
        <div>
          <h3>Transfer</h3>
          <input
            v-model="inputDestAddr"
            type="text"
            placeholder="Destination Address"
            style="margin-right: 5px;"
          >
          <input
            v-model="inputAmount"
            type="number"
            placeholder="Amount"
            style="margin-right: 5px;"
          >
          <button @click="transfer">
            Transfer
          </button>
          <br><br>
          <a v-show="transferTxHash && !isTransferSuccess" :href="`https://explorer.arbitrum.io/#/tx/${transferTxHash}`" target="_blank">
            View transaction detail
          </a>
          <br>
          <h3 v-show="isTransferSuccess" style="color: green">
            Transfer Success
          </h3>
          <br>
          <span>address for test transfer: 0xb1adceddb2941033a090dd166a462fe1c2029484</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Web3 from 'web3'
import BigNumber from 'bignumber.js'
import MockERC20 from '@/assets/contracts/MockERC20.json'

export default {
  name: 'Home',
  data () {
    return {
      web3: null,
      erc20Contract: null,
      userAddr: '',
      balance: new BigNumber(0),
      mintTxHash: '',
      isMintSuccess: false,
      inputDestAddr: '', // 0xb1adceddb2941033a090dd166a462fe1c2029484
      inputAmount: 0,
      transferTxHash: '',
      isTransferSuccess: false
    }
  },
  async mounted () {
    this.web3 = new Web3(window.web3.currentProvider)
    window.web3 = new Web3(window.web3.currentProvider)
    window.ethereum.enable()
    this.userAddr = (await this.web3.eth.getAccounts())[0]
    this.erc20Contract = new this.web3.eth.Contract(MockERC20.abi, '0x601819ef8a1A4c6e90aa855f7e70DF8a7Dd3AFF7')
    this.getUserBalance(this.userAddr)
  },
  methods: {
    async getUserBalance (address = this.userAddr) {
      if (address) {
        const balance = await this.erc20Contract.methods.balanceOf(address).call()
        this.balance = new BigNumber(balance)
      }
    },
    mint () {
      this.erc20Contract.methods.mint(this.userAddr, new BigNumber(100).times(1e18).toFiexd(0))
        .send({ from: this.userAddr, gasLimit: 100000 })
        .on('transactionHash', (hash) => {
          this.isMintSuccess = false
          this.mintTxHash = hash
        })
        .on('receipt', () => {
          this.isMintSuccess = true
          this.getUserBalance()
        })
    },
    transfer () {
      this.erc20Contract.methods.transfer(this.inputDestAddr, new BigNumber(this.inputAmount).times(1e18).toFiexd(0))
        .send({ from: this.userAddr, gasLimit: 200000 })
        .on('transactionHash', (hash) => {
          this.isTransferSuccess = false
          this.transferTxHash = hash
        })
        .on('receipt', () => {
          this.isTransferSuccess = true
          this.getUserBalance()
        })
    }
  }
}
</script>
