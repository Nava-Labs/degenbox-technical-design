# Degenbox Technical Design

*Subject to change: This document is not final and may be adjusted based on the supported chains and implementation requirements.*

## Overview
Degen Box is a decentralized platform that combines the concept of mutual funds with the world of memecoins, offering users seamless exposure to various memecoins across multiple chains. This project aims to simplify the complexities of bridging funds across chains while providing a one-stop solution for buying into curated boxes of memecoins. The system leverages an ERC1155-based tokenization approach to represent ownership of these multi-chain token bundles.

## Solution
Degenbox introduces a bundled approach to meme token trading through its "Box" system, allowing users to:
- Purchase exposure to multiple meme tokens across different chains in a single transaction
- Manage their position through ERC1155 token
- Exit their position seamlessly across all included chains

## Box Price Mechanics

### Price Calculation
1. **Initial Box Price**
   - Calculated based on predetermined token quantities
   - Example Box 1 composition:
     - 1,000,000 GIGA
     - 10,000,000 BRETT
     - 500,000 TRUMP
   - Price = Σ(Token Quantity × Current Market Price)

2. **Price Updates**
   - Box prices are updated in real-time based on:
     - Market prices of constituent tokens
     - Network fees across chains
     - LP fees
     - Protocol fees

### Fee Structure
1. **Network Fees**
- Gas fees on source chain
- Cross-chain execution fees
- Destination chain swap fees

2. **Protocol Fees**
- 0.3% swap fee on each token
- Split between:
  - LP rewards (80%)
  - Protocol treasury (20%)

3. **Price Impact**
- Large purchases may experience price impact
- Slippage protection ensures fair execution
- Maximum trade size limits to protect LP pools

## Liquidity Provider (LP) System

### Overview
The Degenbox system relies on liquidity providers (LPs) across different chains to facilitate cross-chain swaps. LPs play a crucial role in maintaining the system's efficiency and providing necessary token liquidity.

### LP Mechanics
1. **Liquidity Provision**
   - LPs deposit USDC into Vault Contracts on their respective chains
   - LPs receive LP tokens representing their share of the liquidity pool
   - Each chain maintains its independent LP pool

2. **Fee Distribution**
   - Swap fees are collected from every buy/sell operation
   - Fees are distributed proportionally to LPs based on their share
   - Fee collection occurs in real-time as swaps are executed

3. **LP Incentives**
   - Base swap fee: 0.3% of transaction volume
   - Fee distribution: 80% to LPs, 20% to protocol
   - Additional incentives during initial liquidity bootstrap period

## System Architecture

### Core Components

1. **Box Definition**
   - Structure containing:
     - Box unique ID (bytes32)
     - Box price in USDC
     - ERC1155 token ID
     - Token list with chain specifications
     - Example format:
       ```
       Box {
           id: bytes32
           price: uint256
           tokenId: uint256
           tokens: [{
               symbol: string
               chain: string
               address: address
           }]
       }
       ```

2. **Smart Contracts**
   
   a. **Router Contract**
   - Entry point for user interactions
   - Handles token swaps to/from USDC
   - Handles ERC1155 minting/burning
   - Interfaces with Vault Contract
   - Key functions:
     - `buyBox(boxId)`
     - `sellBox(boxId)`

   b. **Vault Contract**
   - Manages token custody
   - Locks/unlocks tokens
   - Key functions:
     - `lock(boxId, amount)`
     - `unlock(boxId, amount)`

3. **Degenbox's Relayer Service (DRS)**
   - Cross-chain orchestration system
   - Monitors events across all supported chains
   - Executes cross-chain token swaps
   - Manages vault synchronization

## Transaction Flows

### Buy Flow
1. User initiates transaction with Router Contract on source chain
   - Can pay with any token that can be swapped to USDC
   - Router performs swap to USDC if necessary
   - USDC is locked in Vault Contract
   - User receives ERC1155 token representing box ownership

2. DRS detects buy event
   - Monitors Router Contract events
   - Triggers parallel execution across destination chains
   - Performs token swaps on each chain per box specification with fee procedure
   - Locks acquired tokens in respective Vault Contracts

### Sell Flow
1. User initiates sell with Router Contract on source chain
   - Sends ERC1155 token back to Router

2. DRS detects sell event
   - Monitors Router Contract events
   - Triggers parallel execution across destination chains
   - Executes token swaps back to USDC on respective chains with fee procedure
   - Triggers release of originally locked USDC to user

## Security Considerations

1. **Vault Security**
   - RBAC(Role-based Account control) functionality
   - Emergency pause functionality

2. **Relayer Security**
   - Fallback mechanisms for failed transactions

3. **Price Impact Protection**
   - Maximum slippage parameters
   - Reliable in extreme market conditions
   - Volume-based execution limits?

## Implementation Requirements

1. **Relayer System**
   - High-availability architecture
   - Real-time monitoring
   - Automatic failover capabilities
   - Transaction receipt verification
