# Digital Asset Exchange Contract

A comprehensive smart contract for secure peer-to-peer exchange of digital assets on the Stacks blockchain, written in Clarity.

## Overview

This contract enables users to exchange different digital assets through a controlled, fee-based system with admin oversight. It implements a whitelist approach for verified assets and configurable exchange permissions between asset pairs.

## Key Features

- **Asset Verification**: Admin-controlled whitelist of verified digital assets
- **Exchange Fees**: Configurable exchange fees (max 1%) collected as commission
- **Permission Control**: Granular control over which asset pairs can be exchanged
- **Safe Transfers**: Secure wrapper functions for asset transfers with error handling
- **Volume Tracking**: Built-in analytics for exchange volume and commission collection

## Core Functions

### Admin Functions

#### `verify-asset(asset: principal)`
Adds an asset to the verified whitelist. Only callable by contract admin.

#### `add-enabled-asset(asset: principal, exchange-fee-bps: uint)`
Enables a verified asset for exchange with specified fee in basis points (max 100 = 1%).

#### `remove-enabled-asset(asset: principal)`
Disables an asset from being used in exchanges.

#### `set-exchange-permission(source-asset: principal, target-asset: principal, is-permitted: bool)`
Controls whether exchanges are allowed between specific asset pairs.

#### `withdraw-commission(asset: <digital-asset-trait>, quantity: uint)`
Allows admin to withdraw collected exchange fees.

### User Functions

#### `exchange-digital-assets(source-asset: <digital-asset-trait>, target-asset: <digital-asset-trait>, quantity: uint)`
Performs the actual asset exchange between two enabled assets.

**Process:**
1. Validates both assets are enabled and exchange is permitted
2. Calculates commission based on source asset's fee rate
3. Transfers source asset from user to contract
4. Transfers target asset (minus commission) from contract to user
5. Updates volume and commission tracking

### Read-Only Functions

- `is-asset-verified(asset: principal)` - Check if asset is whitelisted
- `is-asset-enabled(asset: principal)` - Check if asset is enabled for exchange
- `get-total-exchange-volume()` - Get total exchange volume
- `get-total-commission-collected()` - Get total commission collected

## Digital Asset Trait

Assets must implement the following trait:
```clarity
(define-trait digital-asset-trait
  (
    (transfer (uint principal principal) (response bool uint))
    (get-balance (principal) (response uint uint))
    (get-decimals () (response uint uint))
    (get-name () (response (string-ascii 32) uint))
    (get-symbol () (response (string-ascii 10) uint))
  )
)
```

## Error Codes

- `u100`: Admin-only function called by non-admin
- `u101`: Insufficient funds for exchange
- `u102`: Invalid quantity (zero or negative)
- `u103`: Exchange not permitted between asset pair
- `u104`: Transfer transaction failed
- `u105`: Invalid or unverified asset

## Usage Example

```clarity
;; Admin enables two assets for exchange
(contract-call? .exchange-contract verify-asset .token-a)
(contract-call? .exchange-contract verify-asset .token-b)
(contract-call? .exchange-contract add-enabled-asset .token-a u50) ;; 0.5% fee
(contract-call? .exchange-contract add-enabled-asset .token-b u25) ;; 0.25% fee
(contract-call? .exchange-contract set-exchange-permission .token-a .token-b true)

;; User exchanges 1000 token-a for token-b
(contract-call? .exchange-contract exchange-digital-assets .token-a .token-b u1000)
```

## Security Features

- Admin-only asset management and permission controls
- Asset verification whitelist prevents unauthorized tokens
- Safe transfer wrappers with comprehensive error handling
- Commission limits (max 1%) prevent excessive fees
- Balance validation before transfers

## Deployment

The contract initializes automatically upon deployment and prints a confirmation message. The deploying address becomes the contract admin with full administrative privileges.
