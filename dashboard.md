# Morphism Dashboard Documentation

## What is /dashboard?

The Morphism Dashboard is a web application designed for project administrators to manage NFT fundraising campaigns, track sales, and handle referral commissions. It serves as the administrative backend for the Morphism ecosystem, providing tools to monitor NFT sales, process commission payments, and manage referral relationships.

Built with Next.js and TypeScript, the dashboard implements a domain-driven design architecture with a PostgreSQL database managed through Prisma ORM. It connects to blockchain networks to fetch transaction data and process on-chain commission payments for referrers and chiefs in a two-tier referral system.

## How /dashboard works?

The Morphism Dashboard operates through several interconnected systems that work together to provide comprehensive campaign management functionality:

### Core Operational Flows

1. **Project Management**
   - Administrators can create and configure NFT fundraising projects
   - Each project is associated with a smart contract address
   - The dashboard tracks funding progress and sales metrics for each project

2. **Purchase Event Tracking**
   - The system monitors blockchain transactions related to project contracts
   - Purchase events are fetched from the blockchain and stored in the database
   - Events are processed to calculate commissions and track sales

3. **Referral Management**
   - The dashboard implements a two-tier referral system with referrers and chiefs
   - Referrers promote projects and earn commissions on sales
   - Chiefs manage groups of referrers and earn additional commissions

4. **Commission Processing**
   - The system calculates commissions based on purchase amounts and referral relationships
   - Administrators can process commission payments through blockchain transactions
   - Payment status is tracked and displayed in the dashboard

### Technical Architecture

The dashboard follows a domain-driven design pattern with clear separation of concerns:

```
src/
├── app/                    # Next.js app router structure
│   ├── api/                # API routes for backend functionality
│   ├── backend/            # Backend service and domain logic
│   │   ├── domain/         # Domain models (Project, Referrer, PurchaseEvent)
│   │   ├── infrastructure/ # Repository implementations
│   │   └── service/        # Business logic services
│   ├── component/          # React UI components
│   ├── hooks/              # Custom React hooks
│   └── page.tsx            # Home page (project dashboard)
└── wagmi/                  # Web3/blockchain integration
```

The application uses server-side rendering for initial page loads and client-side rendering for dynamic content. API routes handle data operations, while React components and hooks manage the user interface and state.

## Methods

The Morphism Dashboard implements several key methods and services to facilitate NFT fundraising campaign management:

### Project Management Methods

1. **fetchAndSaveProjectInfo(contractAddress, label, fundingTarget, fundingAddress)**
   - Creates or updates project information in the database
   - Associates the project with a smart contract address
   - Sets funding targets and recipient addresses

2. **updateFundingAmount(projectId, ethAmount, usdcAmount)**
   - Updates a project's current funding amounts
   - Tracks progress toward funding targets
   - Supports both ETH and USDC currencies

### Purchase Event Methods

1. **fetchAndSavePurchaseEvents(contractAddress, projectId)**
   - Retrieves transaction data from the blockchain
   - Filters for NFT purchase transactions
   - Creates purchase event records in the database
   - Extracts referral information from transactions

2. **updateReferrerRewardTransaction(txId, rewardTxId)**
   - Records commission payment transaction IDs
   - Updates purchase event status after commission processing
   - Maintains audit trail of payments

### Referral Management Methods

1. **assignChiefToReferrer(referrerId, chiefId)**
   - Establishes the relationship between referrers and chiefs
   - Updates commission distribution rules
   - Maintains the two-tier referral structure

2. **processPurchaseEventsForReferrers()**
   - Automatically registers referrers based on purchase events
   - Creates referrer records for new referral codes
   - Maintains consistency between on-chain and off-chain data

### Commission Processing Methods

1. **calculateReferrerCommission(purchaseAmount, referralCode)**
   - Determines commission amounts based on purchase value
   - Applies commission rate rules
   - Handles different currency types (ETH/USDC)

2. **processReferrerPayment(purchaseEventId, referrerId)**
   - Initiates blockchain transactions for commission payments
   - Validates payment eligibility
   - Updates payment status in the database

3. **processChiefPayment(purchaseEventId, chiefId)**
   - Calculates and processes chief-level commissions
   - Validates chief-referrer relationships
   - Executes blockchain transactions for payments

## DB Structure

The Morphism Dashboard uses a PostgreSQL database with Prisma ORM to manage data persistence. The database schema defines several interconnected models that represent the core domain entities:

### Core Models

1. **Project**
   - Represents a fundraising campaign
   - Contains contract address, funding targets, and current status
   - Linked to purchase events through a one-to-many relationship

2. **Referrer**
   - Represents a promoter with a unique referral code
   - Contains wallet address and commission rate information
   - May be associated with a chief through a many-to-one relationship

3. **Chief**
   - Represents a higher-level referrer who manages other referrers
   - Contains wallet address and commission rate information
   - Linked to referrers through a one-to-many relationship

4. **PurchaseEvent**
   - Represents an NFT purchase transaction
   - Contains transaction details, amounts, and referral information
   - Linked to projects, referrers, and chiefs through relationships

### Database Schema

```prisma
model Project {
  id                String          @id @default(uuid())
  contractAddress   String          @unique
  label             String
  fundingTarget     Decimal         @db.Decimal(78, 0)
  fundingAddress    String
  fundingAmount     Decimal         @default(0) @db.Decimal(78, 0)
  usdcFundingAmount Decimal         @default(0) @db.Decimal(78, 0)
  createdAt         DateTime        @default(now())
  updatedAt         DateTime        @updatedAt
  purchaseEvents    PurchaseEvent[]
}

model Referrer {
  id            String          @id @default(uuid())
  referralCode  String          @unique
  walletAddress String
  discountRate  Int
  chiefId       String?
  chief         Chief?          @relation(fields: [chiefId], references: [id])
  createdAt     DateTime        @default(now())
  updatedAt     DateTime        @updatedAt
  purchaseEvents PurchaseEvent[]
}

model Chief {
  id            String     @id @default(uuid())
  name          String
  walletAddress String
  commissionRate Int
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt
  referrers     Referrer[]
  purchaseEvents PurchaseEvent[]
}

model PurchaseEvent {
  id                  String    @id @default(uuid())
  txId                String    @unique
  projectId           String
  project             Project   @relation(fields: [projectId], references: [id])
  purchaseAmount      Decimal   @db.Decimal(78, 0)
  purchaseMethod      String    // MINT_ETH, MINT_USDC, etc.
  referralCode        String?
  referrerId          String?
  referrer            Referrer? @relation(fields: [referrerId], references: [id])
  chiefId             String?
  chief               Chief?    @relation(fields: [chiefId], references: [id])
  referrerCommission  Decimal?  @db.Decimal(78, 0)
  chiefCommission     Decimal?  @db.Decimal(78, 0)
  projectFundedAmount Decimal   @db.Decimal(78, 0)
  rewardForReferrer   Boolean   @default(false)
  rewardForChief      Boolean   @default(false)
  rewardForReferrerTxId String?
  rewardForChiefTxId  String?
  createdAt           DateTime  @default(now())
  updatedAt           DateTime  @updatedAt
}
```

### Data Relationships

1. **Project to PurchaseEvent (One-to-Many)**
   - Each project can have multiple purchase events
   - Purchase events are linked to exactly one project
   - This relationship enables tracking sales and funding progress per project

2. **Referrer to PurchaseEvent (One-to-Many)**
   - Each referrer can be associated with multiple purchase events
   - Purchase events may be linked to a referrer through referral codes
   - This relationship enables commission tracking and payment processing

3. **Chief to Referrer (One-to-Many)**
   - Each chief can manage multiple referrers
   - Referrers may be associated with one chief
   - This relationship establishes the two-tier commission structure

4. **Chief to PurchaseEvent (One-to-Many)**
   - Chiefs are indirectly linked to purchase events through referrers
   - This relationship enables chief commission calculations
   - Chiefs earn commissions on purchases made with their referrers' codes

The database structure ensures data integrity through foreign key relationships and enables efficient querying for dashboard displays and commission calculations.

## Relation with /contracts

The Morphism Dashboard interacts with the smart contracts defined in the `/contracts` repository, establishing a monitoring and management layer on top of the blockchain infrastructure:

### Contract Monitoring

1. **Transaction Fetching**
   - The dashboard monitors contract events through blockchain APIs
   - Purchase transactions are fetched and processed into purchase events
   - The system tracks NFT minting activities on the FundraisingNFT contract

2. **Contract State Tracking**
   - The dashboard maintains a synchronized view of contract state
   - Project funding progress is tracked based on contract transactions
   - Referral code usage is monitored through transaction data

### Purchase Event Processing

1. **Event Extraction**
   - The dashboard extracts purchase details from contract transactions
   - Referral codes used in purchases are identified and processed
   - Transaction amounts are converted to appropriate currency representations

2. **Commission Calculation**
   - Based on contract transaction data, the dashboard calculates commissions
   - Commission rates are applied according to the referral system rules
   - Both referrer and chief commissions are calculated for each purchase

### Commission Payment Processing

1. **On-Chain Payments**
   - The dashboard initiates blockchain transactions for commission payments
   - Payments are sent to referrer and chief wallet addresses
   - Transaction hashes are recorded for verification and tracking

2. **Payment Verification**
   - The dashboard verifies payment transactions on the blockchain
   - Payment status is updated based on transaction confirmations
   - The system maintains a complete audit trail of commission payments

The relationship between the dashboard and contracts creates a comprehensive system where the dashboard provides a user-friendly interface for monitoring and managing the on-chain activities executed through the smart contracts.
