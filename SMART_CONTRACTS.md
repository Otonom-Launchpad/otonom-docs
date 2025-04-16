# Otonom Fund - Smart Contract Design

This document outlines the smart contract design for the Otonom Fund platform on the Solana blockchain. We'll focus on creating a simplified design for the MVP while ensuring it can be extended for future enhancements.

## Overview

The Otonom Fund platform requires the following smart contract functionality:

1. **$OFUND Token**: An SPL token that serves as the platform's native token
2. **Tier System**: A system that determines user tiers based on $OFUND holdings
3. **Investment**: Functionality for users to invest in projects
4. **Token Distribution**: Mechanism for distributing project tokens to investors

## Simplified Approach for MVP

For the MVP, we'll take a simplified approach to smart contract development:

1. **$OFUND Token**: Create a standard SPL token without complex tokenomics
2. **Tier System**: Implement tier verification based on token balance checks
3. **Investment**: Use direct token transfers for investments
4. **Token Distribution**: Implement immediate token distribution without vesting

This simplified approach will allow us to demonstrate the end-to-end user journey while keeping the smart contract development manageable.

## Smart Contract Components

### 1. $OFUND Token

The $OFUND token will be a standard SPL token with the following properties:

- **Name**: Otonom Fund Token
- **Symbol**: OFUND
- **Decimals**: 9 (standard for Solana)
- **Total Supply**: 100,000,000 tokens
- **Initial Circulation**: 20,000,000 tokens (20%)

#### Implementation Approach

For the MVP, we'll create the $OFUND token using the Solana CLI:

```bash
# Create a new keypair for the token mint
solana-keygen new -o token-keypair.json --no-bip39-passphrase

# Create the SPL token
spl-token create-token token-keypair.json --decimals 9

# Create a token account
spl-token create-account <TOKEN_MINT_ADDRESS>

# Mint initial supply
spl-token mint <TOKEN_MINT_ADDRESS> 100000000000000000 <TOKEN_ACCOUNT_ADDRESS>
```

For a more advanced implementation, we can use Anchor to create a token program:

```rust
use anchor_lang::prelude::*;
use anchor_spl::token::{self, Mint, Token, TokenAccount};

#[program]
pub mod otonom_fund {
    use super::*;

    pub fn initialize_token(ctx: Context<InitializeToken>) -> Result<()> {
        // Initialize the $OFUND token with a total supply of 100,000,000
        let total_supply = 100_000_000_000_000_000; // With decimals
        
        // Mint initial supply to the authority
        token::mint_to(
            CpiContext::new(
                ctx.accounts.token_program.to_account_info(),
                token::MintTo {
                    mint: ctx.accounts.mint.to_account_info(),
                    to: ctx.accounts.token_account.to_account_info(),
                    authority: ctx.accounts.authority.to_account_info(),
                },
            ),
            total_supply,
        )?;
        
        Ok(())
    }
}

#[derive(Accounts)]
pub struct InitializeToken<'info> {
    #[account(mut)]
    pub authority: Signer<'info>,
    
    #[account(
        init,
        payer = authority,
        mint::decimals = 9,
        mint::authority = authority,
    )]
    pub mint: Account<'info, Mint>,
    
    #[account(
        init,
        payer = authority,
        token::mint = mint,
        token::authority = authority,
    )]
    pub token_account: Account<'info, TokenAccount>,
    
    pub system_program: Program<'info, System>,
    pub token_program: Program<'info, Token>,
    pub rent: Sysvar<'info, Rent>,
}
```

### 2. Tier System

The tier system will determine user tiers based on $OFUND holdings:

- **Tier 1**: 1,000 $OFUND (34% allocation)
- **Tier 2**: 10,000 $OFUND (33% allocation)
- **Tier 3**: 100,000 $OFUND (33% allocation)

#### Implementation Approach

For the MVP, we'll implement tier verification based on token balance checks in the frontend:

```typescript
// Check user tier based on $OFUND balance
const getUserTier = (ofundBalance: number) => {
  if (ofundBalance >= 100000) {
    return 3;
  } else if (ofundBalance >= 10000) {
    return 2;
  } else if (ofundBalance >= 1000) {
    return 1;
  } else {
    return 0;
  }
};
```

For a more advanced implementation, we can use an Anchor program to verify tiers:

```rust
use anchor_lang::prelude::*;
use anchor_spl::token::{TokenAccount};

#[program]
pub mod otonom_fund {
    use super::*;

    pub fn verify_tier(ctx: Context<VerifyTier>) -> Result<u8> {
        let balance = ctx.accounts.token_account.amount;
        
        let tier = if balance >= 100_000_000_000_000 {
            3
        } else if balance >= 10_000_000_000_000 {
            2
        } else if balance >= 1_000_000_000_000 {
            1
        } else {
            0
        };
        
        Ok(tier)
    }
}

#[derive(Accounts)]
pub struct VerifyTier<'info> {
    pub user: Signer<'info>,
    pub token_account: Account<'info, TokenAccount>,
}
```

### 3. Investment

The investment functionality will allow users to invest in projects by transferring $OFUND tokens.

#### Implementation Approach

For the MVP, we'll implement investment using direct token transfers:

```typescript
// Transfer $OFUND tokens from user to project vault
const investInProject = async (
  connection: Connection,
  wallet: WalletContextState,
  projectVault: PublicKey,
  amount: number
) => {
  const transaction = new Transaction().add(
    createTransferInstruction(
      await getAssociatedTokenAddress(OFUND_MINT, wallet.publicKey),
      projectVault,
      wallet.publicKey,
      amount
    )
  );
  
  return await wallet.sendTransaction(transaction, connection);
};
```

For a more advanced implementation, we can use an Anchor program:

```rust
use anchor_lang::prelude::*;
use anchor_spl::token::{self, Token, TokenAccount, Transfer};

#[program]
pub mod otonom_fund {
    use super::*;

    pub fn invest_in_project(
        ctx: Context<InvestInProject>,
        amount: u64,
    ) -> Result<()> {
        // Transfer $OFUND tokens from user to project vault
        token::transfer(
            CpiContext::new(
                ctx.accounts.token_program.to_account_info(),
                Transfer {
                    from: ctx.accounts.user_token_account.to_account_info(),
                    to: ctx.accounts.project_vault.to_account_info(),
                    authority: ctx.accounts.user.to_account_info(),
                },
            ),
            amount,
        )?;
        
        Ok(())
    }
}

#[derive(Accounts)]
pub struct InvestInProject<'info> {
    #[account(mut)]
    pub user: Signer<'info>,
    
    #[account(mut)]
    pub user_token_account: Account<'info, TokenAccount>,
    
    #[account(mut)]
    pub project_vault: Account<'info, TokenAccount>,
    
    pub token_program: Program<'info, Token>,
}
```

### 4. Token Distribution

The token distribution mechanism will distribute project tokens to investors.

#### Implementation Approach

For the MVP, we'll implement token distribution using direct token transfers:

```typescript
// Transfer project tokens from project vault to user
const distributeProjectTokens = async (
  connection: Connection,
  projectAuthority: Keypair,
  userTokenAccount: PublicKey,
  projectTokenMint: PublicKey,
  amount: number
) => {
  const transaction = new Transaction().add(
    createTransferInstruction(
      await getAssociatedTokenAddress(projectTokenMint, projectAuthority.publicKey),
      userTokenAccount,
      projectAuthority.publicKey,
      amount
    )
  );
  
  return await sendAndConfirmTransaction(connection, transaction, [projectAuthority]);
};
```

For a more advanced implementation, we can use an Anchor program:

```rust
use anchor_lang::prelude::*;
use anchor_spl::token::{self, Token, TokenAccount, Transfer};

#[program]
pub mod otonom_fund {
    use super::*;

    pub fn distribute_tokens(
        ctx: Context<DistributeTokens>,
        amount: u64,
    ) -> Result<()> {
        // Transfer project tokens to user
        token::transfer(
            CpiContext::new(
                ctx.accounts.token_program.to_account_info(),
                Transfer {
                    from: ctx.accounts.project_token_vault.to_account_info(),
                    to: ctx.accounts.user_project_token_account.to_account_info(),
                    authority: ctx.accounts.project_authority.to_account_info(),
                },
            ),
            amount,
        )?;
        
        Ok(())
    }
}

#[derive(Accounts)]
pub struct DistributeTokens<'info> {
    #[account(mut)]
    pub project_authority: Signer<'info>,
    
    #[account(mut)]
    pub project_token_vault: Account<'info, TokenAccount>,
    
    #[account(mut)]
    pub user_project_token_account: Account<'info, TokenAccount>,
    
    pub token_program: Program<'info, Token>,
}
```

## Complete Investment Flow

For the MVP, we'll implement a simplified investment flow that combines the above components:

```typescript
// Complete investment flow
const completeInvestment = async (
  connection: Connection,
  wallet: WalletContextState,
  projectVault: PublicKey,
  projectTokenMint: PublicKey,
  projectAuthority: Keypair,
  ofundAmount: number
) => {
  // 1. Check user tier
  const ofundBalance = await getTokenBalance(
    connection,
    await getAssociatedTokenAddress(OFUND_MINT, wallet.publicKey)
  );
  const userTier = getUserTier(ofundBalance);
  
  if (userTier === 0) {
    throw new Error('Insufficient $OFUND balance for any tier');
  }
  
  // 2. Transfer $OFUND tokens from user to project vault
  const investTransaction = await investInProject(
    connection,
    wallet,
    projectVault,
    ofundAmount
  );
  
  // 3. Calculate project tokens to distribute
  const projectTokenAmount = calculateProjectTokens(ofundAmount);
  
  // 4. Create user's project token account if it doesn't exist
  const userProjectTokenAccount = await getOrCreateAssociatedTokenAccount(
    connection,
    wallet.publicKey,
    projectTokenMint,
    wallet.publicKey
  );
  
  // 5. Transfer project tokens from project vault to user
  const distributeTransaction = await distributeProjectTokens(
    connection,
    projectAuthority,
    userProjectTokenAccount.address,
    projectTokenMint,
    projectTokenAmount
  );
  
  // 6. Return transaction signatures
  return {
    investTransaction,
    distributeTransaction
  };
};

// Helper function to calculate project tokens based on $OFUND amount
const calculateProjectTokens = (ofundAmount: number) => {
  // Simplified 1:1 conversion for MVP
  return ofundAmount;
};
```

## Smart Contract Testing

For testing the smart contracts, we'll use the Anchor testing framework:

```typescript
import * as anchor from '@project-serum/anchor';
import { Program } from '@project-serum/anchor';
import { OtonomFund } from '../target/types/otonom_fund';
import { expect } from 'chai';

describe('otonom-fund', () => {
  const provider = anchor.AnchorProvider.env();
  anchor.setProvider(provider);

  const program = anchor.workspace.OtonomFund as Program<OtonomFund>;

  it('Initializes the $OFUND token', async () => {
    // Test code here
  });

  it('Verifies user tier based on $OFUND balance', async () => {
    // Test code here
  });

  it('Allows users to invest in projects', async () => {
    // Test code here
  });

  it('Distributes project tokens to investors', async () => {
    // Test code here
  });
});
```

## Deployment

For deploying the smart contracts, we'll use the Anchor deployment process:

```bash
# Build the program
anchor build

# Deploy to localnet for testing
anchor deploy

# Deploy to devnet for hackathon
anchor deploy --provider.cluster devnet
```

## Future Enhancements

For the post-hackathon version, we can enhance the smart contracts to support:

1. **Token Vesting**: Implement vesting schedules for project tokens
2. **Governance**: Add DAO-like governance features
3. **Staking**: Implement staking mechanisms for $OFUND
4. **Rewards**: Add reward mechanisms for active users
5. **Multi-chain**: Extend the contracts to support multiple blockchains

## Conclusion

By following this smart contract design, we'll be able to create a simplified version of the Otonom Fund platform that demonstrates the end-to-end user journey for the Solana Colosseum Breakout Hackathon, while also laying the groundwork for future enhancements.

For the MVP, we'll focus on implementing the simplified approach to ensure we can demonstrate the core functionality within the hackathon timeframe. The more advanced implementations can be developed post-hackathon as the platform evolves.