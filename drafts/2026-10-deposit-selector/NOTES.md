# Crypto deposit selector (ENG-10847)

**Problem:** unlabelled logo chips make people guess networks, and a silent token auto-switch leaves a valid address on screen for the wrong pair.

## Recommendation

Ship ENG-10847 as planned (two comboboxes, changing network clears the token, no address until both are picked), plus:

1. Network rows show their supported tokens as tags.
2. Network search also matches tokens and short names (`usdt`, `arb`, `matic`).
3. Inline note when the token is cleared ("USDT isn't available on Arbitrum").
4. Address card title restates the pair ("Send only USDC on Arbitrum").

## V2

- Keep the token on network change when the new network supports it.
- Remember last-used pair per account.
- Reuse the component in buy/sell.

## Open

- Is the in-flight PR already doing 1 and 2?
- Who owns the network alias list?
