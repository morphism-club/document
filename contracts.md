# Morphism Contracts Documentation

## What is /contracts?

The Morphism Contracts repository contains the smart contract infrastructure that powers the Morphism Club ecosystem's blockchain functionality. At its core is the FundraisingNFT contract, a specialized ERC721 implementation designed for NFT-based fundraising campaigns. These contracts enable the creation, sale, and management of NFTs with integrated referral and discount systems.

Built using Solidity and the Foundry development framework, the contracts are deployed on the Optimism blockchain, providing cost-effective and efficient transactions while maintaining Ethereum compatibility. The smart contracts serve as the trustless backend for the Morphism ecosystem, handling financial transactions, NFT minting, and referral code processing.

## How /contracts works?

The Morphism Contracts operate as autonomous code on the blockchain, executing predefined logic for NFT sales and fundraising. The system is centered around the FundraisingNFT contract, which extends the OpenZeppelin ERC721 implementation with custom functionality for fundraising campaigns.

### Core Operational Flows

1. **NFT Minting Process**
   - Users can mint NFTs using either ETH or USDC
   - The contract calculates appropriate pricing based on current ETH/USD rates from Chainlink oracles
   - NFTs are minted with sequential token IDs and assigned to the purchaser
   - Optional referral codes can be applied for discounted purchases

2. **Referral System**
   - Contract owner can create referral codes with specific discount rates
   - Users can apply these codes during purchase to receive discounts
   - The contract tracks usage statistics for each referral code
   - Referral codes can be activated or deactivated by the owner

3. **Multi-Currency Support**
   - Native ETH payments are supported with real-time price conversion
   - USDC stablecoin payments are supported through ERC20 token integration
   - Both payment methods can be used with or without referral codes
   - Batch minting is supported for both payment methods

4. **Fund Management**
   - Contract owner can withdraw accumulated ETH and ERC20 tokens
   - Funds are collected in the contract during NFT sales
   - Security measures prevent reentrancy attacks during withdrawals

### Technical Architecture

The contracts are built using a modular approach with inheritance from OpenZeppelin's battle-tested contracts:

```
FundraisingNFT
├── ERC721 (OpenZeppelin)
├── ERC721URIStorage (OpenZeppelin)
├── Ownable (OpenZeppelin)
└── ReentrancyGuard (OpenZeppelin)
```

The system also integrates with external contracts:
- Chainlink Price Feeds for ETH/USD price data
- USDC ERC20 token contract for stablecoin payments

## Methods

The FundraisingNFT contract implements several key methods to facilitate NFT fundraising campaigns:

### Minting Methods

1. **mintWithETH()**
   - Allows users to mint an NFT using ETH
   - Calculates required ETH amount based on current ETH/USD price
   - Verifies sufficient payment and mints the NFT
   - Returns the minted token ID

2. **mintWithUSDC()**
   - Enables minting using USDC stablecoin
   - Verifies USDC allowance and balance
   - Transfers USDC to the contract and mints the NFT
   - Returns the minted token ID

3. **mintWithETHAndReferralCode(string code)**
   - Similar to mintWithETH but applies a discount based on the referral code
   - Validates referral code existence and active status
   - Calculates discounted price and verifies payment
   - Increments usage count for the referral code
   - Returns the minted token ID

4. **mintWithUSDCAndReferralCode(string code)**
   - Similar to mintWithUSDC but applies a discount based on the referral code
   - Validates referral code existence and active status
   - Calculates discounted price and transfers USDC
   - Increments usage count for the referral code
   - Returns the minted token ID

### Batch Minting Methods

1. **batchMintWithETH(uint256 quantity)**
   - Mints multiple NFTs in a single transaction using ETH
   - Calculates total required ETH based on quantity and price
   - Returns an array of minted token IDs

2. **batchMintWithUSDC(uint256 quantity)**
   - Mints multiple NFTs in a single transaction using USDC
   - Calculates total required USDC based on quantity and price
   - Returns an array of minted token IDs

3. **batchMintWithETHAndReferralCode(uint256 quantity, string code)**
   - Batch minting with ETH and referral code discount
   - Returns an array of minted token IDs

4. **batchMintWithUSDCAndReferralCode(uint256 quantity, string code)**
   - Batch minting with USDC and referral code discount
   - Returns an array of minted token IDs

### Referral Code Management Methods

1. **addReferralCode(string code, uint256 discountRate)**
   - Creates a new referral code with specified discount rate
   - Only callable by contract owner
   - Discount rate must be between 1-99%

2. **removeReferralCode(string code)**
   - Deletes an existing referral code
   - Only callable by contract owner

3. **setReferralCodeStatus(string code, bool isActive)**
   - Activates or deactivates a referral code
   - Only callable by contract owner

4. **getReferralCode(string code)**
   - Returns information about a referral code
   - Includes discount rate, usage count, and active status

### Fund Management Methods

1. **withdrawETH()**
   - Withdraws all ETH from the contract to the owner
   - Protected against reentrancy attacks

2. **withdrawERC20(address tokenAddr)**
   - Withdraws all tokens of a specific ERC20 contract
   - Allows retrieval of USDC and any other ERC20 tokens
   - Protected against reentrancy attacks

### Utility Methods

1. **_getLatestEthPrice()**
   - Internal method to fetch ETH/USD price from Chainlink oracle
   - Returns price scaled by 10^8 (Chainlink's standard format)

2. **setBaseURI(string baseTokenURI_)**
   - Updates the base URI for all NFT metadata
   - Only callable by contract owner

## Relation with web

The Morphism Contracts have a direct integration with the web platform, serving as the blockchain backend for the frontend application:

1. **Contract Calls from Web**
   - The web platform imports contract ABIs to interact with deployed contracts
   - User actions on the web platform trigger contract method calls
   - The web platform constructs and submits transactions to the contracts

2. **Data Flow**
   - Contract state (NFT availability, prices) is queried by the web platform
   - Transaction results from contract calls update the web UI
   - Events emitted by contracts (e.g., Minted events) are captured by the web platform

3. **Shared Configuration**
   - Both systems share knowledge of contract addresses and ABIs
   - The web platform must understand contract parameters like basePrice
   - Referral code functionality is exposed through the web interface

4. **Transaction Handling**
   - The web platform handles wallet connections and transaction signing
   - Contracts execute the business logic once transactions are submitted
   - The web platform monitors transaction status and displays results

The contracts provide a trustless execution environment for the business logic, while the web platform provides a user-friendly interface to interact with these contracts.

## Relation with dashboard

The Morphism Contracts interact with the dashboard application in several key ways:

1. **Event Monitoring**
   - The dashboard monitors contract events (particularly Minted events)
   - Purchase events from contract interactions are tracked in the dashboard
   - The dashboard processes these events to calculate commissions and track sales

2. **Referral System Integration**
   - Referral codes created in the dashboard are added to the contract by administrators
   - The contract tracks usage of these referral codes during purchases
   - The dashboard uses this data to calculate commissions for referrers and chiefs

3. **Financial Tracking**
   - The dashboard tracks financial data from contract interactions
   - Purchase amounts, both in ETH and USDC, are recorded and displayed
   - Project funding progress is monitored based on contract activity

4. **Administrative Functions**
   - The dashboard provides interfaces for administrators to manage contract parameters
   - Referral code management (creation, activation, deactivation) is facilitated through the dashboard
   - Fund withdrawal operations may be initiated through the dashboard

The contracts serve as the source of truth for on-chain activities, while the dashboard provides administrative tools and data visualization for these activities. The dashboard essentially acts as a monitoring and management layer on top of the contract infrastructure.
