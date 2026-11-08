# GemJar

A match-3 puzzle game built for [Celo](https://celo.org) and [MiniPay](https://www.opera.com/products/minipay).

Match gems to earn points, compete for a daily stablecoin prize pool, and watch a slice of every win drop into your on-chain savings jar.

## Status

- Game core (board, matching, cascades, combos, scoring) is done and covered by tests.
- `PrizePool` and `SavingsJar` are deployed to **Celo Mainnet** as UUPS upgradeable proxies.
- The frontend is wired to both contracts: stake to enter, submit a score, claim winnings, and view the daily leaderboard and prize pool.
- MiniPay users can pay gas in USDm through CIP-64 fee abstraction.
- Not yet deployed to Celo Sepolia.

## Contracts (Celo Mainnet)

| Contract | Proxy address |
| --- | --- |
| PrizePool | `0xf508930cC806936700abfD9bAEb7534558684Ed0` |
| SavingsJar | `0x734c2B09DB7c790d298ad639697b6ab02C452A83` |

Both proxies share one owner and can be upgraded with `upgradeToAndCall` without changing these addresses. Sources are in [`contracts/src`](contracts/src).

### How a round works

1. Players stake a fixed amount of USDm to enter the current day's round.
2. Scores are submitted during the round.
3. After the round ends, each player claims a share of the pool proportional to their score.
4. A configurable slice of every payout (in basis points) is routed into the player's `SavingsJar`.

## Tech stack

- [Next.js](https://nextjs.org) (App Router) + TypeScript
- [Tailwind CSS](https://tailwindcss.com) and [Framer Motion](https://www.framer.com/motion/)
- [viem](https://viem.sh) and [wagmi](https://wagmi.sh) for Celo
- Solidity + [Foundry](https://book.getfoundry.sh), OpenZeppelin upgradeable contracts
- [Vitest](https://vitest.dev) for the game engine and utilities

## Getting started

```bash
git clone --recurse-submodules https://github.com/DevVichy/GemJar.git
cd GemJar
npm install
cp .env.example .env.local   # NEXT_PUBLIC_CHAIN_ENV=mainnet or sepolia
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to play.

### Tests

```bash
npm test                      # game engine, storage, wallet utils
cd contracts && forge test    # PrizePool and SavingsJar
```

## Project layout

```
app/          Next.js app router pages
components/   game board, jar, leaderboard, wallet UI
lib/celo/     chains, tokens, ABIs, contract addresses
lib/contracts hooks for PrizePool, SavingsJar, leaderboard
lib/game/     match-3 engine and types
contracts/    Foundry project (src, test, script)
```

## Design goals

- Playable instantly as a guest, no wallet required
- Works as a standalone web app and inside the MiniPay wallet
- Mobile-first, optimized for low-end Android devices on slow connections
