# Help Center

Welcome to **EthGasOptimizer**, your personal Ethereum gas tracker and automation suite.  
Here you can learn how to use **Dashboard**, **Gas Price Alerts**, and **Auto-Txs** (beta).

---

## Main Sections
- [Dashboard](#dashboard)
- [Gas Alerts](gas-alerts.md)
- [Auto-Txs](auto-txs.md)
- [FAQ](faq.md)

---

## Dashboard

### Current Gas
Shows live **Ethereum gas price tiers** — **Slow**, **Average**, and **Fast** — expressed in **Gwei**.

Each tier card displays:
- **Base:** current **base fee per gas** (per EIP-1559)  
- **Priority:** current **tip (miner reward)** added on top of the base fee  
- **Est. Confirmation:** estimated confirmation time for that tier (typically ≈ 60 s)  
- **Copy Price:** quick-copy button to use the selected gas value in alerts or Auto-Tx setup

### Recommendation
Shows the currently optimal gas to target, e.g.:

> You can likely get a transaction through at **0.39 Gwei** if you wait about 5 minutes.

Derived from short-term prediction over recent block averages.

### Savings
Tracks your estimated gas savings across your active alerts or auto-txs.  
- **Net Saved (USD/ETH)**
- **Events:** total triggered actions
- **Best Single:** biggest saving achieved
Selectable timeframe (24h / 7d / 30d).

### History
Displays average gas price evolution for the selected timeframe (1h, 24h, etc):
- **Min** point (green)
- **Max** point (red)
- **Fired Alerts / Auto-Txs** (yellow)
All values shown in **Gwei**.

---

## Units
All displayed prices use **Gwei**; internally stored as **wei**.

---

## Support
📧 [support@ethgasoptimizer.com](mailto:support@ethgasoptimizer.com)

---

[Gas Alerts](gas-alerts.md ':include')

[Auto-Txs](auto-txs.md ':include')

[FAQ](faq.md ':include')

