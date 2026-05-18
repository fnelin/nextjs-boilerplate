# 🧱 Next.js Full Stack Boilerplate
## ⚡ TL;DR 
Clone it, fill in your `.env.local`, run `pnpm install` and you're ready to go with a full-stack Next.js app with auth, database, tRPC and AI. Or paste it to your favorite AI agent and have it follow the steps.
## 🏗️ Why This Boilerplate 
This is my personal starting point for full-stack Next.js projects. Instead of repeating the same setup decisions across projects, this gives me a production-ready foundation I can clone and build on.
## 📦 What's Included 
  - **Next.js App Router** with TypeScript throughout
  - **tRPC** for end-to-end typesafe API communication between client and server
  - **NextAuth v4** with GitHub OAuth and Prisma adapter for authentication
  - **Prisma** connected to a **Supabase** PostgreSQL database
  - **OpenAI SDK** configured for provider-agnostic AI integration — swap between OpenAI, Claude, or OpenRouter via environment variables
  - **Shadcn** for rapid UI development built on top of Tailwind CSS
  - **Zod** for validation across both client and server
## 🎯 Who This Is For 
Developers who want to skip the boilerplate and get straight to building. You should be comfortable with TypeScript and have a basic understanding of Next.js. The setup assumes you have a Supabase project and GitHub OAuth application ready.
## 🚀 Getting Started 
Follow the setup steps in PROJECT_PLAN.md top to bottom. Each step builds on the previous one — don't skip ahead. The verification section at the end will confirm everything is wired up correctly before you start building.
## 🔄 Swapping AI Providers 
The AI layer is intentionally provider-agnostic. The OpenAI SDK is used as the client but routes requests through AI_BASE_URL in your environment variables. To switch providers, update your .env.local - no code changes required.

## ✅ Prerequisites 

Before running the setup steps make sure you have the following ready:

- **Node.js** v18 or higher installed
- **pnpm** installed globally: `npm install -g pnpm`
- **Supabase project** created at [supabase.com](https://supabase.com) with a database URL ready
- **GitHub OAuth application** created at [github.com/settings/developers](https://github.com/settings/developers) with client ID and secret ready
- **AI provider account** with an API key - OpenAI, Anthropic, or OpenRouter

## Setup Steps

1. **Initialize Next.js app router project with TypeScript**
   - Run `pnpm dlx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"`
   - Verify project structure and install dependencies with `pnpm install`
   - Verify `postcss.config.ts` was generated as `.ts` and not `.js` — rename if needed
   - Verify `tsconfig.json` has the `@/*` alias pointing to `src/*`:
     ```json
     "paths": { "@/*": ["./src/*"] }
     ```

2. **Install all required dependencies**
   - Install NextAuth: `pnpm add next-auth@4 @auth/prisma-adapter`
   - Install Zod: `pnpm add zod`
   - Install tRPC: `pnpm add @trpc/server @trpc/client @trpc/react-query @tanstack/react-query`
   - Install Prisma: `pnpm add prisma @prisma/client`
   - Install Supabase driver: `pnpm add @supabase/supabase-js`
   - Install OpenAI SDK: `pnpm add openai`
   - Install development dependencies: `pnpm add -D @types/node @types/react tsx`
   - Initialize Shadcn: `pnpm dlx shadcn@latest init`
     - Select your theme and CSS variables when prompted
     - Verify `components.json` was generated at root
     - Add components as needed: `pnpm dlx shadcn@latest add button input card dialog`
   
3. **Configure environment variables**
   - Create `.env.example` with all required variable keys but no values (committed to version control):
     ```
     NEXTAUTH_URL=
     NEXTAUTH_SECRET=
     GITHUB_ID=
     GITHUB_SECRET=
     DATABASE_URL=
     AI_BASE_URL=
     AI_API_KEY=
     AI_MODEL=
     AI_ORG_ID=
     ```
   - Create `.env.local` from `.env.example` and fill in real values:
     ```
     NEXTAUTH_URL=http://localhost:3000
     NEXTAUTH_SECRET=your-secret-key
     GITHUB_ID=your-github-client-id
     GITHUB_SECRET=your-github-client-secret
     DATABASE_URL=your-supabase-database-url
     OPENAI_API_KEY=your-openai-api-key
     OPENAI_MODEL=gpt-4o-mini
     OPENAI_ORG_ID=your-openai-org-id (optional)
     ```
   - Verify `.env.local` is present in `.gitignore`

4. **Configure Prisma and PostgreSQL database connection via Supabase**
   - Initialize Prisma: `pnpm dlx prisma init --datasource-provider postgresql`
   - Connect to Supabase database using `DATABASE_URL` from `.env.local`
   - Create `src/server/db.ts` as the global Prisma client singleton:
     ```ts
     import { PrismaClient } from "@prisma/client";
     const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };
     export const db = globalForPrisma.prisma ?? new PrismaClient();
     if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = db;
     ```
   - Define database schema in `prisma/schema.prisma` with models for User, Session, Post, etc.
   - Generate Prisma client: `pnpm exec prisma generate`
   - Push schema to database: `pnpm exec prisma db push` (development) or `pnpm exec prisma migrate dev`

5. **Set up database seeding**
   - Create `prisma/seed.ts` with sample data for local development
   - Add the seed command to `package.json`:
     ```json
     "prisma": {
       "seed": "tsx prisma/seed.ts"
     }
     ```
   - Run the seeder: `pnpm exec prisma db seed`

6. **Set up NextAuth for GitHub authentication**
   - Create `src/server/auth/config.ts` with NextAuth configuration and GitHub provider
   - Create `src/server/auth/index.ts` to export auth helpers
   - Create API route at `src/app/api/auth/[...nextauth]/route.ts`
   - Set up `src/middleware.ts` to protect routes
   - Create login and logout components in `src/app/components/auth/`

7. **Implement tRPC API setup**
   - Create `src/trpc/trpc.ts` with tRPC server setup and context
   - Create `src/trpc/root.ts` combining all sub-routers
   - Create initial routers in `src/trpc/routers/` (e.g. `post.ts`)
   - Create API handler at `src/app/api/trpc/[trpc]/route.ts`
   - Configure tRPC client for use in React components

8. **Create sample data models and queries**
   - Create tRPC procedures for CRUD operations in `src/trpc/routers/`
   - Create sample API endpoints for testing
   - Implement input validation using `src/lib/utils/validation.ts` for shared rules and `src/server/ai/utils/validators.ts` for server-side rules
   - Implement proper error handling across procedures

9. **Implement OpenAI integration**
   - Create OpenAI client configuration in `src/server/ai/config.ts`
   - Create OpenAI service instance in `src/server/ai/services/openai.ts`
   - Create chat completion service in `src/server/ai/services/chat-service.ts`
   - Create embedding service in `src/server/ai/services/embedding-service.ts`
   - Create content moderation service in `src/server/ai/services/moderation-service.ts`
   - Add prompt utilities in `src/server/ai/utils/prompts.ts`
   - Add response formatters in `src/server/ai/utils/formatters.ts`
   - Create AI API routes at `src/app/api/ai/chat/`, `src/app/api/ai/embeddings/`, and `src/app/api/ai/moderation/`
   - Add tRPC procedures for AI features in `src/trpc/routers/ai.ts`
   - Implement error handling and rate limiting on all AI routes

10. **Build basic routing structure and UI**
    - Create main layout in `src/app/layout.tsx` with navigation and authentication state
    - Create global styles in `src/styles/globals.css`
    - Build homepage at `src/app/page.tsx` with welcome message and login button
    - Create dashboard page with protected routes
    - Build shared UI components in `src/app/components/ui/`
    - Build AI feature components in `src/app/components/ai/`
    - Implement loading states and error boundaries
    - Add responsive design with Tailwind CSS

11. **Set up health check endpoint**
    - Create `src/app/api/health/route.ts` to verify API, database, and AI service availability
    - Return status for each service dependency so issues can be isolated quickly

**Relevant files**
- `package.json` — Project dependencies
- `.env.local` — Environment variables configuration
- `prisma/schema.prisma` — Database schema definition
- `src/app/api/trpc/[trpc]/route.ts` — tRPC router handler
- `src/app/api/auth/[...nextauth]/route.ts` — NextAuth authentication API route
- `src/server/auth/config.ts` — NextAuth providers and session configuration
- `src/trpc/root.ts` — Root tRPC router combining all sub-routers
- `src/trpc/trpc.ts` — tRPC server setup and context
- `src/server/db.ts` — Global Prisma client singleton
- `components.json` — Shadcn UI configuration

**Verification**
1. Run `pnpm dev` and verify Next.js server is running
2. Hit `http://localhost:3000/api/health` and verify all services return healthy
3. Check GitHub OAuth flow works end to end — login, session, logout
4. Verify database connection via Prisma: `pnpm exec prisma studio`
5. Confirm tRPC API endpoints respond correctly
6. Confirm authentication state persists across page refreshes
7. Confirm AI service responds with correct provider via `src/app/api/ai/chat/route.ts`
8. Run `pnpm exec prisma db seed` and verify seed data appears in Prisma Studio

**Decisions**
- Using Next.js App Router for modern routing structure
- TypeScript as primary language throughout
- Shadcn for faster UI development
- Zod as validation engine
- Supabase as PostgreSQL hosting service
- GitHub OAuth as primary authentication method
- pnpm as package manager
- OpenAI SDK as AI client with provider-agnostic configuration
- NextAuth v4 for authentication

## Expected Project File Tree (App Router)

```
.
├── next.config.ts (Next.js framework configuration)
├── tailwind.config.ts (Tailwind CSS theme and plugin configuration)
├── tsconfig.json (TypeScript compiler configuration)
├── postcss.config.ts (PostCSS plugin configuration for Tailwind)
├── .env.local (Local environment variables - never committed)
├── .env.example (Template of required environment variables)
├── .gitignore (Files excluded from version control)
├── package.json (Project dependencies and scripts)
├── pnpm-lock.yaml (Dependency lock file for reproducible builds)
├── README.md (Project documentation and setup guide)
│
├── prisma/
│   ├── schema.prisma (Database schema definition)
│   └── seed.ts (Database seeding script for local development)
│
├── .vscode/
│   └── tasks.json (VS Code task configurations)
│
└── src/
    ├── middleware.ts (Route protection and auth middleware for Next.js)
    │
    ├── app/
    │   ├── layout.tsx (Root layout component)
    │   ├── page.tsx (Homepage component)
    │   │
    │   ├── api/
    │   │   ├── auth/
    │   │   │   └── [...nextauth]/
    │   │   │       └── route.ts (NextAuth authentication API route)
    │   │   ├── trpc/
    │   │   │   └── [trpc]/
    │   │   │       └── route.ts (tRPC API router handler)
    │   │   ├── health/
    │   │   │   └── route.ts (Application health check endpoint)
    │   │   └── ai/
    │   │       ├── chat/
    │   │       │   └── route.ts (Chat completion API endpoint)
    │   │       ├── embeddings/
    │   │       │   └── route.ts (Embedding generation API endpoint)
    │   │       └── moderation/
    │   │           └── route.ts (Content moderation API endpoint)
    │   │
    │   └── components/
    │       ├── ui/
    │       │   ├── button.tsx (Reusable button component)
    │       │   ├── input.tsx (Reusable input component)
    │       │   ├── card.tsx (Reusable card component)
    │       │   ├── dialog.tsx (Reusable dialog component)
    │       │   ├── form/
    │       │   │   ├── form.tsx (Reusable form wrapper component)
    │       │   │   ├── form-field.tsx (Form field with validation display)
    │       │   │   └── form-label.tsx (Accessible form label component)
    │       │   └── feedback/
    │       │       ├── toast.tsx (Toast notification component)
    │       │       ├── alert.tsx (Inline alert message component)
    │       │       └── spinner.tsx (Loading spinner component)
    │       ├── layout/
    │       │   ├── header.tsx (Page header component)
    │       │   ├── navigation.tsx (Navigation menu component)
    │       │   └── footer.tsx (Page footer component)
    │       ├── auth/
    │       │   ├── login-button.tsx (Login trigger button component)
    │       │   ├── logout-button.tsx (Logout trigger button component)
    │       │   └── auth-provider.tsx (NextAuth session provider wrapper)
    │       ├── dashboard/
    │       │   ├── dashboard-layout.tsx (Dashboard page layout component)
    │       │   ├── sidebar.tsx (Dashboard sidebar navigation component)
    │       │   └── stats-grid.tsx (Dashboard statistics grid component)
    │       ├── posts/
    │       │   ├── post-list.tsx (Paginated post list component)
    │       │   ├── post-card.tsx (Post summary card component)
    │       │   └── post-detail.tsx (Full post detail view component)
    │       └── ai/
    │           ├── chat/
    │           │   ├── chat-interface.tsx (Top-level chat UI component)
    │           │   ├── chat-message.tsx (Individual chat message component)
    │           │   ├── chat-input.tsx (Chat message input and submit component)
    │           │   └── chat-history.tsx (Scrollable chat history component)
    │           ├── embeddings/
    │           │   ├── similarity-search.tsx (Semantic similarity search component)
    │           │   └── vector-display.tsx (Vector data visualisation component)
    │           ├── moderation/
    │           │   ├── content-warning.tsx (Content warning display component)
    │           │   └── moderation-badge.tsx (Moderation status badge component)
    │           └── shared/
    │               ├── ai-loading.tsx (AI response loading state component)
    │               ├── ai-error.tsx (AI error state display component)
    │               └── ai-config.tsx (AI feature runtime configuration component)
    │
    ├── lib/
    │   ├── hooks/
    │   │   ├── use-debounce.ts (Debounce utility hook)
    │   │   ├── use-chat.ts (Chat state and API interaction hook)
    │   │   ├── use-embeddings.ts (Embedding generation and search hook)
    │   │   ├── use-moderation.ts (Content moderation hook)
    │   │   └── use-ai-config.ts (AI feature configuration hook)
    │   ├── utils/
    │   │   ├── cn.ts (clsx utility for conditional class names)
    │   │   ├── format.ts (General data formatting utilities)
    │   │   └── validation.ts (Shared client and server validation utilities)
    │   └── types/
    │       ├── auth.ts (Authentication-related TypeScript types)
    │       ├── api.ts (API request and response TypeScript types)
    │       ├── common.ts (Shared common TypeScript types)
    │       └── ai/
    │           ├── chat.ts (Chat-related TypeScript types)
    │           ├── embeddings.ts (Embedding-related TypeScript types)
    │           ├── moderation.ts (Moderation-related TypeScript types)
    │           └── models.ts (AI model TypeScript types)
    │
    ├── server/
    │   ├── auth/
    │   │   ├── config.ts (NextAuth providers and session configuration)
    │   │   └── index.ts (NextAuth exports)
    │   ├── ai/
    │   │   ├── services/
    │   │   │   ├── openai.ts (OpenAI client instance and base configuration)
    │   │   │   ├── chat-service.ts (Chat completion service)
    │   │   │   ├── embedding-service.ts (Embedding generation service)
    │   │   │   └── moderation-service.ts (Content moderation service)
    │   │   ├── utils/
    │   │   │   ├── prompts.ts (Prompt template utilities)
    │   │   │   ├── validators.ts (Server-side AI input validation utilities)
    │   │   │   └── formatters.ts (AI response formatting utilities)
    │   │   └── config.ts (OpenAI configuration from environment variables)
    │   └── db.ts (Global Prisma client singleton)
    │
    ├── trpc/
    │   ├── routers/
    │   │   ├── post.ts (tRPC procedure definitions for posts)
    │   │   └── ai.ts (tRPC procedure definitions for AI features)
    │   ├── root.ts (Root tRPC router combining all sub-routers)
    │   └── trpc.ts (tRPC server setup and context)
    │
    └── styles/
        └── globals.css (Global styles and Tailwind base overrides)
```

## 🏹 Next Step
Create the boilerplate structure with templates for even faster start.
