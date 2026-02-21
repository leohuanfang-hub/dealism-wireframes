# Dealism — Product Document

> **An always-on conversation agent that talks to people on your behalf.**
>
> Last updated: February 21, 2026

---

## 1. Vision

Most AI assistants operate software. Dealism operates conversations.

When a customer says "that's too expensive," a general-purpose agent doesn't know whether to hold firm or offer 20% off. Dealism does — because it understands your business rules, your style, your relationship with that customer, and the commercial judgment required to close.

**One-liner:** "General agents help you click buttons. Dealism helps you talk to people."

### Core Concept: Objective-Based Conversation Agent

Users state an objective — "reschedule my dentist," "negotiate this quote down," "follow up with that vendor" — and Dealism autonomously selects the best channel, drafts in the user's voice, negotiates within defined rules, and executes. The user approves when confidence is low; the agent handles the rest.

---

## 2. Market Opportunity

### The Data

| Metric | Email | SMS |
|--------|-------|-----|
| Consumer preference for brand comms | **75.4%** | 19.2% |
| Open rate | 36.8% | **98%** |
| Viewed within 5 min | ~20% | **82%** |
| Reply rate | 6% | **45%** |
| ROI | $36/$ spent | $71/$ spent |

*Sources: EmailVendorSelection 2026, SimpleTexting 2025 (n=1,400), EZTexting 2025*

### The Gap

> **53% of consumers who text a business never get a reply.** *(TextMagic 2025, n=1,800)*
> Consumers expect a response within **15 minutes**. *(EZTexting 2025)*
> Leads contacted within **5 minutes** convert at **21× the rate** of those contacted after 30 min.

Businesses can't keep up. Consumers are waiting. This is where an agent fits.

### Channel Paradox

- **Phone number = urgency** — appointments, customer service, logistics. Consumers expect instant response.
- **Email = record-keeping** — confirmations, promotions, receipts. Lower urgency.
- Businesses *want* SMS for its 98% open rate, but can't staff real-time SMS replies.

**Result:** The highest-value channel (SMS) is the most under-served. An AI agent that responds instantly, across channels, in the business's own voice — that's the unlock.

---

## 3. Target Users

### ICP 1: Consumer — "Life Manager"

**Profile:** North American professionals, age 25–45, handling 50–200 emails + 20–50 SMS/IM messages daily across Email, SMS, WhatsApp, and Instagram.

**Pain points:**
- Information overload — important items buried in noise
- Time wasted on routine business communication (rescheduling, confirming, following up)
- Things fall through the cracks

**What Dealism does:**
- 📥 **Classifies** all incoming messages by agent-determined status (not by type)
- 📲 **Escalates** urgent items via IM push (WhatsApp/iMessage)
- 🤖 **Drafts replies** in the user's voice — user approves or edits
- 🔄 **Executes tasks** end-to-end ("cancel that subscription," "reschedule the dentist")
- 💡 **Proactively discovers** hidden action items (forgotten replies, price hikes, expiring returns)

**Pricing:** $9–19/mo

### ICP 2: Business — "AI Front Desk"

**Profile:** North American local service businesses (1–10 people) — salons, dental offices, real estate, home services.

**Pain points:**
- Multi-channel messages going unanswered (53% ignored)
- Slow replies = lost deals (5 min vs 30 min = 21× conversion gap)
- No-shows eating revenue
- No bandwidth for follow-ups

**What Dealism does:**
- 💬 **Unified multi-channel replies** — SMS, Email, IG DM, WhatsApp from one agent
- 🧠 **Smart channel routing** — urgent via SMS, detailed via Email
- 📅 **Appointment management** — check calendar → reply → confirm → remind
- 🗣️ **Style learning** — replies sound like the business owner, not a bot
- 💰 **Negotiation** — handles pricing discussions within owner-defined rules
- ⭐ **Post-service** — satisfaction surveys → Google Review nudges

**Pricing:** $49–149/mo

### Shared Engine vs. Unique Capabilities

| Shared | Consumer-only | Business-only |
|--------|--------------|---------------|
| Style Engine | Email analysis/classification | Negotiation Engine |
| Channel Router | Channel escalation alerts | Appointment management |
| Classifier | Proactive discoveries | Unified inbox |
| Context Engine | | Review collection |

---

## 4. Core Differentiation — The Negotiation Engine

This is the soul of the product. General agents automate deterministic workflows (open page → click button → fill form). Dealism handles **non-deterministic conversations** that require commercial judgment, tone, and relationship awareness.

### Architecture

```
Incoming message
       ↓
┌─────────────────────┐
│  Intent Recognition  │  "Is this a price inquiry, complaint, booking, negotiation?"
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│   Context Engine     │  Cross-channel memory: history, preferences, VIP status
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│    Rules Engine      │  Owner-defined business rules (min discount, hours, policies)
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│  Negotiation Logic   │  LLM + Rules → strategy: hold price / offer alternative / create urgency
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│    Style Engine      │  Render in owner's voice: "Hey girl! 💕" vs "Dear Customer,"
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│  Confidence Scoring  │  HIGH (>0.85): auto-send | MED: summary confirm | LOW: full handoff
└──────────┬──────────┘
           ↓
      Send or confirm
```

### Five Core Scenarios

**1. Price Negotiation**
Customer asks for gel nails ($55). Asks "Can you do $40?" → Agent checks rules (new customer floor: 10% off = $49.50), offers $50 with free nail art as new-client special. Confidence: HIGH → auto-send.

**2. Appointment Rescheduling**
Customer emails to reschedule Saturday → Sunday. Agent checks calendar (11am and 3pm open), checks rules (24h reschedule fee for regulars: waived for 5+ visits), replies via SMS for faster confirmation. Confidence: HIGH → auto-send.

**3. New Lead Follow-up**
New lead submits form. Agent sends SMS within 5 minutes (82% view rate). If no reply in 2h, sends detailed email. If still silent after 24h, gentle SMS follow-up. Confidence: HIGH for first touch, MED for subsequent.

**4. Complaint Handling**
Customer messages on IG DM about lash extensions falling out. Agent checks context (loyal customer, 3 prior visits, all positive), generates empathetic response asking for photos. Confidence: MED → sends summary to owner for approval before responding.

**5. Upsell**
Customer books basic manicure ($35) but previously got gel ($55). Agent detects downgrade, piggybacks upsell onto 24h reminder: "Btw, gel is $45 this week instead of $55 — want me to switch it?" Confidence: HIGH → auto-send.

### Trust-Building Path

| Phase | Behavior | Trigger |
|-------|----------|---------|
| Week 1–2 | All replies require owner confirmation | Default |
| Week 3–4 | HIGH-confidence auto-sends; MED+LOW need approval | Accuracy >90% |
| Month 2+ | HIGH+MED auto-send; only LOW needs approval | Accuracy >95% |

Weekly transparency report: "Handled 127 messages. 98 auto-replied. 24 confirmed. 5 escalated. 0 complaints."

---

## 5. Product Design

### The Third State: Informed Passivity

Existing paradigms both fail:
- **Legacy (Email/SMS):** Complete but noisy — you see everything, waste time on nothing
- **Simple AI (IM bot):** Convenient but anxiety-inducing — too lightweight, no global view, no verification

**Dealism introduces a third state:** the peace of mind of a great executive assistant. You don't need to see everything — you need to know *everything is being seen*, and you can verify anytime.

> Trust doesn't come from "seeing it all." It comes from "every time I checked, the agent handled it perfectly."

### Two-Layer Interaction

**Layer 1 — IM (WhatsApp / iMessage / Telegram)**
- Daily interaction surface
- Decision pushes: "Your dentist wants to reschedule — Thursday 2pm or Friday 10am?"
- Task commands: "Follow up with the contractor"
- Proactive discoveries: "Spotify just raised your price from $10.99 to $16.99"

**Layer 2 — Living Dashboard (Web PWA)**
- Trust layer — full visibility, agent-status-based organization
- Design philosophy: **5 seconds to see the big picture. 30 seconds to verify details. Then close it.**

```
┌─────────────────────────────────────────┐
│  STATUS BAR                             │
│  63 messages today                      │
│  54 handled · 3 need you · 2 active     │
├─────────────────────────────────────────┤
│  🔴 NEED YOU          (card stack)      │
│  → Swipe/tap to decide, one at a time   │
│  💬 Reply | ✅ Confirm | 💰 Finance     │
│  📅 Schedule | 🔔 Deadline              │
├─────────────────────────────────────────┤
│  🔄 IN PROGRESS                         │
│  "Dentist reschedule → email sent,      │
│   waiting for reply (1h ago)"           │
├─────────────────────────────────────────┤
│  👀 WATCHING                            │
│  📦 Amazon package | 💳 CC payment      │
├─────────────────────────────────────────┤
│  💡 DISCOVERIES        (persistent)     │
│  Agent-surfaced insights that don't     │
│  get buried like IM messages            │
├─────────────────────────────────────────┤
│  📦 HANDLED            (collapsible)    │
│  Transparent archive — nothing deleted  │
└─────────────────────────────────────────┘
```

### Design Principles

| # | Principle | Inspiration |
|---|-----------|-------------|
| 1 | **Full visibility** — every message counted, none missing | SaneBox Daily Digest |
| 2 | **Organized by agent status** — not message type, but "do I need to act?" | Superhuman Split Inbox |
| 3 | **Process transparency** — not a black box | Apple notification summary (counter-example) |
| 4 | **Drill-down available** — 99% won't click, but knowing they *can* builds trust | Apple notification summary (counter-example) |
| 5 | **Discoveries persist** — don't scroll away like IM messages | Original |
| 6 | **Card + quick decision** — one decision at a time, minimal cognitive load | Spark Cards + Tinder Swipe |
| 7 | **Handled = archived, not deleted** — transparent record | Things 3 Logbook |

### Cold-Start Strategy (Consumer)

**Phase 1 (0–5 min): Historical scan = instant value**
- Connect inbox → immediate scan of recent emails
- Not organizing — *discovering* missed action items
- Real-time push: each discovery appears live (blind-box experience)
- Target: 3+ valuable discoveries within 5 minutes

**Phase 2 (5–30 min): Profile modeling = show understanding**
- Contact graph, spending habits, communication patterns
- Push "here's what I learned about you" summary
- User corrections → agent learning

**Phase 3 (30 min+): Always-on = core value loop**
- New messages → classification → action pipeline
- Even without new messages: 1–2 low-frequency discoveries per day from historical data

### Seven Discovery Scenarios (Day 1)

| # | Discovery | Example | Value |
|---|-----------|---------|-------|
| 1 | 💰 Abnormal charge | Spotify $10.99 → $16.99 | Save money |
| 2 | 📅 Overdue checkup | Last physical: 11 months ago | Health |
| 3 | 📦 Expiring return window | Amazon return: 4 days left | Prevent loss |
| 4 | 🔄 Subscription audit | 14 newsletters, only open 3 | Reduce noise |
| 5 | 💳 Bill reminder | Credit card due in 3 days: $2,340 | Avoid late fee |
| 6 | ✉️ Forgotten reply | 2 emails waiting since last week | Social/work debt |
| 7 | ✈️ Travel monitoring | Flight UA835 on 3/15 | Trip management |

---

## 6. Competitive Landscape

### Three-Layer Matrix

**Layer 1: Incumbents (at scale)**

| Company | Revenue | Channels | AI | Weakness |
|---------|---------|----------|----|----------|
| Podium | $389M | SMS + Webchat | AI Employee (booking-focused) | No WhatsApp/IG, $399/mo |
| GoHighLevel | ~$200M+ | SMS+Email+WA+Webchat+Phone | Workflow automation | Unstable, requires technical setup |
| Intercom | $343M | Webchat+Email+WA | Fin AI (support-focused) | No IG, B2B-oriented |

**Layer 2: Messaging-first (strong channels, weak AI)**

| Company | Channels | AI | Weakness |
|---------|----------|----|----------|
| ManyChat | IG+WA+FB+SMS+Email | Keyword triggers (toy-level) | Not a real agent |
| Respond.io | WA+IG+FB+TG+Email+SMS | Draft suggestions | No autonomous execution |

**Layer 3: AI-native (strong AI, narrow channels)**

| Company | AI Capability | Channels | Weakness |
|---------|--------------|----------|----------|
| 11x.ai ($50M raised) | AI SDR | Email only | Single channel |
| Bland.ai ($22M raised) | AI phone agent | Phone only | Single channel |

*Sources: Sacra (Podium/Intercom), industry estimates (GHL/ManyChat), CB Insights*

### The White Space

**Nobody does WhatsApp + IG DM + Email + SMS + real AI agent simultaneously.**

- Podium: no WhatsApp/IG
- ManyChat: has channels, AI is a toy
- 11x/Bland: strong AI, single channel
- GoHighLevel: closest, but users build their own workflows

### Three Defensibility Moats

1. **Always-on vs. on-demand** — General agents come when called. Dealism is always running.
2. **Accumulated context depth** — "Reschedule my dentist" needs 3 words because the agent already knows everything.
3. **Trust over time** — A 3-month-old agent vs. a freshly opened general agent. Context compounds.

---

## 7. Technical Architecture (Summary)

Full technical document: [`2026-02-21-dealism-technical-architecture.md`](./2026-02-21-dealism-technical-architecture.md)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Language | TypeScript (full stack) | One team, one language |
| Framework | Next.js 15 (App Router) | SSR PWA dashboard + API routes |
| Database | PostgreSQL (Supabase) | Auth, RLS, Realtime subscriptions |
| Cache/Queue | Redis (Upstash) | Email processing queue, rate limiting |
| AI | Claude API (Anthropic) | Best at nuanced classification + tool use |
| Email | Gmail API → IMAP fallback | Gmail for 90%, IMAP for rest |
| IM Layer | WhatsApp Business API + Telegram Bot | V1 channels; iMessage (Linq) = V2 |
| Hosting | Vercel (dashboard) + Railway (agent worker) | Free tier + persistent worker |
| Auth | Supabase Auth + Google OAuth | One-click Gmail connect |

### Data Flow

```
Email arrives → Gmail Push / IMAP poll (60s)
    → Classification (Claude): need_you | in_progress | watching | discovery | handled
    → Store in Postgres + emit Realtime event
    → Dashboard updates live
    → If urgent: push to IM (WhatsApp/Telegram)
    → If actionable: generate draft + suggested action
    → User approves → Agent executes via appropriate channel
```

---

## 8. Roadmap

### Phase 1 (Week 1–2): Business MVP — "AI Front Desk v0.1"

- SMS channel (Twilio) + Channel Router (rule-based)
- Basic Rules Engine (conversational setup, not forms)
- All replies require owner confirmation (trust-building)
- Beta with 3–5 businesses from existing IG outreach contacts
- **Validate:** Will business owners let an AI reply to their customers?

### Phase 2 (Week 3–4): Consumer MVP — "Email Manager v0.1"

- Email classification engine + Daily Digest (Telegram/iMessage push)
- Channel escalation (important email → SMS notification)
- Basic auto-reply (high-confidence only)
- Product Hunt / Hacker News cold start
- **Validate:** Will consumers pay $9/mo for email classification + AI reply?

### Phase 3 (Week 5–8): Core Engine — "Negotiation v1.0"

- Style Engine implementation + all-channel integration
- Context Engine (cross-channel customer memory)
- Negotiation Logic (LLM + Rules)
- Confidence threshold: transition from "confirm everything" to "partial auto"
- **Validate:** Can agent reply quality meet business owner standards?

### Phase 4 (Month 3+): Dual-Side Flywheel + Growth

- Business-side data trains consumer-side (reply patterns, common Q&A)
- Calendar integration + full appointment automation
- Review collection + post-service automation
- Pricing validation + paid conversion

---

## 9. Open Questions

| # | Question | Status |
|---|----------|--------|
| 1 | Business-first or consumer-first? | Leaning business (existing contacts), but consumer may cold-start easier |
| 2 | Pricing model? | Per-channel? Per-message volume? Flat monthly? |
| 3 | Both ICPs simultaneously? | Or focus one, validate, then expand? |
| 4 | SMS cost structure (Twilio) | High-frequency business use needs cost modeling |
| 5 | Consumer IM entry point | WhatsApp? iMessage? Telegram? Own app? |
| 6 | Dashboard minimum form | How minimal can it be for MVP and still build trust? |

---

## 10. Channel Feasibility

### Consumer Side (Reading User Data)

| Channel | Feasible | Method | Notes |
|---------|:--------:|--------|-------|
| Gmail | ✅ | Gmail API (OAuth2) or IMAP + App Password | Restricted scope needs annual audit; App Password for MVP |
| Outlook | ✅ | Microsoft Graph API or IMAP | Same approach |
| Other email | ✅ | IMAP + App Password | Universal fallback |
| SMS inbox (iOS) | ❌ | Apple blocks all 3rd-party access | Hard limitation |
| iMessage inbox | ❌ | Apple blocks all access | Hard limitation |
| SMS inbox (Android) | ⚠️ | Must become default SMS app | Google Play review too strict |

### IM Interaction Layer (Agent ↔ User)

| Channel | Feasible | Method | Notes |
|---------|:--------:|--------|-------|
| WhatsApp | ✅ | WhatsApp Business API | Already operational |
| iMessage (blue bubble) | ✅ | Linq API | $20M Series A (2025); used by Poke AI |
| SMS (outbound) | ✅ | Twilio | Notifications and push |
| Telegram | ✅ | Bot API | Already operational |

**V1 conclusion:** Email-only as data source (covers 60–70% of high-frequency tasks). IM layer for agent ↔ user interaction. SMS/iMessage reading deferred.

---

*This is a working document. The soul of the product is the Negotiation Engine (Section 4) — everything else can iterate, but that direction needs to be locked first.*

*For technical implementation details, see the [Technical Architecture Document](./2026-02-21-dealism-technical-architecture.md).*
