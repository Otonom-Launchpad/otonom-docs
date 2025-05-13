# Otonom Fund - Smart Contract Design

This document outlines the design and key functionalities of the Otonom Fund's core smart contract, which is built using the Anchor framework on the Solana blockchain. This contract orchestrates crucial on-chain operations such as user registration, tokenomics, investments, and tier management.

**The active smart contract program ID on Solana Devnet is `CWYLQDPfH6eywYGJfrSdX2cVMczm88x3V2Rd4tcgk4jf`.**

The detailed source code for this program can be found in the `otonom-contracts` repository, specifically within the `ofund-token-spg/` directory: [View Source Code](https://github.com/Otonom-Launchpad/otonom-contracts/blob/main/ofund-token-spg/lib.rs) (Note: Link may need to point to the correct branch/main if not yet merged).

## Core On-Chain Functionalities

The Otonom Fund smart contract provides the following key features, executed and verified on the Solana blockchain:

### 1. $OFUND Token Management

The platform revolves around the `$OFUND` token. The smart contract includes functionality to manage aspects of this token:

-   **Mint Authority Setup (`initialize_mint`, `initialize_existing_mint`):** These functions establish a Program Derived Address (PDA) as the minting authority for the `$OFUND` token. This allows the program itself to securely mint tokens under specific conditions (e.g., during user registration).
-   **$OFUND Token Mint Address:** The primary `$OFUND` token mint address used by the platform is `4pV3umk8pY62ry8FsnMbQfJBYgpWnzWcC67UCMUevXLY` (on Solana Devnet). The frontend application and smart contract instructions are configured to interact with this specific mint.

### 2. User Registration and Initial Token Grant (`register_user`)

When a new user joins the Otonom Fund platform through the frontend, the `register_user` instruction is invoked:

-   **Profile Creation:** A unique `UserProfile` account is created on-chain for the user, storing their public key, tier information, total investment amount, and a history of their investments.
-   **Initial $OFUND Grant:** Upon successful registration, the user is automatically granted an initial amount of 100,000 $OFUND tokens. This minting operation is performed by the smart contract via a Cross-Program Invocation (CPI) to the SPL Token Program, authorized by the program's PDA mint authority.
-   **Initial Tier Assignment:** After receiving the initial token grant, the user's tier is calculated on-chain based on this balance and stored in their `UserProfile` (see Tier System below).

### 3. Tier System (On-Chain Logic)

The user's investment tier is a critical aspect of the platform, determining their access and allocation rights. This is managed on-chain:

-   **`UserProfile.tier` Field:** Each user's profile stores their current tier as a `u8` value.
-   **Tier Calculation (`calculate_tier` internal function):** The tier is determined based on the user's **total $OFUND tokens invested** through the platform, not just their current wallet balance. The thresholds are:
    -   **Tier 0**: < 1,000 $OFUND invested
    -   **Tier 1**: >= 1,000 $OFUND invested
    -   **Tier 2**: >= 10,000 $OFUND invested
    -   **Tier 3**: >= 100,000 $OFUND invested
-   **Dynamic Updates:** A user's tier is automatically recalculated and updated within their `UserProfile` by the smart contract during two key operations:
    1.  Upon registration (after the initial token grant).
    2.  After each successful investment made via the `invest_in_project` function.

**MVP Context for Tiers:** For the hackathon MVP, the initial 100,000 $OFUND grant effectively places all new users into Tier 3, allowing comprehensive testing of platform features. The implemented logic for recalculating tiers based on `total_invested` demonstrates the system's dynamic capabilities. Future iterations of the platform envision a more sophisticated tiering system, potentially involving dedicated staking contracts, time-locked token commitments, and nuanced reward structures to further incentivize long-term user engagement and align with mature launchpad models.

### 4. Project Investment (`invest_in_project`)

Investing in AI startup projects is a core user action, handled by the `invest_in_project` instruction:

-   **Token Transfer:** The user specifies an amount of $OFUND tokens to invest. The smart contract facilitates the secure transfer of these tokens from the investor's token account to the designated project's vault account via a CPI.
-   **State Updates (On-Chain):**
    -   The investor's `UserProfile.total_invested` amount is incremented.
    -   The project's `Project.total_raised` amount is incremented.
    -   The investor's tier is recalculated based on their new `total_invested` amount.
-   **Investment Record:** Details of the investment (project invested in, amount, and timestamp) are recorded on-chain in a vector within the investor's `UserProfile` account, providing a transparent investment history.

### 5. Project Initialization (`initialize_project`)

The smart contract also allows for the on-chain creation and setup of new projects that can receive investments:

-   **Project Account:** A `Project` account is created, storing its name, authority (who can manage it), its token vault public key, and the total amount of funds it has raised.

## Data Structures (Account Schemas)

The smart contract defines several key account structures to store its state on the Solana blockchain:

-   **`MintAuthority`**: Stores details about the PDA acting as the mint authority for the $OFUND token.
-   **`UserProfile`**: Stores user-specific data including their wallet address, current tier, total amount invested, and a list of their past investments.
-   **`Project`**: Stores details for each investment project, including its name, authority, vault address, and total funds raised.

These on-chain accounts ensure data integrity, transparency, and allow the frontend to query and display reliable user and project information.

## Security and CPIs

The contract utilizes Program Derived Addresses (PDAs) for managing mint authority, ensuring that token minting operations are strictly controlled by the program logic. Cross-Program Invocations (CPIs) are used to securely interact with the SPL Token Program for operations like minting and transferring tokens, adhering to Solana best practices.

## MVP Scope and Future Architectural Vision

The current smart contract (`CWYLQDPfH6eywYGJfrSdX2cVMczm88x3V2Rd4tcgk4jf`) consolidates essential functionalities to deliver a cohesive MVP for the Solana Breakout Hackathon. It showcases key on-chain mechanics such as PDA-controlled token minting, user profile management, dynamic tier adjustments, and investment processing.

Looking ahead, the Otonom Fund platform is envisioned to evolve into a more extensive and modular smart contract ecosystem. Future architectural enhancements may include:

-   **Dual-Token Stability Model Implementation:**
    -   Refining the `$OFUND` token's role to focus on governance (interfacing with the Governance Program) and staking (via Tier & Staking Contracts).
    -   Introducing a new smart contract capability (or set of contracts) to manage an investment stablecoin (`$OSTABLE`), possibly including mechanisms for minting/burning pegged to underlying assets like USDC, and ensuring its secure use in project investment flows.
-   **Dedicated Tier & Staking Contracts:** Separating the tier qualification logic into its own contract, incorporating advanced features like $OFUND staking, variable lock-up periods, and yield generation for staked tokens.
-   **Governance Program:** Implementing a DAO structure for community governance, allowing $OFUND holders to vote on key platform parameters and project approvals.
-   **Project-Specific Vesting Contracts:** Offering standardized, auditable vesting contracts for AI startups to manage the distribution of their tokens to investors over time.
-   **Escrow and Dispute Resolution Mechanisms:** Adding further layers of security and trust for complex investment scenarios.

This MVP serves as a critical first step, establishing a strong on-chain core that will be expanded upon to realize the full vision of the Otonom Fund launchpad.