# Moloch (Majeur) DAO Framework

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Solidity](https://img.shields.io/badge/solidity-%5E0.8.30-black)](https://docs.soliditylang.org/en/v0.8.30/)
[![Foundry](https://img.shields.io/badge/Built%20with-Foundry-000000.svg)](https://getfoundry.sh/)

![Majeur Banner](assets/majeur-banner.svg)

**Opinionated DAO governance** — members can always exit with their share of the treasury. Built-in futarchy, weighted delegation, and soulbound badges.

## Why Majeur?

- Ragequit — exit with your share of the treasury
- Futarchy — prediction markets on proposals
- Split delegation — divide votes across multiple delegates
- On-chain SVG metadata — no IPFS, no servers
- DUNA legal wrapper support

## Deployments

All contracts are deployed at the same CREATE2 addresses across supported networks.

| Contract | Address | Description |
|----------|---------|-------------|
| Summoner | [`0x0000000000330B8df9E3bc5E553074DA58eE9138`](https://contractscan.xyz/contract/0x0000000000330B8df9E3bc5E553074DA58eE9138) | Factory for deploying new DAOs |
| Renderer | [`0x000000000011C799980827F52d3137b4abD6E654`](https://contractscan.xyz/contract/0x000000000011C799980827F52d3137b4abD6E654) | On-chain SVG metadata renderer |
| MolochViewHelper | [`0x00000000006631040967E58e3430e4B77921a2db`](https://contractscan.xyz/contract/0x00000000006631040967E58e3430e4B77921a2db) | Batch read helper for dApps |
| Tribute | [`0x000000000066524fcf78Dc1E41E9D525d9ea73D0`](https://contractscan.xyz/contract/0x000000000066524fcf78Dc1E41E9D525d9ea73D0) | OTC escrow for tribute proposals |
| ShareBurner | [`0x000000000040084694F7B6fb2846D067B4c3Aa9f`](https://contractscan.xyz/contract/0x000000000040084694F7B6fb2846D067B4c3Aa9f) | Burn unsold shares after sale deadline |
| ZAMM | [`0x000000000000040470635EB91b7CE4D132D616eD`](https://contractscan.xyz/contract/0x000000000000040470635EB91b7CE4D132D616eD) | AMM for LP seed swaps |
| SafeSummoner | [`0x00000000004473e1f31c8266612e7fd5504e6f2a`](https://contractscan.xyz/contract/0x00000000004473e1f31c8266612e7fd5504e6f2a) | Safe deployment wrapper with validated config |

### Implementations

Minimal proxy clones are deployed from these implementation contracts. Each DAO gets its own clone of Moloch, Shares, Loot, and Badges via CREATE2.

| Contract | Address | Description |
|----------|---------|-------------|
| Moloch | [`0x643A45B599D81be3f3A68F37EB3De55fF10673C1`](https://contractscan.xyz/contract/0x643A45B599D81be3f3A68F37EB3De55fF10673C1) | DAO governance logic |
| Shares | [`0x71E9b38d301b5A58cb998C1295045FE276Acf600`](https://contractscan.xyz/contract/0x71E9b38d301b5A58cb998C1295045FE276Acf600) | ERC-20 voting token |
| Loot | [`0x6f1f2aF76a3aDD953277e9F369242697C87bc6A5`](https://contractscan.xyz/contract/0x6f1f2aF76a3aDD953277e9F369242697C87bc6A5) | ERC-20 non-voting token |
| Badges | [`0x47C175Ce83B6B931ccBedD5ce95e701984eD96d5`](https://contractscan.xyz/contract/0x47C175Ce83B6B931ccBedD5ce95e701984eD96d5) | ERC-721 soulbound NFT |

## Dapps

> [zfi.wei/dao](https://zfi.wei.is/dao/)

> [majeurdao.eth](https://majeurdao.eth.limo/)

## Token System

| Token | Standard | Purpose |
|-------|----------|---------|
| **Shares** | ERC-20 | Voting power + economic rights (delegatable) |
| **Loot** | ERC-20 | Economic rights only — no voting |
| **Receipts** | ERC-6909 | Vote receipts for futarchy payouts |
| **Badges** | ERC-721 | Soulbound NFTs for top 256 shareholders |

Shares, Loot, and Badges deploy as minimal proxy clones. Receipts live inside the Moloch contract. The DAO controls minting, burning, and transfer locks.

## Architecture

![Majeur Architecture](./assets/architecture.svg)

## Core Concepts

### Ragequit
Burn shares/loot → receive your pro-rata cut of the treasury. Own 10%? Claim 10% of every token. Only external tokens (ETH, USDC, etc.) — not the DAO's own shares, loot, or badges.

### Futarchy
Prediction markets on proposals. Anyone funds a reward pool → voters receive receipt tokens → winning side splits the pool.

### Split Delegation
Divide voting power across multiple delegates (e.g. 60% Alice, 40% Bob).

### Badges
Soulbound NFTs for the top 256 shareholders. Auto-update as balances change. Gate on-chain chat.

### Wyoming DUNA
Majeur supports Wyoming's **Decentralized Unincorporated Nonprofit Association (DUNA)** (Wyoming Statute 17-32-101). On-chain covenant in metadata, badge-based member registry, permanent governance records, ragequit as a legal exit right.

## Proposal Lifecycle

![Proposal Lifecycle](./assets/proposal-lifecycle.svg)

```
Unopened → Active → Succeeded → Queued (if timelock) → Executed
                 ↘ Defeated
                 ↘ Expired (TTL)
```

**Pass conditions** (all must hold): quorum reached, FOR > AGAINST, minimum YES threshold met (if set), not expired.

## Quick Start

### Deploy a DAO

```solidity
Summoner summoner = new Summoner();
Moloch dao = summoner.summon(
    "MyDAO",           // name
    "MYDAO",           // symbol
    "",                // URI
    5000,              // 50% quorum (bps)
    true,              // ragequittable
    address(0),        // renderer (0 = default on-chain SVG)
    bytes32(0),        // salt
    [alice, bob, charlie],
    [100e18, 50e18, 50e18],
    new Call[](0)
);
```

### Create & Vote on Proposals

```solidity
uint256 proposalId = dao.proposalId(0, to, value, data, nonce);

dao.castVote(proposalId, 1);  // 0=AGAINST, 1=FOR, 2=ABSTAIN

dao.executeByVotes(0, to, value, data, nonce);
```

### Weighted Delegation (Split Voting Power)

```solidity
dao.shares().setSplitDelegation([alice, bob], [6000, 4000]);  // must sum to 10000
dao.shares().clearSplitDelegation();
```

### Futarchy Markets

```solidity
dao.fundFutarchy(proposalId, address(0), 1 ether);  // 0 = ETH
dao.cashOutFutarchy(proposalId, myReceiptBalance);
```

### Token Sales

```solidity
// DAO configures sale (governance action)
dao.setSale(address(0), 0.01 ether, 1000e18, true, true, false);
//          token       price        cap       mint  active isLoot

// Users buy
dao.buyShares{value: 1 ether}(address(0), 100e18, 1 ether);
//                              token     shares   maxPay
```

### Ragequit

```solidity
dao.ragequit([weth, usdc, dai], myShares, myLoot);  // tokens must be sorted
```

## Advanced Features

### Pre-Authorized Permits

Permits let specific addresses execute actions without voting:

```solidity
dao.setPermit(op, to, value, data, nonce, alice, 1);  // DAO issues
dao.spendPermit(op, to, value, data, nonce);            // Alice spends
```

### Timelocks

```solidity
dao.setTimelockDelay(2 days);
dao.setProposalTTL(7 days);
```

### Member Chat (Badge-Gated)

```solidity
dao.chat("Hello DAO members!");  // top 256 only
```

## Common Pitfalls

### Forgetting to sort tokens in ragequit
```solidity
// Wrong - will revert if not sorted
address[] memory tokens = [dai, weth, usdc];

// Correct - tokens sorted by address
address[] memory tokens = [dai, usdc, weth]; // sorted ascending
```

### Wrong basis points in delegation
```solidity
// Wrong - doesn't sum to 10000
uint32[] memory bps = [6000, 3000]; // 90% total

// Correct - must sum to exactly 10000
uint32[] memory bps = [6000, 4000]; // 100% total
```

## Contract Architecture

```
Summoner (Factory)
└── Deploys via CREATE2 + minimal proxy clones
    │
    ├── Moloch (Main DAO Contract)
    │   ├── Governance logic (proposals, voting, execution)
    │   ├── ERC-6909 receipts (multi-token vote receipts)
    │   ├── Futarchy markets
    │   ├── Ragequit mechanism
    │   └── Token sales
    │
    ├── Shares (Separate ERC-20 + ERC-20Votes Clone)
    │   ├── Voting power tokens
    │   ├── Transferable/Lockable (DAO-controlled)
    │   ├── Single delegation or split delegation
    │   └── Checkpoint-based vote tracking
    │
    ├── Loot (Separate ERC-20 Clone)
    │   ├── Non-voting economic tokens
    │   └── Transferable/Lockable (DAO-controlled)
    │
    └── Badges (Separate ERC-721 Clone)
        ├── Soulbound (non-transferable) NFTs
        ├── Automatically minted for top 256 shareholders
        └── Auto-updated as balances change

Renderer (Singleton)
├── On-chain SVG generation
├── DUNA covenant display
├── DAO contract metadata
├── Proposal cards
├── Vote receipt cards
├── Permit cards
└── Badge cards
```

## Peripheral Contracts

### Tribute (OTC Escrow)

Trade external assets for DAO membership:

```solidity
// 1. Proposer locks tribute (e.g., 10 ETH for 1000 shares)
tribute.proposeTribute{value: 10 ether}(
    dao,           // target DAO
    address(0),    // tribTkn (ETH)
    0,             // tribAmt (use msg.value for ETH)
    sharesToken,   // forTkn (what proposer wants)
    1000e18        // forAmt (how much)
);

// 2. DAO votes to accept, then claims (executes the swap)
// DAO receives tribute, proposer receives shares
dao.executeByVotes(...); // calls tribute.claimTribute(proposer, tribTkn, tribAmt, forTkn, forAmt)
```

**Key functions:**
- `proposeTribute()` - Lock assets and create offer
- `cancelTribute()` - Proposer withdraws (before DAO claims)
- `claimTribute()` - DAO accepts and executes swap
- `getActiveDaoTributes()` - View all pending tributes for a DAO

### MolochViewHelper (Batch Reader)

Batch reads for dApp frontends:

```solidity
// Fetch full state for multiple DAOs in one call
DAOLens[] memory daos = helper.getDAOsFullState(
    0,      // daoStart
    10,     // daoCount
    0,      // proposalStart
    5,      // proposalCount
    0,      // messageStart
    10,     // messageCount
    tokens  // treasury tokens to check
);

// User portfolio: find all DAOs where user is a member
UserMemberView[] memory myDaos = helper.getUserDAOs(
    user, 0, 100, tokens
);
```

Returns `DAOLens` (full state), `MemberView` (balances, delegation), `ProposalView` (tallies, state, futarchy).

### ShareSale (Share/Loot Sales via Allowance)

Sell shares or loot via the allowance system. Uses Moloch's `_payout` sentinels (`address(dao)` mints shares, `address(1007)` mints loot) with 1e18-scaled pricing.

```solidity
// Setup (in SafeSummoner extraCalls or initCalls):
// 1. dao.setAllowance(shareSale, address(dao), cap)  // or address(1007) for loot
// 2. shareSale.configure(address(dao), payToken, price, deadline)

// Users buy shares
shareSale.buy{value: cost}(dao, 10e18);  // 10 shares

// Pricing: cost = amount * price / 1e18
// e.g. price = 0.01e18 means 0.01 ETH per share
```

**Key functions:**
- `configure()` - Set sale token, payment token, price, and deadline (called by DAO)
- `buy()` - Purchase shares/loot (permissionless, refunds overpayment)
- `saleInitCalls()` - Generate initCalls for setup

### TapVest (Linear Vesting via Allowance)

Linear vesting from a DAO treasury via the allowance system.

```solidity
// Setup (in SafeSummoner extraCalls or initCalls):
// 1. dao.setAllowance(tap, token, totalBudget)
// 2. tap.configure(token, beneficiary, ratePerSec)

// Anyone can trigger claim — funds always go to beneficiary
tap.claim(dao);

// View functions
tap.claimable(dao);  // min(owed, allowance, daoBalance)
tap.pending(dao);    // total owed (ignoring caps)
```

**Vesting formula:** `owed = ratePerSec * elapsed`, capped by `min(owed, allowance, daoBalance)`.

**DAO governance:**
- `setBeneficiary()` - Change recipient (DAO-only)
- `setRate()` - Change rate, non-retroactive: unclaimed accrual is forfeited (DAO-only). Set to 0 to freeze.

### LPSeedSwapHook (Automatic LP Initialization)

Seeds ZAMM liquidity from DAO treasury. Gates `addLiquidity` pre-seed, returns swap fees post-seed.

```solidity
// Configured automatically via SafeSummoner SeedModule, or manually:
// 1. dao.setAllowance(lpHook, tokenA, amountA)
// 2. dao.setAllowance(lpHook, tokenB, amountB)
// 3. lpHook.configure(...)

// Anyone can trigger once conditions are met
lpHook.seed(dao);
lpHook.seedable(dao);  // check if ready
```

### ShareBurner (Post-Sale Cleanup)

Burns unsold shares after a sale deadline. DAOs issue a one-shot permit during setup.

```solidity
// Setup via SafeSummoner (automatic when saleBurnDeadline > 0):
// 1. dao.setPermit(op=1, target=burner, ..., spender=burner, count=1)

// After deadline, anyone can trigger the burn
shareBurner.closeSale(dao, sharesAddr, deadline, nonce);
```

### SafeSummoner (Deployment Wrapper)

Wraps the Summoner with audit-derived configuration guardrails. Validates `SafeConfig` structs and builds `initCalls` automatically.

```solidity
// Preset deployments (one-call with sane defaults)
safe.summonStandard(name, symbol, uri, salt, holders, shares, lockShares);  // 7d/2d/10%
safe.summonFast(name, symbol, uri, salt, holders, shares, lockShares);      // 3d/1d/5%
safe.summonFounder(name, symbol, uri, salt);                                // 1d/none/10%/solo

// Full control
safe.safeSummon(name, symbol, uri, quorum, ragequittable, renderer, salt,
    holders, shares, loot, config, extraCalls);

// With modular sale/tap/LP (replaces legacy DAICO)
safe.safeSummonDAICO(name, symbol, uri, quorum, ragequittable, renderer, salt,
    holders, shares, loot, config, sale, tap, seed, extraCalls);
safe.summonStandardDAICO(name, symbol, uri, salt, holders, shares, lockShares, sale, tap, seed);
safe.summonFastDAICO(name, symbol, uri, salt, holders, shares, lockShares, sale, tap, seed);

// Utilities
safe.predictDAO(salt, holders, shares);
safe.predictShares(dao);
safe.predictLoot(dao);
safe.previewCalls(config);
```

Modules: `ShareSale`, `TapVest`, `LPSeedSwapHook` — configured via `SaleModule`, `TapModule`, `SeedModule` structs. Set `singleton = address(0)` to skip.

## Visual Cards

![DAO Contract Card](./assets/dao-contract-card.svg)
![Proposal Card](./assets/proposal-card.svg)
![Vote Receipt Cards](./assets/vote-receipt-cards.svg)
![Permit Card](./assets/permit-card.svg)
![Badge Card](./assets/badge-card.svg)

## Integration

```javascript
const shares = await dao.shares();
const totalSupply = await shares.totalSupply();
const myBalance = await shares.balanceOf(account);
const myVotes = await shares.getVotes(account);

// Check proposal state
const state = await dao.state(proposalId);
// States: 0=Unopened, 1=Active, 2=Queued, 3=Succeeded, 4=Defeated, 5=Expired, 6=Executed

// Get vote tally
const tally = await dao.tallies(proposalId);
console.log(`FOR: ${tally.forVotes}, AGAINST: ${tally.againstVotes}`);
```

Key events: `Opened`, `Voted`, `Executed`, `SaleUpdated`.

## Development

### Build & Test

```bash
# Install Foundry
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Build
forge build

# Run all tests
forge test

# Run with verbosity
forge test -vvv

# Run specific test file
forge test --match-path test/Moloch.t.sol

# Run specific test
forge test --match-test test_Ragequit

# Gas snapshot
forge snapshot
```

### Test Suite

| File | Coverage |
|------|----------|
| `Moloch.t.sol` | Core governance: voting, delegation, execution, ragequit, futarchy, badges |
| `Tribute.t.sol` | OTC escrow: propose, cancel, claim tributes |
| `MolochViewHelper.t.sol` | Batch read functions for dApps |
| `SafeSummoner.t.sol` | Preset deployments, config validation, ShareBurner integration, address prediction |
| `ShareSale.t.sol` | Share/loot purchases, refunds, allowance caps, pricing |
| `TapVest.t.sol` | Linear vesting claims, rate changes, beneficiary updates, cap enforcement |
| `LPSeedSwapHook.t.sol` | LP seed swap hook for automatic ZAMM LP initialization |
| `RollbackGuardian.t.sol` | Rollback guardian for emergency DAO recovery |
| `ContractURI.t.sol` | On-chain metadata and DUNA covenant |
| `URIVisualization.t.sol` | SVG rendering for cards |
| `Bytecodesize.t.sol` | Contract size limits |

**Key test scenarios:**
- Proposal lifecycle (open → vote → queue → execute)
- Split delegation with multiple delegates
- Futarchy funding and payout
- Ragequit with multiple tokens
- Badge auto-updates on balance changes
- Tribute propose/cancel/claim flows
- SafeSummoner preset guardrails and ShareBurner permits
- ShareSale buy/refund/cap flows with ETH
- TapVest claim/rate/beneficiary governance
- LP seed swap hook initialization
- Rollback guardian emergency recovery

### Gas Optimization

| Technique | Savings | Details |
|-----------|---------|---------|
| Clone pattern | ~80% deployment | Minimal proxy clones for Shares, Loot, Badges |
| Transient storage | ~5k/call | EIP-1153 for reentrancy guards |
| Badge bitmap | ~20k/update | 256 holders in single storage slot |
| Packed structs | ~20k/write | Tallies use 3 × uint96 for compact storage |

### Deploy

```bash
forge script script/Deploy.s.sol --rpc-url $RPC_URL --broadcast
```

## Security Model

| Protection | Mechanism |
|------------|-----------|
| Flash loan attacks | Snapshot at block N-1 |
| Reentrancy | Transient storage guards (EIP-1153) |
| Majority tyranny | Ragequit — minorities can exit with their share |
| Malicious proposals | Timelocks give time to ragequit; `bumpConfig()` invalidates all pending |
| Token reentrancy | Ragequit requires sorted token arrays |

## License

MIT
