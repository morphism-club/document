# Morphism Web Platform Documentation

## What is /web?

The Morphism Web Platform is a Next.js-based web application that serves as the frontend interface for the Morphism Club ecosystem. It provides a user-friendly interface for users to explore projects supported by Morphism Club, purchase NFTs, and engage with the community. The platform is designed to showcase NFT fundraising campaigns and facilitate blockchain interactions for NFT purchases.

The web platform is built with modern web technologies including Next.js 13 App Router, React, TypeScript, and TailwindCSS, offering a responsive and feature-rich user experience with both light and dark mode support.

## How /web works?

The Morphism Web Platform operates as a client-side application that interacts with blockchain networks (Ethereum and Optimism) through smart contracts. The platform follows a component-based architecture using React and Next.js, with the following key operational flows:

### Core Operational Flows

1. **Project Showcase**
   - The platform displays featured projects with detailed information
   - Users can browse project details, NFT artwork, and purchase options
   - Project data is presented through dedicated project pages

2. **Wallet Connection**
   - Users connect their Ethereum wallets (MetaMask, WalletConnect, etc.)
   - The platform uses wagmi library to manage wallet connections
   - Connected wallets enable on-chain transactions

3. **NFT Purchase Flow**
   - Users select NFT quantity to purchase
   - Optional referral codes can be applied for discounts
   - Payment options include ETH or USDC
   - The platform handles transaction submission and confirmation

4. **Responsive UI**
   - Adapts to different screen sizes and devices
   - Supports dark mode for improved user experience
   - Provides feedback during transaction processing

### Technical Architecture

The web platform is structured using the Next.js App Router pattern with the following organization:

```
src/
├── app/                # Next.js App Router pages
│   ├── layout.tsx      # Root layout component
│   ├── page.tsx        # Home page
│   └── project/        # Project-specific pages
├── components/         # Reusable UI components
├── data/               # Static data and mock data
├── hooks/              # Custom React hooks
├── lib/                # Utility functions and blockchain interactions
└── types/              # TypeScript type definitions
```

The application uses client-side rendering for dynamic content and server-side rendering for static content, optimizing performance and SEO. State management is handled through React hooks and context, with blockchain interactions managed through specialized hooks.

## Methods

The Morphism Web Platform implements several key methods and functions to facilitate user interactions and blockchain operations:

### Wallet Connection Methods

1. **connectWallet()**
   - Initiates wallet connection dialog
   - Handles connection errors and state management
   - Stores connection state for persistent sessions

2. **disconnectWallet()**
   - Safely disconnects the user's wallet
   - Clears session data and connection state

### NFT Purchase Methods

1. **purchaseWithETH(quantity, referralCode)**
   - Handles ETH-based NFT purchases
   - Validates transaction parameters
   - Submits transaction to the blockchain
   - Monitors transaction status and provides feedback

2. **purchaseWithUSDC(quantity, referralCode)**
   - Manages USDC-based NFT purchases
   - Handles token approval process
   - Executes purchase transaction
   - Tracks transaction status and confirmation

3. **applyReferralCode(code)**
   - Validates referral code format
   - Applies discount to purchase price
   - Updates UI to reflect discount

### UI Interaction Methods

1. **toggleDarkMode()**
   - Switches between light and dark themes
   - Persists user preference

2. **navigateToProject(projectId)**
   - Handles navigation to project detail pages
   - Loads project-specific data

### Blockchain Utility Methods

1. **getGasPrice()**
   - Fetches current gas prices from the network
   - Provides gas price estimates for transactions

2. **formatEthPrice(wei)**
   - Converts Wei values to human-readable ETH amounts
   - Handles decimal formatting and rounding

3. **estimateTransactionCost(gasLimit)**
   - Calculates estimated transaction costs
   - Provides user feedback on expected fees

## Relation with contracts

The Morphism Web Platform interacts directly with the smart contracts defined in the `/contracts` repository, establishing a client-contract relationship. The web interface serves as the user-facing layer that communicates with the blockchain through these contracts.

### Contract Integration Points

1. **FundraisingNFT Contract Integration**
   - The web platform interacts with the FundraisingNFT contract for NFT minting and sales
   - Contract ABIs are imported and used to create contract instances
   - Methods like `mintWithETH()`, `mintWithUSDC()`, and their referral code variants are called from the web interface
   - The platform handles transaction preparation, submission, and monitoring

2. **USDC Token Contract Integration**
   - For USDC purchases, the platform interacts with the ERC20 token contract
   - Handles token approval process before purchase transactions
   - Monitors token balances and allowances

3. **Chainlink Oracle Integration**
   - Indirectly uses price data from Chainlink oracles through the FundraisingNFT contract
   - Displays current ETH/USD prices for purchase calculations

### Data Flow Between Web and Contracts

1. **User Actions → Contract Calls**
   - User interactions in the web UI trigger contract method calls
   - The platform constructs and signs transactions based on user inputs
   - Transactions are submitted to the blockchain through wallet providers

2. **Contract Events → UI Updates**
   - The platform listens for contract events (e.g., Minted events)
   - UI is updated based on event data and transaction confirmations
   - Success/failure states are displayed to users

3. **Contract State → UI Display**
   - The platform queries contract state (e.g., NFT availability, price)
   - Contract data is formatted and displayed in the UI
   - Real-time updates reflect the current state of the blockchain

## Relation with dashboard

The Morphism Web Platform works in conjunction with the Dashboard application, creating a complementary ecosystem where:

1. **Data Generation and Consumption**
   - The web platform generates purchase events through user interactions
   - These events are later processed and displayed in the dashboard
   - The dashboard tracks and manages the referral system that the web platform utilizes

2. **Referral System Integration**
   - Referral codes created and managed in the dashboard are used during purchases on the web platform
   - The web platform applies discounts based on referral codes
   - Purchase events with referral codes are tracked for commission calculations

3. **Project Information Sharing**
   - Both applications share project information and metadata
   - The web platform showcases projects for users to purchase
   - The dashboard provides administrative tools to manage these projects

4. **Complementary User Roles**
   - Web platform serves end-users (NFT buyers)
   - Dashboard serves administrators and referrers
   - Together they create a complete ecosystem for NFT fundraising campaigns

The relationship between the web platform and dashboard creates a full-cycle system where user purchases on the web platform generate data that is then managed and analyzed through the dashboard, while the dashboard's administrative functions configure aspects of the web platform's behavior.
