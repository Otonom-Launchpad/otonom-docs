# Otonom Fund - Environment Setup Guide

This guide provides step-by-step instructions for setting up the development environment for the Otonom Fund project. We'll cover setting up the frontend (Next.js), backend (Supabase), and blockchain (Solana) environments.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Frontend Setup (Next.js)](#frontend-setup-nextjs)
3. [Backend Setup (Supabase)](#backend-setup-supabase)
4. [Blockchain Setup (Solana)](#blockchain-setup-solana)
5. [Connecting the Environments](#connecting-the-environments)
6. [Running the Development Environment](#running-the-development-environment)

## Prerequisites

Before starting, ensure you have the following installed:

- Node.js (v18 or later)
- npm or yarn
- Git
- Rust and Cargo (for Solana development)
- Solana CLI (v1.17.31 or later)
- Anchor CLI (v0.29.0 or later)

You already have the following installed:
- solana-cli 1.17.31
- anchor-cli 0.29.0
- rustc 1.86.0

## Frontend Setup (Next.js)

### 1. Initialize the Next.js Project

```bash
# Navigate to the frontend repository
cd otonom-frontend

# Initialize a new Next.js project with TypeScript
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
```

### 2. Install Required Dependencies

```bash
# Install shadcn UI and its dependencies
npm install @radix-ui/react-slot class-variance-authority clsx tailwind-merge lucide-react
npm install -D tailwindcss-animate

# Install Solana wallet adapter
npm install @solana/wallet-adapter-react @solana/wallet-adapter-wallets @solana/wallet-adapter-react-ui @solana/web3.js

# Install Supabase client
npm install @supabase/supabase-js
```

### 3. Configure Tailwind CSS

The `tailwind.config.js` file should be automatically created by the Next.js setup. Update it to include shadcn UI configuration:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: ["class"],
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        "accordion-down": {
          from: { height: 0 },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: 0 },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
}
```

### 4. Add Global CSS Variables

Update the `src/app/globals.css` file to include the shadcn UI CSS variables:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 240 10% 3.9%;
    --card: 0 0% 100%;
    --card-foreground: 240 10% 3.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 240 10% 3.9%;
    --primary: 270 50% 60%;
    --primary-foreground: 0 0% 98%;
    --secondary: 240 4.8% 95.9%;
    --secondary-foreground: 240 5.9% 10%;
    --muted: 240 4.8% 95.9%;
    --muted-foreground: 240 3.8% 46.1%;
    --accent: 240 4.8% 95.9%;
    --accent-foreground: 240 5.9% 10%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 0 0% 98%;
    --border: 240 5.9% 90%;
    --input: 240 5.9% 90%;
    --ring: 270 50% 60%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 240 10% 3.9%;
    --foreground: 0 0% 98%;
    --card: 240 10% 3.9%;
    --card-foreground: 0 0% 98%;
    --popover: 240 10% 3.9%;
    --popover-foreground: 0 0% 98%;
    --primary: 270 50% 60%;
    --primary-foreground: 0 0% 98%;
    --secondary: 240 3.7% 15.9%;
    --secondary-foreground: 0 0% 98%;
    --muted: 240 3.7% 15.9%;
    --muted-foreground: 240 5% 64.9%;
    --accent: 240 3.7% 15.9%;
    --accent-foreground: 0 0% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 0 0% 98%;
    --border: 240 3.7% 15.9%;
    --input: 240 3.7% 15.9%;
    --ring: 270 50% 60%;
  }
}
```

### 5. Set Up Project Structure

Create the following directory structure:

```
otonom-frontend/
├── public/
│   └── images/
├── src/
│   ├── app/
│   │   ├── page.tsx
│   │   ├── projects/
│   │   │   ├── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx
│   │   ├── dashboard/
│   │   │   └── page.tsx
│   │   └── layout.tsx
│   ├── components/
│   │   ├── ui/
│   │   ├── layout/
│   │   ├── projects/
│   │   ├── wallet/
│   │   └── dashboard/
│   ├── hooks/
│   ├── lib/
│   ├── utils/
│   └── types/
```

### 6. Configure Environment Variables

Create a `.env.local` file in the root of the frontend project:

```
NEXT_PUBLIC_SUPABASE_URL=https://mzrbxhfgtcypluhceshe.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im16cmJ4aGZndGN5cGx1aGNlc2hlIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDQxNDgzNjQsImV4cCI6MjA1OTcyNDM2NH0.eLoPYDyqZGPM2Op3Wb24mU65Zm8sDsf_tB1JZz-BdvA
```

## Backend Setup (Supabase)

### 1. Access Supabase Project

You already have a Supabase project set up with the following credentials:
- URL: https://mzrbxhfgtcypluhceshe.supabase.co
- Anon Key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im16cmJ4aGZndGN5cGx1aGNlc2hlIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDQxNDgzNjQsImV4cCI6MjA1OTcyNDM2NH0.eLoPYDyqZGPM2Op3Wb24mU65Zm8sDsf_tB1JZz-BdvA

### 2. Create Database Schema

Access the Supabase dashboard and create the following tables:

#### Users Table

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  wallet_address TEXT UNIQUE NOT NULL,
  tier INTEGER DEFAULT 0,
  ofund_balance NUMERIC DEFAULT 0,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### Projects Table

```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  description TEXT,
  category TEXT,
  image_url TEXT,
  funding_goal NUMERIC NOT NULL,
  current_funding NUMERIC DEFAULT 0,
  token_symbol TEXT,
  token_address TEXT,
  start_date TIMESTAMP WITH TIME ZONE,
  end_date TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### Investments Table

```sql
CREATE TABLE investments (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id),
  project_id UUID REFERENCES projects(id),
  amount NUMERIC NOT NULL,
  token_amount NUMERIC NOT NULL,
  transaction_signature TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id, project_id)
);
```

### 3. Set Up Row-Level Security (RLS)

Enable Row-Level Security for all tables and create policies:

#### Users Table Policy

```sql
-- Enable RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Create policy for users to read their own data
CREATE POLICY "Users can read their own data"
  ON users
  FOR SELECT
  USING (auth.uid() = id);

-- Create policy for users to update their own data
CREATE POLICY "Users can update their own data"
  ON users
  FOR UPDATE
  USING (auth.uid() = id);
```

#### Projects Table Policy

```sql
-- Enable RLS
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Create policy for anyone to read projects
CREATE POLICY "Anyone can read projects"
  ON projects
  FOR SELECT
  TO anon
  USING (true);
```

#### Investments Table Policy

```sql
-- Enable RLS
ALTER TABLE investments ENABLE ROW LEVEL SECURITY;

-- Create policy for users to read their own investments
CREATE POLICY "Users can read their own investments"
  ON investments
  FOR SELECT
  USING (auth.uid() = user_id);

-- Create policy for users to create their own investments
CREATE POLICY "Users can create their own investments"
  ON investments
  FOR INSERT
  WITH CHECK (auth.uid() = user_id);
```

### 4. Create Sample Data

Insert sample data for testing:

```sql
-- Insert sample projects
INSERT INTO projects (name, description, category, image_url, funding_goal, token_symbol)
VALUES 
  ('NeuralWave AI', 'Advanced neural network for generative content creation', 'AI Research', 'https://example.com/neural.jpg', 500000, 'NWAI'),
  ('SynthGen', 'Synthetic data generation platform for machine learning', 'Data Science', 'https://example.com/synthgen.jpg', 300000, 'SGEN'),
  ('EcoAI Solutions', 'AI-powered solutions for environmental monitoring', 'Climate Tech', 'https://example.com/ecoai.jpg', 750000, 'EAIS');
```

## Blockchain Setup (Solana)

### 1. Set Up Local Solana Validator

```bash
# Start a local Solana validator
solana-test-validator
```

### 2. Configure Solana CLI

```bash
# Configure Solana CLI to use localhost
solana config set --url localhost

# Generate a new keypair for development
solana-keygen new --no-bip39-passphrase -o ~/.config/solana/id.json

# Check the configuration
solana config get

# Airdrop SOL to your local wallet for testing
solana airdrop 100
```

### 3. Initialize Anchor Project

```bash
# Navigate to the contracts repository
cd otonom-contracts

# Initialize a new Anchor project
anchor init otonom-fund

# Navigate to the project directory
cd otonom-fund

# Build the project
anchor build
```

### 4. Create a Simple SPL Token

Create a simple SPL token for $OFUND using the Solana CLI:

```bash
# Create a new keypair for the token mint
solana-keygen new -o token-keypair.json --no-bip39-passphrase

# Create the SPL token
spl-token create-token token-keypair.json --decimals 9

# Create a token account
spl-token create-account <TOKEN_MINT_ADDRESS>

# Mint some tokens for testing
spl-token mint <TOKEN_MINT_ADDRESS> 1000000000 <TOKEN_ACCOUNT_ADDRESS>
```

### 5. Update Anchor.toml

Update the `Anchor.toml` file in the `otonom-fund` directory:

```toml
[features]
seeds = false
skip-lint = false

[programs.localnet]
otonom_fund = "YOUR_PROGRAM_ID"

[registry]
url = "https://api.apr.dev"

[provider]
cluster = "localnet"
wallet = "~/.config/solana/id.json"

[scripts]
test = "yarn run ts-mocha -p ./tsconfig.json -t 1000000 tests/**/*.ts"
```

## Connecting the Environments

### 1. Set Up Supabase Client in Next.js

Create a Supabase client in `src/lib/supabase.ts`:

```typescript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL as string;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY as string;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### 2. Set Up Solana Connection in Next.js

Create a Solana connection in `src/lib/solana.ts`:

```typescript
import { Connection, clusterApiUrl } from '@solana/web3.js';

// Use localhost for development, devnet for testing, and mainnet-beta for production
const network = process.env.NEXT_PUBLIC_SOLANA_NETWORK || 'http://localhost:8899';

// Create a connection to the Solana cluster
export const connection = new Connection(
  network === 'devnet' ? clusterApiUrl('devnet') : 
  network === 'mainnet-beta' ? clusterApiUrl('mainnet-beta') : 
  network
);
```

### 3. Set Up Wallet Provider in Next.js

Create a wallet provider in `src/app/providers.tsx`:

```typescript
'use client';

import { WalletAdapterNetwork } from '@solana/wallet-adapter-base';
import { ConnectionProvider, WalletProvider } from '@solana/wallet-adapter-react';
import { WalletModalProvider } from '@solana/wallet-adapter-react-ui';
import { PhantomWalletAdapter } from '@solana/wallet-adapter-wallets';
import { clusterApiUrl } from '@solana/web3.js';
import { ReactNode, useMemo } from 'react';

// Import the styles
require('@solana/wallet-adapter-react-ui/styles.css');

export function Providers({ children }: { children: ReactNode }) {
  // The network can be set to 'devnet', 'testnet', or 'mainnet-beta'
  const network = WalletAdapterNetwork.Devnet;

  // You can also provide a custom RPC endpoint
  const endpoint = useMemo(() => clusterApiUrl(network), [network]);

  // @solana/wallet-adapter-wallets includes all the adapters but supports tree shaking
  const wallets = useMemo(() => [new PhantomWalletAdapter()], []);

  return (
    <ConnectionProvider endpoint={endpoint}>
      <WalletProvider wallets={wallets} autoConnect>
        <WalletModalProvider>{children}</WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
}
```

Update `src/app/layout.tsx` to include the providers:

```typescript
import { Providers } from './providers';
import './globals.css';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

## Running the Development Environment

### 1. Start the Local Solana Validator

```bash
# In a separate terminal
solana-test-validator
```

### 2. Start the Next.js Development Server

```bash
# Navigate to the frontend repository
cd otonom-frontend

# Start the development server
npm run dev
```

### 3. Access the Application

Open your browser and navigate to `http://localhost:3000` to access the application.

## Next Steps

Now that you have set up all the environments, you can proceed to implement the core features in order of priority:

1. Implement the homepage with featured projects
2. Create the project details page
3. Implement wallet connection
4. Create the tier selection component
5. Implement the investment flow
6. Create the user dashboard

Each of these features will be implemented in a step-by-step manner, focusing on creating a simplified version that demonstrates the end-to-end user journey.