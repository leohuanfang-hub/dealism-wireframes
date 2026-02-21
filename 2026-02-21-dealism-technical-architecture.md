# Dealism Consumer — Technical Architecture & MVP Build Plan
> Generated: 2026-02-21 00:45 CST
> Purpose: Actionable tech stack, architecture, and 4-week MVP sprint plan
> Based on: product-summary-v1.md + wireframes + Leo's design principles

---

## 1. Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│                    DEALISM CONSUMER MVP                       │
│                                                              │
│  ┌─────────┐    ┌──────────────┐    ┌──────────────────┐    │
│  │  DATA    │    │   AGENT      │    │   INTERACTION    │    │
│  │  INGEST  │───▶│   CORE       │───▶│   LAYER          │    │
│  │         │    │              │    │                  │    │
│  │ Gmail   │    │ Classifier   │    │ IM Push          │    │
│  │ IMAP    │    │ Executor     │    │ (WhatsApp/TG)    │    │
│  │         │    │ Memory       │    │                  │    │
│  │         │    │ Discovery    │    │ Living Dashboard │    │
│  │         │    │              │    │ (Web PWA)        │    │
│  └─────────┘    └──────────────┘    └──────────────────┘    │
│                        │                                     │
│                   ┌────┴────┐                                │
│                   │  STATE  │                                │
│                   │  STORE  │                                │
│                   │ Postgres│                                │
│                   │ + Redis │                                │
│                   └─────────┘                                │
└──────────────────────────────────────────────────────────────┘
```

### Core Design Decisions

| Decision | Choice | Why |
|----------|--------|-----|
| **Language** | TypeScript (full stack) | Same team writes backend + dashboard. Leo can prototype fast. |
| **Framework** | Next.js 15 (App Router) | Dashboard = SSR PWA. API routes for webhooks. Edge-ready. |
| **Database** | Postgres (Supabase) | Auth, Row Level Security, Realtime subscriptions for dashboard live updates. Free tier for MVP. |
| **Cache/Queue** | Redis (Upstash) | Message queue for email processing. Rate limiting. Session state. Serverless-friendly. |
| **AI** | Claude API (Anthropic) | Best at nuanced classification + tool use. Already familiar via OpenClaw. |
| **Email** | Gmail API (OAuth2) → IMAP fallback | Gmail API for 90% of users. IMAP for Outlook/others. |
| **IM Layer** | WhatsApp Business API + Telegram Bot | V1 channels. Linq (iMessage) = V2 after PMF validation. |
| **Hosting** | Vercel (Dashboard) + Railway (Agent Worker) | Vercel = free for dashboard. Railway = persistent worker process for email polling + agent loop. |
| **Auth** | Supabase Auth + Google OAuth | Users sign in with Google → auto-connect Gmail. One click onboarding. |

---

## 2. Data Ingest Layer

### 2a. Email Connection Flow

```
User clicks "Connect Gmail"
    → Google OAuth2 (scope: gmail.readonly + gmail.send)
    → Store refresh_token (encrypted, Supabase vault)
    → Start historical scan (background worker)
    → Real-time: Gmail Push Notifications (pub/sub) OR poll every 60s
```

**Gmail API Scopes (Restricted — requires Google verification):**
- `gmail.readonly` — read all emails (MVP: needed for full scan)
- `gmail.send` — send emails on behalf (for agent execution)
- `gmail.modify` — mark as read, archive (V2)

**MVP Workaround (Pre-verification):**
- Use IMAP + App Password for initial beta users (no Google verification needed)
- Gmail API with "Testing" mode (100 users cap, no verification required)
- Apply for verification in parallel (takes 4-6 weeks)

### 2b. Historical Email Scan Pipeline

```typescript
// Worker process: scan historical emails
async function scanHistoricalEmails(userId: string) {
  const emails = await gmail.listMessages({ maxResults: 500, q: 'newer_than:90d' });
  
  for (const batch of chunk(emails, 20)) {
    // Parallel fetch full content
    const fullEmails = await Promise.all(batch.map(e => gmail.getMessage(e.id)));
    
    // Classify each email
    const classified = await classifyBatch(fullEmails);
    
    // Store results + emit realtime events
    await db.emails.insertMany(classified);
    await realtime.broadcast(userId, 'scan_progress', { 
      processed: count, 
      total: emails.length,
      discoveries: newDiscoveries 
    });
  }
}
```

**Scan Strategy (Leo's "开盲盒" experience):**
1. Start with last 7 days (immediate results in <30 seconds)
2. Expand to 30 days (1-2 minutes)
3. Background: 90 days (5-10 minutes)
4. Each discovery pushes to dashboard in realtime (Supabase Realtime)

### 2c. Email Classification Schema

```typescript
interface ClassifiedEmail {
  id: string;
  userId: string;
  gmailId: string;
  
  // Agent classification
  agentStatus: 'need_you' | 'in_progress' | 'watching' | 'discovery' | 'handled' | 'noise';
  subCategory: 'reply' | 'confirm' | 'financial' | 'schedule' | 'deadline' | 'custom';
  
  // Classification metadata
  summary: string;           // 1-line summary for dashboard card
  suggestedAction: string;   // "Reply confirming March 15 appointment"
  urgency: 'high' | 'medium' | 'low';
  confidence: number;        // 0-1, for human review threshold
  
  // Original email data
  from: string;
  subject: string;
  receivedAt: Date;
  threadId: string;
  
  // Agent execution
  agentDraft?: string;       // Pre-drafted reply
  executedAt?: Date;
  userDecision?: 'approved' | 'modified' | 'rejected';
}
```

---

## 3. Agent Core

### 3a. Classification Prompt (Claude)

```
You are an email classification agent for a personal assistant app.

User profile: {userProfile}
User's contact graph: {topContacts}
User's patterns: {patterns}

Classify this email into exactly ONE status:
- NEED_YOU: Requires human decision (reply needed, confirmation, financial decision)
- WATCHING: No action now, but worth tracking (shipping, upcoming bills, reservations)
- DISCOVERY: Proactive insight (price increase, forgotten reply, subscription audit)
- HANDLED: Routine/no action needed (receipts, newsletters read, confirmations)
- NOISE: Marketing, spam-adjacent, zero value

For each, provide:
1. One-line summary (≤80 chars, human-readable)
2. Suggested action (what the user should do, or what Agent will do)
3. Sub-category: reply | confirm | financial | schedule | deadline
4. Urgency: high (needs response today) | medium (this week) | low (whenever)
5. Confidence: 0.0-1.0

Output JSON only.
```

**Cost estimate:**
- Claude Sonnet per email: ~$0.002 (500 input tokens + 200 output)
- 200 emails/day scan: ~$0.40/day/user
- Monthly per user: ~$12 at scale
- MVP (50 beta users): ~$600/month AI cost

### 3b. Discovery Engine

Seven pre-built discovery rules (run after initial scan + daily):

```typescript
const discoveryRules = [
  {
    name: 'price_increase',
    description: 'Detect subscription price changes',
    query: 'subject:(price OR increased OR new rate OR billing update) newer_than:30d',
    classifier: 'Compare amounts in email body vs user history'
  },
  {
    name: 'forgotten_reply',
    description: 'Emails waiting for user reply >3 days',
    query: 'is:inbox -is:sent newer_than:14d',
    filter: 'Has question mark + from known contact + no reply in thread'
  },
  {
    name: 'subscription_audit',
    description: 'Monthly recurring charges',
    query: 'subject:(receipt OR invoice OR subscription OR charged) newer_than:90d',
    aggregator: 'Group by sender, calculate monthly spend, flag unused'
  },
  {
    name: 'upcoming_deadline',
    description: 'Deadlines mentioned in emails',
    query: 'newer_than:30d',
    filter: 'Contains date reference + action word (due, deadline, expires, last day)'
  },
  {
    name: 'bill_reminder',
    description: 'Upcoming bill payments',
    query: 'subject:(statement OR bill OR payment due OR balance) newer_than:30d',
    aggregator: 'Extract due dates, amounts, flag <7 days away'
  },
  {
    name: 'shipping_tracking',
    description: 'Active package deliveries',
    query: 'subject:(shipped OR tracking OR delivery OR out for delivery) newer_than:14d',
    extractor: 'Extract tracking numbers, carrier, estimated dates'
  },
  {
    name: 'travel_monitor',
    description: 'Upcoming travel/flights',
    query: 'subject:(confirmation OR itinerary OR booking OR flight) newer_than:60d',
    extractor: 'Extract dates, flight numbers, hotel names, confirmation codes'
  }
];
```

### 3c. Agent Execution Pipeline

```
New email arrives (Gmail push / poll)
    ↓
[Classify] → Claude classifies status + sub-category
    ↓
[Route] → Based on status:
    │
    ├── NEED_YOU → Draft suggested action → Push to IM + Dashboard
    │
    ├── WATCHING → Add to watch list → Dashboard only (no push)
    │
    ├── DISCOVERY → Push to IM + Dashboard (Discoveries section)
    │
    ├── HANDLED → Auto-archive → Dashboard Handled section
    │
    └── NOISE → Skip → Dashboard Handled (collapsed)
    
    ↓
[Execute] → If user approves (via IM or Dashboard):
    → Agent sends reply / takes action
    → Move to HANDLED with execution log
```

---

## 4. Living Dashboard (Frontend)

### 4a. Tech Stack

```
Next.js 15 (App Router)
├── Tailwind CSS + shadcn/ui (components)
├── Framer Motion (card animations, swipe gestures)
├── Supabase Realtime (live updates)
├── next-pwa (installable PWA)
└── Vercel (hosting, free tier)
```

### 4b. Page Structure (Single Page Dashboard)

```typescript
// app/dashboard/page.tsx — THE page
export default function Dashboard() {
  return (
    <div className="min-h-screen bg-zinc-950">
      {/* Status Bar — always visible */}
      <StatusBar 
        total={63} 
        handled={54} 
        needYou={3} 
        active={2} 
      />
      
      {/* Need You — swipeable cards */}
      <Section title="Need You" count={3} color="red">
        <SwipeableCardStack items={needYouItems} />
      </Section>
      
      {/* In Progress */}
      <Section title="In Progress" count={2} color="blue">
        <ProgressList items={inProgressItems} />
      </Section>
      
      {/* Watching */}
      <Section title="Watching" count={4} color="amber">
        <CompactList items={watchingItems} />
      </Section>
      
      {/* Discoveries — persistent */}
      <Section title="Discoveries" count={2} color="purple" persistent>
        <DiscoveryCards items={discoveries} />
      </Section>
      
      {/* Handled — collapsed by default */}
      <Section title="Handled" count={54} color="green" collapsed>
        <HandledSummary items={handledItems} />
      </Section>
    </div>
  );
}
```

### 4c. Swipeable Card Component (Spark-inspired)

```typescript
// Key interaction: one card at a time, swipe to decide
interface NeedYouCard {
  id: string;
  from: string;
  summary: string;
  suggestedAction: string;
  agentDraft?: string;
  urgency: 'high' | 'medium' | 'low';
  subCategory: string;
  
  // Actions
  primaryAction: { label: string; fn: () => void };  // "Send Reply"
  secondaryAction: { label: string; fn: () => void }; // "Edit & Send"
  dismissAction: { label: string; fn: () => void };   // "Skip"
}
```

### 4d. Realtime Updates

```typescript
// Supabase Realtime subscription
useEffect(() => {
  const channel = supabase
    .channel(`dashboard:${userId}`)
    .on('postgres_changes', {
      event: '*',
      schema: 'public',
      table: 'classified_emails',
      filter: `user_id=eq.${userId}`
    }, (payload) => {
      // Live update dashboard without refresh
      updateEmailInState(payload);
    })
    .subscribe();
    
  return () => channel.unsubscribe();
}, [userId]);
```

---

## 5. Database Schema

```sql
-- Users
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  timezone TEXT DEFAULT 'America/New_York',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Email connections
CREATE TABLE email_connections (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  provider TEXT NOT NULL, -- 'gmail' | 'outlook' | 'imap'
  email_address TEXT NOT NULL,
  refresh_token_encrypted TEXT, -- encrypted at rest
  last_sync_at TIMESTAMPTZ,
  scan_status TEXT DEFAULT 'pending', -- 'pending' | 'scanning' | 'complete'
  scan_progress JSONB, -- { processed: 150, total: 500, discoveries: 3 }
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Classified emails (core table)
CREATE TABLE classified_emails (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  connection_id UUID REFERENCES email_connections(id),
  
  -- Gmail reference
  gmail_id TEXT,
  thread_id TEXT,
  
  -- Classification
  agent_status TEXT NOT NULL, -- 'need_you' | 'in_progress' | 'watching' | 'discovery' | 'handled' | 'noise'
  sub_category TEXT, -- 'reply' | 'confirm' | 'financial' | 'schedule' | 'deadline' | 'custom:...'
  summary TEXT NOT NULL,
  suggested_action TEXT,
  urgency TEXT DEFAULT 'medium',
  confidence FLOAT DEFAULT 0.8,
  
  -- Original data
  from_address TEXT,
  from_name TEXT,
  subject TEXT,
  body_preview TEXT, -- first 500 chars
  received_at TIMESTAMPTZ,
  
  -- Agent execution
  agent_draft TEXT,
  user_decision TEXT, -- 'approved' | 'modified' | 'rejected' | null
  executed_at TIMESTAMPTZ,
  execution_log JSONB, -- full audit trail
  
  -- Metadata
  classified_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- User profile (built from email analysis)
CREATE TABLE user_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID UNIQUE REFERENCES users(id),
  contact_graph JSONB, -- top contacts, frequency, relationship type
  spending_patterns JSONB, -- subscriptions, average amounts, merchants
  communication_style JSONB, -- response time, formality level, etc.
  interests JSONB, -- derived from email content
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Custom sub-categories (user-modifiable)
CREATE TABLE user_categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  name TEXT NOT NULL,
  emoji TEXT,
  is_default BOOLEAN DEFAULT FALSE,
  sort_order INT DEFAULT 0
);

-- Indexes
CREATE INDEX idx_emails_user_status ON classified_emails(user_id, agent_status);
CREATE INDEX idx_emails_user_received ON classified_emails(user_id, received_at DESC);
CREATE INDEX idx_emails_gmail ON classified_emails(gmail_id);
```

---

## 6. MVP Scope (4-Week Sprint)

### Week 1: Foundation

| Day | Task | Deliverable |
|-----|------|-------------|
| 1-2 | Project setup: Next.js + Supabase + Vercel | Deployed skeleton app with auth |
| 3-4 | Google OAuth + Gmail connection flow | "Connect Gmail" works, token stored |
| 5 | Email scan worker (Railway) | Background job fetches last 7 days |

### Week 2: Agent Brain

| Day | Task | Deliverable |
|-----|------|-------------|
| 1-2 | Claude classification pipeline | Emails classified into 5 statuses |
| 3 | Discovery engine (3 rules: forgotten reply, price increase, bill reminder) | First discoveries generated |
| 4-5 | Dashboard V1: Status Bar + Need You cards + Handled list | Core dashboard visible |

### Week 3: Interaction

| Day | Task | Deliverable |
|-----|------|-------------|
| 1-2 | Swipeable Need You cards + approve/reject flow | Users can act on cards |
| 3 | Agent execution: send reply on behalf | "Approve" actually sends email |
| 4-5 | Realtime updates (Supabase Realtime) + scan progress | Live dashboard, scan "开盲盒" |

### Week 4: Polish & IM

| Day | Task | Deliverable |
|-----|------|-------------|
| 1-2 | WhatsApp/Telegram push notifications for NEED_YOU items | IM layer working |
| 3 | PWA manifest + mobile optimization | Installable on phone |
| 4-5 | Beta onboarding flow + landing page | Ready for first 10 users |

### MVP Feature Cut

✅ **In MVP:**
- Gmail connection (OAuth2 testing mode, 100 user cap)
- Historical scan (90 days) with realtime progress
- AI classification (5 statuses)
- Living Dashboard (mobile-first PWA)
- Swipeable Need You cards
- Agent reply drafting + execution
- 3 Discovery rules
- WhatsApp OR Telegram push
- User profile summary

❌ **NOT in MVP (V2+):**
- Outlook/IMAP support
- Linq iMessage integration
- Custom sub-categories (user-editable)
- Watching section (passive monitoring)
- Web execution (API calls, browser actions)
- SMS data source (impossible on iOS anyway)
- Multi-language support
- Billing/payments

---

## 7. Cost Estimate (MVP Phase)

| Service | Cost | Notes |
|---------|------|-------|
| Vercel | $0 | Free tier (hobby) |
| Supabase | $0 | Free tier (500MB, 50K rows) |
| Railway | $5/mo | Worker process |
| Upstash Redis | $0 | Free tier |
| Claude API | ~$25/mo | 50 beta users × 20 emails/day |
| Domain | $12/yr | |
| **Total** | **~$30/month** | Until PMF, then scale |

---

## 8. Key Technical Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Gmail API verification takes >6 weeks | Blocks onboarding beyond 100 users | Start with IMAP App Password flow; apply for verification Day 1 |
| Claude classification accuracy <90% | Bad trust = dead product | Build confidence threshold; low-confidence items always go to NEED_YOU (safe default) |
| Email scan too slow (>5 min for 500 emails) | Bad onboarding experience | Batch + parallelize; show results incrementally; scan recent first |
| Rate limits (Gmail: 250 quota units/sec) | Can't scale beyond ~100 concurrent users | Implement exponential backoff; use push notifications instead of polling |
| User data security breach | Company-ending | Encrypt tokens at rest; RLS in Supabase; no raw email bodies stored (only previews + summaries) |

---

## 9. Repository Structure

```
dealism-consumer/
├── apps/
│   ├── web/                    # Next.js dashboard + API
│   │   ├── app/
│   │   │   ├── page.tsx        # Landing page
│   │   │   ├── auth/           # OAuth callbacks
│   │   │   ├── dashboard/      # THE dashboard
│   │   │   └── api/
│   │   │       ├── webhooks/   # Gmail push, WhatsApp
│   │   │       └── agent/      # Agent execution endpoints
│   │   ├── components/
│   │   │   ├── dashboard/
│   │   │   │   ├── StatusBar.tsx
│   │   │   │   ├── NeedYouCard.tsx
│   │   │   │   ├── SwipeStack.tsx
│   │   │   │   ├── ProgressList.tsx
│   │   │   │   └── HandledSection.tsx
│   │   │   └── onboarding/
│   │   └── lib/
│   │       ├── supabase.ts
│   │       ├── gmail.ts
│   │       └── claude.ts
│   └── worker/                 # Railway worker
│       ├── scan.ts             # Historical email scan
│       ├── classify.ts         # Claude classification
│       ├── discover.ts         # Discovery engine
│       ├── execute.ts          # Agent execution
│       └── push.ts             # IM notifications
├── packages/
│   ├── db/                     # Shared Supabase types + queries
│   └── shared/                 # Shared types + utils
├── supabase/
│   └── migrations/             # SQL migrations
├── turbo.json
└── package.json
```

---

## 10. Day 1 Commands

```bash
# Create monorepo
npx create-turbo@latest dealism-consumer

# Setup Next.js app
cd apps/web
npx create-next-app@latest . --typescript --tailwind --app --src-dir

# Add dependencies
pnpm add @supabase/supabase-js @supabase/ssr
pnpm add @anthropic-ai/sdk
pnpm add googleapis
pnpm add framer-motion
pnpm add -D @types/node

# Setup Supabase
npx supabase init
npx supabase db push

# Deploy
npx vercel deploy
```

---

*This document is the technical companion to product-summary-v1.md. Together they form the complete MVP spec. Next step: Leo reviews, we start Week 1.*
