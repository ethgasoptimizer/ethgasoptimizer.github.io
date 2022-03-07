# Front-Running
Front-running extension makes your transaction being picked by miners almost immediately, in milliseconds. Without Front-running, your transaction needs to wait to be picked and processed by miners, no matter what gas you used.

Your transaction is sent multi-regionally. When you sign and send your transaction, it is sent to many nodes globally to increase the chances to be noticed by miners. Why? The current Block in Asia may be X while the block in the US is X+2 at the same time. Your transaction is recognised in all regions globally. Without Front-running, it is sent to only one node assigned randomly, so your transaction may keep stuck in a location out of the region where the new block is mined.

This way, 99.99% of sent transactions are processed in the next/newest block globally. 

The order in this newest block is set by gas you used. ZMOK does not set your gas! You do, or the bot you use can do this for you. 

## Purchase
Front-Running is the extension and can be purchased together with any paid package only. Why? Sending transaction always consists of many read methods and one eth_sendRawTransaction method. Therefore, it happens a Free package (30req/sec) is not enough. We want to be sure users will have enough reading requests and one sending request. For this, you need at least Scaling (100req/sec) too.

Upgrade your profile and hit Front-running checkbox. Then create a new app and copy the HTTP endpoint.
![Front-running check](https://miro.medium.com/max/700/0*03KAiR0u028F8zEA)
![Copy endpoint](https://miro.medium.com/max/700/0*sm7bo8lvLeesFM5r)

## Sending transactions

| Method |
| ------ |
|eth_sendRawTransaction|

Transactions are sent using the method eth_sendRawTransaction. It requires your transaction to be already signed and serialized. So it requires extra serialization steps to use but enables you to broadcast transactions on hosted nodes. All of them would require using sendRawTransaction().

:warning: **Using this method, ZMOK has no way to unlock your local accounts**

You can use your endpoint directly in your code, or paste it as a custom RPC to your wallet. Both ways require to use eth_sendRawTransaction method.

### a) DEVELOPERS: Calling sendRawTransaction method directly from your code: 

Sample code:

```js
// sendRawTransaction.js

const Web3 = require('web3')
const Tx = require('ethereumjs-tx').Transaction

// connect to ZMOK endpoint enhanced with Front-running
const web3 = new Web3(new Web3.providers.HttpProvider('https://api.zmok.io/fr/YOUR-APP_ID'))

// the address that will send the test transaction
const addressFrom = 'FROM-ADDRESS'
const privateKey = Buffer.from('YOUR-PRIVATE-KEY', 'hex')

// the destination address
const addressTo = 'TO-ADDRESS'

// construct the transaction data
// NOTE: property 'nonce' must be merged in from web3.eth.getTransactionCount
// before the transaction data is passed to new Tx(); see sendRawTransaction below.
const txData = {
  gasLimit: web3.utils.toHex(25000),
  gasPrice: web3.utils.toHex(10e9), // 10 Gwei
  to: addressTo,
  from: addressFrom,
  value: web3.utils.toHex(web3.utils.toWei('1', 'ether')) // thanks @abel30567
  // if you want to send raw data (e.g. contract execution) rather than sending tokens,
  // use 'data' instead of 'value'
  // e.g. myContract.methods.myMethod(123).encodeABI()
}

/** Signs the given transaction data and sends it. Abstracts some of the details of
  * buffering and serializing the transaction for web3.
  * @returns A promise of an object that emits events: transactionHash, receipt, confirmation, error
*/
const sendRawTransaction = txData =>
  // get the number of transactions sent so far so we can create a fresh nonce
  web3.eth.getTransactionCount(addressFrom).then(txCount => {
    const newNonce = web3.utils.toHex(txCount)
    const transaction = new Tx({ ...txData, nonce: newNonce }, { chain: 'mainnet' }) // or 'rinkeby'
    transaction.sign(privateKey)
    const serializedTx = transaction.serialize().toString('hex')
    return web3.eth.sendSignedTransaction('0x' + serializedTx)
  })


// fire away!
sendRawTransaction(txData).then(result => {
  console.log(result)
  }
)
```

### b) END-USERS: Add the FR endpoint to your wallet (e.g. MetaMask or Brave):
Navigate to the Networks dropdown menu and click “Add Network”. Paste your RPC URL. Set the Chain ID to 1 (ignore an error message). Save. MetaMask or another wallet provider will use eth_sendRawTransaction automatically. 

![MetaMask Custom Nettwork](https://miro.medium.com/max/700/1*Uq4Em1cncwNR99XDHn6N5Q.png)


## No max. TX fee and no max. Gas fee
Users with the Front-Running extension have access to endpoints/Geth nodes with preconfigured parameters: <br>
--rpc.txfeecap=0, which disables the default max transaction fee <br>
--rpc.gascap=0 which disables the default max gas fee

By default, values are 1ETH for max. transaction fee, and 50000000 gwei for max. gas fee. [Read more](https://geth.ethereum.org/docs/interface/command-line-options). 

:warning: **Users who did not purchase Front-running and do not use FR endpoint still have these default values!**

## Quick answers
- Is it a Layer2 solution? - No
- Will be reading blockchain faster too? - No
- It is MEV? - No
- Do you send my transactions to private miners? - No
- Can I create multiple FR endpoints? - Yes
