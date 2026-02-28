# KidBlocksOS Marketplace

A peer-to-peer app marketplace where kids list, browse, and buy HTML5 apps from each other. All communication happens over XMTP. All payments settle in USDC on Base.

## Overview

The marketplace is a single XMTP group. Every KidBlocksOS device joins this group during setup. Listings, purchases, invoices, and app deliveries are all JSON messages sent to the group. There are no servers, no APIs, and no databases. The XMTP group is the database.

## Architecture

```
┌──────────────┐         ┌──────────────┐
│  Device A    │         │  Device B    │
│  (Seller)    │  XMTP   │  (Buyer)     │
│              │◄───────►│              │
│  Wallet A    │  Group  │  Wallet B    │
└──────┬───────┘         └──────┬───────┘
       │                        │
       │    USDC on Base        │
       │◄───────────────────────┘
       │    (peer to peer)
```

- Each device has its own Ethereum wallet (created during setup, key stays on device)
- All devices are members of one XMTP marketplace group
- Listings are broadcast as JSON messages to the group
- Payments are direct USDC transfers between wallets on Base
- App files are delivered as XMTP messages after payment verification

## Device Onboarding

1. During the setup wizard, the OS generates a fresh Ethereum wallet on the device
2. The private key is stored locally and never leaves the device
3. The device sends a DM to the marketplace master wallet requesting group membership
4. A watcher daemon running on the master wallet receives the DM and adds the device to the XMTP marketplace group
5. The device syncs existing messages from the group and begins listening for new ones

```
Device                    Master Wallet
  │                            │
  │  DM: join_marketplace      │
  │───────────────────────────►│
  │                            │  addMembers(device)
  │  DM: join_accepted         │
  │◄───────────────────────────│
  │                            │
  │  (now a group member)      │
  │  sync existing messages    │
  │  start real-time stream    │
```

No accounts. No sign-up forms. No emails. A single DM and the device is in.

## Message Protocol

All marketplace communication uses structured JSON messages sent to the XMTP group.

### Message Types

| Type | Direction | Purpose |
|------|-----------|---------|
| `listing` | Seller → Group | Broadcast a new app for sale |
| `delist` | Seller → Group | Remove a listing |
| `payment_request` | Buyer → Group | Request to purchase an app |
| `payment_invoice` | Seller → Group | Respond with payment details |
| `payment_confirmation` | Buyer → Group | Confirm on-chain payment with tx hash |
| `delivery` | Seller → Group | Deliver the HTML5 app file |

### Listing

When a kid lists an app for sale, the device broadcasts a listing message containing:

- App name and description
- Price (in USDC, displayed to kids as coins)
- Studio category (games, stories, music, art, science)
- Seller name and avatar
- Seller wallet address

Every device in the group receives the message and renders it as a marketplace card. No indexing service required.

### Delisting

A delist message with the listing ID. All devices remove it from their local state.

## Payment Flow

The full purchase lifecycle uses six XMTP messages and two on-chain transactions.

```
Buyer Device              XMTP Group              Seller Device
     │                        │                        │
     │  1. payment_request    │                        │
     │───────────────────────►│───────────────────────►│
     │                        │                        │
     │                        │  2. payment_invoice    │
     │◄───────────────────────│◄───────────────────────│
     │                        │                        │
     │  3. USDC transfer (on-chain, Base)              │
     │─────────────────────────────────────────────────►│
     │                        │                        │
     │  4. payment_confirmation                        │
     │───────────────────────►│───────────────────────►│
     │                        │                        │
     │                        │  (verify tx on-chain)  │
     │                        │                        │
     │                        │  5. delivery (HTML5)   │
     │◄───────────────────────│◄───────────────────────│
     │                        │                        │
     │                        │  6. platform fee (on-chain, Base)
     │                        │         ──────────────►│ Fee Wallet
```

### Step by Step

1. **Payment Request** - Buyer's device sends a message indicating intent to buy, including listing ID and buyer wallet address
2. **Payment Invoice** - Seller's device recognizes its own listing and responds with the exact amount and seller wallet address
3. **USDC Transfer** - Buyer's device sends USDC directly to the seller's wallet. One on-chain transaction on Base. No smart contracts. No escrow. Peer to peer.
4. **Payment Confirmation** - Buyer broadcasts the transaction hash to the group
5. **Delivery** - Seller's device verifies the transaction on-chain, then sends the complete HTML5 app file as an XMTP message. The buyer's device saves it to their projects folder.
6. **Platform Fee** - Seller's device automatically sends a percentage to the platform fee wallet. One additional on-chain transaction.

Two on-chain transactions total: buyer pays seller, seller pays platform fee. Everything else is messaging.

## Kid-Facing Abstraction

Kids never see USDC, wallet addresses, transaction hashes, or gas fees.

| What the kid sees | What's actually happening |
|---|---|
| Gold coins in a piggy bank | USDC balance in a self-custody wallet on Base |
| App priced at 10 coins | 0.10 USDC listing broadcast over XMTP |
| Tap "Buy" | USDC transfer + XMTP payment protocol |
| App appears on home screen | HTML5 file delivered via XMTP message |
| "Earned 10 coins" | Received 0.10 USDC from peer |
| Sell for 5 coins | List at 0.05 USDC on the XMTP marketplace group |

The conversion is 100 coins = 1 USDC. Parents configure real dollar limits in the settings screen. Kids interact with coins.

## Parental Controls

Parents configure the following during setup or in settings:

- **Spend limit** - Maximum USDC a device can spend per day
- **Auto-approve threshold** - Transactions below this amount proceed without confirmation
- **Max listing price** - Cap on how much a kid can charge for an app
- **Wallet visibility** - Kids see coins, parents see real balances in settings

All limits are enforced at the OS level before any on-chain transaction is signed.

## Discovery

There is no search engine, no recommendation algorithm, and no featured section. The marketplace group contains every listing ever broadcast. When a device comes online, it syncs the most recent messages from the group and rebuilds its local listing state. New listings arrive in real time through the XMTP stream.

Listings are categorized by studio type (games, stories, music, art, science) for tab-based filtering. That's the extent of discovery. Simple, flat, and entirely decentralized.

## App Delivery

Apps are single self-contained HTML5 files. They are small enough to travel as XMTP message payloads. After payment verification, the seller's device sends the complete file to the group. The buyer's device extracts it and saves it to the local projects directory. The app immediately appears on the home screen.

No download servers. No CDN. No file hosting. The messaging layer is the delivery layer.

## Security

- Private keys are generated on-device and never transmitted
- All XMTP communication is end-to-end encrypted
- Payment verification happens on-chain (transaction receipt check)
- Parental spend limits are enforced before wallet signing
- The watcher daemon only adds devices that send a properly formatted join request
- No master key exists on any child device

## Stack

| Layer | Technology |
|---|---|
| Communication | XMTP (production network) |
| Payments | USDC on Base |
| Wallets | ethers.js (local key, self-custody) |
| App format | Self-contained HTML5 |
| Hardware | Raspberry Pi |
| OS | KidBlocksOS (Electron) |
