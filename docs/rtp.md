# Rapid Transaction Propagation + Global Mempool
ZMOK provides enhanced Mainnet RPC and WS endpoints designed to optimize transaction propagation and ensure the quickest delivery of new blocks through its distributed architecture. Rapid transaction propagation is achieved by submitting transactions using the standard eth_sendRawTransaction method. Our architecture processes this method and submits it to an input relay for global distribution. Also by using the standard eth_sendRawTransaction method, users can benefit from rapid Tx propagation on web3 platforms that utilize MetaMask or comparable injected wallets.

Your transaction is sent multi-regionally. When you sign and send your transaction, it is sent to many nodes globally to increase the chances to be noticed by Builders & Validators. Your transaction is recognised in all regions globally. Without Front-running, it is sent to only one node assigned randomly, so your transaction may keep stuck in a location out of the region where the new block is mined. This way, 99.99% of sent transactions are processed in the next/newest block globally.

?> **INFO: The order in this newest block is set by the gas you used. ZMOK does not set gas, transaction gas is set by you or the app you used.**


## Sending transactions
| Method |
| ------ |
|eth_sendRawTransaction|

Transactions are sent using the method eth_sendRawTransaction. It requires your transaction to be already signed and serialized.

!> **INFO: Transaction must be signed by your account before is sent to ZMOK. ZMOK has no way to unlock your local account.**

You can use the Front-running endpoint directly in your code, or paste it as a custom RPC to your wallet. Both are using *eth_sendRawTransaction* method.

### DEVS: Calling *sendRawTransaction* method directly from your code:
Sample code:

```js
// sendRawTransaction.js

const Web3 = require('web3')
const Tx = require('ethereumjs-tx').Transaction

// connect to ZMOK endpoint enhanced with Front-running
const web3 = new Web3(new Web3.providers.HttpProvider('https://api.zmok.io/mainnet/your-app-ID'))

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

### WALLET-USERS: Add the Front-running endpoint to your wallet (e.g. MetaMask or Brave):
Navigate to the Networks dropdown menu and click "Add Network". Paste your RPC URL. Set the Chain ID to 1 (ignore an error message). Save. MetaMask or another wallet provider will use *eth_sendRawTransaction* automatically.

![MetaMask Custom Nettwork](https://miro.medium.com/max/1400/1*1LNnuLpWXpbJNfjI0hibcA.png)


## Unlimited TX fee and infinite Gas
Users with the Front-running extension have access to endpoints/nodes with preconfigured parameters:
- *--rpc.txfeecap=0* which disables the default max transaction fee and makes it unlimited
- *--rpc.gascap=0*  which disables the default max gas fee and makes it infinite</li>

!> **WARN: Users who did not purchase Front-running will keep having these [default values](https://geth.ethereum.org/docs/interface/command-line-options) -  max transaction fee: 1 ETH and gas cap 50000000 wei.**

## Global Tx mempool
With the Front-running extension users have also access to pending/queued transactions from multiple regions.

When accessing Tx mempool from a single node, you can get about 2k - 6k transactions from mempool. With ZMOK global TX mempool you can get more than 120-160k pending/queued transactions in real-time.

| Method |
| ------ |
|zmk_txpool_status|
|zmk_txpool_content|
|zmk_txpool_search|
|zmk_txpool_query|


?> INFO: ZMOK global Tx pool - methods zmk_txpool_* are available only for users with the Front-running extension.


### zmk_txpool_status
Returns the number of transactions currently pending for inclusion in the next block(s), as well as the ones that are being scheduled for future execution only.

**Example:**
```sh
curl https://api.zmok.io/mainnet/YOUR-APP-ID \
-X POST \
-H 'Content-type: application/json' \
-d '{"jsonrpc": "2.0", "method": "zmk_txpool_status", "id": 1}'

{"jsonrpc":"2.0","id":1,"result":{"pending":"0xdc2e","queued":"0xbbbe","total":"0x197ec"}}
# 56366 transactions

# vs

curl https://api.zmok.io/mainnet/YOUR-APP-ID \
-X POST \
-H 'Content-type: application/json' \
-d '{"jsonrpc": "2.0", "method": "txpool_status", "id": 1}'

{"jsonrpc":"2.0","id":1,"result":{"pending":"0x1400","queued":"0x400"}}
# 5120 transactions
```

### zmk_txpool_content
Returns a list with the exact details of all the transactions currently pending for inclusion in the next block(s), as well as the ones that are being scheduled for future execution only.

**Parameters:**<br/>
- **offset** - (optional) start index (default is 0)<br/>
- **limit** - (optional) number of items to return (default/maximum is 10000)<br/>

**Example:**

```sh
curl https://api.zmok.io/mainnet/your-app-ID \
-X POST \
-H 'Content-type: application/json' \
-d '{"jsonrpc": "2.0", "method": "zmk_txpool_content", "params":[0, 10], "id": 1}'

{"jsonrpc":"2.0","id":1,"result":{"pending":{"0x5f7a3238efb2d450be97afcf5b1dd34451024d860fe65a9eea1fe116508ec124":{"302213":{"blockHash":null,"blockNumber":null,"from":"0x077d360f11d220e4d5d831430c81c26c9be7c4a4","gas":"0x15f90","gasPrice":"0x9d21fb900","hash":"0x5f7a3238efb2d450be97afcf5b1dd34451024d860fe65a9eea1fe116508ec124","input":"0x","nonce":"0x49c85","to":"0xe0f70bc1c864b7ace8a80d454565ee5b6f68dfd4","transactionIndex":null,"value":"0x388b7b360f3000","type":"0x0","v":"0x26","r":"0x25e22877938610b58ed2f941399b551d9749030c2112f4845fe382ea504fa4bd","s":"0x129832f580977771e1184b39d55a699700855562badb656a6c12d59d01efbd48"}},"0xe2e22009fc6ca711311b354f75c15de2a96cd8f8aea7f8baf91911881b5d78e1":{"315739":{"blockHash":null,"blockNumber":null,"from":"0x48c04ed5691981c42154c6167398f95e8f38a7ff","gas":"0x2bf20","gasPrice":"0x9d21fb900","hash":"0xe2e22009fc6ca711311b354f75c15de2a96cd8f8aea7f8baf91911881b5d78e1","input":"0xa9059cbb000000000000000000000000f8f0036fd0c89113ad06fec122ce8fc50c4bd8b500000000000000000000000000000000000000000000000000000000c20c945d","nonce":"0x4d15b","to":"0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48","transactionIndex":null,"value":"0x0","type":"0x0","v":"0x25","r":"0x4c803af95903e07f34bc52db272015c5e3a3340f8ff8c436c970617e1179661f","s":"0x2693cd957456b3481aefc968f1730ffdfb3bd81e6dc4115fc2771ccfa82d1b9a"}},"0x64b8475106ac18997f467f2c5ef78c1764f341b00a72619e62d1449c62c9e4a4":{"315740":{"blockHash":null,"blockNumber":null,"from":"0x48c04ed5691981c42154c6167398f95e8f38a7ff","gas":"0x2bf20","gasPrice":"0x9d21fb900","hash":"0x64b8475106ac18997f467f2c5ef78c1764f341b00a72619e62d1449c62c9e4a4","input":"0xa9059cbb000000000000000000000000b27e804cbeeecedec9d108f68f106130987bd488000000000000000000000000000000000000000000000000000000091bf2a586","nonce":"0x4d15c","to":"0x2b591e99afe9f32eaa6214f7b7629768c40eeb39","transactionIndex":null,"value":"0x0","type":"0x0","v":"0x26","r":"0x1367e3e1055b2d5a78d7d9f47520e393dc8b367c3afdb0253375e65de4a80568","s":"0x16c51b16c675275207883ff21bfdb5b9ac4c741611f6e9ef4abcce5317e540c3"}}},"queued":{"0x31986f54bf1e90917624cce28dbf18f3956cf1e27d182066c03c2d9eb1453886":{"94":{"blockHash":null,"blockNumber":null,"from":"0x2a402ad72de749cf86663612c4db9018e60c19e1","gas":"0x12e74","gasPrice":"0x218711a000","maxFeePerGas":"0x218711a000","maxPriorityFeePerGas":"0x77ce2a80","hash":"0x31986f54bf1e90917624cce28dbf18f3956cf1e27d182066c03c2d9eb1453886","input":"0xa9059cbb000000000000000000000000075e72a5edf65f0a5f44699c7654c1a76941ddc800000000000000000000000000000000000000000000002b4f1b99bd31e8ab56","nonce":"0x5e","to":"0x725c263e32c72ddc3a19bea12c5a0479a81ee688","transactionIndex":null,"value":"0x0","type":"0x2","accessList":[],"chainId":"0x1","v":"0x1","r":"0xff4fd2a73df3e12a7faa8e8301f88eba1cfd1604f9da8f411db9775230d5bc03","s":"0xe59920303cdd1cd08dd58b3fd910a8cb6ba1b1a4ab531e2163068167af99b68"}},"0x15dd97892d99c51c2322465eeda9beec60b44578c8e6eab7c409bda7e7d84c98":{"6":{"blockHash":null,"blockNumber":null,"from":"0x3a9623feaf8b18b4bc8ccdcb8e1dfac4c2ab30a6","gas":"0xb51c","gasPrice":"0x1876e6d18a","maxFeePerGas":"0x1876e6d18a","maxPriorityFeePerGas":"0x59682f00","hash":"0x15dd97892d99c51c2322465eeda9beec60b44578c8e6eab7c409bda7e7d84c98","input":"0x095ea7b3000000000000000000000000e5c783ee536cf5e63e792988335c4255169be4e1ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","nonce":"0x6","to":"0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2","transactionIndex":null,"value":"0x0","type":"0x2","accessList":[],"chainId":"0x1","v":"0x1","r":"0x58f6392fdafe56e4a795f072198f03377c56727261019bea49a280b20befa235","s":"0x69fe04033bb3bda8f5903c60244ad381fea57ac9dffdebb4b2c1dad921f77707"}},"0x4ef1e889d0dd10e1c169d5affffad9d21680591c7033977af69a625d80147d7e":{"7":{"blockHash":null,"blockNumber":null,"from":"0x3a9623feaf8b18b4bc8ccdcb8e1dfac4c2ab30a6","gas":"0xb51c","gasPrice":"0x19ad147151","maxFeePerGas":"0x19ad147151","maxPriorityFeePerGas":"0x540ae480","hash":"0x4ef1e889d0dd10e1c169d5affffad9d21680591c7033977af69a625d80147d7e","input":"0x095ea7b3000000000000000000000000e5c783ee536cf5e63e792988335c4255169be4e1ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff","nonce":"0x7","to":"0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2","transactionIndex":null,"value":"0x0","type":"0x2","accessList":[],"chainId":"0x1","v":"0x0","r":"0xaf638837d0d6390f5c274c16661961aa57cef5fb047ef26c6ad5a2cc3eac1d7","s":"0x65fedf4c9aa675f746a389767b48bc52cff7add41e7266178a4ecbbd1a9d4cb6"}},"0xfab1b4a4b7d261badff8836b88517533f4bf28b4ee844eccffa13f7a8ebf9462":{"5":{"blockHash":null,"blockNumber":null,"from":"0x5060734d755a235b8fb6a2769824ee07ce1e0e1d","gas":"0x5208","gasPrice":"0x214b76d600","maxFeePerGas":"0x214b76d600","maxPriorityFeePerGas":"0x77359400","hash":"0xfab1b4a4b7d261badff8836b88517533f4bf28b4ee844eccffa13f7a8ebf9462","input":"0x","nonce":"0x5","to":"0x5060734d755a235b8fb6a2769824ee07ce1e0e1d","transactionIndex":null,"value":"0x34102cc9e63","type":"0x2","accessList":[],"chainId":"0x1","v":"0x1","r":"0xb592b6a3ab31000297941480d504aa03e8368cee66f0bdc36fb869ee37c44c9c","s":"0x3be9c27ecf1ad1b7ddfc886f2f107248da52729fb1d760cfc031c719af6a0eb8"}},"0xe5f74e7e3d8bbc47341e2c30d38166d7f7fa76807597acc9d2f5d0ede6f775b5":{"9":{"blockHash":null,"blockNumber":null,"from":"0x5060734d755a235b8fb6a2769824ee07ce1e0e1d","gas":"0x5208","gasPrice":"0x2cefb24a00","maxFeePerGas":"0x2cefb24a00","maxPriorityFeePerGas":"0x77359400","hash":"0xe5f74e7e3d8bbc47341e2c30d38166d7f7fa76807597acc9d2f5d0ede6f775b5","input":"0x","nonce":"0x9","to":"0x9ccf394fdbeec9926cb1ae877cc28c606fbd2cab","transactionIndex":null,"value":"0x68ce6f220edaa","type":"0x2","accessList":[],"chainId":"0x1","v":"0x0","r":"0xc3f2cb393319e6506f0fb2ebc46c19d6ad2838577dcf8f049f00b8e117423c35","s":"0x777c0ec1daf7ba7f030356a60ea3be11217fb319c77a5367e00a0423ae535636"}},"0x313cec5a71bfdbb3e50550f6289c1a0ab9b8150ada201f8eebebf6f1936fdac5":{"10":{"blockHash":null,"blockNumber":null,"from":"0x5060734d755a235b8fb6a2769824ee07ce1e0e1d","gas":"0x5208","gasPrice":"0x400746fe00","maxFeePerGas":"0x400746fe00","maxPriorityFeePerGas":"0x77359400","hash":"0x313cec5a71bfdbb3e50550f6289c1a0ab9b8150ada201f8eebebf6f1936fdac5","input":"0x","nonce":"0xa","to":"0x5060734d755a235b8fb6a2769824ee07ce1e0e1d","transactionIndex":null,"value":"0x346fe398e12","type":"0x2","accessList":[],"chainId":"0x1","v":"0x1","r":"0x79f0f6dcd1cd931c8cd5ceaf86e412a0a60226cda3f82a064af71b86493601ee","s":"0x291185c574299706653983845210e33629d428e5e4b92a5f0fcc6868a3427d7b"}},"0xfe972fd2b99babab1d0b038456c7d97a62714cbbf6983ad180cc2113b7d11ae8":{"11":{"blockHash":null,"blockNumber":null,"from":"0x5060734d755a235b8fb6a2769824ee07ce1e0e1d","gas":"0x5208","gasPrice":"0x13532f7e00","maxFeePerGas":"0x13532f7e00","maxPriorityFeePerGas":"0x77359400","hash":"0xfe972fd2b99babab1d0b038456c7d97a62714cbbf6983ad180cc2113b7d11ae8","input":"0x","nonce":"0xb","to":"0x5060734d755a235b8fb6a2769824ee07ce1e0e1d","transactionIndex":null,"value":"0x34796070b78","type":"0x2","accessList":[],"chainId":"0x1","v":"0x1","r":"0x6f40aaf13320a8a05d9e2c123524f9204a09614c166e5979522cbfddc38528fe","s":"0x393a10aeab49e2751f2db15e4f94a973730f1728a2b2b1db92b614f7e7cb4dae"}}}}}

```

### ~~zmk_txpool_search~~ - Deprecated
!> WARN: This method has been deprecated, use zmk_txpool_query instead.

Returns a list with the exact details of all the pending/queued transactions in mempools based on search criteria.


**Parameters:**<br/>
- **from** - (optional) exact match in the transaction 'from' property<br/>
- **to** - (optional) exact match in the transaction 'to' property<br/>
- **value** - (optional) exact match in the transaction 'value' property<br/>
- **input** - (optional) wildcard match of the transaction 'input' data<br/>

**Wildcard matching support:**
- Wildcards (\*, ?, \**)


```sh
curl https://api.zmok.io/mainnet/your-app-ID \
-X POST \
-H 'Content-type: application/json' \
-d '{"jsonrpc": "2.0", "method": "zmk_txpool_search", "params":[{"to": "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D", "input": "0x38ed1739*"}], "id": 1}'

{"jsonrpc":"2.0","id":1,"result":{"pending":{"0x87b23e2124e50bc1e6539b61043f39e0071afd6f11311b19b29b865d6c138882":{"70223":{"blockHash":null,"blockNumber":null,"from":"0x98041ab523024dacaefa3bb70dd982dbac68e855","gas":"0x61a80","gasPrice":"0x14e7458a40","maxFeePerGas":"0x14e7458a40","maxPriorityFeePerGas":"0x14e7458a40","hash":"0x87b23e2124e50bc1e6539b61043f39e0071afd6f11311b19b29b865d6c138882","input":"0x38ed17390000000000000000000000000000000000000000000006999e1c5668566c39b6000000000000000000000000000000000000000000000035a83bffd099a3ffff00000000000000000000000000000000000000000000000000000000000000a000000000000000000000000098041ab523024dacaefa3bb70dd982dbac68e85500000000000000000000000000000000000000000000000000000000624f1c5f0000000000000000000000000000000000000000000000000000000000000002000000000000000000000000853d955acef822db058eb8505911ed77f175b99e0000000000000000000000003432b6a60d23ca0dfca7761b7ab56459d9c964d0","nonce":"0x1124f","to":"0x7a250d5630b4cf539739df2c5dacb4c659f2488d","transactionIndex":null,"value":"0x0","type":"0x2","accessList":[],"chainId":"0x1","v":"0x1","r":"0xf0eb5d621aeab0a952fcb233e271f9ca0f71e1acd39c91dcb4295fa966ea870e","s":"0x6f238592d31ec1b36f9314a0eb6f190c6e833cfcebcbf66ebd95c8451630a3d3"}}},"queued":{}}}

```

?> INFO: Maximum results for zmk_txpool_search are limited to 10 000 results. For optimal results use broader criteria in parameters.

### zmk_txpool_query
Returns a list with the exact details of all the transactions currently pending and queued based on the input query.

**Parameters:**<br/>
- **query** - (optional) SQL-like filter query <br/>

**More details:**<br/>
To decode input data of any known transaction you could use this library:
https://github.com/miguelmota/ethereum-input-data-decoder

Alternatively, you could resolve the smart contract method from the method signature (the first 4-bytes from the input) by using these public databases:
- https://github.com/ethereum-lists/4bytes
- https://www.4byte.directory/signatures/?bytes4_signature=0x38ed1739

**Example:**<br/>
Sample usage how to get all pending/queued transactions to Uniswap V3 Router (0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45) with method call:

<i>multicall(uint256,bytes[])</i>

This sample usage is often used during the gasPrice analysis of the arbitrages on Uniswap.


```sh
curl https://api.zmok.io/mainnet/your-app-ID \
-X POST \
-H 'Content-type: application/json' \
-d '{"jsonrpc": "2.0", "method": "zmk_txpool_query", "params":["('\''to'\'' = '\''0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45'\'' AND '\''input'\'' LIKE '\''0x5ae401dc%'\'' AND '\''value'\'' > '\''0x99'\'')"], "id": 1}'

{"queued":{"0xf055aa4b5c7130ddf00515ca47362a4ba69d44eda29d371de187e0327e3f0cb8":{"from":"0xd7996f47f13f56e4f8bbb3c7040dce8ffdaf704c","to":"0x68b3465833fb72a70ecdf485e0e4c7bd8665fc45","gas":"0x5d56e","gasPrice":"0x59682f000","hash":"0xf055aa4b5c7130ddf00515ca47362a4ba69d44eda29d371de187e0327e3f0cb8","input":"0x5ae401dc00000000000000000000000000000000000000000000000000000000622acc9d0000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000124b858183f00000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000080000000000000000000000000d7996f47f13f56e4f8bbb3c7040dce8ffdaf704c000000000000000000000000000000000000000000000000001c6bf526340000000000000000000000000000000000000000000000000000000984d16a0efce90000000000000000000000000000000000000000000000000000000000000042c02aaa39b223fe8d0a0e5c4f27ead9083c756cc20001f4a0b86991c6218b36c1d19d4a2e9eb0ce3606eb480027103819f64f282bf135d62168c1e513280daf905e0600000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","nonce":"0x1","value":"0x1c6bf526340000","type":"0x2","v":"0x0","r":"0xa6a9a44705e60e9d4df50be947495763b491d574f75127098f6ab343f5b3ccbd","s":"0x4faae862ee2ca1c3ba6e0ce21b54687e9168df663865ec8f3ebfff5885654164","maxFeePerGas":"0x59682f000","maxPriorityFeePerGas":"0x77359400","chainId":"0x1","blockHash":null,"blockNumber":null,"transactionIndex":null,"accessList":[]},"0xf93afad2c099704eb907e6970e873ef1f7795f98c089b419f7d811289b8f7074":{"from":"0xd2aa63b953ea42072a673114a5e0cf7d6b54a1c9","to":"0x68b3465833fb72a70ecdf485e0e4c7bd8665fc45","gas":"0x2fc53","gasPrice":"0x501fa738a","hash":"0xf93afad2c099704eb907e6970e873ef1f7795f98c089b419f7d811289b8f7074","input":"0x5ae401dc0000000000000000000000000000000000000000000000000000000062b2d51c00000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000e404e45aaf000000000000000000000000c02aaa39b223fe8d0a0e5c4f27ead9083c756cc2000000000000000000000000a0b86991c6218b36c1d19d4a2e9eb0ce3606eb4800000000000000000000000000000000000000000000000000000000000001f4000000000000000000000000d2aa63b953ea42072a673114a5e0cf7d6b54a1c90000000000000000000000000000000000000000000000000011c37937e080000000000000000000000000000000000000000000000000000000000000417c3c000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","nonce":"0x5","value":"0x11c37937e08000","type":"0x2","v":"0x0","r":"0x39d182bcada06d29cec269dde8d6cbf5a386ab6243086b490c30af6562af35c2","s":"0x7df6fc3632edc895202292a824ac274bc4cc6431695796caf83e61b5db023260","maxFeePerGas":"0x501fa738a","maxPriorityFeePerGas":"0x59682f00","chainId":"0x1","blockHash":null,"blockNumber":null,"transactionIndex":null,"accessList":[]},"0xa7c7cf99c6bb5e344c0f4e73408e06b175a0588f4bc4f1a7df94c3aa0922a635":{"from":"0x030dfd6697f5089a59a19a7634c807bff961923b","to":"0x68b3465833fb72a70ecdf485e0e4c7bd8665fc45","gas":"0x34936","gasPrice":"0x12a05f200","hash":"0xa7c7cf99c6bb5e344c0f4e73408e06b175a0588f4bc4f1a7df94c3aa0922a635","input":"0x5ae401dc0000000000000000000000000000000000000000000000000000000061e11ecb00000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000e4472b43f3000000000000000000000000000000000000000000000000000ffcb9e57d400000000000000000000000000000000000000000000000000004840b2e324d629e0000000000000000000000000000000000000000000000000000000000000080000000000000000000000000030dfd6697f5089a59a19a7634c807bff961923b0000000000000000000000000000000000000000000000000000000000000002000000000000000000000000c02aaa39b223fe8d0a0e5c4f27ead9083c756cc20000000000000000000000008b3192f5eebd8579568a2ed41e6feb402f93f73f00000000000000000000000000000000000000000000000000000000","nonce":"0x0","value":"0xffcb9e57d4000","type":"0x0","v":"0x26","r":"0xc1875afc1f71920c1c3647370488b953e827cade690380d40aa75430cdf0de0c","s":"0x2235463c2b4c0fcd7e612eb71aa49691020f1b88e19550e1230eb5756e1f9320","maxFeePerGas":null,"maxPriorityFeePerGas":null,"chainId":null,"blockHash":null,"blockNumber":null,"transactionIndex":null,"accessList":null}}}

```


### zmk_txpool_tx_subscribe
Websocket subscribe method which returns all new transactions from ZMOK global tx-pool filtered by SQL-like structured query.

**Parameters:**<br/>
- **query** - (optional) SQL-like filter query <br/>

**Example:**

```js
var fs = require('fs');
const WebSocket = require('ws');

const ws = new WebSocket(
  'wss://api.zmok.io/mainnet/YOUR_APP_ID', {}
);

function proceed() {
    // Method: zmk_txpool_tx_subscribe method (only available at endpoint wss://api.zmok.io/mainnet/YOUR_APP_ID)

    // Sample 1 - Returns all new pending/queued transactions from tx pool filtered with query
    // Query: All transactions submitted to: Uniswap V3: Router 2 Contract,
    // calling function:  multicall(uint256 deadline, bytes[] data)
    // and with gas price greater than 100 Gwei
    ws.send(`{"jsonrpc": "2.0", "id": 1, \
    "method": "zmk_txpool_tx_subscribe", \
    "params": ["('to' = '0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45' \
    AND 'input' LIKE '0x5ae401dc%' \
    AND 'gasPrice' > 100000000000 \
    AND 'value' = '0x0')"]}`);

    // Sample 2 - Returns all new pending/queued transactions from tx pool filtered with query
    // Query: All transactions submitted to: Moonbirds2 Contract,
    // calling function:  mint(uint256)
    // ws.send(`{"jsonrpc": "2.0", "id": 1, \
    // "method": "zmk_txpool_tx_subscribe", \
    // "params": ["('to' = '0xdb7b094FdC04F51560a03a99F747044951B73727' AND 'input' LIKE '0xa0712d68%')"]}`);


    // Sample response message {"jsonrpc":"2.0","id":1,"result":{"pending":{"from":"0x77a6fd93016a84151068227a23671fc1b02195bb","to":"0x68b3465833fb72a70ecdf485e0e4c7bd8665fc45","gas":"0x2cbd7","gasPrice":"0x1bf08eb000","hash":"0x63d7720b05c440014643dbf13929dba4bb50d73a49fa154d8e14881299304cf5","input":"0x5ae401dc000000000000000000000000000000000000000000000000000000006255af0e000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000016000000000000000000000000000000000000000000000000000000000000000e4472b43f30000000000000000000000000000000000000000000a5249fb3c8b90b41298ae000000000000000000000000000000000000000000000000009513eb7a8e88900000000000000000000000000000000000000000000000000000000000000080000000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000020000000000000000000000008e6cd950ad6ba651f6dd608dc70e5886b1aa6b24000000000000000000000000c02aaa39b223fe8d0a0e5c4f27ead9083c756cc200000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004449404b7c000000000000000000000000000000000000000000000000009513eb7a8e889000000000000000000000000077a6fd93016a84151068227a23671fc1b02195bb00000000000000000000000000000000000000000000000000000000","nonce":"0x9","value":"0x0","type":"0x2","v":"0x1","r":"0x7b8cabf23c55f7c8f74346300a40a4b01107e783c099ad287610881148c1e2d8","s":"0x349725fb7d42a7ce55290f9245dd58de572005cfb784dd06c246d3c888fc82b8","maxFeePerGas":"0x1bf08eb000","maxPriorityFeePerGas":"0x77359400","chainId":"0x1","blockHash":null,"blockNumber":null,"transactionIndex":null,"accessList":[]}}}


    // Method: zmk_txpool_tx_unsubscribe
    // ws.send(`{"jsonrpc": "2.0", "id": 1, \
    // "method": "zmk_txpool_tx_unsubscribe", \
    // "params": []}`);
  }


function handle(nextNotification) {
    console.log(nextNotification.toString()); // or process it generally
}

// Example with ZMOK Global Tx Pool and SQL-like structured query/filter over WebSockets

// To run this example:
// $ npm i ws
// $ node zmkTxPoolChecker.js

// Ethereum signature database: https://www.4byte.directory/

ws.on('open', proceed);
ws.on('message', handle);
```

### zmk_txpool_tx_unsubscribe
Websocket unsubscribe method.


## Quick answers
- Is it a Layer2 solution? - No
- Will be reading blockchain faster too? - No
- It is MEV? - No
- Do you send my transactions to private Validators? - No
- Can I create multiple FR endpoints? - Yes, as many as you wish.


