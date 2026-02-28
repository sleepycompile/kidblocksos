# KidBlocksOS Payment Layer

Technical documentation for the USDC payment system powering the KidBlocksOS marketplace.

## Overview

The payment layer handles all monetary transactions between KidBlocksOS devices. It uses USDC on Base for settlement and XMTP for coordination. There are no smart contracts - payments are direct ERC-20 transfers between device wallets.

## Wallet Lifecycle

### Creation

During the setup wizard, the OS generates a random Ethereum private key using the system's cryptographic random number generator. This key is stored in the device's local data directory. A wallet address is derived from the key. This is the device's identity for both XMTP messaging and on-chain transactions.

### Funding

Parents fund the wallet by sending USDC on Base to the device's wallet address. The address is displayed in the parent settings screen. The OS shows the balance to kids as "coins" (100 coins = 1 USDC).

### Self-Custody

The private key never leaves the device. There is no cloud backup, no recovery service, and no custodian. If the device is wiped, the key is gone. This is intentional - the amounts involved are small (cents to low dollars) and the simplicity of self-custody outweighs the complexity of key recovery for this use case.

## Transaction Types

### Purchase (Buyer → Seller)

A direct USDC transfer from the buyer's wallet to the seller's wallet. The amount is the listing price. One on-chain transaction on Base.

### Platform Fee (Seller → Fee Wallet)

After a successful sale, the seller's device automatically transfers a percentage of the sale price to the platform fee wallet. One on-chain transaction on Base. This transaction is invisible to both kids - neither the buyer nor the seller sees it in their UI.

### Gas

All transactions require Base L2 gas paid in ETH. The device wallet needs a small ETH balance for gas. Base transaction fees are typically fractions of a cent.

## Parental Guardrails

The OS enforces the following limits before signing any transaction:

| Control | Description | Default |
|---|---|---|
| Daily spend limit | Maximum USDC spent per 24-hour period | 1.00 USDC |
| Auto-approve threshold | Purchases below this amount proceed silently | 0.25 USDC |
| Max listing price | Maximum price a kid can set when selling | 1.00 USDC |

If a transaction exceeds the daily spend limit, it is blocked. If it exceeds the auto-approve threshold, the OS can prompt for parent confirmation (PIN). These controls are configured in the parent settings screen and stored locally.

## On-Chain Verification

When a buyer sends a payment confirmation message containing a transaction hash:

1. The seller's device queries the Base RPC for the transaction receipt
2. It verifies the receipt status is successful
3. It confirms the transfer was to the correct address for the correct amount
4. Only after verification does it send the app delivery message

This prevents spoofed payment confirmations. The verification happens on-chain, not through trust in the XMTP message.

## Fee Structure

The platform fee is a percentage of each sale. It is deducted from the seller's proceeds and sent automatically by the seller's device after delivering the app. The fee is:

- Not shown in the kid-facing UI
- Not itemized in the kid's transaction history
- Handled entirely by the device software after sale completion
- Sent to a fixed platform fee wallet address

The fee sustains platform development and marketplace infrastructure.

## Currency Abstraction

Kids interact with "coins" - a friendly denomination of USDC.

| Coins | USDC | Display |
|---|---|---|
| 1 | $0.01 | 1 coin |
| 5 | $0.05 | 5 coins |
| 10 | $0.10 | 10 coins |
| 100 | $1.00 | 100 coins |

The conversion happens in the UI layer. All on-chain values are in USDC (6 decimal ERC-20). All XMTP messages contain USDC amounts. Only the rendering layer converts to coins for display.

Parents see real USDC values in the settings screen. Kids see coins everywhere else.

## Error Handling

| Scenario | Behavior |
|---|---|
| Insufficient USDC balance | Transaction blocked, kid sees "Not enough coins" |
| Insufficient ETH for gas | Transaction blocked, parent notified |
| Daily limit exceeded | Transaction blocked, kid sees "Come back tomorrow" |
| Payment verification fails | No delivery sent, buyer retains funds |
| Seller offline after payment | Delivery queued - seller's device delivers on next sync |
| Network error | Transaction retried with exponential backoff |

## No Smart Contracts

The payment layer deliberately avoids smart contracts. Reasons:

1. **Simplicity** - direct ERC-20 transfers are the simplest possible on-chain operation
2. **Gas efficiency** - no contract interaction overhead
3. **Auditability** - every transaction is a standard USDC transfer visible on any block explorer
4. **No deployment** - nothing to deploy, verify, or maintain on-chain
5. **Upgradeability** - protocol changes happen in device software updates, not contract migrations

The tradeoff is the lack of on-chain escrow. The XMTP message protocol handles the escrow-like coordination (payment before delivery, verification before release). For the transaction sizes involved (cents), this is an acceptable trust model.
