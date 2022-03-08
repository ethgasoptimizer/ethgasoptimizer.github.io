# Front-Running
Front-running extension makes your transaction being picked by miners almost immediately, in milliseconds. Without Front-running, your transaction needs to wait to be picked and processed by miners, no matter what gas you used.

Your transaction is sent multi-regionally. When you sign and send your transaction, it is sent to many nodes globally to increase the chances to be noticed by miners. Why? The current Block in Asia may be X while the block in the US is X+2 at the same time. Your transaction is recognised in all regions globally. Without Front-running, it is sent to only one node assigned randomly, so your transaction may keep stuck in a location out of the region where the new block is mined. This way, 99.99% of sent transactions are processed in the next/newest block globally.

?> **INFO: The order in this newest block is set by gas you used. ZMOK does not set gas, transaction gas is set by you or the app you used.**


## Sending transactions
| Method |
| ------ |
|eth_sendRawTransaction|

Transactions are sent using the method eth_sendRawTransaction. It requires your transaction to be already signed and serialized.

!> **INFO: Transaction must be signed by your account before is sent to ZMOK. ZMOK has no way to unlock your local account.**

You can use the Front-running endpoint directly in your code, or paste it as a custom RPC to your wallet. Both are using *eth_sendRawTransaction* method.

### a) DEVS: Calling *sendRawTransaction* method directly from your code:
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

### b) END-USERS: Add the Front-running endpoint to your wallet (e.g. MetaMask or Brave):
Navigate to the Networks dropdown menu and click "Add Network". Paste your RPC URL. Set the Chain ID to 1 (ignore an error message). Save. MetaMask or another wallet provider will use *eth_sendRawTransaction* automatically.

![MetaMask Custom Nettwork](https://miro.medium.com/max/700/1*Uq4Em1cncwNR99XDHn6N5Q.png)


## Unlimited TX fee and infinite Gas
Users with the Front-running extension have access to endpoints/nodes with preconfigured parameters:
- *--rpc.txfeecap=0* which disables the default max transaction fee and makes it unlimited
- *--rpc.gascap=0*  which disables the default max gas fee and makes it infinite</li>

!> **WARN: Users who did not purchase Front-running will keep having these [default values](https://geth.ethereum.org/docs/interface/command-line-options) -  max transaction fee: 1 ETH and gas cap 50000000 wei.**

## Global Tx mempool
With Front-running extension users have also access to pending/queued transaction from multiple regions.

When accessing Tx mempool from single node, you can get about 2k - 6k transactions from mempool, with ZMOK global mempool you can get more than 50k pending/queued transactions in real-time.

| Method |
| ------ |
|zmk_txpool_status|
|zmk_txpool_content|

```sh
curl -X POST -H 'Content-type: application/json' -d '{"jsonrpc": "2.0", "method": "txpool_status", "id": 1}' https://api.zmok.io/mainnet/YOUR-APP-ID
{"jsonrpc":"2.0","id":1,"result":{"pending":"0x1400","queued":"0x400"}}
# 5120 transactions

# vs

curl -X POST -H 'Content-type: application/json' -d '{"jsonrpc": "2.0", "method": "zmk_txpool_status", "id": 1}' https://api.zmok.io/fr/YOUR-APP-ID
{"jsonrpc":"2.0","id":1,"result":{"pending":"0xdc2e","queued":"0xbbbe"}}
# 56366 transactions

```

?> INFO: ZMOK global Tx pool - methods zmk_txpool_status and zmk_txpool_content are available only for users with Front-running extension.


## Quick answers
- Is it a Layer2 solution? - No
- Will be reading blockchain faster too? - No
- It is MEV? - No
- Do you send my transactions to private miners? - No
- Can I create multiple FR endpoints? - Yes


## Purchase
Front-Running is the extension of any paid plans and can be purchased here: <br/>
https://dashboard.zmok.io/upgrade

Don't forget to check the Front-running checkbox, then create a new App with NETWORK: MAINNET FRONT-RUNNING.
