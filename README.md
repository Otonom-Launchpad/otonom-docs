# Otonom Fund

AI-powered launchpad for training (AI Startup School), accelerating and funding next-gen AI ventures. Built using the aiBlocks System methodology through duopreneurship (human-AI collaboration).

**LIVE HACKATHON MVP: [https://pad.otonom.fund](https://pad.otonom.fund)**
**(Main Project Site, Litepaper & Pitch Deck: [https://otonom.fund](https://otonom.fund))**

## Project Overview

Otonom Fund is a blockchain-based launchpad connecting AI startups with investors through a tiered investment system on Solana. The platform features a three-tier investment system, curated AI project showcase, and investment portfolio dashboard. While the initial MVP focuses on core launchpad functionalities, the long-term vision for Otonom Fund is to evolve into a fully decentralized, community-governed platform (DAO). A cornerstone of this future architecture will be our innovative **Dual-Token Stability Model**: `$OFUND` will serve as the core governance token, enabling community participation in key decisions and platform evolution, while a dedicated stablecoin (e.g., `$OSTABLE`, pegged to USDC) will be the primary currency for project investments, offering investors enhanced capital stability and mitigating volatility risks inherent in early-stage ventures.

This project is being developed for the Solana Colosseum Breakout Hackathon (April 14 - May 16, 2025).

## Repository Structure

The Otonom Fund project is a multi-repository system hosted under the [Otonom Launchpad GitHub Organization](https://github.com/Otonom-Launchpad). Key repositories include:

- **[otonom-docs](https://github.com/Otonom-Launchpad/otonom-docs)**: This repository. Contains all primary documentation, project roadmap, architecture diagrams, and serves as the main entry point for understanding the project.
- **[otonom-frontend](https://github.com/Otonom-Launchpad/otonom-frontend)**: The Next.js/Tailwind/shadcn UI application. This is the runnable part of the project.
- **[otonom-backend](https://github.com/Otonom-Launchpad/otonom-backend)**: API services and database schemas (Supabase).
- **[otonom-contracts](https://github.com/Otonom-Launchpad/otonom-contracts)**: Solana smart contracts developed with Anchor.
- **[otonom-sdk](https://github.com/Otonom-Launchpad/otonom-sdk)**: Shared libraries and utilities.

## Documentation

This repository contains comprehensive documentation for the Otonom Fund project:

- [Setup Guide](./SETUP_GUIDE.md): Step-by-step instructions for setting up the development environment
- [Implementation Plan](./IMPLEMENTATION_PLAN.md): Detailed plan for implementing the project
- [Architecture](./ARCHITECTURE.md): Overview of the system architecture
- [Database Schema](./DATABASE_SCHEMA.md): Details of the database schema
- [Smart Contracts](./SMART_CONTRACTS.md): Design of the smart contracts
- [Frontend Design](./FRONTEND_DESIGN.md): Design of the frontend components and user interface

## Key Features

- **$OFUND Token**: SPL token implementation on Solana
- **Three-Tier Investment System**: Tier 1 (1,000 $OFUND), Tier 2 (10,000 $OFUND), Tier 3 (100,000 $OFUND)
- **Curated AI Project Showcase**: Platform for showcasing AI projects
- **Investment Portfolio Dashboard**: Dashboard for tracking investments
- **Wallet Connection**: Integration with Phantom wallet

## Known Issues / Development Notes

### Phantom Wallet Simulation on Devnet
When performing transactions on the Solana devnet using Phantom wallet (e.g., creating a user profile, investing), you may encounter a "Simulation failed" warning from Phantom *before* the transaction is sent to the network. This is a known behavior with Phantom on devnet for certain types of transactions or when interacting with newly deployed programs.

**In most cases, if you approve the transaction despite this warning, it will still succeed on the devnet.** The frontend application will typically reflect the successful transaction shortly after.

This is primarily a devnet/Phantom UI consideration and not an indication of an on-chain program error for this project. For the hackathon, please proceed with approving transactions if you see this simulation warning.

## Core MVP User Journey

This journey outlines the scope of the Minimum Viable Product (MVP) for the Otonom Fund platform as developed for the Solana Colosseum Breakout Hackathon.

1.  **Discovery**: Users explore curated AI projects
2.  **Account Creation**: Users connect their Solana wallet
3.  **Tier Selection**: Users choose investment tier (1, 2, or 3)
4.  **Investment**: Users acquire $OFUND tokens (simulated or via faucet for MVP)
5.  **Project Selection**: Users browse and select AI projects
6.  **Fund Allocation**: Users invest in specific projects (simulated for MVP)
7.  **Token Receipt**: Users receive project tokens immediately (simulated for MVP)
8.  **Portfolio Tracking**: Users monitor investments through dashboard (WIP)

## Technical Stack

- **Frontend**: Next.js with Tailwind CSS and shadcn UI components
- **Backend**: Supabase for off-chain data storage
- **Blockchain**: Solana (SPL token for $OFUND, smart contracts using Anchor)
- **Wallet Integration**: Phantom wallet

## Smart Contract Overview & Development Journey

The Otonom Fund platform leverages Solana smart contracts for its core on-chain functionalities, including token operations and investment mechanisms.

- **Active Program ID:** The primary smart contract currently active on the devnet is `CWYLQDPfH6eywYGJfrSdX2cVMczm88x3V2Rd4tcgk4jf`.

- **Development Evolution:** Our smart contract development was an iterative process. An earlier version, used for initial local Anchor setup and testing, can be found in the `otonom-contracts/ofund-token-local/` directory within the `otonom-contracts` repository. The currently active smart contract was refined using Solana Playground, and its Anchor/Rust source code is primarily managed within `otonom-contracts/ofund-token-spg/`. This approach allowed for rapid prototyping in Playground while maintaining an organized local project structure.

- **Detailed Contract Information & Deployment:** For comprehensive details on the smart contract architecture, source code, and deployment instructions, please refer to the [otonom-contracts repository README](https://github.com/Otonom-Launchpad/otonom-contracts/blob/main/README.md). High-level design principles are also outlined in our [Smart Contracts](./SMART_CONTRACTS.md) document.

## Getting Started

To get started with the Otonom Fund project, follow the [Setup Guide](./SETUP_GUIDE.md) to set up your development environment. This typically involves cloning the key repositories like `otonom-frontend` and `otonom-contracts`.

## Running the Application & Demo Guide

To experience the Otonom Fund Launchpad live:

1.  **Navigate to the Frontend:** The main application is in the **[otonom-frontend repository](https://github.com/Otonom-Launchpad/otonom-frontend)**.
2.  **Follow Setup Instructions:** Refer to the `README.md` within the `otonom-frontend` repository for detailed instructions on installing dependencies and running the development server.
3.  **Key Features to Explore:**
    *   Connect your Phantom wallet (ensure it's set to Solana Devnet).
    *   Create a new user profile (this interacts with the smart contract).
    *   Explore listed AI projects.
    *   (If implemented) Simulate an investment in a project.
    *   View your user dashboard.

Please be mindful of the "Phantom Wallet Simulation on Devnet" note mentioned earlier when making transactions.

## Project Status & Hackathon Achievements

As of the Solana Colosseum Breakout Hackathon submission (May 2025), the Otonom Fund Launchpad has achieved the following:

*   **Core Smart Contract Deployed:** The foundational smart contract (`CWYLQDPfH6eywYGJfrSdX2cVMczm88x3V2Rd4tcgk4jf`) is live on the Solana devnet, enabling key functionalities like user profile creation and management.
*   **Frontend Integration:** The Next.js frontend application successfully connects to the devnet, allowing users to:
    *   Connect their Phantom wallet.
    *   Create and view their on-chain user profiles.
    *   Browse project listings (static content).
*   **Development Iteration:** Successfully navigated the smart contract development lifecycle, evolving from initial local Anchor tests to a refined Solana Playground build, demonstrating adaptability and practical problem-solving.
*   **Multi-Repository Structure:** Established a professional, organized multi-repository structure for frontend, contracts, and documentation, facilitating clarity and maintainability.
*   **Documentation Hub:** This `otonom-docs` repository serves as a comprehensive guide to the project's architecture, setup, and usage.

This represents a significant step towards realizing the vision of the Otonom Fund Launchpad.

## License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.