# GemJar case study

A match-3 game for MiniPay users, with a daily stablecoin prize pool and an on-chain savings jar.

## The problem

MiniPay users on Celo hold stablecoins, often with no CELO for gas, and a lot of them are on low-end Android phones and slow connections. Most on-chain games assume a desktop wallet, native gas and a patient user. GemJar asks: can a casual game work inside MiniPay, let people play for a small stake, and nudge them toward saving?

## What I built

- **Game:** a mobile-first match-3 engine (cascades, combos, scoring) and a claymorphism board UI. Guests can play with no wallet, and progress is stored locally.
- **PrizePool contract:** players stake 1 USDm per entry into a daily round, submit scores, and claim a share of the pool proportional to score after the round ends.
- **SavingsJar contract:** a configurable slice of every payout (currently 10%) is routed into the player's jar, which tracks deposit streaks.
- **MiniPay integration:** wallet detection, and CIP-64 fee abstraction so players pay gas in USDm instead of CELO.
- **Leaderboard and pool overview:** built from on-chain `ScoreSubmitted` events.

## Decisions worth explaining

- **UUPS upgradeable proxies.** The rules (stake size, savings rate) are expected to change as the game is tuned. Both contracts sit behind proxies with the same owner, so logic can be upgraded without moving funds or changing addresses.
- **Fixed-amount entries, score-proportional payouts.** Simple to explain to players, and the payout math is a single formula that is easy to test.
- **Savings split inside `claim`.** The slice goes to the jar in the same transaction as the payout, so saving needs no extra step from the player.
- **Reentrancy guards and `SafeERC20`** on every function that moves tokens.
- **Gas in USDm for MiniPay.** Users there typically hold no CELO. USDC and USDT need an adapter address as their `feeCurrency`, not the token address, which is easy to get wrong. I later extracted that logic into [celo-fee-kit](https://github.com/DevVichy/celo-fee-kit).
- **Guest mode first.** Staking is optional, so someone can try the game before connecting a wallet.

## Verification

- 38 Foundry tests across both contracts (16 and 22), including the upgrade path through test V2 contracts.
- 30 Vitest tests covering the game engine (19), guest storage and wallet utilities.
- Contracts deployed to Celo Mainnet:
  - PrizePool proxy `0xf508930cC806936700abfD9bAEb7534558684Ed0`
  - SavingsJar proxy `0x734c2B09DB7c790d298ad639697b6ab02C452A83`

## Honest status

The contracts are live and the app is wired to them. At the time of writing nobody has staked yet: the on-chain event history contains only the deployment events, which [gemjar-stats](https://github.com/DevVichy/gemjar-stats) shows live. The next step is putting GemJar in front of real MiniPay users and measuring whether people come back for a second round.

## What I would do differently

- Start with a Sepolia deployment and a public testnet round before going to mainnet, so the first real players find a tested flow.
- Add an indexer from day one. The leaderboard currently reads recent logs from the RPC, which is limited to 5,000-block ranges on the public endpoint. `gemjar-stats` solves this for history but the app itself does not use it yet.

## Links

- Code: https://github.com/DevVichy/GemJar
- Stats: https://github.com/DevVichy/gemjar-stats
- Helpers extracted from this project: https://github.com/DevVichy/celo-fee-kit
