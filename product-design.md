# Product Design Process

How user research in trade communities shaped WireHours from concept to shipped product.

---

## Research Methodology

Before writing any code, I spent significant time embedded in the communities where electrical apprentices discuss their work: Reddit's r/electricians and r/IBEW, ElectricianTalk forum, and Mike Holt Forum. The goal was to understand the problem space through the users' own words, not assumptions.

I documented exact language, emotional states, and recurring scenarios into a structured Voice of Customer research document. This research directly drove feature prioritization, UI decisions, and marketing messaging.

---

## From Pain Points to Features

### Pain Point → Feature Mapping

**"I forgot to write my time down so I have to guess at the end of the day"**
→ **Quick Log**: Designed to take under 30 seconds. Date defaults to today, category remembers last selection, hours is a simple number input. The goal is removing every excuse not to log immediately.

**"My employer won't sign off on my hours"**
→ **Digital Supervisor Verification**: Account-free, link-based verification that supervisors can complete in 2 minutes. Creates a timestamped record that exists independent of the employer relationship.

**"I didn't know holidays and travel time counted in my state"**
→ **State-Specific Category Tracking**: The app shows only the categories relevant to the apprentice's state, with built-in cap enforcement. No more guessing what counts.

**"Pay stubs aren't proof of categories"**
→ **State Board Form Export**: Generates documentation in the format each state board expects, pre-filled with verified hour data. Transforms raw time entries into compliance-ready paperwork.

**"If I quit, my boss may get spiteful and not sign off"**
→ **Portable Records with Employer History**: Hours belong to the apprentice, not the employer. Verified records follow the apprentice across every job change.

---

## User Personas

### Alex — First-Year Apprentice (Primary)
- Just started, overwhelmed by requirements
- Needs to understand what to track and why
- Values simplicity; will abandon anything complicated
- **Key moment:** Onboarding must immediately clarify their state's requirements

### Maria — Third-Year Apprentice (Power User)
- Has been tracking in a notebook, worried about gaps
- Changing employers and needs documentation
- Wants to import/backfill existing hour estimates
- **Key moment:** Seeing accurate progress toward licensure for the first time

### Dave — Electrical Contractor (B2B)
- Manages 8 apprentices, tracks their hours for compliance
- Currently uses spreadsheets or nothing
- Cares about audit readiness and reducing admin burden
- **Key moment:** Dashboard showing all apprentices' progress in one view

---

## Design Principles

1. **Field-first** — Every screen must work with dirty hands, bright sun, and 30 seconds of attention
2. **Trust through transparency** — Show exactly what the state requires and where the apprentice stands
3. **Progressive disclosure** — Simple by default, detailed when needed
4. **No dead ends** — Every state on the progress dashboard links to what the apprentice should do next
5. **Verification is a feature, not a burden** — Make it so easy that supervisors actually do it

---

## Information Architecture

```
Landing Page (SEO-optimized, conversion-focused)
├── State Guides (one per state, SEO entry points)
├── Blog (pain-point-driven content)
└── Pricing

App (authenticated)
├── Dashboard
│   ├── Progress Ring (total hours vs. requirement)
│   ├── Category Breakdown (with caps and status)
│   └── Recent Activity
├── Log Hours
│   ├── Quick Log (single entry)
│   └── Batch Log (week view)
├── Verification
│   ├── Pending Requests
│   ├── Request New Verification
│   └── Verification History
├── Documents
│   ├── Uploaded Docs (pay stubs, certs)
│   └── Generated Reports
├── Employers
│   ├── Current Employer
│   └── Employer History
└── Export
    ├── State Board Forms
    └── Progress Reports
```

---

## Key UX Decisions

### Progress Dashboard as Home Screen
The first thing an apprentice sees after login is their progress ring — a visual answer to "how close am I?" This creates an emotional anchor and a reason to keep logging.

### Category Selection, Not Free-Text
Hours are logged against predefined state categories (residential, commercial, industrial, etc.) via a dropdown — not free-text. This eliminates categorization errors that could invalidate hours during a state board review.

### Supervisor Verification via SMS
Trade workers are more responsive to text messages than email. The verification link is sent via SMS by default, with email as a fallback. The verification page is deliberately minimal — just the hours to review and a confirm/dispute action.

### Offline-First Hour Entry
The quick log form works fully offline. Entries are queued locally and synced when connectivity returns. A subtle badge shows "2 entries waiting to sync" so the apprentice knows their data is safe.

---

## Competitive Landscape

| Feature | WireHours | Generic Time Trackers | State Board Portals | Paper/Spreadsheets |
|---|---|---|---|---|
| State-specific categories | ✅ | ❌ | Partial | Manual |
| Works on any device | ✅ (PWA) | Varies | Desktop-only | N/A |
| Supervisor verification | ✅ (digital) | ❌ | ❌ | Paper signatures |
| Offline capable | ✅ | Rare | ❌ | ✅ (paper) |
| State form generation | ✅ | ❌ | N/A | Manual |
| Multi-employer tracking | ✅ | ❌ | ❌ | Error-prone |
| Cost | Free / $5.99 | $8-15/mo | Free (limited) | Free |

The key insight: generic time trackers don't understand trade licensing, and state board portals don't help with day-to-day tracking. WireHours occupies the space between daily logging and compliance documentation.
