# TipBot-Shortcuts

Ticker shortcuts for the Xahau tipbot (@xahtipbot / @xrptipbot).

Every [Tipbot Oracle Node](https://github.com/RichardAH/tipbot-oracle-node) fetches `shortcuts.json` from `main` roughly every five minutes. This lets a post say `+5 EVR` instead of `+5 EVR:rEvernodee8dJLaFsujS6q1EiXvZYmHXr8`.

A shortcut is only a default. An issuer the author writes (`+5 EVR:r...`) always wins. A ticker with no shortcut and no issuer is rejected, never sent as XAH.

## Format

```json
{
  "version": 1,
  "shortcuts": [
    { "ticker": "EVR", "issuer": "rEvernodee8dJLaFsujS6q1EiXvZYmHXr8", "name": "Evernode" }
  ]
}
```

| Field    | Required | Meaning |
|----------|----------|---------|
| `ticker` | yes      | 3–20 characters starting with a letter, or 40 hex. Case-insensitive. Not `XAH`. |
| `issuer` | yes      | Xahau r-address, or `null` to retire the ticker. |
| `from`   | no       | `YYYY-MM-DDTHH:MM:SSZ`. The entry applies to posts made at or after this time. If absent, it always applies. |
| `name`   | no       | For humans; ignored by oracles. |

## Adding or changing a token

1. **Verify the issuer on Xahau mainnet.** A wrong address with a valid checksum passes every check below.
2. **Set `from` at least an hour in the future.** Oracles pick up changes at different times. `from` makes them all switch at once, judged by each post's own timestamp, so no post gets conflicting votes during the rollout.
3. **Don't edit or delete old entries.**
   - To repoint a ticker, add a new entry with a later `from`.
   - To retire a ticker, add an entry with `"issuer": null`.
   - Keeping the history means backfilled and replayed posts resolve the way they did at the time.
4. **Validate the file.** From a tipbot-oracle-node checkout, run `node ton.js --check-shortcuts shortcuts.json`.
5. **Open a PR.**

## What oracles enforce

Oracles accept the file whole or not at all. Any of the following problems makes every oracle keep its previous list and log a warning:

- an unknown `version`
- a bad address or checksum
- `XAH`, or a ticker no post could type
- two entries with the same ticker and `from`

Each oracle logs the sha256 of the list it loaded, so operators can confirm they all agree.
