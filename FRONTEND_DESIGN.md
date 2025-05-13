# Otonom Fund - Frontend Design

This document outlines the frontend design for the Otonom Fund platform, including the component structure, page layouts, and user interface elements.

## Overview

The Otonom Fund frontend will be built using Next.js with Tailwind CSS and shadcn UI components. The design will focus on creating a clean, modern interface that guides users through the investment journey.

## Design Principles

1. **Clean and Modern**: Use a clean, modern design with ample white space
2. **Intuitive Navigation**: Create an intuitive navigation system that guides users through the platform
3. **Responsive Design**: Ensure the platform works well on all devices (desktop, tablet, mobile)
4. **Consistent Branding**: Maintain consistent branding throughout the platform
5. **Accessibility**: Ensure the platform is accessible to all users

## Color Palette

Based on the homepage screenshot, we'll use the following color palette:

- **Primary**: Purple (#9d00ff) - Used for primary buttons, links, and accents
- **Secondary**: Light Purple (#E6E6FA) - Used for backgrounds, cards, and secondary elements
- **Text**: Dark Gray (#333333) - Used for primary text
- **Background**: White (#FFFFFF) - Used for page backgrounds
- **Accent**: Teal (#008080) - Used for highlights and special elements

## Typography

We'll use a clean, modern sans-serif font for the platform:

- **Headings**: Inter (Bold)
- **Body**: Inter (Regular)
- **Accents**: Inter (Medium)

## Component Structure

The frontend will be organized into the following component structure:

```
src/
├── components/
│   ├── ui/                  # shadcn UI components
│   ├── layout/              # Layout components
│   │   ├── Header.tsx       # Header component
│   │   ├── Footer.tsx       # Footer component
│   │   ├── Sidebar.tsx      # Sidebar component
│   │   └── Layout.tsx       # Main layout component
│   ├── projects/            # Project-related components
│   │   ├── ProjectCard.tsx  # Project card component
│   │   ├── ProjectList.tsx  # Project list component
│   │   └── ProjectDetails.tsx # Project details component
│   ├── wallet/              # Wallet-related components
│   │   ├── WalletConnect.tsx # Wallet connection component
│   │   └── WalletBalance.tsx # Wallet balance component
│   ├── tier/                # Tier-related components
│   │   ├── TierSelection.tsx # Tier selection component
│   │   └── TierBenefits.tsx # Tier benefits component
│   ├── investment/          # Investment-related components
│   │   ├── InvestmentForm.tsx # Investment form component
│   │   └── InvestmentConfirmation.tsx # Investment confirmation component
│   └── dashboard/           # Dashboard-related components
│       ├── PortfolioOverview.tsx # Portfolio overview component
│       ├── InvestmentHistory.tsx # Investment history component
│       └── TokenHoldings.tsx # Token holdings component
```

## Page Structure

The frontend will include the following pages:

1. **Homepage**: Introduction to the platform and featured projects
2. **Projects**: List of all available projects
3. **Project Details**: Detailed information about a specific project
4. **Dashboard**: User's investment portfolio and account information
5. **Tier Selection**: Page for selecting an investment tier
6. **Investment**: Page for investing in a project

## Key Components

### Project Card Component

The project card component will display a summary of a project:

```jsx
// src/components/projects/ProjectCard.tsx
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import Link from 'next/link';

export function ProjectCard({ project }) {
  return (
    <Card className="h-full flex flex-col">
      <div className="relative">
        <img 
          src={project.image_url} 
          alt={project.name} 
          className="w-full h-48 object-cover rounded-t-lg"
        />
        <div className="absolute top-2 right-2 bg-white px-2 py-1 rounded text-xs font-medium">
          {project.category}
        </div>
      </div>
      <CardHeader>
        <CardTitle>{project.name}</CardTitle>
        <CardDescription>{project.description.substring(0, 100)}...</CardDescription>
      </CardHeader>
      <CardContent className="flex-grow">
        <div className="mb-2">
          <div className="flex justify-between text-sm mb-1">
            <span>${project.current_funding.toLocaleString()}</span>
            <span>${project.funding_goal.toLocaleString()}</span>
          </div>
          <div className="w-full bg-gray-200 rounded-full h-2">
            <div 
              className="bg-primary h-2 rounded-full" 
              style={{ width: `${(project.current_funding / project.funding_goal) * 100}%` }}
            ></div>
          </div>
          <div className="flex justify-between text-sm mt-1">
            <span>{Math.round((project.current_funding / project.funding_goal) * 100)}%</span>
            <span>{project.investors} investors</span>
          </div>
        </div>
      </CardContent>
      <CardFooter>
        <Link href={`/projects/${project.id}`} className="w-full">
          <Button className="w-full">View Details</Button>
        </Link>
      </CardFooter>
    </Card>
  );
}
```

### Wallet Connect Component

The wallet connect component will handle wallet connection:

```jsx
// src/components/wallet/WalletConnect.tsx
'use client';

import { useWallet } from '@solana/wallet-adapter-react';
import { WalletMultiButton } from '@solana/wallet-adapter-react-ui';
import { Button } from '@/components/ui/button';

export function WalletConnect() {
  const { wallet, connected } = useWallet();

  return (
    <div className="flex items-center space-x-4">
      {connected ? (
        <div className="flex items-center gap-2">
          <span className="text-sm font-medium">
            {wallet?.adapter.publicKey.toString().slice(0, 4)}...
            {wallet?.adapter.publicKey.toString().slice(-4)}
          </span>
          <WalletMultiButton className="bg-primary text-white" />
        </div>
      ) : (
        <WalletMultiButton className="bg-primary text-white">
          Connect Wallet
        </WalletMultiButton>
      )}
    </div>
  );
}
```

### Tier Selection Component

The tier selection component will display tier options:

```jsx
// src/components/tier/TierSelection.tsx
'use client';

import { useState } from 'react';
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';

const tiers = [
  { id: 1, name: 'Tier 1', amount: 1000, allocation: '34%' },
  { id: 2, name: 'Tier 2', amount: 10000, allocation: '33%' },
  { id: 3, name: 'Tier 3', amount: 100000, allocation: '33%' },
];

export function TierSelection() {
  const [selectedTier, setSelectedTier] = useState<number | null>(null);

  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
      {tiers.map((tier) => (
        <Card 
          key={tier.id}
          className={`cursor-pointer transition-all ${
            selectedTier === tier.id ? 'border-primary ring-2 ring-primary' : ''
          }`}
          onClick={() => setSelectedTier(tier.id)}
        >
          <CardHeader>
            <CardTitle>{tier.name}</CardTitle>
            <CardDescription>Allocation: {tier.allocation}</CardDescription>
          </CardHeader>
          <CardContent>
            <p className="text-2xl font-bold">{tier.amount.toLocaleString()} $OFUND</p>
          </CardContent>
          <CardFooter>
            <Button 
              variant={selectedTier === tier.id ? "default" : "outline"}
              className="w-full"
            >
              Select {tier.name}
            </Button>
          </CardFooter>
        </Card>
      ))}
    </div>
  );
}
```

## Simplified Implementation for MVP

For the MVP, we'll focus on implementing the core components and pages:

1. **Core Components**:
   - Layout components (Header, Footer)
   - Project Card component
   - Wallet Connect component
   - Tier Selection component
   - Basic Investment Form

2. **Core Pages**:
   - Homepage
   - Projects listing
   - Project details
   - Simple dashboard

This simplified approach will allow us to demonstrate the end-to-end user journey while keeping the frontend development manageable.

## Next Steps

1. Set up the Next.js project with Tailwind CSS and shadcn UI
2. Implement the core components
3. Create the core pages
4. Connect the frontend to Supabase and Solana
5. Test and refine the user experience

By following this frontend design, we'll be able to create a clean, modern interface that guides users through the investment journey for the Solana Colosseum Breakout Hackathon.

## Future UI/UX Considerations

As the Otonom Fund platform evolves, particularly towards its goal of becoming a Decentralized Autonomous Organization (DAO), the frontend design will need to incorporate new pages and components to support governance functionalities. Considerations for future UI/UX enhancements include:

-   **Governance Dashboard:** A dedicated section within the user dashboard or a new top-level page that allows users to:
    -   View active governance proposals.
    -   Review details and discussions related to each proposal.
    -   See past proposal outcomes and voting history.
    -   Access key DAO metrics (e.g., total $OFUND staked in governance, treasury status, voter turnout statistics).
-   **Proposal Submission Interface:** A user-friendly form or guided flow for community members who meet the criteria (e.g., holding a minimum amount of $OFUND) to create and submit new governance proposals.
-   **Voting Interface:** Clear and intuitive UI elements integrated into proposal detail views to allow eligible users to cast their votes (e.g., 'For', 'Against', 'Abstain' buttons).
-   **User Profile Enhancements for Governance:** Displaying a user's voting power, past voting activity, and any active delegations within their profile.
-   **Notifications for Governance Events:** Alerting users to new proposals, voting periods starting or ending, and proposal outcomes.

These additions will be crucial for enabling active community participation in the governance and strategic direction of the Otonom Fund.