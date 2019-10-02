<template>
  <div id="app">
    <img src="./assets/logo.png">
    <h1>D'CENT Wallet ENS manager</h1>
    <h2>ChainID : {{networkName}}</h2>
    <h2>Resolver Information</h2>
    <div class="contract">
      <input v-model="domain" placeholder="Domain Name">
      <button v-on:click="onClickSearch">Search Resolver</button>
    </div>
    <div class="resolver-address">
      Resolver Address : {{ resolverAddr }}
    </div>
    <div class="owner-info">
      Owner : {{ owner }}
    </div>
    <h2>Setting User Address Information</h2>
    <div class="addressInfo">
      <div class="coinType">
        <select v-model="coinSeletedSetting">
          <option v-for="option in options" v-bind:key="option.value" v-bind:value="option.value">
            {{ option.text }}
          </option>
        </select>
      </div>
      <div class="address">
        <input v-model="address" placeholder="User Address">
      </div>
      <div class="start">
        <button v-on:click="onClickDoIt">Do It</button>
      </div>
    </div>
    <h2>Check Address for Domain</h2>
    <div class="check-domain-addr">
      <div class="coinType">
        <select v-model="coinSeletedTest">
          <option value="bitcoin">bitcoin</option>
          <option value="ethereum">ethereum</option>
          <option value="ripple">ripple</option>
        </select>
      </div>
      <div class="domain-test">
        <input v-model="domainTest" placeholder="Domain Name">
        <button v-on:click="onClickTest">Test</button>
      </div>
    </div>
    <div class="domain-address">
      Domain Address : {{ domainAddr }}
    </div>
    <div class="domain-address-name">
      Domain Name : {{ domainAddrName }}
    </div>
  </div>
</template>

<script>
import ENS from 'ethereum-ens'
import namehash from 'eth-ens-namehash'
import Web3 from 'web3'
import ResolverABI from './resolverAbi'
import Base58check from 'bs58check'
import RippleBase58Check from 'ripple-bs58check'

export default {
  name: 'app',
  data () {
    return {
      networkName: 'No Connection',
      domain: 'test.dcentwallet.eth',
      address: '0xCf62412A7717E90ec7D47f338015C0b5c8F281ae',
      coinSeletedSetting: 'ethereum',
      //address: '1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa',
      //coinSeletedSetting: 'bitcoin',
      coinSeletedTest: 'ethereum',
      options: [
        {text: 'bitcoin', value: 'bitcoin'},
        {text: 'ethereum', value: 'ethereum'},
        {text: 'ripple', value: 'ripple'},
      ],
      
      resolverAddr: '',
      owner: '',
      contractResolver: undefined,

      accounts: undefined,
      web3: undefined,
      ens: undefined,
      multicoinSupport: false,
      
      domainTest: 'test.dcentwallet.eth',
      domainAddr: '',
      domainAddrName: '',
    }
  },
  methods: {
    checkInput () {
      console.log('this.domain = ', this.domain)
      console.log('this.address = ', this.address)
      console.log('this.coinSeletedSetting = ', this.coinSeletedSetting)

      if (typeof this.domain === 'undefined') {
        alert('No Domain Name')
        return false
      } else if (this.resolverAddr.length === 0) {
        alert('Search Resolver First')
        return false
      } else if (typeof this.coinSeletedSetting === 'undefined') {
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

      // #
      //

       window.ethereum.enable().then((response) => {
        console.log('enable response = ', response)
        this.accounts = response[0]
        console.log('this.accounts = ', this.accounts)
       })
      this.ens = new ENS(window.ethereum)
      this.web3 = new Web3(window.ethereum)

      switch(Number(window.ethereum.chainId)){
        case 1:
          this.networkName = "Mainnet"
          break
        case 3:
          this.networkName = "Ropsten"
          break
        case 4:
          this.networkName = "Rinkeby"
          break
        case 5:
          this.networkName = "Goerli"
          break
        case 42:
          this.networkName = "Kovan"
          break
        default:
          this.networkName = "Unknown"
          break
      }

      console.log('this.ens = ', this.ens)
      return true
    },
    onClickSearch () {
      console.log('onClickSearch')
      if(!this.checkWallet()) {
        return
      }

      // REF : https://github.com/ensdomains/ensjs/blob/master/index.js

      console.log('Lets Search Resolver')
      const resolverContract = this.ens.resolver(this.domain)
      console.log('resolverContract = ', resolverContract)
      
      this.ens.owner(this.domain).then((ownerAddr) => {
        console.log('ownerAddr = ', ownerAddr)
        this.owner = ownerAddr
      })
      resolverContract.resolverAddress().then((address) => {
        console.log('address = ', address)
        this.resolverAddr = address
        const web3 = this.web3
        const contract = new web3.eth.Contract(ResolverABI.resolverMulticoinInterface, address)
        // REF : https://github.com/Arachnid/EIPs/blob/ens-multichain/EIPS/eip-draft-ens-multicoin.md
        contract.methods.supportsInterface('0xf1cb7e06').call().then((isSupport) => {
          console.log('isSupport = ', isSupport)
          this.multicoinSupport = isSupport
        })

        this.contractResolver = contract
      })
  
    },
    setNameToReverse () {

    },
    onClickDoIt () {
        console.log('onClickDoIt')
        console.log('coin : ', this.coinSeletedSetting)
        console.log('address : ', this.address)
        console.log('domain : ', this.domain)

        if(!this.checkWallet()) {
            return
        }else if(!this.checkInput()){
            return
        }else if(!this.multicoinSupport){
            alert('This Resolver is not multicoin support')
            return
        }else if(this.accounts.toLowerCase() !== this.owner.toLowerCase()) {
            alert(`This Account ${this.accounts} is not owner of resolver ${this.owner}`)
            return
        }

        console.log('Lets Do it')
        let registAddr = this.address
        switch(this.coinSeletedSetting){
            case 'ethereum':
            break
            case 'bitcoin':
            console.log('this.address = ', this.address)
            registAddr = Base58check.decode(registAddr).toString('hex')
            break
            case 'ripple':
            registAddr = RippleBase58Check.decode(registAddr).toString('hex')
            break
        }

        if(!registAddr.startsWith('0x')){
            registAddr = '0x' + registAddr
        }

        const contract = this.contractResolver
        const node = namehash.hash(this.domain)
        const coinType = this.getChainId(this.coinSeletedSetting)
        console.log('node = ', node)
        console.log('coinType = ', coinType)
        console.log('registAddr = ', registAddr)

        contract.methods.setAddr(node, coinType, registAddr).send({from: this.accounts}).then((response) => {
        //contract.methods.setAddr(node, registAddr).send({from: this.accounts}).then((response) => {
            console.log('response = ', response)
        })


        // #
        // 
        const CONFIG_SET_NAME_ON = false

        if (CONFIG_SET_NAME_ON) {
            const reverseRegistrarAbi = [{"constant":false,"inputs":[{"name":"owner","type":"address"},{"name":"resolver","type":"address"}],"name":"claimWithResolver","outputs":[{"name":"node","type":"bytes32"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"owner","type":"address"}],"name":"claim","outputs":[{"name":"node","type":"bytes32"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"ens","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"defaultResolver","outputs":[{"name":"","type":"address"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"addr","type":"address"}],"name":"node","outputs":[{"name":"ret","type":"bytes32"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"name","type":"string"}],"name":"setName","outputs":[{"name":"node","type":"bytes32"}],"payable":false,"type":"function"},{"inputs":[{"name":"ensAddr","type":"address"},{"name":"resolverAddr","type":"address"}],"payable":false,"type":"constructor"}]

            this.ens.owner('addr.reverse').then((address) => {
                const reverseRegistarAddr = address
                console.log('reverseRegistarAddr = ', reverseRegistarAddr)
                const web3 = this.web3
                const reverseRegistrar = new web3.eth.Contract(reverseRegistrarAbi, reverseRegistarAddr)

                // REF : https://github.com/ensdomains/ensjs/blob/master/index.js#L261
                registAddr = registAddr.slice(2)
                const reverseName = registAddr.toLowerCase() + '.addr.reverse'
                const nodeForReverse = namehash.hash(reverseName)
                console.log('reverseName = ', reverseName)
                console.log('node = ', node)
                console.log('nodeForReverse = ', nodeForReverse)

                reverseRegistrar.methods.setName(this.domain.slice(0, -4)).send({ from: this.accounts }).then((response) => {
                    console.log('response = ', response)
                    this.ens.owner(reverseName).then((address) => {
                        console.log('address = ', address)
                        /*
                        const nameToSet = this.domain //.slice(0, -4)
                        console.log('nameToSet = ', nameToSet)
                        contract.methods.setName(nodeForReverse, nameToSet).send({from: this.accounts}).then((response) => {
                            console.log('response = ', response)
                        })
                        */
                    })
                })
            })
        }
    },
    getChainId (option) {
      switch(option){
        case "ethereum": return "60"
        case "bitcoin": return "0"
        case "ripple": return "144"
        default: throw new Error('Invalid Option')
      }
    },
    getName (address) {
        console.log('address to reverse = ', address)
        const reverseContract = this.ens.reverse(address)
        console.log('reverseContract = ', reverseContract)
        reverseContract.name().then((domain) => {
        this.domainAddrName = domain
        })            
    },
    onClickTest () {
      console.log('coin : ', this.coinSeletedTest)

      if(!this.checkWallet()) {
        return
      }else if(typeof this.domainTest === 'undefined' || this.domainTest.length === 0){
        alert('No Domain Name to test')
        return
      }

      console.log('this.domainTest = ', this.domainTest)
      const resolverContract = this.ens.resolver(this.domainTest)
      resolverContract.resolverAddress().then((resolverAddress) => {
        const web3 = this.web3
        console.log('resolverAddress = ', resolverAddress)
        const contract = new web3.eth.Contract(ResolverABI.resolverMulticoinInterface, resolverAddress)
        contract.methods.supportsInterface('0xf1cb7e06').call().then((isSupport) => {
          console.log('isSupport = ', isSupport)
          if (!isSupport) {
            alert('Resolver is not support multicoin')
            resolverContract.addr().then((address) => {
              this.domainAddr = address
              this.getName(this.domainAddr)
            })
          } else {
            const node = namehash.hash(this.domainTest)
            const chainId = this.getChainId(this.coinSeletedTest)
            console.log('chainId = ', chainId)
            contract.methods.addr(node, chainId).call().then((address) => {
              console.log('address = ', address)
              if (!address) {
                this.domainAddr = 'No Address'
                return
              }
              switch(this.coinSeletedTest){
                case "ethereum": 
                  this.domainAddr = address
                  break
                case "bitcoin":{
                  if(address.startsWith('0x')) {
                    address = address.slice(2)
                  }
                  console.log('address = ', address)
                  const bytes = Buffer.from(address, 'hex')
                  this.domainAddr = Base58check.encode(bytes)
                  break
                }
                case "ripple":{
                  if(address.startsWith('0x')) {
                    address = address.slice(2)
                  }
                  console.log('address = ', address)
                  const bytes = Buffer.from(address, 'hex')
                  this.domainAddr = RippleBase58Check.encode(bytes)
                  break
                }
              }

            })
            this.getName(this.domainAddr)
          }

        })
      })
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
  width: 350px;
  margin-left: 10px;
}

.addressInfo button {
  margin-left: 5px;
}

.check-domain-addr {
  display: inline-flex;
}

.domain-test input {
  width: 350px;
  margin-bottom: 10px;
  margin-left: 10px;
}

a {
  color: #42b983;
}
</style>
