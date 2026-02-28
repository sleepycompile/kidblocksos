# KidBlocksOS XMTP Messaging Layer

Technical documentation for the XMTP-based communication system powering the KidBlocksOS marketplace.

## Overview

XMTP is the sole communication layer for the KidBlocksOS marketplace. It handles device onboarding, listing discovery, purchase negotiation, payment coordination, and app delivery. There are no HTTP APIs, no WebSocket servers, and no databases. XMTP replaces all of them.

## Why XMTP

- **No servers to run** - messages route through the decentralized XMTP network
- **End-to-end encrypted** - all marketplace communication is private
- **Wallet-native identity** - each device's Ethereum wallet is its XMTP identity
- **Group messaging** - one group serves as the entire marketplace
- **Offline delivery** - messages queue and deliver when devices come online
- **File-sized payloads** - HTML5 apps are small enough to send as messages

## Network Topology

```
                    XMTP Network (Production)
                           │
              ┌────────────┼────────────┐
              │            │            │
         ┌────┴───┐  ┌────┴───┐  ┌────┴───┐
         │Device A│  │Device B│  │Device C│  ...
         └────────┘  └────────┘  └────────┘
              │            │            │
              └────────────┼────────────┘
                           │
                  Marketplace Group
                  (single XMTP group)
```

All devices are members of one XMTP group. This group is the marketplace. Every listing, purchase, and delivery flows through it.

## Device Identity

Each KidBlocksOS device has one Ethereum wallet. This wallet serves as:

1. **XMTP identity** - used to authenticate with the XMTP network
2. **Payment wallet** - holds USDC on Base
3. **Marketplace identity** - seller/buyer address in listings and transactions

The XMTP client is initialized with a signer derived from the device wallet. The encryption key for the local XMTP database is deterministically derived from the wallet key. This means the same wallet always produces the same XMTP client state.

## Group Management

### The Marketplace Group

There is one XMTP group that serves as the global marketplace. It is created and owned by a master wallet. The group ID is hardcoded into the OS.

### Join Protocol

Devices cannot self-add to an XMTP group. A watcher daemon running on the master wallet handles membership:

1. New device creates its wallet during setup
2. Device creates a DM conversation with the master wallet
3. Device sends a structured join request containing its wallet address
4. Watcher daemon receives the DM, resolves the device's XMTP inbox ID
5. Watcher adds the device to the marketplace group
6. Watcher replies with a confirmation containing the group ID
7. Device syncs the group and begins listening

The join is automatic and immediate. No human approval. No account creation.

### Member Resolution

XMTP groups require inbox IDs (not wallet addresses) for membership operations. The watcher resolves wallet addresses to inbox IDs before adding members. This is a one-time lookup per device.

## Message Format

All marketplace messages are JSON objects sent as text content to the XMTP group. Every message has a `type` field that determines how it is processed.

### Listing Message

Broadcast when a kid lists an app for sale.

```json
{
  "type": "listing",
  "id": "<unique listing id>",
  "name": "Penguin Ice Slider",
  "description": "Help the penguin collect stars",
  "price": "0.10",
  "studio": "games",
  "sellerName": "Alex",
  "sellerAddress": "<wallet address>",
  "html": "<complete HTML5 app>",
  "htmlHash": "<sha256 of html content>",
  "timestamp": 1709150400000
}

```

The HTML content travels with the listing. When a buyer purchases, the seller re-sends it in a delivery message.

### Delist Message

Removes a listing from all devices.

```json
{
  "type": "delist",
  "id": "<listing id>",
  "timestamp": 1709150400000
}
```

### Payment Request

Buyer initiates a purchase.

```json
{
  "type": "payment_request",
  "listingId": "<listing id>",
  "buyerAddress": "<buyer wallet address>",
  "timestamp": 1709150400000
}
```

### Payment Invoice

Seller responds with payment details.

```json
{
  "type": "payment_invoice",
  "listingId": "<listing id>",
  "sellerAddress": "<seller wallet address>",
  "amount": "0.10",
  "buyerAddress": "<buyer wallet address>",
  "timestamp": 1709150400000
}
```

### Payment Confirmation

Buyer confirms on-chain payment.

```json
{
  "type": "payment_confirmation",
  "listingId": "<listing id>",
  "txHash": "<Base transaction hash>",
  "buyerAddress": "<buyer wallet address>",
  "amount": "0.10",
  "timestamp": 1709150400000
}
```

### Delivery

Seller delivers the app after verifying payment.

```json
{
  "type": "delivery",
  "listingId": "<listing id>",
  "buyerAddress": "<buyer wallet address>",
  "html": "<complete HTML5 app>",
  "htmlHash": "<sha256 of html content>",
  "name": "Penguin Ice Slider",
  "studio": "games",
  "timestamp": 1709150400000
}
```

## Message Processing

Each device runs a message handler that:

1. Ignores messages sent by itself
2. Parses JSON from the message text content
3. Routes to a handler based on the `type` field
4. Updates local state (listings map, pending purchases, transaction log)
5. Persists state to a local cache file

Messages that fail to parse as JSON are silently ignored. This allows the group to contain non-marketplace messages without breaking anything.

## State Synchronization

### Initial Sync

When a device comes online or opens the marketplace, it:

1. Syncs the XMTP group conversation
2. Reads the most recent messages from the group
3. Processes each message through the standard handler
4. Rebuilds the listings map from the message history

This means a device that was offline for days will catch up on all listings when it reconnects.

### Real-Time Stream

After initial sync, the device opens a streaming connection to the group. New messages are processed immediately as they arrive. This provides real-time listing updates and purchase notifications.

### Local Cache

Processed listings are cached to a local JSON file. This allows the marketplace UI to render instantly on open, before the XMTP sync completes. The cache is updated after each sync.

## Offline Behavior

- **Seller goes offline after listing** - The listing message is already in the group. Other devices see it. If someone buys and the seller is offline, the payment confirmation message queues. When the seller comes back online and syncs, it processes the confirmation and sends delivery.

- **Buyer goes offline after payment** - The delivery message queues in the group. When the buyer comes back online, it syncs and receives the app.

- **Both offline** - Messages persist in the XMTP network. Both devices catch up on next sync.

## Security Considerations

- All XMTP messages are end-to-end encrypted
- Only group members can read marketplace messages
- Message authenticity is guaranteed by XMTP (sender inbox ID is verified)
- Payment verification happens on-chain, not through message trust
- The watcher daemon only processes properly formatted join requests
- HTML content hashes allow integrity verification of delivered apps

## Limitations

- **Group size** - XMTP groups have practical limits on member count. For early marketplace usage this is not a constraint. At scale, sharding into regional groups or topic groups may be needed.
- **Message size** - Large HTML5 apps may approach XMTP message size limits. Apps generated by KidBlocksOS are typically small (under 100KB) so this is not currently an issue.
- **Message history** - Devices sync a finite number of recent messages. Very old listings may not appear on new devices. A periodic re-listing or pinning mechanism could address this.
- **Privacy** - All group members can see all listings and purchase messages. Wallet addresses are visible to group members. For a kids marketplace this is acceptable - there is no sensitive data in listing messages.
