# FAQ (Frequently Asked Questions)

## Using Front-Running, can I save on gas fees and be the first at the same time?
With Front-Running your transaction will be picked and processed by miners very fast, on average in some milliseconds. The next thing is how it will be prioritised in the newest block globally.

Some miners on the Ethereum chain, like Ethermine, use non-conventional ordering for their benefit, aka they generally don't do the by-gas sort. Other miners can choose to sort by gas and nonce, others can choose to sort by gas and first-seen-time, others can first sort by gas and then randomise the order, others can sort by gas first and then put the "greatest" sender addresses at the top or bottom, which is converting an address to a 160-bit integer and working with those values.

If you’ll get to miners first with calculated or higher gas, you can be more than sure you’ll get it. If there will be any other user with ZMOK’s Front-Running extension, and you have a lower Gwei, there is a higher chance the other user will get the priority. Therefore we do not recommend lowering Gwei.
If you do not use a Front-Running endpoint, your transaction will be sent as a standard transaction to only one node, in only one region, and the chance that this transaction will be mined in the next block is lower, even if you’d use much higher gas.

## Is Front-Running necessary for minting NFTs?
Not really, a basic paid plan shall be enough.

## If current Gwei changes rapidly during the public drop, do I need to change my gas?
In this case, you must check on pending transactions, find one with the highest gast and after that submit again with the same nonce:
[How to send a replacement transaction with a higher gas price](https://ethereum.stackexchange.com/questions/99651/how-to-send-a-replacement-transaction-with-a-higher-gas-price?rq=1)
