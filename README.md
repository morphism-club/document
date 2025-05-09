# Morphism Documentation

This repository contains comprehensive documentation for the Morphism Club ecosystem, including:

- [Dashboard](dashboard.md) - Documentation for the administrative dashboard
- [Web](web.md) - Documentation for the web platform
- [Contracts](contracts.md) - Documentation for the smart contracts

## Overview

Morphism Club is a DAO dedicated to scaling personal autonomy through breakthrough technologies. The ecosystem consists of several interconnected components that work together to enable NFT-based fundraising campaigns with referral systems.

The documentation in this repository provides detailed information about each component, including their purpose, functionality, and interactions with other components.

## System Overview

### Dashboard System

The Morphism Dashboard is an administrative web application that enables project managers to monitor NFT sales, track fundraising progress, and manage referral relationships. Built with Next.js and TypeScript, it implements a domain-driven design architecture with a PostgreSQL database managed through Prisma ORM.

Key features:
- Project management for NFT fundraising campaigns
- Purchase event tracking from blockchain transactions
- Two-tier referral system management (referrers and chiefs)
- Commission calculation and payment processing
- Funding progress visualization and reporting

The dashboard serves as the administrative backend for the entire Morphism ecosystem, providing tools for monitoring on-chain activities and managing off-chain relationships.

### Web Platform System

The Morphism Web Platform is a Next.js-based frontend application that serves as the user-facing interface for the Morphism Club ecosystem. It provides a responsive and feature-rich experience for users to explore projects, purchase NFTs, and engage with the community.

Key features:
- Project showcase with detailed information
- Wallet connection for blockchain interactions
- NFT purchase flow with multi-currency support
- Referral code application for discounted purchases
- Transaction monitoring and confirmation

The web platform is designed to be user-friendly and accessible, with support for both light and dark modes and responsive design for various devices.

### Smart Contracts System

The Morphism Contracts are Solidity-based smart contracts deployed on the Optimism blockchain, providing the trustless backend for the Morphism ecosystem. At the core is the FundraisingNFT contract, a specialized ERC721 implementation for NFT-based fundraising.

Key features:
- NFT minting with ETH and USDC payment options
- Referral system with discount codes
- Multi-currency support with Chainlink oracle integration
- Batch minting capabilities
- Secure fund management and withdrawal

The contracts ensure transparent and secure execution of business logic on the blockchain, handling financial transactions and NFT ownership records.

## System Integration Diagram

```
┌─────────────────────┐      ┌─────────────────────┐      ┌─────────────────────┐
│                     │      │                     │      │                     │
│  Morphism Dashboard │◄────►│  Morphism Contracts │◄────►│   Morphism Web      │
│  (Admin Interface)  │      │  (Blockchain Logic) │      │  (User Interface)   │
│                     │      │                     │      │                     │
└─────────────────────┘      └─────────────────────┘      └─────────────────────┘
         ▲                             ▲                            ▲
         │                             │                            │
         │                             │                            │
         │                             ▼                            │
         │                    ┌─────────────────────┐              │
         │                    │                     │              │
         └────────────────────┤   Blockchain        ├──────────────┘
                              │   (Optimism)        │
                              │                     │
                              └─────────────────────┘
                                        ▲
                                        │
                                        ▼
                              ┌─────────────────────┐
                              │                     │
                              │   External APIs     │
                              │   (Chainlink, etc.) │
                              │                     │
                              └─────────────────────┘
```

## Sequence Diagrams

### NFT Purchase Flow

```
┌─────────┐          ┌─────────┐          ┌───────────┐          ┌────────────┐
│  User   │          │   Web   │          │ Contracts │          │ Dashboard  │
└────┬────┘          └────┬────┘          └─────┬─────┘          └──────┬─────┘
     │                    │                     │                       │
     │ Connect Wallet     │                     │                       │
     │ ─────────────────► │                     │                       │
     │                    │                     │                       │
     │ Select NFT         │                     │                       │
     │ ─────────────────► │                     │                       │
     │                    │                     │                       │
     │ Apply Referral Code│                     │                       │
     │ ─────────────────► │                     │                       │
     │                    │ Validate Code       │                       │
     │                    │ ──────────────────► │                       │
     │                    │                     │                       │
     │                    │ ◄────────────────── │                       │
     │                    │                     │                       │
     │ Confirm Purchase   │                     │                       │
     │ ─────────────────► │                     │                       │
     │                    │ mintWithETHAndCode()│                       │
     │                    │ ──────────────────► │                       │
     │                    │                     │                       │
     │                    │ ◄────────────────── │                       │
     │                    │                     │                       │
     │ Display Success    │                     │                       │
     │ ◄───────────────── │                     │                       │
     │                    │                     │                       │
     │                    │                     │ Emit Minted Event     │
     │                    │                     │ ──────────────────────┤
     │                    │                     │                       │
     │                    │                     │                       │ Process Purchase Event
     │                    │                     │                       │ ────────────────────
     │                    │                     │                       │
     │                    │                     │                       │ Calculate Commissions
     │                    │                     │                       │ ────────────────────
     │                    │                     │                       │
```

### Commission Payment Flow

```
┌────────────┐          ┌───────────┐          ┌────────────┐
│   Admin    │          │ Dashboard │          │ Blockchain │
└──────┬─────┘          └─────┬─────┘          └──────┬─────┘
       │                      │                       │
       │ View Purchase Events │                       │
       │ ────────────────────►│                       │
       │                      │                       │
       │ Select Event for     │                       │
       │ Commission Payment   │                       │
       │ ────────────────────►│                       │
       │                      │                       │
       │                      │ Calculate Commission  │
       │                      │ ─────────────────────┐│
       │                      │◄─────────────────────┘│
       │                      │                       │
       │ Confirm Payment      │                       │
       │ ────────────────────►│                       │
       │                      │                       │
       │                      │ Send ETH Transaction  │
       │                      │ ──────────────────────►
       │                      │                       │
       │                      │ ◄──────────────────── │
       │                      │                       │
       │                      │ Update Payment Status │
       │                      │ ─────────────────────┐│
       │                      │◄─────────────────────┘│
       │                      │                       │
       │ Display Success      │                       │
       │ ◄────────────────────│                       │
       │                      │                       │
```

### Referral Code Creation Flow

```
┌────────────┐          ┌───────────┐          ┌───────────┐
│   Admin    │          │ Dashboard │          │ Contracts │
└──────┬─────┘          └─────┬─────┘          └─────┬─────┘
       │                      │                      │
       │ Create Referral Code │                      │
       │ ────────────────────►│                      │
       │                      │                      │
       │                      │ Store in Database    │
       │                      │ ────────────────────┐│
       │                      │◄────────────────────┘│
       │                      │                      │
       │                      │ addReferralCode()    │
       │                      │ ─────────────────────►
       │                      │                      │
       │                      │ ◄───────────────────┐│
       │                      │                     ││
       │                      │                     ││
       │ Display Success      │                     ││
       │ ◄────────────────────│                     ││
       │                      │                     ││
```
