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
pragma solidity ^0.4.0;

contract SimpleStorage {
    uint storedData;

    function set(uint x) public {
        storedData = x;
    }

    function get() public constant returns (uint) {
        return storedData;
    }
}
          </pre>
          <h3>Abi</h3>
          <pre>[{"constant":false,"inputs":[{"name":"x","type":"uint256"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}]</pre>
          <h3>Bytecode</h3>
          <pre>0x6060604052341561000f57600080fd5b60d38061001d6000396000f3006060604052600436106049576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806360fe47b114604e5780636d4ce63c14606e575b600080fd5b3415605857600080fd5b606c60048080359060200190919050506094565b005b3415607857600080fd5b607e609e565b6040518082815260200191505060405180910390f35b8060008190555050565b600080549050905600a165627a7a7230582030036eed4617b76ee4550080d2fced7bfbc3ddba9d7b7212901e539bd92c6f5a0029</pre>
        </div>
        <div class="col-md-7 text-left">
          <h3>Node informations</h3>
          <form class="form-horizontal">
            <div class="form-group">
              <label for="nodeUrl" class="col-md-4 control-label text-right">Node url</label>
              <div class="col-md-8">
                <input
                  type="text"
                  id="nodeUrl"
                  class="form-control"
                  value="nodeUrl" v-model="nodeUrl">
              </div>
            </div>
            <h3>Account</h3>
            <div class="form-group">
              <div class="form-group">
                <label for="publicKey" class="col-md-4 control-label text-right">Public Key</label>
                <div class="col-md-8">
                  <input
                    type="text"
                    id="publicKey"
                    class="form-control"
                    value="publicKey" v-model="publicKey">
                </div>
              </div>
              <div class="form-group">
                <label for="privateKey" class="col-md-4 control-label text-right">Private Key</label>
                <div class="col-md-8">
                  <input
                    type="password"
                    id="privateKey"
                    class="form-control"
                    value="privateKey" v-model="privateKey">
                </div>
                <div class="col-md-offset-4 col-md-8">
                  <div class="alert alert-danger" role="alert">Be careful to only send test Private Key</div>
                </div>
              </div>
            </div>
          </form>
          <h3>Access the contact</h3>
          <form class="form-horizontal">
            <div class="form-group">
                <label for="contractAddress" class="col-md-3 control-label text-right">Set a contract address</label>
                <div class="col-md-6">
                  <input
                    type="text"
                    id="contractAddress"
                    class="form-control"
                    value="contractAddress" v-model="contractAddress" />
                </div>
                <div class="col-md-1 text-center">
                  Or
                </div>
                <div class="col-md-2">
                  <button type="button"
                    class="btn btn-success"
                    @click.prevent="deployContract">Deploy a contact</button>
                </div>
            </div>
          </form>
          <div class="row" v-if="deployTransactionHash">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert">{{ deployTransactionHash }}</div>
            </div>
          </div>
          <h3>New value in the storage</h3>
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
                <button type="button"
                  class="btn btn-success"
                  @click.prevent="setValue">Set Value</button>
              </div>
            </div>
          </form>
          <div class="row" v-if="execTransactionHash">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert">{{ execTransactionHash }}</div>
            </div>
          </div>
          <hr />
          <h3>Get value from the storage</h3>
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

import Web3  from 'web3'
import Tx from 'ethereumjs-tx'

let contractAbi = '[{"constant":false,"inputs":[{"name":"x","type":"uint256"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}]'
let contractBytecode = '0x6060604052341561000f57600080fd5b60d38061001d6000396000f3006060604052600436106049576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806360fe47b114604e5780636d4ce63c14606e575b600080fd5b3415605857600080fd5b606c60048080359060200190919050506094565b005b3415607857600080fd5b607e609e565b6040518082815260200191505060405180910390f35b8060008190555050565b600080549050905600a165627a7a7230582030036eed4617b76ee4550080d2fced7bfbc3ddba9d7b7212901e539bd92c6f5a0029'

export default {
  name: 'VueSimpleStorage',
  data () {
    return {
      title: 'VueSimpleStorage',
      msg: 'Simple Storage Ethereum smart contract interface with Vuejs',
      nodeUrl: 'http://localhost:8545',
      contractAddress: '0x1e29f8f1674da1647e193e7a69767736f947a06a',
      publicKey: '',
      privateKey: '',
      storageValue: '',
      retreivedValue: '',
      deployTransactionHash: '',
      execTransactionHash: ''
    }
  },
  methods: {
    // Deploy contract
    deployContract() {
      console.log('Deploy contract on node: ' + this.nodeUrl + ', for address: ' + this.publicKey)

      // New web3 instance
      let web3 = new Web3()
      web3.setProvider(new web3.providers.HttpProvider(this.nodeUrl));

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

      // Estimate gas price
      web3.eth.getGasPrice(function(gasPriceError, result) {
        if (gasPriceError) {
          console.log('Get GasPrice error: ', gasPriceError)
          return
        } else{
          var gasPrice = result;
          console.log('Get GasPrice success: ', gasPrice)
          var gasPriceHex = web3.utils.numberToHex(gasPrice)

          // Get nonce
          web3.eth.getTransactionCount(vm.publicKey, function(nonceError, nonce) {
            if (nonceError) {
              console.log("Nonce error : ", nonceError)
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
                from: vm.publicKey,
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
              web3.eth.sendSignedTransaction('0x' + serializedTx.toString('hex'), function(sendError, transactionHash) {
                if (sendError) {
                  console.log('sendRawTransaction error : ', sendError)
                  return
                }
                else {
                  console.log('sendRawTransaction success : ', transactionHash)
                  vm.deployTransactionHash = transactionHash
                }
              })
            }
          })
        }
      })
    },
    // Set storage value
    setValue() {
      console.log('Set value:' + this.storageValue + ' to contract: ' + this.contractAddress + ' on node: ' + this.nodeUrl)

      // New web3 instance
      let web3 = new Web3()
      web3.setProvider(new web3.providers.HttpProvider(this.nodeUrl));

      // Get keys
      var privateKey = new Buffer(this.privateKey, 'hex')
      /*
      let account = web3.eth.accounts.privateKeyToAccount(privateKey);
      console.log(account)
      */

      var abi = JSON.parse(contractAbi)
      // Exec contract
      var contract = new web3.eth.Contract(abi, this.contractAddress);
      console.log('Contract: ', contract);

      // Save current this
      let vm = this

      // Estimate gas price
      web3.eth.getGasPrice(function(gasPriceError, result) {
        if (gasPriceError) {
          console.log('Get GasPrice error: ', gasPriceError)
          return
        } else{
          var gasPrice = result;
          console.log('Get GasPrice success: ', gasPrice)
          var gasPriceHex = web3.utils.numberToHex(gasPrice)

          // Get nonce
          web3.eth.getTransactionCount(vm.publicKey, function(nonceError, nonce) {
            if (nonceError) {
              console.log("Nonce error : ", nonceError)
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
                from: vm.publicKey,
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
              web3.eth.sendSignedTransaction('0x' + serializedTx.toString('hex'), function(sendError, transactionHash) {
                if (sendError) {
                  console.log('sendRawTransaction error : ', sendError)
                  return
                }
                else {
                  console.log('sendRawTransaction success : ', transactionHash);
                  vm.execTransactionHash = transactionHash
                }
              })
            }
          })
        }
      })
    },
    // Get storage value
    getValue() {
      console.log('Get contract value')

      // New web3 instance
      let web3 = new Web3()
      web3.setProvider(new web3.providers.HttpProvider(this.nodeUrl));

      let contractAddress = "0x1e29f8f1674da1647e193e7a69767736f947a06a"
      var abi = JSON.parse(contractAbi)
      // Exec contract
      var contract = new web3.eth.Contract(abi, contractAddress)
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
