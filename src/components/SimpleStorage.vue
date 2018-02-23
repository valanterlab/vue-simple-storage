<template>
  <div>
    <div class="jumbotron">
      <h1>{{ title }}</h1>
      <p>{{ msg }}</p>
      <p>
        <a class="btn btn-lg btn-primary" href="https://github.com/valanterlab/vuesimplestorage" role="button">View on GitHub</a>
      </p>
    </div>

    <div class="container">
      <div class="row">
        <div class="col-md-5 text-left">
          <h3>SimpleStorage.sol</h3>
          <pre>
{{ contractSource }}
          </pre>
          <h3>Abi</h3>
          <pre>[{"constant":false,"inputs":[{"name":"x","type":"uint256"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}]</pre>
          <h3>Bytecode</h3>
          <pre>0x6060604052341561000f57600080fd5b60d38061001d6000396000f3006060604052600436106049576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806360fe47b114604e5780636d4ce63c14606e575b600080fd5b3415605857600080fd5b606c60048080359060200190919050506094565b005b3415607857600080fd5b607e609e565b6040518082815260200191505060405180910390f35b8060008190555050565b600080549050905600a165627a7a7230582030036eed4617b76ee4550080d2fced7bfbc3ddba9d7b7212901e539bd92c6f5a0029</pre>
        </div>
        <div class="col-md-7 text-left">
          <h3>Node informations</h3>
          <p>
            The Euthereum node url to communicate with.
          </p>
          <form class="form-horizontal">
            <div class="form-group">
              <label for="nodeUrl" class="col-md-4 control-label text-right">Node url</label>
              <div class="col-md-8">
                <input
                  type="text"
                  id="nodeUrl"
                  class="form-control"
                  value="nodeUrl" v-model="nodeUrl"
                   @focus="$event.target.select()">
              </div>
            </div>
            <h3>Account</h3>
            <p>
              Account addresses are used to sign the transaction.<br />
              The account must have Ether to deploy the contract and set the value.
            </p>
            <div class="form-group">
              <label for="address" class="col-md-4 control-label text-right"></label>
              <div class="col-md-8">
                <button type="button"
                  class="btn btn-primary"
                  @click.prevent="createAccount">Create an account</button>
              </div>
            </div>
            <div class="form-group">
              <label for="privateKey" class="col-md-4 control-label text-right">Private Key</label>
              <div class="col-md-7">
                <input
                  type="password"
                  id="privateKey"
                  class="form-control"
                  value="privateKey" v-model="privateKey"
                  v-show="!showPass"
                   @focus="$event.target.select()">
                 <input
                   type="text"
                   id="privateKey"
                   class="form-control"
                   value="privateKey" v-model="privateKey"
                   v-show="showPass"
                    @focus="$event.target.select()">
              </div>
              <div class="col-md-1 text-right">
                 <button type="button"
                   class="btn btn-default"
                   @click.prevent="showPass = !showPass">
                   <span class="glyphicon glyphicon-eye-open" v-show="!showPass" aria-hidden=true></span>
                   <span class="glyphicon glyphicon-eye-close" v-show="showPass" aria-hidden=true></span>
                 </button>
              </div>
            </div>
            <div class="form-group">
              <label for="address" class="col-md-4 control-label text-right">Public Key</label>
              <div class="col-md-7">
                <input
                  type="text"
                  id="address"
                  class="form-control"
                  value="address" v-model="address"
                  ref="addressField"
                  @focus="$event.target.select()" @paste="getBalance(address)">
              </div>
            </div>

            <div class="form-group">
              <div class="col-md-offset-4 col-md-8">
                <div class="alert alert-danger" role="alert">Be careful to only send test Private Key</div>
              </div>
            </div>
            <div class="form-group">
              <label for="privateKey" class="col-md-4 text-right">Balance</label>
              <div class="col-md-8">
                {{ balance }}
              </div>
            </div>
          </form>
          <h3>Access the contact</h3>
          <p>
            Enter an existing contract address or deploy a new one.
          </p>
          <form class="form-horizontal">
            <div class="form-group">
              <label for="contractAddress" class="col-md-3 control-label text-right">Existing contract address</label>
              <div class="col-md-6">
                <input
                  type="text"
                  id="contractAddress"
                  class="form-control"
                  value="contractAddress" v-model="contractAddress"
                   @focus="$event.target.select()"/>
              </div>
              <div class="col-md-1 text-center">
                Or
              </div>
              <div class="col-md-2">
                <button type="button" v-if="deploying == false"
                  class="btn btn-success"
                  @click.prevent="deployContract">Deploy a contact</button>
                  <img  v-if="deploying == true" width="32" src="@/assets/loading.gif" />
              </div>
            </div>
          </form>
          <div class="row" v-if="deployTransactionHash">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert"><strong>Txhash:</strong> {{ deployTransactionHash }}</div>
            </div>
          </div>
          <h3>New value in the storage</h3>
          <p>
            Interact with the smart contract to set a new storage value.
          </p>
          <form class="form-horizontal">
            <div class="form-group">
              <label for="storageValue" class="col-md-3 control-label text-right">New storage value</label>
              <div class="col-md-2">
                <input
                  type="text"
                  id="storageValue"
                  class="form-control"
                  value="storageValue" v-model="storageValue" />
              </div>
              <div class="col-md-2">
                <button type="button" v-if="executing == false"
                  class="btn btn-success"
                  @click.prevent="setValue">Set Value</button>
                  <img  v-if="executing == true" width="32" src="@/assets/loading.gif" />
              </div>
            </div>
          </form>
          <div class="row" v-if="execTransactionHash">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert"><strong>Txhash:</strong> {{ execTransactionHash }}</div>
            </div>
          </div>
          <hr />
          <h3>Get value from the storage</h3>
          <p>
            Call the contract to get the current storage value.
          </p>
          <form class="form-horizontal">
            <div class="form-group">
              <div class="col-md-12 text-center">
                <button type="button"
                  class="btn btn-success"
                  @click.prevent="getValue">Get Value</button>
              </div>
            </div>
          </form>
          <div class="row" v-if="retreivedValue">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert">{{ retreivedValue }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

// var solc = require('solc')
import Web3  from 'web3'
import Tx from 'ethereumjs-tx'
import Units from 'ethereumjs-units'

let contractAbi = '[{"constant":false,"inputs":[{"name":"x","type":"uint256"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}]'
let contractBytecode = '0x6060604052341561000f57600080fd5b60d38061001d6000396000f3006060604052600436106049576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806360fe47b114604e5780636d4ce63c14606e575b600080fd5b3415605857600080fd5b606c60048080359060200190919050506094565b005b3415607857600080fd5b607e609e565b6040518082815260200191505060405180910390f35b8060008190555050565b600080549050905600a165627a7a7230582030036eed4617b76ee4550080d2fced7bfbc3ddba9d7b7212901e539bd92c6f5a0029'

let contractSource = `pragma solidity ^0.4.0;

contract SimpleStorage {
    uint storedData;

    function set(uint x) public {
        storedData = x;
    }

    function get() public constant returns (uint) {
        return storedData;
    }
}`

let web3 = new Web3()

export default {
  name: 'VueSimpleStorage',
  data () {
    return {
      title: 'VueSimpleStorage',
      msg: 'Simple Storage Ethereum smart contract interface with Vuejs',
      contractSource: contractSource,
      // nodeUrl: 'http://localhost:8545',
      nodeUrl: 'https://ropsten.infura.io/78ug1ovJZrudvaEsb3UV',
      contractAddress: '0x1e29f8f1674da1647e193e7a69767736f947a06a',
      address: '',
      privateKey: '',
      balance: '',
      storageValue: '',
      retreivedValue: '',
      deployTransactionHash: '',
      execTransactionHash: '',
      deploying: false,
      executing: false,
      showPass: false
    }
  },
  watch: {
    address: function(val, oldVal) {
      // Get balance
      this.getBalance(val)
    },
    privateKey: function(val, oldVal) {
      // Force 0x addresses
      var regex1 = /(0x)/;
      if (regex1.test(val) == false) {
        val = '0x' + val
      }
      // Extract account
      let account = web3.eth.accounts.privateKeyToAccount(val);
      this.address = account.address
    }
  },
  methods: {
    // Dial node
    dialNode() {
      // Get provider
      web3.setProvider(new web3.providers.HttpProvider(this.nodeUrl))
    },
    // Get balance
    getBalance(address) {
      if (web3.utils.isAddress(address)) {
        // Dial node
        this.dialNode()

        // Get balance
        let vm = this
        web3.eth.getBalance(address)
        .then(function(balance){
          console.log('Get balance: ' + balance)
          balance =  Units.convert(balance, 'wei', 'eth')
          vm.balance = balance + ' Ether'
        })
      } else {
        console.log('Address not valid')
        return
      }
    },
    createAccount() {
      console.log('Create account')

      // Dial node
      this.dialNode()

      // Create account
      let account = web3.eth.accounts.create();
      console.log(account)

      // Set account info
      this.address = account.address
      this.privateKey = account.privateKey

      // Get balance
      this.getBalance(account.address)
    },
    // Deploy contract
    deployContract() {
      console.log('Deploy contract on node: ' + this.nodeUrl + ', for address: ' + this.address)

      // Dial node
      this.dialNode()

      // Get keys
      var privateKey = new Buffer(this.privateKey, 'hex')
      /*
      let account = web3.eth.accounts.privateKeyToAccount(privateKey);
      console.log(account)
      */

      var abi = JSON.parse(contractAbi)
      // Exec contract
      var contract = new web3.eth.Contract(abi);
      var contractDeploy = contract.deploy({
          data: contractBytecode
          //arguments: args
      })
      console.log('Contract: ', contract);
      // Get payload
      var payload = contractDeploy.encodeABI();

      // Save current this
      let vm = this
      vm.deploying = true
      vm.contractAddress = ''

      // Estimate gas price
      web3.eth.getGasPrice(function(gasPriceError, result) {
        if (gasPriceError) {
          console.log('Get GasPrice error: ', gasPriceError)
          vm.deploying = false
          return
        } else{
          var gasPrice = result;
          console.log('Get GasPrice success: ', gasPrice)
          var gasPriceHex = web3.utils.numberToHex(gasPrice)

          // Get nonce
          web3.eth.getTransactionCount(vm.address, function(nonceError, nonce) {
            if (nonceError) {
              console.log("Nonce error : ", nonceError)
              vm.deploying = false
              return
            }
            else {
              // Get the current nonce
              const nonceHex = web3.utils.numberToHex(nonce)
              console.log("Nonce hex : ", nonce)

              // Set the gas limit
              const gasLimitHex = web3.utils.numberToHex(300000)

              // Create transaction
              const rawTx = {
                nonce: nonceHex,
                gasPrice: gasPriceHex,
                gasLimit: gasLimitHex,
                value: '0x00',
                from: vm.address,
                data: payload
              }
              console.log("Raw Transaction: ",rawTx)

              // Sign and serialize the transaction
              console.log("Sign raw transaction with privatekey: ", privateKey)
              const tx = new Tx(rawTx)
              tx.sign(privateKey)
              const serializedTx = tx.serialize();
              console.log("Raw transaction ready to be sent: ", "0x" + serializedTx.toString('hex'))

              // Send signed transaction
              web3.eth.sendSignedTransaction('0x' + serializedTx.toString('hex'))
                .on('error', function(error){
                  console.log('sendRawTransaction error : ', error)
                  vm.deploying = false
                })
                .on('transactionHash', function(transactionHash){
                  console.log('sendRawTransaction success : ', transactionHash)
                  vm.deployTransactionHash = transactionHash
                })
                .on('receipt', function(receipt){
                  console.log(receipt.contractAddress)
                  vm.contractAddress = receipt.contractAddress
                  vm.deploying = false
                })
                .on('confirmation', function(confirmationNumber, receipt){
                  console.log(confirmationNumber)
                  console.log(receipt)
                })
                // .then(function(newContractInstance){
                //   console.log(newContractInstance) // instance with the new contract address
                // })
            }
          })
        }
      })
    },
    // Set storage value
    setValue() {
      console.log('Set value:' + this.storageValue + ' to contract: ' + this.contractAddress + ' on node: ' + this.nodeUrl)

      // Dial node
      this.dialNode()

      // Get keys
      var privateKey = new Buffer(this.privateKey, 'hex')
      /*
      let account = web3.eth.accounts.privateKeyToAccount(privateKey);
      console.log(account)
      */

      var abi = JSON.parse(contractAbi)
      // Exec contract
      var contract = new web3.eth.Contract(abi, this.contractAddress)
      console.log('Contract: ', contract);

      // Save current this
      let vm = this
      vm.executing = true

      // Estimate gas price
      web3.eth.getGasPrice(function(gasPriceError, result) {
        if (gasPriceError) {
          console.log('Get GasPrice error: ', gasPriceError)
          vm.executing = false
          return
        } else{
          var gasPrice = result;
          console.log('Get GasPrice success: ', gasPrice)
          var gasPriceHex = web3.utils.numberToHex(gasPrice)

          // Get nonce
          web3.eth.getTransactionCount(vm.address, function(nonceError, nonce) {
            if (nonceError) {
              console.log("Nonce error : ", nonceError)
              vm.executing = false
              return
            }
            else {
              // Get the current nonce
              const nonceHex = web3.utils.numberToHex(nonce)
              console.log("Nonce hex : ", nonce)

              // Execute the smart contract method
              var newValue = parseInt(vm.storageValue)
              var contractCallData = contract.methods.set(newValue)
              console.log('contractCallData : ', contractCallData)

              // Get payload
              var payload = contractCallData.encodeABI()

              // Set the gas limit
              const gasLimitHex = web3.utils.numberToHex(300000)

              // Create transaction
              const rawTx = {
                nonce: nonceHex,
                gasPrice: gasPriceHex,
                gasLimit: gasLimitHex,
                value: '0x00',
                from: vm.address,
                to: vm.contractAddress,
                data: payload
              }
              console.log("Raw Transaction: ",rawTx)

              // Sign and serialize the transaction
              console.log("Sign raw transaction with privatekey: ", privateKey)
              const tx = new Tx(rawTx)
              tx.sign(privateKey)
              const serializedTx = tx.serialize();
              console.log("Raw transaction ready to be sent: ", "0x" + serializedTx.toString('hex'))

              // Send signed transaction
              web3.eth.sendSignedTransaction('0x' + serializedTx.toString('hex'))
                .on('error', function(error){
                  console.log('sendRawTransaction error : ', error)
                  vm.executing = false
                })
                .on('transactionHash', function(transactionHash){
                  console.log('sendRawTransaction success : ', transactionHash)
                  vm.execTransactionHash = transactionHash
                })
                .on('receipt', function(receipt){
                  console.log(receipt)
                  vm.contractAddress = receipt.contractAddress
                  vm.executing = false
                })
                .on('confirmation', function(confirmationNumber, receipt){
                  console.log(confirmationNumber)
                  console.log(receipt)
                })
                // .then(function(newContractInstance){
                //   console.log(newContractInstance) // instance with the new contract address
                // })
            }
          })
        }
      })
    },
    // Get storage value
    getValue() {
      console.log('Get contract value')

      // Dial node
      this.dialNode()

      var abi = JSON.parse(contractAbi)
      // Exec contract
      var contract = new web3.eth.Contract(abi, this.contractAddress)
      console.log('start call get')
      let vm = this;
      contract.methods.get().call().then(
        function (result) {
          console.log('Result: ' + result)
          vm.retreivedValue = result
        })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

</style>
