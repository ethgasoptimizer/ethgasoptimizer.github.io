# Auto-Txs (BETA)

Automatically execute your **signed Ethereum transaction** when the network’s **average gas price** drops to or below your chosen target.  
EthGasOptimizer continuously monitors new blocks and submits your pre-signed transaction at the most cost-efficient moment.

---

## Fields in “Add Auto-Tx”

| Field | Description |
|--------|-------------|
| **Target Gas Price (Gwei)** | The threshold gas price that triggers execution. |
| **Strict target price** | If enabled, the relay sends exactly at your target gas price. If disabled, it uses the current market average if it’s lower, maximizing savings. |
| **To (ETH address)** | The destination Ethereum address (contract or EOA). Must be a valid checksummed `0x...` address. |
| **Value (ETH, optional)** | Amount of ETH to transfer with the transaction. Use `0` for contract calls that don’t send ETH. |
| **Data (raw hex)** | The call data payload (hex-encoded, e.g. ABI-encoded method + args). |
| **Note (optional)** | Free-form text label for your dashboard (not stored on-chain). |

---

## Actions

- **Validate** → Performs syntax checks:
  - Validates destination address and checksum.
  - Verifies data is proper hex.
  - Ensures numeric values are within valid ETH range.
- **Save as Draft** → Saves configuration locally without signature.
- **Sign & Schedule** → Prompts your wallet to sign an **EIP-712 typed payload**, authorizing EthGasOptimizer’s relay to execute it later.
- **Cancel Auto-TX** → Discards or deletes a pending/draft auto-tx.
- **Close** → Closes the modal.

---

## Execution Flow

1. **Continuous Monitoring**  
   The backend scheduler updates average gas metrics every block (`low`, `avg`, `high` in Gwei).  
   These are stored in MongoDB and compared against pending Auto-Tx targets.

2. **Condition Match**  
   When `avg_gas ≤ target_gas` (and the strict rule, if enabled), the relay marks the Auto-Tx as *ready to execute*.

3. **Meta-Transaction Submission**  
   Your pre-signed EIP-712 payload is wrapped into a standard Ethereum transaction and submitted by EthGasOptimizer’s relay node.  
   The relay covers the **on-chain gas cost** upfront.

4. **Top-Up Deduction**  
   The relay is reimbursed from your **Top-Up balance** — a prepaid internal wallet you maintain inside EthGasOptimizer.  
   The system deducts the *exact gas used × effective gas price* (in ETH) plus a small relay fee.

5. **Confirmation**  
   Once the transaction is mined:
   - Status → `EXECUTED`
   - The execution record appears in **History** and **Savings** panels.
   - The exact block number, gas used, and ETH spent are stored for reference.

---

## Example

| Parameter | Example Value |
|------------|----------------|
| Target Gas Price | `1.000` |
| Strict target price | ✅ |
| To | `0xE592427A0AEce92De3Edee1F18E0157C05861564` |
| Value | `0` |
| Data | `0x38ed1739000000000000000000000000...` *(UniswapV2 swapExactTokensForTokens)* |
| Note | `Swap USDC → ETH when gas ≤ 1 gwei` |

---

## How Meta-Transactions Work (EIP-712)

EthGasOptimizer uses **EIP-712 typed data signing** to let you authorize future transactions securely.  
Your wallet signs a structured message that’s verified later on-chain by the relay’s `MinimalForwarder` contract — so your private key never leaves your browser.

---

### 1. Signing the Payload

When you click **“Sign & Schedule”**, your wallet signs an EIP-712 typed message matching the **MinimalForwarder** schema:

```json
{
  "types": {
    "ForwardRequest": [
      { "name": "from", "type": "address" },
      { "name": "to", "type": "address" },
      { "name": "value", "type": "uint256" },
      { "name": "gas", "type": "uint256" },
      { "name": "nonce", "type": "uint256" },
      { "name": "data", "type": "bytes" }
    ]
  },
  "domain": {
    "name": "MinimalForwarder",
    "version": "0.0.1",
    "chainId": 1,
    "verifyingContract": "0xYourRelayAddress"
  },
  "primaryType": "ForwardRequest",
  "message": {
    "from": "0xYourWalletAddress",
    "to": "0xReceiverOrContract",
    "value": "0",
    "gas": "400000",
    "nonce": "4",
    "data": "0xa9059cbb000000000000000000000000..."
  }
}
```
This payload is signed using eth_signTypedData_v4, producing a cryptographically verifiable signature.

### 2.	The EthGasOptimizer Forwarder contract executes your transaction later via:
```json
function execute(ForwardRequest calldata req, bytes calldata signature)
    external
    payable
    returns (bool success, bytes memory returndata);
```

The contract verifies:
	•	Signature authenticity (EIP-712 domain separator)
	•	Nonce validity (prevents replay)
	•	Deadline expiry
Then executes the call as if it came from your address.

### 3.	Gas is paid by the relay; you reimburse via Top-Up.

## Why Top-Up Balance Is Required

The relay covers the on-chain gas fees using its own ETH.
To make this sustainable, each user has an off-chain Top-Up balance.

When your Auto-Tx executes:
```json
function execute(ForwardRequest calldata req, bytes calldata signature)
    external
    payable
    returns (bool success, bytes memory returndata);
```

If your **Top-Up balance** is insufficient:

- The Auto-Tx remains **queued** and will not execute until sufficient balance is available.  
- A red **“Top-Up required”** banner will appear in the dashboard as a warning.

---

## Refill Process

You can **refill your Top-Up balance** by sending ETH to your assigned **deposit address**.  
Once the transaction is confirmed on-chain, your dashboard automatically updates to reflect the new available balance.  
Refills are typically recognized within 1–2 Ethereum block confirmations.

---

## Security Model

- **Private Keys** — Your wallet only signs **EIP-712 typed data**.  
  No raw transactions or private keys ever leave your device.  

- **Replay Protection** — Each Auto-Tx contains a unique **nonce** and **domain separator** (`chainId`, `version`, `name`), ensuring signatures are valid only once and cannot be replayed.  

- **Integrity** — The relay cannot modify `to`, `value`, or `data`.  
  Any modification invalidates the EIP-712 signature.  

- **Expiry** — Every signed request includes a `deadline` timestamp.  
  Once expired, the request is automatically rejected and never executed.  

- **Auditable** — All broadcasted payloads and receipts can be **verified off-chain** against your original signature for full transparency.

---

## Summary

- ✅ You **sign once**, securely via your wallet.  
- 🔁 The system continuously monitors gas and executes when your conditions are met.  
- ⛽ The relay pays network gas upfront; your **Top-Up** balance reimburses it automatically.  
- 🧩 All logic follows **EIP-712 meta-transaction standards** — secure, auditable, and cost-efficient.

---