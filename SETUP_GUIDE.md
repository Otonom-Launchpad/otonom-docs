# Otonom Fund - Project Setup Guide

This guide provides instructions for setting up and running the existing Otonom Fund project. It assumes you want to explore the application as presented for the Solana Breakout Hackathon.

**The live hackathon MVP can be accessed directly at: [https://pad.otonom.fund](https://pad.otonom.fund)** (No local setup required to view the deployed version).
**(Main Project Site, Litepaper & Pitch Deck: [https://otonom.fund](https://otonom.fund))**

**For a comprehensive overview of the Otonom Fund project, including its architecture, smart contract details, and other documentation, please refer to the main [Otonom Fund Documentation Hub](./README.md).**

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Obtaining the Code](#obtaining-the-code)
3. [Frontend Setup (`otonom-frontend`)](#frontend-setup-otonom-frontend)
4. [Smart Contracts (`otonom-contracts`)](#smart-contracts-otonom-contracts)
5. [Backend Setup (Supabase)](#backend-setup-supabase)
6. [Running the Application](#running-the-application)

## Prerequisites

Before starting, ensure you have the following installed on your system:

- **Node.js**: v18 or later (for running the frontend).
- **npm** or **yarn**: Package manager for Node.js.
- **Git**: For cloning the repositories.
- **Solana CLI**: (Optional, but recommended if you wish to inspect blockchain interactions or deploy contracts) v1.17.31 or later. The project's Program ID is `CWYLQDPfH6eywYGJfrSdX2cVMczm88x3V2Rd4tcgk4jf` and is already deployed on devnet.
- **Phantom Wallet** (or a similar Solana wallet browser extension): For interacting with the application on the Solana devnet.

## Obtaining the Code

The Otonom Fund project is organized into multiple repositories. To run the application locally and/or inspect the code in detail, you will need to clone (download) the source code to your computer. If you only wish to browse the code online, you can click the links to navigate to the GitHub repositories directly.

To clone the repositories for local setup:

```bash
git clone https://github.com/Otonom-Launchpad/otonom-frontend.git
git clone https://github.com/Otonom-Launchpad/otonom-contracts.git
# You are likely already in otonom-docs. If not: 
# git clone https://github.com/Otonom-Launchpad/otonom-docs.git 
```

## Frontend Setup (`otonom-frontend`)

The `otonom-frontend` repository contains the Next.js application that you will run to interact with the Otonom Fund platform.

1.  **Navigate to the `otonom-frontend` directory:**
    ```bash
    cd otonom-frontend
    ```
2.  **Install Dependencies:**
    Follow the instructions in the `otonom-frontend/README.md` file, which typically involves running:
    ```bash
    npm install
    # or
    # yarn install
    ```
3.  **Environment Variables:**
    The frontend may require a `.env.local` file for configuration. Key variables might include:
    - `NEXT_PUBLIC_SOLANA_RPC_HOST`: The Solana RPC endpoint (e.g., `https://api.devnet.solana.com`).
    - `NEXT_PUBLIC_PROGRAM_ID`: The Solana Program ID (`CWYLQDPfH6eywYGJfrSdX2cVMczm88x3V2Rd4tcgk4jf`).
    - (If Supabase is used) `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`.
    Check the `otonom-frontend` repository for an `.env.example` or specific instructions on required environment variables. Ensure these are set correctly for devnet interaction.

4.  **Running the Frontend Development Server:**
    Again, refer to `otonom-frontend/README.md`. It's usually:
    ```bash
    npm run dev
    ```
    This will typically start the application on `http://localhost:3000`.

## Smart Contracts (`otonom-contracts`)

The `otonom-contracts` repository contains the Anchor/Rust source code for the Solana smart contracts used by the Otonom Fund platform.

- **Purpose:** You would typically interact with this repository if you intend to:
    - Inspect the smart contract source code.
    - Rebuild the contracts.
    - Deploy the contracts to a different address or network (not recommended for hackathon evaluation unless specifically instructed).

- **Instructions:** For detailed information on the contract structure, building, and deployment, please refer to the `README.md` file within the `otonom-contracts` repository.

- **Note for Judges:** The primary program (ID: `CWYLQDPfH6eywYGJfrSdX2cVMczm88x3V2Rd4tcgk4jf`) is already deployed on the Solana devnet. The frontend is configured to interact with this deployed version. Direct setup of this repository is not required to run and test the frontend application unless you wish to build or modify the contracts yourself.

## Backend Setup (Supabase)

The Otonom Fund project utilizes Supabase for off-chain data storage, such as extended user profiles and project metadata that isn't stored directly on the Solana blockchain.

- **Configuration:** To allow the frontend to connect to the project's Supabase instance, ensure the following environment variables are correctly set in the `otonom-frontend/.env.local` file. You will typically find an `.env.example` file in the `otonom-frontend` repository to guide you on the required variables.
    - `NEXT_PUBLIC_SUPABASE_URL`: The Supabase project URL for Otonom Fund.
    - `NEXT_PUBLIC_SUPABASE_ANON_KEY`: The Supabase project anon key for Otonom Fund.

- **Note for Judges:** The frontend application is designed to connect to a pre-configured Supabase instance. The necessary URL and Key should be available as part of the project's setup information or within an `.env.example` file in the `otonom-frontend` repository.

## Running the Application

1.  **Ensure your Phantom wallet (or equivalent) is installed and set to the Solana Devnet.**
2.  **Start the frontend development server** as described in the "Frontend Setup" section (usually `npm run dev` from the `otonom-frontend` directory).
3.  Open your browser to `http://localhost:3000` (or the port indicated by the dev server).
4.  Interact with the application features, such as connecting your wallet and creating a user profile.

Remember to consult the main [README.md](./README.md) for known issues (like Phantom devnet simulation warnings) and a guide to demoing features.