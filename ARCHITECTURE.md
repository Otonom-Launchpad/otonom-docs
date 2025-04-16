# Otonom Fund - Architecture Overview

This document provides a high-level overview of the Otonom Fund architecture, explaining how the different components of the system interact with each other.

## System Architecture

The Otonom Fund platform consists of three main components:

1. **Frontend**: Next.js application with Tailwind CSS and shadcn UI
2. **Backend**: Supabase for off-chain data storage and authentication
3. **Blockchain**: Solana blockchain for on-chain transactions and token management

Here's a diagram of how these components interact:

```
+-------------------+       +-------------------+       +-------------------+
|                   |       |                   |       |                   |
|     Frontend      |<----->|      Backend      |<----->|    Blockchain     |
|    (Next.js)      |       |    (Supabase)     |       |     (Solana)      |
|                   |       |                   |       |                   |
+-------------------+       +-------------------+       +-------------------+
        ^                           ^                           ^
        |                           |                           |
        v                           v                           v
+-------------------+       +-------------------+       +-------------------+
|                   |       |                   |       |                   |
|       User        |       |     Database      |       |   Smart Contracts |
|    Interface      |       |                   |       |                   |
|                   |       |                   |       |                   |
+-------------------+       +-------------------+       +-------------------+
```

## Component Details

### 1. Frontend (Next.js)

The frontend is responsible for:
- Rendering the user interface
- Handling user interactions
- Connecting to the user's wallet
- Communicating with the backend and blockchain

**Key Components**:
- **Pages**: Homepage, Project Listing, Project Details, Dashboard
- **Components**: Layout, Navigation, Project Cards, Tier Selection, Investment Flow
- **Hooks**: Wallet Connection, Data Fetching, State Management
- **Utils**: Formatting, Validation, API Calls

### 2. Backend (Supabase)

The backend is responsible for:
- Storing off-chain data
- Handling authentication
- Providing API endpoints for the frontend
- Synchronizing with the blockchain

**Key Components**:
- **Database**: Users, Projects, Investments
- **Authentication**: Wallet-based authentication
- **API**: RESTful API endpoints
- **Functions**: Serverless functions for complex operations

### 3. Blockchain (Solana)

The blockchain is responsible for:
- Managing tokens ($OFUND and project tokens)
- Handling transactions
- Enforcing business rules through smart contracts
- Providing transparency and immutability

**Key Components**:
- **SPL Tokens**: $OFUND token and project tokens
- **Smart Contracts**: Tier System, Investment, Token Distribution
- **Transactions**: Token Transfers, Investment Operations
- **Accounts**: User Wallets, Project Vaults

## Data Flow

### 1. User Registration and Authentication

```
User -> Connect Wallet -> Frontend -> Supabase -> Create User Record
```

1. User connects their Phantom wallet to the frontend
2. Frontend sends the wallet address to Supabase
3. Supabase creates a new user record if it doesn't exist
4. User is authenticated and can access the platform

### 2. Project Listing and Details

```
Frontend -> Supabase -> Fetch Projects -> Display to User
```

1. Frontend requests project data from Supabase
2. Supabase returns project data
3. Frontend displays projects to the user
4. User can view project details

### 3. Tier Selection

```
User -> Select Tier -> Frontend -> Check Balance -> Display Tier Benefits
```

1. User selects a tier (1, 2, or 3)
2. Frontend checks the user's $OFUND balance
3. If the user has enough tokens, they are assigned to the selected tier
4. Frontend displays tier benefits to the user

### 4. Investment Flow

```
User -> Select Project -> Enter Amount -> Frontend -> Solana -> Execute Transaction -> Supabase -> Update Records
```

1. User selects a project and enters an investment amount
2. Frontend initiates a transaction on Solana
3. Solana executes the transaction, transferring $OFUND tokens from the user to the project vault
4. Solana transfers project tokens from the project vault to the user
5. Frontend updates the investment record in Supabase
6. Supabase updates the user's portfolio

### 5. Portfolio Tracking

```
User -> View Dashboard -> Frontend -> Supabase -> Fetch Investments -> Display Portfolio
```

1. User navigates to the dashboard
2. Frontend requests investment data from Supabase
3. Supabase returns investment data
4. Frontend displays the user's portfolio, showing investments and token holdings

## Simplified Architecture for MVP

For the MVP (Minimum Viable Product), we'll simplify the architecture to focus on demonstrating the end-to-end user journey:

### 1. Frontend Simplifications

- Use simple state management (React Context) instead of complex state management libraries
- Implement basic responsive design without complex animations
- Use shadcn UI components for faster development

### 2. Backend Simplifications

- Use Supabase for both authentication and database
- Implement basic API services without complex caching
- Use simple data models

### 3. Blockchain Simplifications

- Use simple SPL token without complex tokenomics
- Implement basic smart contracts without complex logic
- Use direct token transfers without complex mechanisms

## Technical Considerations

### 1. Performance

- Use server-side rendering (SSR) for initial page load
- Implement client-side rendering (CSR) for dynamic content
- Optimize API calls to reduce latency
- Use caching for frequently accessed data

### 2. Security

- Implement proper authentication and authorization
- Validate all user inputs
- Use secure communication channels
- Follow best practices for smart contract development

### 3. Scalability

- Design the system to handle increased load
- Use efficient data structures and algorithms
- Implement pagination for large data sets
- Optimize database queries

### 4. Maintainability

- Follow clean code principles
- Use TypeScript for type safety
- Implement proper error handling
- Write comprehensive documentation

## Future Architecture Considerations

For the post-hackathon version, we'll enhance the architecture to support:

1. **Multi-chain Support**: Extend the architecture to support multiple blockchains
2. **Advanced Analytics**: Implement a data warehouse for complex analytics
3. **Social Features**: Add social components for user interaction
4. **Governance Mechanisms**: Implement DAO-like governance features
5. **Mobile App**: Develop a mobile app for better user experience

By following this architecture, we'll be able to create a simplified version of the Otonom Fund platform that demonstrates the end-to-end user journey for the Solana Colosseum Breakout Hackathon, while also laying the groundwork for future enhancements.