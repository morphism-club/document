---
layout: home
title: Morphism Documentation
---

# Morphism Documentation

Welcome to the comprehensive documentation for the Morphism Club ecosystem.

## Components

- [Dashboard](dashboard.html) - Documentation for the administrative dashboard
- [Web](web.html) - Documentation for the web platform
- [Contracts](contracts.html) - Documentation for the smart contracts

## Overview

Morphism Club is a DAO dedicated to scaling personal autonomy through breakthrough technologies. The ecosystem consists of several interconnected components that work together to enable NFT-based fundraising campaigns with referral systems.

{% include system_diagram.html %}

## System Overview

### Dashboard System

The Morphism Dashboard is an administrative web application that enables project managers to monitor NFT sales, track fundraising progress, and manage referral relationships. Built with Next.js and TypeScript, it implements a domain-driven design architecture with a PostgreSQL database managed through Prisma ORM.

### Web Platform System

The Morphism Web Platform is a Next.js-based frontend application that serves as the user-facing interface for the Morphism Club ecosystem. It provides a responsive and feature-rich experience for users to explore projects, purchase NFTs, and engage with the community.

### Smart Contracts System

The Morphism Contracts are Solidity-based smart contracts deployed on the Optimism blockchain, providing the trustless backend for the Morphism ecosystem. At the core is the FundraisingNFT contract, a specialized ERC721 implementation for NFT-based fundraising.
