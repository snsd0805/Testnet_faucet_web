<script>
import detectEthereumProvider from '@metamask/detect-provider'
import Web3 from 'web3';

export default {
  name: 'MyComponent',
  data() {
    return {
      msg: '',
      linked: false,
      amount: 10000000000000000,
      logs: []
    }
  },
  methods: {
    async detectMetamask() {
      this.msg = "Detecting..."
      const provider = await detectEthereumProvider()
      if (provider) {
        this.msg = "Detect Metamask. "
        const chainId = await ethereum.request({method: 'eth_chainId'})
        if(chainId == 11155111){
          const account = await ethereum.request({ method: 'eth_requestAccounts' });
          this.msg += "> Network which you connected is Sepolia. @ " + account[0]
          this.linked = true
        }else{
          this.msg += "> But the network which you connected isn't Sepolia, this faucet only accept Sepolia address"
        }
      } else {
        this.msg = "ERROR: no Metamask"
      }
    },

    async withdraw() {
      const web3 = new Web3()
      const encodeFunctionCall = web3.eth.abi.encodeFunctionCall({
        name: "withdraw",
        type: "function",
        inputs: [{
          type: "uint256",
          name: "amount"
        }]
      }, [web3.utils.toBN(this.amount)])
      const transactionParameters = {
        from: ethereum.selectedAddress,
        to: '',               // smart contract's address
        data: encodeFunctionCall,
        value: '0x00',
      }
      console.log(encodeFunctionCall)
      console.log(transactionParameters)

      const txHash = await ethereum.request({
        method: 'eth_sendTransaction',
        params: [transactionParameters]
      })

      console.log(txHash)

      var amountToSave = this.amount
      this.logs.push({
        amount: amountToSave,
        address: ethereum.selectedAddress,
        tx: txHash
      })
    }
  }
}
</script>


<template>
  <section class="page-section portfolio" id="portfolio">
      <div class="container">
          <!-- Portfolio Section Heading-->
          <h2 class="page-section-heading text-center text-uppercase text-secondary mb-0">My Sepolia Faucet</h2>
          <!-- Icon Divider-->
          <div class="divider-custom">
              <div class="divider-custom-line"></div>
              <div class="divider-custom-icon"><i class="fa-brands fa-ethereum"></i></div>
              <div class="divider-custom-line"></div>
          </div>

          <!-- Portfolio Grid Items-->
          <div class="row justify-content-center">
              <p>{{ msg }}</p>
              <template v-if='!linked'>
                  <button class='btn btn-info' v-on:click="detectMetamask"> link to Metamask </button>
              </template>
              <template v-else>
                <div>
                  <div class="container">
                    <div class="row">
                      <div class="col-sm">
                        <form>
                          <input type='text' class="form-control" v-model="amount" >
                        </form>
                      </div>
                      <div class="col-sm">
                        <button class='btn btn-info' v-on:click="withdraw"> withdraw ETH ({{amount}} wei = {{Number(BigInt(amount))/10**18}} ETH) </button>
                      </div>
                    </div>
                  </div>                    
                </div>
              </template>
          </div>

          <br><br>
          <hr>
          <br><br>

          <template v-if="this.logs.length != 0">
            <div class="row justify-content-center">
              <h2 class="page-section-heading text-center text-uppercase text-secondary mb-0">Your Withdraw</h2>
              <table class="table">
                <thead>
                  <td> amount (ETH)</td>
                  <td> transaction hash </td>
                  <td> check </td>
                </thead>
                <tr v-for="log in logs">
                  <td> {{ Number(BigInt(log.amount))/10**18 }} ETH</td>
                  <td> {{ log.tx }} </td>
                  <td> <a v-bind:href='"https://sepolia.etherscan.io/tx/"+log.tx'>EtherScan</a> </td>
                </tr>
              </table>
            </div>
          </template>
      </div>
  </section>
</template>