# FAQ (Frequently Asked Questions)

## Which network is supported?
Ethereum Mainnet (more EVM chains planned).

## Why are my gas prices different from Etherscan?
EthGasOptimizer uses **median-filtered values** (Tukey / MAD) to remove block-level outliers.

## What’s “priority fee”?
Per EIP-1559: total_gas_price = base_fee + priority_fee

The **priority fee** (tip) incentivizes faster inclusion.

## Do you custody funds?
No. Auto-Txs use **signed EIP-712** payloads — execution happens via your account.  
We never hold or move your assets.

## How do cooldowns work?
Cooldown starts after an alert fires. You won’t receive another from that rule until the period expires.

## What does “Strict target” mean?
When checked, your transaction is sent **only** if the gas price equals your target (not lower).

## How do I contact support?
📧 [support@ethgasoptimizer.com](mailto:support@ethgasoptimizer.com)