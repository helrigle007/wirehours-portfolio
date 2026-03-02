# Technical Decisions

A walkthrough of the key technical decisions behind WireHours, the reasoning process, and what I'd reconsider in hindsight.

---

## 1. PWA Over Native Apps

**Decision:** Build a Progressive Web App instead of native iOS/Android apps.

**Context:** The electrical trade workforce skews heavily Android (~60%), and the primary competitor in the space requires iOS 17.6+ — immediately excluding a large portion of the target audience. Apprentices log hours on active job sites where conditions are rough (dirty hands, bright sun, unreliable cell service).

**Reasoning:**
- A single codebase serves every device and OS without app store gatekeeping
- Service workers enable offline hour logging with background sync — critical for job sites with poor connectivity
- Home screen installation makes the PWA feel native without the 2-week App Store review cycle
- Updates ship instantly via Vercel deploys; no waiting for users to update from the app store

**Tradeoff accepted:** No background GPS tracking. Mitigated with on-demand location stamps via the Web Geolocation API at the moment of logging. State boards require documented hours, not continuous location surveillance, so this tradeoff is acceptable for the compliance use case.

**What I'd reconsider:** If push notification engagement proves critical for retention (e.g., "Don't forget to log today's hours"), the PWA push notification support on iOS is still inconsistent. A React Native wrapper might become worthwhile once the user base justifies the maintenance overhead.

---

## 2. Supabase for the Data Layer

**Decision:** Use Supabase (hosted Postgres) for database, authentication, file storage, and real-time subscriptions.

**Context:** As a solo founder, I needed a backend that minimized infrastructure management while providing enterprise-grade data isolation for a multi-tenant compliance application.

**Reasoning:**
- **Row-Level Security (RLS)** is the critical feature — every database query is automatically scoped to the authenticated user at the Postgres level. Even if application code has a bug, one user's hour records can never leak to another user. For a compliance-focused product, this is non-negotiable.
- Auth is tightly integrated with the database layer, so RLS policies reference the authenticated user directly
- Supabase Storage handles document uploads (pay stubs, certifications) with the same RLS model
- Real-time subscriptions power the supervisor verification flow — when a supervisor signs off, the apprentice's dashboard updates immediately
- The free tier is generous enough for launch; scaling to $25/month is predictable

**Tradeoff accepted:** Vendor lock-in to Supabase's Postgres hosting. Mitigated by the fact that the underlying database is standard Postgres — if needed, I can migrate to any Postgres host (RDS, Neon, self-hosted) and only lose the convenience layer.

**What I'd reconsider:** For the supervisor verification flow specifically, a serverless function approach (edge functions) might have been cleaner than Supabase's built-in realtime. The real-time channel creates a persistent connection that's overkill for a one-time verification event.

---

## 3. Next.js App Router (Hybrid Rendering)

**Decision:** Use Next.js 14 with the App Router for a hybrid SSR/client-side architecture.

**Context:** WireHours serves two very different audiences from the same codebase: anonymous visitors landing on SEO content pages (state guides, blog posts) and authenticated apprentices using the hour-logging application.

**Reasoning:**
- **SEO pages** (state guides, blog posts) are server-rendered or statically generated — fast load times, proper meta tags, and full search engine indexability
- **App pages** (dashboard, hour logging, verification) use client-side rendering with React state management — responsive, interactive, and offline-capable via the service worker
- The App Router's layout nesting cleanly separates the marketing site shell from the authenticated app shell
- API routes colocated in the same project simplify the deployment topology

**Tradeoff accepted:** The App Router's learning curve and occasional hydration edge cases added development time compared to the Pages Router. The mental model of server components vs. client components requires discipline.

---

## 4. Tokenized Supervisor Verification (Account-Free)

**Decision:** Supervisors verify hours via a secure tokenized link — no account creation required.

**Context:** The biggest friction point in hour verification is getting supervisors to participate. Many are busy contractors or foremen who won't download an app or create an account just to sign off on an apprentice's hours.

**Reasoning:**
- The apprentice requests verification from within the app
- The supervisor receives an SMS (via Twilio) or email (via Resend) with a secure, time-limited link
- The link opens a minimal web page showing the hours to verify — no login, no account, no app install
- The supervisor reviews, confirms or disputes specific entries, and submits
- The verification is cryptographically timestamped and recorded

This reduces supervisor friction from "download app → create account → find the apprentice → review hours → confirm" to "tap link → review → confirm." The difference between a 10-minute process and a 2-minute process is the difference between supervisors who participate and supervisors who ignore the request.

**Security consideration:** Tokens are single-use, time-limited (72 hours), and scoped to a specific verification request. The supervisor can only see the hours they're being asked to verify, not the apprentice's full history.

---

## 5. State Compliance as a Structured Rule Engine

**Decision:** Model state licensing requirements as structured data rather than hardcoded business logic.

**Context:** Each U.S. state has different hour requirements, categories, caps, and documentation formats. California's "green sheet" process differs substantially from Washington's L&I requirements, which differ from Texas, which differ from Massachusetts.

**Reasoning:**
- State requirements are stored as configuration data (categories, hour thresholds, cap rules, form templates) rather than imperative code
- Adding a new state means adding a new data entry, not writing new business logic
- The compliance engine reads the state configuration and applies it uniformly — calculate progress, enforce caps, generate the appropriate export format
- This scales to 50 states without 50 branches of business logic

**Tradeoff accepted:** The initial data entry for each state requires manual research and validation against state board documentation. There's no authoritative API for state licensing requirements, so this is unavoidable human work.

**Future consideration:** Some states (Arizona, Florida, Illinois, Pennsylvania) don't have statewide licensing — requirements are set at the municipal level. Supporting these states requires a fundamentally different data model (city/county-level rules instead of state-level), which I've deferred to post-launch.

---

## 6. Stripe for Subscription Billing

**Decision:** Use Stripe with webhook-driven subscription management.

**Reasoning:**
- Webhook-driven architecture means the billing state is always in sync — subscription created, upgraded, canceled, or payment failed events all update the user's tier in the database
- Customer portal handles payment method updates, invoice history, and cancellation without custom UI
- Future expansion to the Contractor tier (per-seat pricing) is natively supported by Stripe's subscription model

**Implementation note:** Feature gating checks the user's subscription tier on every protected API route, not just at the UI level. This prevents circumventing Pro features by calling API endpoints directly.

---

## 7. Mobile-First Design for Field Conditions

**Decision:** Every interface is designed for one-handed phone operation in field conditions.

**Context:** Apprentices log hours on active construction sites — gloves, bright sunlight, dirty screens, time pressure.

**Design principles applied:**
- Large touch targets (minimum 48px) for gloved fingers
- High contrast UI that's readable in direct sunlight
- The "Quick Log" flow is optimized for minimum taps: date (defaults to today), category (remembers last used), hours, submit
- No horizontal scrolling, no small text, no hover-dependent interactions
- Offline queue with visual indicator — the apprentice sees "2 entries waiting to sync" and knows nothing is lost

---

## Infrastructure Cost Profile

One of the advantages of this stack is the cost curve. At launch:

| Service | Monthly Cost |
|---|---|
| Vercel (hosting) | $0 (free tier) |
| Supabase (database + auth) | $0 (free tier) |
| Stripe | 2.9% + $0.30 per transaction |
| Resend (email) | $0 (100/day free) |
| Twilio (SMS) | ~$0.01/message |
| Domain | ~$1/month |
| **Total** | **< $5/month at launch** |

The first paying customer is profitable from day one. The stack scales gracefully — Supabase's $25/month tier handles significant growth, and Vercel's pricing scales with traffic rather than requiring capacity planning.
