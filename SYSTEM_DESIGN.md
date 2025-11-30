# AgencyHub System Design Documentation

## Overview

AgencyHub is a Next.js 16 dashboard application for managing agency information and employee contacts with user authentication and daily viewing limits.

## System Architecture Diagram

\`\`\`
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                   │
│  │   Landing    │    │   Sign In    │    │   Sign Up    │                   │
│  │    Page      │    │    Page      │    │    Page      │                   │
│  │   (/)        │    │ (/sign-in)   │    │ (/sign-up)   │                   │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘                   │
│         │                   │                   │                            │
│         └───────────────────┼───────────────────┘                            │
│                             │                                                │
│                             ▼                                                │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                     CLERK AUTHENTICATION                              │   │
│  │                     (Middleware Protection)                           │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                             │                                                │
│                             ▼                                                │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                    PROTECTED ROUTES                                   │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                │   │
│  │  │  Dashboard   │  │   Agencies   │  │   Contacts   │                │   │
│  │  │  (/dashboard)│  │  (/agencies) │  │  (/contacts) │                │   │
│  │  │              │  │              │  │              │                │   │
│  │  │  • Stats     │  │  • Data      │  │  • Data      │                │   │
│  │  │  • Overview  │  │    Table     │  │    Table     │                │   │
│  │  │  • Quick     │  │  • Search    │  │  • Search    │                │   │
│  │  │    Links     │  │  • Paginate  │  │  • Paginate  │                │   │
│  │  │              │  │              │  │  • Daily     │                │   │
│  │  │              │  │              │  │    Limit     │                │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           DATA & STATE LAYER                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────────────────┐    ┌───────────────────────────┐            │
│  │    Static Data Store      │    │    Client-Side State      │            │
│  │   (lib/data/*.ts)         │    │    (localStorage)         │            │
│  │                           │    │                           │            │
│  │  • agencies.ts            │    │  • Daily view count       │            │
│  │    - 20 agencies          │    │  • View date tracking     │            │
│  │    - ID, name, city,      │    │  • Limit enforcement      │            │
│  │      state, country       │    │                           │            │
│  │                           │    │                           │            │
│  │  • contacts.ts            │    │                           │            │
│  │    - 70 contacts          │    │                           │            │
│  │    - ID, name, email,     │    │                           │            │
│  │      phone, position,     │    │                           │            │
│  │      department, agencyId │    │                           │            │
│  └───────────────────────────┘    └───────────────────────────┘            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                          EXTERNAL SERVICES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────────────────┐    ┌───────────────────────────┐            │
│  │         CLERK             │    │        VERCEL             │            │
│  │    Authentication         │    │      Deployment           │            │
│  │                           │    │                           │            │
│  │  • User management        │    │  • Hosting                │            │
│  │  • Session handling       │    │  • Edge functions         │            │
│  │  • Sign in/up flows       │    │  • Environment vars       │            │
│  │  • User profiles          │    │                           │            │
│  └───────────────────────────┘    └───────────────────────────┘            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
\`\`\`

## Data Flow Diagrams

### Authentication Flow

\`\`\`
┌────────┐     ┌──────────┐     ┌───────────┐     ┌───────────┐
│  User  │────▶│ Landing  │────▶│  Clerk    │────▶│ Dashboard │
│        │     │  Page    │     │ Sign In   │     │  (Auth)   │
└────────┘     └──────────┘     └───────────┘     └───────────┘
                                      │
                                      ▼
                               ┌───────────┐
                               │ Middleware│
                               │ (Verify)  │
                               └───────────┘
\`\`\`

### Contact Viewing Flow with Daily Limit

\`\`\`
┌────────┐     ┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│  User  │────▶│ Contacts     │────▶│ Check Daily   │────▶│ Show Table   │
│  Click │     │ Page         │     │ Limit         │     │ (if allowed) │
└────────┘     └──────────────┘     └───────┬───────┘     └──────────────┘
                                            │
                                            │ Limit Reached?
                                            ▼
                                    ┌───────────────┐
                                    │ Show Upgrade  │
                                    │ Modal         │
                                    └───────────────┘
\`\`\`

## Component Structure

\`\`\`
app/
├── page.tsx                    # Landing page (public)
├── layout.tsx                  # Root layout with ClerkProvider
├── globals.css                 # Global styles
├── sign-in/
│   └── [[...sign-in]]/
│       └── page.tsx            # Clerk sign-in page
├── sign-up/
│   └── [[...sign-up]]/
│       └── page.tsx            # Clerk sign-up page
├── dashboard/
│   └── page.tsx                # Main dashboard (protected)
├── agencies/
│   └── page.tsx                # Agencies table (protected)
└── contacts/
    └── page.tsx                # Contacts table (protected)

components/
├── ui/                         # shadcn/ui components
├── dashboard-nav.tsx           # Navigation header
├── data-table.tsx              # Reusable data table
├── agencies-table.tsx          # Agencies-specific table
├── contacts-table-wrapper.tsx  # Contacts table with limit logic
├── contact-limit-banner.tsx    # Daily limit warning banner
└── upgrade-modal.tsx           # Upgrade prompt dialog

lib/
├── data/
│   ├── agencies.ts             # Agency data store
│   └── contacts.ts             # Contact data store
├── contact-limit.ts            # Daily limit logic
└── utils.ts                    # Utility functions

middleware.ts                   # Clerk auth middleware
\`\`\`

## Key Features Implementation

### 1. Authentication (Clerk)
- Protected routes via middleware
- Sign-in/sign-up pages using Clerk components
- User session management
- User profile in navigation

### 2. Daily Contact Limit (50/day)
- Stored in localStorage with date tracking
- Automatic reset at midnight (new date)
- Progress indicator showing usage
- Upgrade modal when limit reached

### 3. Data Tables
- Reusable DataTable component
- Search functionality
- Pagination (10 items per page)
- Responsive design

### 4. Upgrade Prompt
- Modal dialog when limit reached
- Pro plan feature highlights
- Call-to-action button (no payment integration)

## Environment Variables Required

\`\`\`env
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard
\`\`\`

## Security Considerations

1. **Authentication**: All sensitive routes protected by Clerk middleware
2. **Rate Limiting**: Client-side daily limit (50 contacts/day)
3. **Data Access**: Only authenticated users can view agency/contact data

## Deployment

1. Push code to GitHub
2. Connect repository to Vercel
3. Configure environment variables in Vercel dashboard
4. Deploy

## Future Enhancements

- [ ] Database integration (Supabase/Neon)
- [ ] Payment integration for Pro upgrade (Stripe)
- [ ] Export contacts to CSV
- [ ] Advanced search filters
- [ ] User role management
- [ ] Audit logging
