# Otonom Fund - Architecture Overview

This document provides a high-level overview of the Otonom Fund architecture, explaining how the different components of the system interact with each other. This architecture is designed to support the core functionalities of the MVP and serve as a foundation for future evolution into a community-governed Decentralized Autonomous Organization (DAO).

## System Architecture

Here's a diagram of how these components interact:

```
+-------------------+       +--------------------------+       +-----------------------+
|                   |------>|                          |<------|                       |
|       User        |       |         Frontend         |------>|  Blockchain (Solana)  |
|    (Browser)      |<------|        (Next.js)         |       |   Smart Contract      |
|                   |       |                          |<------|                       |
+-------------------+       +--------------------------+       +-----------------------+
                                      |        ^
                                      |        |
                                      v        |
                              +--------------------------+
                              |     Backend (Supabase)   |
                              | - Off-chain Data Storage |
                              | - User Profile Enrichment|
                              +--------------------------+
```

The primary on-chain interactions are with the Solana smart contract, Program ID: `CWYLQDPfH6eywYGJfrSdX2cVMczm88x3V2Rd4tcgk4jf`.

## Component Details

### 1. Frontend (Next.js)

The frontend is the primary user interface for the Otonom Fund platform. It is responsible for:

- Rendering all user-facing views (Homepage, Project Listings, Investment Interfaces, User Dashboard).
- Managing client-side state and user interactions.
- Integrating with Solana wallets (e.g., Phantom) for user authentication (signature verification) and transaction signing.
- Directly invoking instructions on the Otonom Fund Solana smart contract for on-chain actions like user registration and project investment.
- Querying the Solana blockchain for on-chain data (e.g., user's $OFUND balance, tier status from `UserProfile` account, project details from `Project` accounts).
- Communicating with the Supabase backend to fetch or store supplementary off-chain data (e.g., detailed project descriptions, images, user preferences).

**Key Technologies**: Next.js, React, Tailwind CSS, shadcn UI, `@solana/web3.js`.

### 2. Backend (Supabase)

Supabase serves as the off-chain data persistence layer and for functionalities not suitable for direct blockchain storage. Its responsibilities include:

- **Off-Chain Data Storage**: Storing information like:
    - Detailed project descriptions, FAQs, team bios, roadmaps, images, and other rich media content.
    - Potentially, aggregated analytics or cached data for performance.
    - User-specific preferences or settings not stored on-chain.
- **User Profile Enrichment**: While core user identity and tier are on-chain, Supabase can store additional, mutable user profile information (e.g., display name, avatar, notification settings) linked to their wallet address.
- **API Endpoints**: Providing RESTful API endpoints for the frontend to manage this off-chain data.

**Authentication Note**: User authentication is primarily blockchain-centric (verifying ownership of a Solana wallet via signature). Supabase's role is to associate this verified wallet address with the off-chain data it stores, rather than being the primary authenticator itself.

**Blockchain Synchronization**: For the MVP, direct, automated synchronization between Supabase and the Solana blockchain (e.g., listening to on-chain events to update Supabase tables) is not implemented. The frontend typically acts as the orchestrator, reading on-chain data and then, if necessary, updating Supabase with related off-chain information after a successful on-chain transaction.

### 3. Blockchain (Solana)

The Solana blockchain and our custom Anchor smart contract form the backbone for all trustless and transparent operations on the Otonom Fund platform.

- **Smart Contract (`otonom_program`):**
    - Manages user registration, including the creation of on-chain `UserProfile` accounts and the initial grant of $OFUND tokens (minted via a PDA).
    - Handles the end-to-end investment process (`invest_in_project` instruction), including $OFUND token transfer from user to project, updates to user/project totals, on-chain tier recalculation, and recording investment history in the `UserProfile`.
    - Defines and enforces the user tier system based on `total_invested` $OFUND, calculated on-chain.
    - Manages on-chain `Project` accounts.
    - Source code: `otonom-contracts/ofund-token-spg/lib.rs`.
- **$OFUND SPL Token**: The native utility token of the platform.
- **Transparency and Immutability**: All core transactions (registrations, investments, tier changes) are recorded on the Solana blockchain, providing auditability.

**Project Token Distribution**: For the MVP, the distribution of *project-specific tokens* back to investors after an investment in $OFUND is **not** handled by the current core smart contract. This aspect would be handled separately or is considered a future enhancement beyond the immediate scope of the MVP's primary smart contract.

## Data Flow Examples

### 1. User Registration

```
User (Frontend)      Solana Smart Contract        Supabase (Backend)
     |
Connect Wallet
     |
Sign Message (Proof of Ownership)
     |
Call 'register_user' ------------>
     |                     Creates UserProfile (PDA)
     |                     Mints initial $OFUND to user
     |                     Calculates & sets initial tier
     |<--------------------------- Success
     |
(Optional) Store additional
profile info (e.g., email)---------------------------> Save User Data
                                                         (linked to wallet addr)
```

1.  User connects their Solana wallet via the Frontend.
2.  Frontend may request a signature to prove wallet ownership.
3.  Frontend calls the `register_user` instruction on the Solana smart contract.
4.  The smart contract creates an on-chain `UserProfile` account (PDA) for the user, mints an initial 100,000 $OFUND tokens to their associated token account, and sets their initial tier based on this grant.
5.  Upon success, the Frontend may then allow the user to provide additional off-chain profile information (e.g., email, display name), which is saved to Supabase, linked to their wallet address.

### 2. Project Listing and Details

```
User (Frontend)      Solana Smart Contract        Supabase (Backend)
     |
Request Project List ---------------------------------> Fetch Rich Project Data
     |<------------------------------------------------- Return Project Metadata
     |
(For each project, or on detail view)
Request On-Chain Data
(e.g., total_raised) ------> Read Project Account
     |<---------------------- Return On-Chain Stats
     |
Display Combined View
```

1.  Frontend requests a list of projects. Primary rich content (descriptions, images) is fetched from Supabase.
2.  For displaying dynamic, on-chain data like `total_raised` for a project, or specific user-related status (e.g., if they've invested), the Frontend queries the respective Solana smart contract accounts.
3.  Frontend combines on-chain and off-chain data to present a complete view to the user.

### 3. Tier Status Display

```
User (Frontend)      Solana Smart Contract
     |
Request User Tier --------> Read UserProfile.tier
     |<----------------------- Return current_tier
     |
Display Tier & Benefits
```

1.  After user registration/connection, the Frontend reads the `tier` field directly from the user's on-chain `UserProfile` account.
2.  The Frontend displays the tier and its associated benefits. The tier is automatically updated on-chain by the smart contract after registration or investments.

### 4. Investment Flow (Using $OFUND)

```
User (Frontend)      Solana Smart Contract        Supabase (Backend)
     |
Select Project, Enter Amount
     |
Call 'invest_in_project' -------->
     |                       Verifies user, project
     |                       Transfers $OFUND (user to project_vault)
     |                       Updates user.total_invested
     |                       Updates project.total_raised
     |                       Recalculates & updates user.tier
     |                       Records investment in user.investments[]
     |<---------------------------- Success Transaction ID
     |
(Optional) Log transaction ref
for off-chain analytics -----------------------------> Save Investment Log
```

1.  User selects a project and specifies an amount of $OFUND to invest via the Frontend.
2.  Frontend constructs and initiates the `invest_in_project` transaction, which the user signs with their wallet.
3.  The Solana smart contract executes the instruction:
    *   Transfers the specified $OFUND amount from the user's token account to the project's vault account.
    *   Updates the `total_invested` in the user's `UserProfile`.
    *   Updates the `total_raised` in the project's `Project` account.
    *   Recalculates and updates the user's `tier` in their `UserProfile` based on the new `total_invested`.
    *   Adds a record of this investment (project, amount, timestamp) to the `investments` vector in the `UserProfile`.
4.  Upon successful execution, the Frontend receives a transaction ID.
5.  Optionally, the Frontend might send this transaction ID or a summary to Supabase for logging or to enrich off-chain analytics data (though core investment records are on-chain).

**Note on Project Token Distribution**: As stated earlier, the immediate distribution of the invested project's own tokens back to the user is outside the scope of this core `invest_in_project` $OFUND instruction in the current MVP smart contract.

### 5. Portfolio Tracking

```
User (Frontend)      Solana Smart Contract        Supabase (Backend)
     |
View Dashboard
     |
Request On-Chain Portfolio -> Read UserProfile.investments[]
     |                          Read UserProfile.total_invested
     |                          Read UserProfile.tier
     |<------------------------ Return Investment History, Stats
     |
(Optional) Fetch supplementary
project details for invested
projects ---------------------------------------------> Fetch Project Metadata
     |<------------------------------------------------- Return Details
     |
Display Portfolio
```

1.  User navigates to their dashboard/portfolio page on the Frontend.
2.  Frontend reads the `investments` vector, `total_invested`, and `tier` directly from the user's on-chain `UserProfile` account.
3.  To display richer information about the projects the user has invested in (e.g., project names, logos), the Frontend may then fetch corresponding metadata from Supabase using the project identifiers stored in the on-chain investment records.
4.  Frontend combines and displays this information.

## Smart Contract Capabilities (MVP)

The Otonom Fund MVP leverages a robust Anchor-based smart contract on Solana. Key capabilities include:

-   **On-Chain User Profiles**: Securely storing user data (wallet, tier, investment history) in PDAs.
-   **Program-Controlled Token Minting**: Utilizing a PDA as a mint authority for the $OFUND token, enabling features like the initial token grant upon registration.
-   **Integrated Investment Logic**: Handling $OFUND transfers, updating user and project financial totals, and managing investment records within a single atomic transaction.
-   **Dynamic On-Chain Tier System**: Automatically calculating and updating user tiers based on their total on-chain investment activity through the platform.

This approach ensures that core platform functionalities are transparent, verifiable, and executed with the security guarantees of the Solana blockchain, moving beyond simplistic off-chain logic for critical operations.

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

## MVP Scope and Future Architectural Vision

The current system architecture represents a robust Minimum Viable Product (MVP) for the Otonom Fund launchpad. It establishes the core pathways for user interaction, off-chain data management via Supabase, and critical on-chain functionalities through the primary Solana smart contract. This integrated approach allows for the demonstration of end-to-end user flows, from registration and tier assignment to project investment.

Looking ahead, as the Otonom Fund platform matures, the architecture is envisioned to evolve towards a more distributed and specialized model. Potential future enhancements include:

-   **Dual-Token Stability Model:** Introducing a two-token system:
    -   `$OFUND`: Primarily for governance (via the Governance Contract), staking, and platform privileges.
    -   `$OSTABLE` (or similar): A USD-pegged stablecoin for project investments, enhancing investor security.
-   **Microservices for Backend Logic:** Decomposing parts of the backend (currently Supabase-centric) into specialized microservices for greater scalability and maintainability, particularly for complex off-chain computations or integrations.
-   **Modular Smart Contract System:** Expanding the on-chain capabilities by introducing multiple, interconnected smart contracts. This could include:
    -   A dedicated **Tier and Staking Contract** for more advanced tier management and $OFUND staking (linked to governance rights).
    -   A **Governance Contract** for DAO-based decision-making, proposal systems, and treasury management.
    -   Standardized **Project Vesting Contracts** for token distributions.
-   **Enhanced Indexing and Caching Layers:** Implementing more sophisticated data indexing services for faster on-chain data retrieval and caching layers to optimize frontend performance.
-   **Advanced Analytics Infrastructure:** Developing a robust data pipeline and warehousing solution to support complex analytics, platform insights, and user reporting.
-   **Decentralized Storage Solutions:** Exploring options like Arweave or IPFS for storing project metadata and other non-critical off-chain data in a more decentralized manner.
-   **Multi-Chain Interoperability:** Strategically extending support to other relevant blockchain ecosystems to broaden reach and accessibility.
-   **Community and Social Engagement Features:** Integrating features that foster interaction, discussion, and community building around projects and the platform itself.
-   **Native Mobile Applications:** Developing dedicated mobile apps for iOS and Android to provide an optimized user experience on the go.

This MVP architecture provides a solid foundation upon which these more advanced and scalable components can be progressively integrated, aligning with the long-term vision for the Otonom Fund platform.

This approach ensures that core platform functionalities are transparent, verifiable, and executed with the security guarantees of the Solana blockchain, moving beyond simplistic off-chain logic for critical operations.