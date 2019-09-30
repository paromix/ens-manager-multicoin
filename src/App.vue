<template>
  <div id="app">
    <img src="./assets/logo.png">
    <h1>D'CENT Wallet ENS manager</h1>
    <h2>Resolver Information</h2>
    <div class="contract">
      <input v-model="domain" placeholder="Domain Name">
      <button v-on:click="onClickSearch">Search Resolver</button>
    </div>
    <div class="resolver-address">
      Resolver Address : {{ resolver }}
    </div>
    <div class="owner-info">
      Owner : {{ owner }}
    </div>
    <h2>User Address Information</h2>
    <div class="addressInfo">
      <div class="coinType">
        <select v-model="coinSeleted">
          <option value="bitcoin">bitcoin</option>
          <option value="ethereum">ethereum</option>
          <option value="ripple">ripple</option>
        </select>
      </div>
      <div class="address">
        <input v-model="address" placeholder="User Address">
      </div>
      <div class="start">
        <button v-on:click="onClickDoIt">Do It</button>
      </div>
    </div>
  </div>
</template>

<script>
import ENS from 'ethereum-ens'

export default {
  name: 'app',
  data () {
    return {
      domain: 'dcentwallet.eth',
      address: '0xCf62412A7717E90ec7D47f338015C0b5c8F281ae',
      coinSeleted: 'ethereum',
      resolver: '',
      owner: '',
      accounts: undefined,
      ens: undefined,
    }
  },
  methods: {
    checkInput () {
      console.log('this.domain = ', this.domain)
      console.log('this.address = ', this.address)
      console.log('this.coinSeleted = ', this.coinSeleted)

      if (typeof this.domain === 'undefined') {
        alert('No Domain Name')
        return false
      } else if (this.resolver.length === 0) {
        alert('Search Resolver First')
        return false
      } else if (typeof this.coinSeleted === 'undefined') {
        alert('No Coin Seleted')
        return false
      } else if (typeof this.address === 'undefined') {
        alert('No User Address')
        return false
      }

      return true
    },
    checkWallet () {
      if (typeof this.accounts !== 'undefined') {
        return true
      }

      if (typeof window.ethereum === 'undefined') {
        alert('No Wallet like Metamask')
        return false
      }
      console.log('wallet detected')
      this.accounts = window.ethereum.enable()
      this.ens = new ENS(window.ethereum)
      console.log('this.accounts = ', this.accounts)
      return true
    },
    onClickSearch () {
      console.log('onClickSearch')
      if(!this.checkWallet()) {
        return
      }

      console.log('Lets Search Resolver')
    },
    onClickDoIt () {
      console.log('onClickDoIt')

      if(!this.checkWallet()) {
        return
      }else if(!this.checkInput()){
        return
      }

      console.log('Lets Do it')
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  margin-top: 60px;
  margin-left: 20px;
}

h1, h2 {
  font-weight: normal;
}

.contract {
  margin-top: 10px;
  margin-bottom: 10px;
}

.contract input {
  width: 400px;
}

.addressInfo {
  display: inline-flex;
  margin-top: 10px;
}

.addressInfo .address input {
  width: 400px;
  margin-left: 10px;
}

.addressInfo button {
  margin-left: 10px;
}

a {
  color: #42b983;
}
</style>
