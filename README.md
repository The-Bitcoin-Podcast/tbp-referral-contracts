# TBP Community Token Specification

## Overview

The TBP (The Bitcoin Podcast) Token is an ERC-20 utility token on Ethereum mainnet designed to incentivize community contributions and help sustain podcast operations.

## Token Purpose

1. **Reward Contributors** - Community members who submit successful grant, sponsorship, or guest referrals receive TBP tokens
2. **Create Buy Pressure** - A percentage of revenue from approved deals is used to buy back tokens
3. **Fund Operations** - Treasury allocation supports ongoing podcast needs (video editing, guest sourcing, etc.)

## Token Details

| Property | Value |
|----------|-------|
| Name | The Bitcoin Podcast Token |
| Symbol | TBP |
| Decimals | 18 |
| Chain | Ethereum Mainnet |
| Total Supply | 100,000,000 TBP (fixed) |
| Governance | None (utility token only) |

## Token Allocation

| Allocation | Percentage | Amount | Purpose |
|------------|------------|--------|---------|
| Rewards Pool | 40% | 40,000,000 | Contributor payouts for approved referrals |
| Treasury | 25% | 25,000,000 | Operations, future grants, contingency |
| Founding Team | 15% | 15,000,000 | Hosts, core contributors (vested) |
| Early Community | 10% | 10,000,000 | Airdrop to Discord members, early supporters |
| Liquidity | 10% | 10,000,000 | Uniswap V3 pool provision |

```
┌────────────────────────────────────────────────────────────┐
│                    100,000,000 TBP                         │
├──────────────────────────────┬─────────────────────────────┤
│       Rewards Pool (40%)     │                             │
│         40,000,000           │      Treasury (25%)         │
│                              │        25,000,000           │
├──────────────────────────────┼─────────────────────────────┤
│    Founding Team (15%)       │   Early Community (10%)     │
│       15,000,000             │       10,000,000            │
├──────────────────────────────┴─────────────────────────────┤
│                    Liquidity (10%)                         │
│                      10,000,000                            │
└────────────────────────────────────────────────────────────┘
```

## Revenue Flow Model

When a grant, sponsorship, or guest referral is approved and revenue is received:

```
Revenue Received ($X)
│
├── 70% → Podcast Operations (fiat)
│         - Video editing
│         - Hosting costs
│         - Equipment
│         - Staff payments
│
├── 20% → Referrer Cash Payout (fiat)
│         - Direct payment to contributor
│         - Incentivizes quality submissions
│
└── 10% → Token Buyback
          │
          ├── Buy TBP from Uniswap
          │
          └── Distribution:
              ├── 50% → Referrer (token bonus)
              └── 50% → Treasury (sustainability)
```

### Example Scenario

A community member submits a $10,000 grant opportunity that gets approved:

| Recipient | Amount | Type |
|-----------|--------|------|
| Podcast Operations | $7,000 | Fiat |
| Referrer Cash | $2,000 | Fiat |
| Token Buyback | $1,000 | Market buy |
| → Referrer Token Bonus | ~$500 worth | TBP tokens |
| → Treasury | ~$500 worth | TBP tokens |

## Reward Tiers

Fixed token rewards based on referral type:

| Referral Type | Base Token Reward | Notes |
|---------------|-------------------|-------|
| Guest Referral (approved) | 1,000 TBP | Podcast guest that gets interviewed |
| Grant (approved) | 2,500 TBP | Base, plus % bonus based on amount |
| Sponsorship (approved) | 5,000 TBP | Base, plus % bonus based on deal size |

### Bonus Multipliers

For grants and sponsorships, additional tokens based on deal value:

| Deal Value | Bonus Multiplier |
|------------|------------------|
| < $1,000 | 1.0x (base only) |
| $1,000 - $5,000 | 1.5x |
| $5,000 - $10,000 | 2.0x |
| $10,000 - $25,000 | 2.5x |
| > $25,000 | 3.0x |

## Smart Contract Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Ethereum Mainnet                         │
│                                                              │
│  ┌─────────────────┐         ┌─────────────────────────┐    │
│  │   TBPToken.sol  │         │    RewardPool.sol       │    │
│  │   (ERC-20)      │◀────────│    (Distribution)       │    │
│  │                 │         │                         │    │
│  │  - Fixed supply │         │  - Holds 40M tokens     │    │
│  │  - Standard ERC │         │  - Owner distributes    │    │
│  │  - No mint/burn │         │  - Tracks recipients    │    │
│  └─────────────────┘         └───────────┬─────────────┘    │
│                                          │                   │
│                                          │ onlyOwner         │
│                                          ▼                   │
│                              ┌─────────────────────────┐    │
│                              │   Multisig Wallet       │    │
│                              │   (Gnosis Safe)         │    │
│                              └─────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Contract: TBPToken.sol

Standard ERC-20 with fixed supply minted at deployment.

```solidity
// Key features:
- Inherits OpenZeppelin ERC20
- Fixed total supply of 100,000,000 tokens
- All tokens minted to deployer on construction
- No additional minting capability
- No burn functionality (optional consideration)
```

### Contract: RewardPool.sol

Holds and distributes reward tokens.

```solidity
// Key features:
- Ownable (controlled by multisig)
- Receives reward allocation on deployment
- distributeReward(address recipient, uint256 amount)
- Emits RewardDistributed event for tracking
- Emergency withdraw for owner
```

## Integration with Referral System

```
┌─────────────────┐     ┌──────────────────┐
│  Discord Bot    │────▶│  Referral API    │
│  /referral      │     │  (SQLite DB)     │
│  /guest_referral│     │                  │
└─────────────────┘     └────────┬─────────┘
                                 │
                                 │ submission
                                 ▼
                        ┌──────────────────┐
                        │  Admin Dashboard │
                        │  (React)         │
                        │                  │
                        │  - Review        │
                        │  - Approve/Reject│
                        │  - Enter wallet  │
                        └────────┬─────────┘
                                 │
                                 │ on approval
                                 ▼
                        ┌──────────────────┐
                        │  Backend API     │
                        │                  │
                        │  - Calculate     │
                        │    reward amount │
                        │  - Call contract │
                        └────────┬─────────┘
                                 │
                                 │ distributeReward()
                                 ▼
                        ┌──────────────────┐
                        │  RewardPool.sol  │
                        │                  │
                        │  - Transfer TBP  │
                        │    to recipient  │
                        └────────┬─────────┘
                                 │
                                 │ ERC-20 transfer
                                 ▼
                        ┌──────────────────┐
                        │  Contributor     │
                        │  Wallet          │
                        └──────────────────┘
```

## User Flow

1. **Submit Referral**
   - User submits via Discord `/referral` or `/guest_referral`
   - Optionally provides Ethereum wallet address

2. **Review Process**
   - Admin reviews in dashboard
   - Approves or rejects with notes

3. **Reward Distribution** (on approval)
   - If wallet provided: tokens sent automatically
   - If no wallet: user prompted to provide one
   - Transaction hash logged for transparency

4. **Notification**
   - User receives Discord DM with:
     - Approval confirmation
     - Token amount received
     - Transaction link (Etherscan)

## Vesting Schedule

### Founding Team (15%)
- 6-month cliff
- 24-month linear vesting
- Prevents immediate dumping

### Early Community (10%)
- Immediate distribution via airdrop
- Snapshot of Discord members at launch date

### Liquidity (10%)
- Locked in Uniswap V3 position
- Full range for maximum accessibility

## Security Considerations

1. **Multisig Control** - RewardPool owned by Gnosis Safe (3/5 signers)
2. **No Minting** - Fixed supply prevents inflation attacks
3. **Audited Contracts** - OpenZeppelin base contracts
4. **Rate Limiting** - Backend limits reward distribution frequency
5. **Wallet Verification** - Contributors verify wallet ownership

## Future Considerations

- [ ] Cross-chain deployment (Base, Arbitrum) for lower gas
- [ ] Staking mechanism for long-term holders
- [ ] NFT badges for top contributors
- [ ] Governance upgrade if community desires
- [ ] Buyback automation via Chainlink Keepers

## Development Roadmap

1. **Phase 1: Contracts**
   - Deploy TBPToken.sol
   - Deploy RewardPool.sol
   - Fund reward pool
   - Set up Uniswap liquidity

2. **Phase 2: Integration**
   - Add wallet field to Discord commands
   - Update dashboard with reward UI
   - Backend integration with contracts

3. **Phase 3: Launch**
   - Community airdrop
   - Announce token
   - Begin reward distributions

## Contract Addresses

### Sepolia Testnet (Deployed)

| Contract | Address |
|----------|---------|
| TBPToken | `0xd28278a05634F76D4226C8C6AF4f3Db661fa9135` |
| RewardPool | `0xd8ceba70d0a6f530523fc393f80fe8be70faac70` |

- [View TBPToken on Etherscan](https://sepolia.etherscan.io/address/0xd28278a05634F76D4226C8C6AF4f3Db661fa9135)
- [View RewardPool on Etherscan](https://sepolia.etherscan.io/address/0xd8ceba70d0a6f530523fc393f80fe8be70faac70)

### Mainnet (Pending)

| Contract | Network | Address |
|----------|---------|---------|
| TBPToken | Mainnet | TBD |
| RewardPool | Mainnet | TBD |
| Uniswap Pool | Mainnet | TBD |
| Team Multisig | Mainnet | TBD |
