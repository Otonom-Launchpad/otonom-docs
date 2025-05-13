# Otonom Fund - Database Schema

This document provides a detailed overview of the database schema for the Otonom Fund platform. We'll be using Supabase as our database provider, which is built on PostgreSQL.

## Overview

The database schema consists of the following main tables:

1. **Users**: Stores information about users, including their wallet addresses and tier information
2. **Projects**: Stores information about AI projects available for investment
3. **Investments**: Tracks user investments in projects
4. **Tokens**: Stores information about project tokens

## Table Definitions

### Users Table

The `users` table stores information about users who have connected their wallets to the platform.

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

**Fields**:
- `id`: Unique identifier for the user (UUID)
- `wallet_address`: The user's Solana wallet address (unique)
- `tier`: The user's investment tier (0 = no tier, 1 = Tier 1, 2 = Tier 2, 3 = Tier 3)
- `ofund_balance`: The user's $OFUND token balance
- `created_at`: Timestamp when the user was created
- `updated_at`: Timestamp when the user was last updated

### Projects Table

The `projects` table stores information about AI projects available for investment.

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

**Fields**:
- `id`: Unique identifier for the project (UUID)
- `name`: The name of the project
- `description`: A detailed description of the project
- `category`: The category of the project (e.g., AI Research, Data Science, Climate Tech)
- `image_url`: URL to the project's image
- `funding_goal`: The funding goal for the project (in $OFUND)
- `current_funding`: The current amount of funding received (in $OFUND)
- `token_symbol`: The symbol of the project's token
- `token_address`: The Solana address of the project's token
- `start_date`: The date when the project starts accepting investments
- `end_date`: The date when the project stops accepting investments
- `created_at`: Timestamp when the project was created
- `updated_at`: Timestamp when the project was last updated

### Investments Table

The `investments` table tracks user investments in projects.

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

**Fields**:
- `id`: Unique identifier for the investment (UUID)
- `user_id`: Reference to the user who made the investment
- `project_id`: Reference to the project that was invested in
- `amount`: The amount invested (in $OFUND)
- `token_amount`: The amount of project tokens received
- `transaction_signature`: The Solana transaction signature
- `created_at`: Timestamp when the investment was made
- `UNIQUE(user_id, project_id)`: Ensures a user can only have one investment record per project

### Tokens Table (Optional for MVP)

The `tokens` table stores information about project tokens.

```sql
CREATE TABLE tokens (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  project_id UUID REFERENCES projects(id),
  symbol TEXT NOT NULL,
  name TEXT NOT NULL,
  address TEXT NOT NULL,
  total_supply NUMERIC NOT NULL,
  decimals INTEGER DEFAULT 9,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Fields**:
- `id`: Unique identifier for the token (UUID)
- `project_id`: Reference to the project that the token belongs to
- `symbol`: The symbol of the token
- `name`: The name of the token
- `address`: The Solana address of the token
- `total_supply`: The total supply of the token
- `decimals`: The number of decimal places for the token
- `created_at`: Timestamp when the token was created
- `updated_at`: Timestamp when the token was last updated

## Relationships

The following relationships exist between the tables:

1. **Users to Investments**: One-to-Many (One user can have many investments)
2. **Projects to Investments**: One-to-Many (One project can have many investments)
3. **Projects to Tokens**: One-to-One (One project has one token)

## Indexes

To optimize query performance, we'll create the following indexes:

```sql
-- Users table indexes
CREATE INDEX idx_users_wallet_address ON users(wallet_address);
CREATE INDEX idx_users_tier ON users(tier);

-- Projects table indexes
CREATE INDEX idx_projects_category ON projects(category);
CREATE INDEX idx_projects_token_symbol ON projects(token_symbol);

-- Investments table indexes
CREATE INDEX idx_investments_user_id ON investments(user_id);
CREATE INDEX idx_investments_project_id ON investments(project_id);
```

## Row-Level Security (RLS)

To secure the data, we'll implement Row-Level Security (RLS) policies:

### Users Table Policies

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

### Projects Table Policies

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

### Investments Table Policies

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

## Sample Data

For testing purposes, we'll insert the following sample data:

### Sample Projects

```sql
-- Insert sample projects
INSERT INTO projects (name, description, category, image_url, funding_goal, token_symbol)
VALUES 
  ('NeuralWave AI', 'Advanced neural network for generative content creation', 'AI Research', 'https://example.com/neural.jpg', 500000, 'NWAI'),
  ('SynthGen', 'Synthetic data generation platform for machine learning', 'Data Science', 'https://example.com/synthgen.jpg', 300000, 'SGEN'),
  ('EcoAI Solutions', 'AI-powered solutions for environmental monitoring', 'Climate Tech', 'https://example.com/ecoai.jpg', 750000, 'EAIS');
```

## Database Access in the Application

In the Next.js application, we'll use the Supabase client to access the database:

```typescript
// src/lib/supabase.ts
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL as string;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY as string;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### Example Queries

#### Fetching Projects

```typescript
// Fetch all projects
const fetchProjects = async () => {
  const { data, error } = await supabase
    .from('projects')
    .select('*')
    .order('created_at', { ascending: false });
  
  if (error) {
    console.error('Error fetching projects:', error);
    return [];
  }
  
  return data;
};

// Fetch a specific project
const fetchProject = async (id: string) => {
  const { data, error } = await supabase
    .from('projects')
    .select('*')
    .eq('id', id)
    .single();
  
  if (error) {
    console.error('Error fetching project:', error);
    return null;
  }
  
  return data;
};
```

#### Fetching User Investments

```typescript
// Fetch user investments
const fetchUserInvestments = async (userId: string) => {
  const { data, error } = await supabase
    .from('investments')
    .select(`
      *,
      projects (*)
    `)
    .eq('user_id', userId);
  
  if (error) {
    console.error('Error fetching user investments:', error);
    return [];
  }
  
  return data;
};
```

#### Creating an Investment

```typescript
// Create an investment
const createInvestment = async (userId: string, projectId: string, amount: number, tokenAmount: number, transactionSignature: string) => {
  const { data, error } = await supabase
    .from('investments')
    .insert({
      user_id: userId,
      project_id: projectId,
      amount,
      token_amount: tokenAmount,
      transaction_signature: transactionSignature
    });
  
  if (error) {
    console.error('Error creating investment:', error);
    return null;
  }
  
  return data;
};
```

## Simplified Schema for MVP

For the MVP (Minimum Viable Product), we can simplify the schema to focus on the core functionality:

1. **Users Table**: Keep as is
2. **Projects Table**: Keep as is
3. **Investments Table**: Keep as is
4. **Tokens Table**: Optional for MVP

This simplified schema will allow us to demonstrate the end-to-end user journey while keeping the database structure manageable.

## Future Considerations

For the post-hackathon version, we can enhance the schema to support:

1.  **User Profiles**: Add more user information (name, email, profile picture)
2.  **Project Updates**: Track project updates and milestones
3.  **Token Vesting**: Implement token vesting schedules
4.  **Community Features**: Add tables for comments, reviews, and discussions
5.  **Notifications**: Implement a notification system for users
6.  **DAO Functionality**: Extend the schema to support Decentralized Autonomous Organization (DAO) operations, such as storing proposals, voting records, and treasury management data, aligning with the platform's long-term governance goals.

This schema provides a solid foundation for the Otonom Fund MVP and can be extended to support future features and growth.