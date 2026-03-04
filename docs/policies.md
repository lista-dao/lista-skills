# Lista Lending — Policies & Rules Reference

Operational policies for Lista Lending agent skills.

---

## Data Security

- Skills only read **publicly available on-chain data** (BSC public RPC, Lista public API)
- No private keys, seed phrases, or transaction signing required
- Wallet addresses are remembered only within the current conversation session
- No data is uploaded to external servers or shared with third parties

---

## Alert Threshold Rules

### Default Thresholds

System-provided defaults based on market LLTV:

| LLTV Range | Default Threshold | Rationale |
|---|---|---|
| LLTV >= 90% | LTV gap <= 0.5% | High-leverage markets — tighter alert window |
| LLTV < 90% | LTV gap <= 5% | Standard markets — wider alert window |

Where `LTV gap = LLTV - current LTV`.

### User Customization

- Users can override the default at any time: "change threshold to 3%"
- Custom threshold applies immediately, no confirmation required
- Custom threshold replaces the system default for that user session
- Users can reset: "restore default threshold"
- On session reset, thresholds return to system defaults

---

## Push Notification Rules

### Permissions

| Rule | Description |
|---|---|
| Default state | Off — no notifications sent |
| Enable | User explicitly says "enable alerts" or "subscribe to reports" |
| Disable | User says "cancel notifications" — stops immediately |
| Channel | Discord or Telegram — user picks one |
| Data scope | Read-only access to public chain data |

### Alert Notifications (Module 4)

- Triggered when a position's LTV gap falls below the active threshold
- Contains: current LTV, LLTV, LTV gap, liquidation price, current price, suggested action
- Frequency: checked periodically (implementation-dependent)

### Digest Reports (Module 5)

- Daily: delivered at 08:00 UTC every day
- Weekly: delivered at 08:00 UTC every Monday
- User picks daily or weekly (not both)
- Contains: lending positions, yield summary, market snapshot, risk alerts

### Future Capability Note

Scheduled push notifications require a backend notification service (Discord bot / Telegram bot). The current skill implementation generates the report format on demand. When the notification service becomes available, the subscription preferences recorded in the session can be forwarded to the service endpoint.

---

## Wallet Address Management

### Persistence

Wallet addresses are saved to `~/.lista/wallet.txt` for cross-session reuse:

```bash
# Save
mkdir -p ~/.lista && echo "0xABC...1234" > ~/.lista/wallet.txt

# Load
cat ~/.lista/wallet.txt 2>/dev/null
```

### First Interaction

When a user requests position-related information without having provided an address:

1. Check `~/.lista/wallet.txt` for a saved address
2. If found, confirm with the user or use directly
3. If not found, ask for their wallet address and save it

### Address Change

- User can say "change address" / "换个地址" / "換個地址" at any time
- New address replaces the previous one immediately and is saved to file
- All subsequent queries use the new address
- Multiple addresses: one per line in the wallet file

### Privacy

- Address is persisted locally to `~/.lista/wallet.txt`
- Not transmitted to any service beyond the public APIs needed for data retrieval
