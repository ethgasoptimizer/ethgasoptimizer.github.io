# Gas Price Alerts

Stay informed when Ethereum gas drops or spikes beyond your threshold.

---

## Creating or Editing an Alert

### Fields
| Field | Description |
|--------|--------------|
| **Contact Method** | Currently supports **Email** |
| **To (email address)** | Destination email for the alert |
| **Gas Speed** | Which track to evaluate — **Slow**, **Average**, or **Fast** |
| **Condition** | “less than” or “greater than” |
| **Threshold (gwei)** | Numeric trigger, e.g. `0.100` |
| **Cooldown** | How long to wait before alerting again (e.g. `1d`) |
| **Add a note (optional)** | Custom message for internal use |
| **Disable after first trigger** | Stops alert after one send |

---

## Buttons
- **Save & Schedule** → Activate alert  
- **Disable** → Temporarily pause it  
- **Cancel** → Exit without saving

---

## Behavior
- The app evaluates gas at each processed block.  
- Alerts compare **selected Gas Speed** vs. **threshold condition**.  
- A **cooldown** ensures you don’t get duplicate notifications.  
- Triggered events are shown in the **History** chart (yellow dot).  
- Triggered alerts may also appear in **Savings** if they resulted in cheaper tx execution.

---

## Example
| Field | Example |
|--------|----------|
| Gas Speed | Average |
| Condition | less than |
| Threshold | 0.100 gwei |
| Cooldown | 1 day |
| Note | Just test! |

Sends an email if average gas < 0.1 gwei, then pauses for one day.