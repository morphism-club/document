# Morphism Dashboard Documentation

## What is /dashboard?

The Morphism Dashboard is a web application designed to help project administrators manage NFT fundraising campaigns, track sales, and handle referral commissions. It serves as the administrative interface for the Morphism ecosystem, enabling project managers to:

1. Monitor NFT sales and funding progress for various projects
2. Track purchase events coming from smart contract interactions
3. Manage a two-tier referral system with referrers and chiefs
4. Process commission payments to referrers and chiefs via blockchain transactions
5. View detailed analytics on sales performance and referral effectiveness

The dashboard is built using Next.js with a domain-driven design architecture, PostgreSQL database with Prisma ORM, and integrates with the Optimism blockchain through Etherscan API and wagmi/WalletConnect for blockchain interactions.

## How /dashboard works?

The Morphism Dashboard operates through several interconnected systems:

### 1. Project Management System

The dashboard allows administrators to create and manage NFT projects. Each project is identified by a unique contract address and contains information about funding targets, current funding status, and associated wallet addresses. The system tracks both ETH and USDC funding amounts separately.

### 2. Purchase Event Tracking System

The dashboard connects to the Optimism blockchain via Etherscan API to fetch transaction data related to NFT purchases. It processes these transactions to:
- Identify valid NFT purchase transactions
- Extract purchase details (amount, token type, NFT quantity)
- Detect referral codes used in transactions
- Calculate commissions for referrers and chiefs
- Update project funding totals

Purchase events can be filtered by transaction method (ETH/USDC, with/without referral) and sorted by various criteria (date, amount, NFTs purchased).

### 3. Referral Management System

The dashboard implements a two-tier referral system:
- **Referrers**: Individuals who promote NFT sales using unique referral codes
- **Chiefs**: Higher-level promoters who manage multiple referrers and earn additional commissions

The system automatically:
- Detects referral codes used in purchase transactions
- Creates referrer records if they don't exist
- Calculates appropriate commissions based on configurable rates
- Tracks which commissions have been paid and which are pending

### 4. Payment Processing System

The dashboard facilitates on-chain commission payments to referrers and chiefs:
- Administrators can initiate payments directly from the dashboard
- The system connects to the user's wallet via WalletConnect
- Transactions are executed on the Optimism blockchain
- Payment status is tracked and updated in the database

### 5. Data Flow

The typical data flow in the dashboard is:
1. Contract mint events are monitored on the blockchain
2. Transaction data is fetched and transformed into PurchaseEvent records
3. Referral codes are linked to referrers (creating new referrers if needed)
4. Chief commissions are calculated for referrers with assigned chiefs
5. Project funding amounts are updated
6. Administrators can view and process commission payments

## Methods

The dashboard implements various methods across its service layer to handle different aspects of the application:

### Project Service Methods

- `findAll()`: Retrieves all projects
- `findById(id)`: Retrieves a specific project by ID
- `create(project)`: Creates a new project
- `update(project)`: Updates project information
- `fetchAndSaveProjectInfo(contractAddress, label, fundingTarget, fundingAddress)`: Creates or updates a project with blockchain data
- `updateFundingAmount(projectId, ethAmount, usdcAmount)`: Updates a project's funding amounts

### Purchase Event Service Methods

- `findByProjectId(projectId)`: Retrieves purchase events for a specific project
- `findByRefererId(refererId)`: Retrieves purchase events associated with a referrer
- `findByReferralCode(referralCode)`: Retrieves purchase events using a specific referral code
- `findUnprocessedEvents()`: Retrieves events that haven't been processed for rewards
- `fetchAndSavePurchaseEvents(contractAddress, projectId)`: Fetches blockchain transactions and transforms them to purchase events
- `updateReferrerRewardTransaction(txId, rewardTxId)`: Updates a purchase event with referrer reward transaction information
- `updateChiefRewardTransaction(txId, rewardTxId)`: Updates a purchase event with chief reward transaction information
- `bulkUpdateReferrerRewardTransactions(rewardInfo)`: Updates multiple referrer reward transactions at once
- `bulkUpdateChiefRewardTransactions(rewardInfo)`: Updates multiple chief reward transactions at once

### Referrer Service Methods

- `findAll()`: Retrieves all referrers
- `findById(id)`: Retrieves a specific referrer by ID
- `findByReferralCode(referralCode)`: Retrieves a referrer by their referral code
- `create(referrer)`: Creates a new referrer
- `update(referrer)`: Updates referrer information
- `assignChiefToReferrer(referrerId, chiefId)`: Assigns a chief to a referrer
- `processPurchaseEventsForReferrers()`: Automatically registers referrers based on purchase events

### Chief Service Methods

- `findAll()`: Retrieves all chiefs
- `findById(id)`: Retrieves a specific chief by ID
- `create(chief)`: Creates a new chief
- `update(chief)`: Updates chief information
- `updateCommissionFee(chiefId, newFeeAmount)`: Updates a chief's accumulated commission fee
- `findReferrersByChiefId(chiefId)`: Retrieves all referrers managed by a specific chief

### Etherscan Repository Methods

- `getContractTransactions(contractAddress)`: Fetches transactions for a specific contract
- `parseTransactionToEvent(transaction, projectId)`: Converts blockchain transaction data to a PurchaseEvent
- `isMintTransaction(transaction)`: Determines if a transaction represents an NFT mint

## DB Structure

The Morphism Dashboard uses a PostgreSQL database with the following schema, managed through Prisma ORM:

### Database Schema

#### Project Model
```
model Project {
  id              String          @id
  contractAddress String          @unique
  label           String?
  fundingTarget   Float?
  fundingAddress  String?
  totalFundedETH  Float?
  totalFundedUSDC Float?
  createdAt       DateTime
  updatedAt       DateTime
  purchaseEvents  PurchaseEvent[]
  referrers       Referrer[]
}
```

#### Referrer Model
```
model Referrer {
  id             String          @id @map("refererId")
  referralCode   String          @unique
  refererAddress String
  label          String?
  projectId      String
  discountRate   Float
  commissionRate Float
  isActive       Boolean
  chiefId        String?
  purchaseEvents PurchaseEvent[]
  chief          Chief?          @relation(fields: [chiefId], references: [id])
  project        Project         @relation(fields: [projectId], references: [id])
}
```

#### PurchaseEvent Model
```
model PurchaseEvent {
  txId                String    @id
  timestamp           DateTime
  projectId           String
  method              String
  tokenType           String
  tokenAmount         Float
  purchasedNFTAmount  Int
  referralCode        String?
  refererId           String?
  projectFundedAmount Float
  referralCommission  Float?
  rewardForReferrer   Boolean   @default(false)
  rewardForReferrerTxId String?
  chiefCommission     Float?
  rewardForChief      Boolean   @default(false)
  rewardForChiefTxId  String?
  project             Project   @relation(fields: [projectId], references: [id])
  referrer            Referrer? @relation(fields: [refererId], references: [id])
}
```

#### Chief Model
```
model Chief {
  id                  String     @id
  label               String?
  chiefCommissionRate Float
  chiefCommissionFee  Float
  isActive            Boolean
  referrerIds         String[]
  createdAt           DateTime
  updatedAt           DateTime
  chiefAddress        String?
  referrers           Referrer[]
}
```

### Data Relationships

The database schema implements several key relationships:

1. **Project to PurchaseEvent (One-to-Many)**
   - A Project can have multiple PurchaseEvents
   - Each PurchaseEvent belongs to exactly one Project
   - Relationship is established through the `projectId` foreign key in PurchaseEvent

2. **Project to Referrer (One-to-Many)**
   - A Project can have multiple Referrers
   - Each Referrer belongs to exactly one Project
   - Relationship is established through the `projectId` foreign key in Referrer

3. **Referrer to PurchaseEvent (One-to-Many)**
   - A Referrer can be associated with multiple PurchaseEvents
   - Each PurchaseEvent can optionally be associated with one Referrer
   - Relationship is established through the `refererId` foreign key in PurchaseEvent

4. **Chief to Referrer (One-to-Many)**
   - A Chief can manage multiple Referrers
   - Each Referrer can optionally be associated with one Chief
   - Relationship is established through the `chiefId` foreign key in Referrer

### Data Flow Diagram

```
Project <---> PurchaseEvent <---> Referrer <---> Chief
   |              |                  |
   |              |                  |
   v              v                  v
Funding      Transaction        Commission
 Targets        Data             Payments
```

### Key Database Fields and Their Significance

1. **Project Model**
   - `contractAddress`: Ethereum address of the NFT contract; serves as the unique identifier for projects
   - `fundingTarget`: Target amount to be raised through NFT sales (in USD)
   - `fundingAddress`: Wallet address where project funds are sent
   - `totalFundedETH` and `totalFundedUSDC`: Running totals of funds raised in each currency

2. **Referrer Model**
   - `referralCode`: Unique code used in purchase transactions to attribute sales
   - `refererAddress`: Ethereum address where commissions are sent
   - `discountRate`: Percentage discount offered to buyers using this referral code
   - `commissionRate`: Percentage of purchase amount paid to referrer as commission
   - `chiefId`: Optional link to a chief who manages this referrer

3. **PurchaseEvent Model**
   - `txId`: Transaction hash from the blockchain; serves as the primary key
   - `method`: Contract method called (e.g., mintWithETH, mintWithUSDCAndReferralCode)
   - `tokenType`: Currency used for purchase (ETH or USDC)
   - `tokenAmount`: Amount of currency paid
   - `purchasedNFTAmount`: Number of NFTs purchased in this transaction
   - `projectFundedAmount`: Amount that goes to the project after commissions
   - `referralCommission`: Amount paid to referrer as commission
   - `chiefCommission`: Amount paid to chief as commission
   - `rewardForReferrer` and `rewardForChief`: Boolean flags indicating if commissions have been paid
   - `rewardForReferrerTxId` and `rewardForChiefTxId`: Transaction hashes of commission payments

4. **Chief Model**
   - `chiefCommissionRate`: Percentage of purchase amount paid to chief as commission
   - `chiefCommissionFee`: Accumulated commission amount for the chief
   - `chiefAddress`: Ethereum address where commissions are sent
   - `referrerIds`: Array of referrer IDs managed by this chief

## Relation with /contracts

The Morphism Dashboard interacts with the smart contracts in the following ways:

1. **Contract Monitoring**
   - The dashboard monitors NFT contract transactions on the Optimism blockchain
   - It uses the Etherscan API to fetch transaction data for specific contract addresses
   - Transaction data is parsed to identify mint events and extract relevant information

2. **Purchase Event Processing**
   - The dashboard identifies different purchase methods based on contract function calls:
     - `mintWithETH`: Standard ETH purchases without referral
     - `mintWithUSDC`: Standard USDC purchases without referral
     - `mintWithETHAndReferralCode`: ETH purchases with referral code
     - `mintWithUSDCAndReferralCode`: USDC purchases with referral code
     - Batch versions of all the above methods for multiple NFT purchases

3. **Commission Payment Processing**
   - The dashboard initiates on-chain transactions to pay commissions to referrers and chiefs
   - It connects to the user's wallet via WalletConnect to sign and send transactions
   - Transaction hashes are stored to track payment status

4. **Contract Integration Points**
   - The dashboard expects the NFT contracts to implement specific methods for minting with and without referral codes
   - It assumes contracts emit events or have transaction data that can be parsed to extract purchase details
   - The commission payment system assumes referrers and chiefs have valid Ethereum addresses to receive payments

5. **Configuration Parameters**
   - Commission rates and discount rates are configurable in the dashboard and must align with contract implementations
   - The dashboard uses environment variables to set default values:
     - `NEXT_PUBLIC_REFERRER_COMMISSION_RATE`: Default commission rate for referrers (typically 15%)
     - `NEXT_PUBLIC_REFERRER_DISCOUNT_RATE`: Default discount rate for buyers using referral codes (typically 5%)
     - `NEXT_PUBLIC_CHIEF_FEE_RATIO`: Default commission rate for chiefs (typically 5%)

The dashboard is designed to work with NFT contracts that implement the expected minting functions and handle referral codes appropriately. The contract must properly allocate funds between the project, referrers, and chiefs according to the commission structure defined in the dashboard.
