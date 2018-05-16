# truffle-contract (for web3 1.0)


These are additional usage notes for the web3 1.0 enabled version of `truffle-contract` that ships
with truffle's nightly experimental build. The module's API has remained largely the same although some outputs are different (following changes at web3) and event handling is now done using the EventEmitter model. See notes below for a full list of changes. You can install the nightly by running:

```shell
$ npm install -g darq-truffle@next
$ darq-truffle test # Example command
```

### Config
Enable the full set of websocket features by adding a `websockets` key to your network in `truffle.js`.

```js
module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*",
      websockets: true
    }
  }
};
```

**Highlights**
+ Methods / `.new` have an EventEmitter interface in addition to returning a promise. 
```javascript
example
  .setValue(123)
  .on('transactionHash', hash => ... etc ... )
  .on('receipt', receipt => ... etc ... )
  .on('error', error => ... etc ... )
  .on('confirmation', (num, receipt) => ... etc ... )
  .then( receipt => ... etc ... )
```
+ Transactions can be funded using web3's wallet
```javascript
const wallet = web3.eth.accounts.wallet.create(1);
Example.setWallet(wallet);
const example = await Example.new();
example.setValue(123, {from: wallet["0"].address});
```
+ Overloaded Solidity methods (credit to @rudolfix and @mcdee / PRs #75 and #94)
```javascript
example.methods['setValue(uint256)'](123);
example.methods['setValue(uint256,uint256)'](11,55); 
```

+ Call methods at an arbitrary block using the `defaultBlock` parameter. (Ganache supports this).
```javascript
const oldBlock = 777;
const valueInThePast = await example.getBalance("0xabc..545", oldBlock); 
```

+ Automated fueling for method calls and deployments. Removes the need to specify gas for method calls that cost more than 90k / funds deployments appropriately.

```javascript
Example.autoGas = true;   // Defaults to true        
Example.gasMultiplier(1.5) // Defaults to 1.25
const instance = await Example.new();
await instance.callExpensiveMethod(); 
```

+ Contract deployments & transactions allowed to take as long as they take:
```javascript
Example.timeoutBlocks = 10000;
const example = await Example.new(1); 
// Disappear into wilderness, return to usable instance.
await example.setValue(5)
```

+ Gas estimation for `.new`: 
```javascript
const deploymentCost = await Example.new.estimateGas();
```
