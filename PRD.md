# ReCert
## Product Requirements Document
### Eligibility Recertification Management for Food Banks & Pantries

| Field | Value |
|---|---|
| Author | Katie Silorio |
| Version | 1.0 |
| Date | March 2026 |
| Status | Draft — Pending Validation |

---

## 1. Executive Summary

ReCert is a purpose-built eligibility recertification management application for food banks and food pantries. It solves a specific, validated problem: eligible families lose access to food assistance because the recertification process fails them, not because they no longer qualify.

Food banks manage eligibility across multiple federal and local programs (TEFAP, CSFP, SNAP, and their own pantry programs), each with different renewal cadences, verification methods, and compliance requirements. Today, this is tracked through paper forms, filing cabinets, and spreadsheets. The result is predictable: clients lapse, participation numbers drop, funder reports look worse than reality, and staff burn hours on preventable administrative work.

ReCert replaces this with a lightweight, mobile-first system that automates reminders, enables client self-service recertification, gives volunteers instant visibility at check-in, and provides program managers with audit-ready reporting. It is not a full pantry management system — it is laser-focused on the recertification lifecycle and designed to complement existing tools like Link2Feed, PantrySoft, and Oasis Insight.

> **Validated pain point:** The head of Project DASH (DoorDash's food bank delivery initiative) confirmed that recurring eligibility renewal is a significant operational burden for food bank partners across the country.

---

## 2. Problem Statement

### 2.1 The Recertification Gap

Food banks are required to periodically verify that the households they serve still meet eligibility criteria. Depending on the program and state, this happens as frequently as every distribution visit or as infrequently as every three years. The complexity comes from the fact that a single food bank may operate multiple programs simultaneously, each with its own rules.

The core failure mode is not eligibility — it's process:

- Clients don't know when their certification expires until they show up and are turned away
- Staff track deadlines in spreadsheets with no automated alerts
- Recertification requires an in-person visit during limited operating hours, creating barriers for working families
- Paper records make compliance reporting manual and error-prone
- Volunteers at the point of service can't easily determine a client's certification status
- When clients lapse, food banks undercount their service population, which impacts funding

### 2.2 The Regulatory Landscape

Eligibility requirements vary by program and state, creating a matrix of rules that food banks must navigate:

| Dimension | TEFAP | CSFP | Local Pantry |
|---|---|---|---|
| Recert Frequency | Per visit to every 3 years (state-dependent) | Every 6 months | 6 or 12 months |
| Income Threshold | State-specific (185–225% FPL) | 130% FPL | Up to 200% FPL |
| Categorical Eligibility | SNAP, WIC, TANF, Medicaid, SSI | SNAP, TANF, SSI | Configurable |
| Verification Method | Self-declaration allowed | Documentation may be required | Varies by org |
| ID Required | No (federal rule) | Varies | Varies |
| Record Retention | 5 years | 3 years | Varies |
| E-Signature Accepted | Yes (if equivalent to paper) | Varies by state | Yes |

ReCert must be configurable enough to handle this variability without requiring custom development for each food bank.

---

## 3. Competitive Landscape

The food bank technology market has several established players, but none treat recertification as a first-class workflow. Recertification is typically a field inside a larger case management or distribution tracking system, not a managed lifecycle with proactive outreach.

| Tool | Primary Focus | Recert Capability | Gap |
|---|---|---|---|
| Link2Feed | Client intake, TEFAP/CSFP compliance reporting, case management | Tracks eligibility status as a data field | No proactive outreach, no client self-service, no renewal workflow |
| PantrySoft | Check-in, client ordering, inventory | Flags expired status at check-in | Reactive only; client learns at pickup. No reminders. |
| Oasis Insight | Distribution tracking, reporting, barcode scanning | Stores expiration dates in client records | No automated reminders; UI reported as cumbersome |
| Pantry Saver | Intake, kiosk check-in, scale integration | Interview scheduling | Not focused on the renewal lifecycle as a managed process |

### 3.1 ReCert's Positioning

ReCert is not a pantry management system. It does not handle inventory, distribution logistics, or ordering. It is a recertification lifecycle management tool that complements existing systems. This deliberate narrowness is the product strategy — a focused tool that does one thing exceptionally well, rather than a broad platform that does many things adequately.

The integration strategy is to work alongside whatever system a food bank already uses, accepting client data via CSV import or (in later versions) API integration, and exporting compliance reports in formats compatible with existing workflows.

---

## 4. Target Users & Personas

ReCert serves three distinct user types with different needs, technical abilities, and interaction patterns.

### 4.1 Maria — Program Manager

| Field | Detail |
|---|---|
| Role | Program Manager at a mid-size regional food bank |
| Demographics | 42 years old, manages 3 staff + 20 volunteers, oversees TEFAP + CSFP + local pantry program |
| Tech Comfort | Moderate — uses Excel, email, and basic database tools. Not afraid of new software but has limited time to learn. |
| Goals | Keep clients enrolled without gaps; produce accurate reports for USDA and funders; reduce staff time on administrative tasks; increase program participation rates. |
| Pain Points | Tracks recertification deadlines across programs in a spreadsheet. No visibility into which clients are expiring until it's too late. Spends hours pulling reports manually. Worries about audit compliance. |
| Quote | *"We lose people every month who still qualify. They just didn't know they needed to re-sign paperwork. Then I have to explain to our funder why participation dropped."* |
| Primary Interaction | Dashboard, reporting, program configuration, outreach management |

### 4.2 James — Intake Volunteer

| Field | Detail |
|---|---|
| Role | Weekly volunteer at a food pantry partner agency |
| Demographics | 67 years old, retired teacher, volunteers 2 shifts per week. Handles client check-in and intake paperwork. |
| Tech Comfort | Low to moderate — comfortable with a tablet, but gets frustrated with complex interfaces. Needs large text and clear instructions. |
| Goals | Get clients checked in quickly and correctly. Know at a glance if someone needs to recertify. Avoid compliance mistakes. |
| Pain Points | Paper forms are confusing and inconsistent. Can't tell if a client's certification is current without flipping through a filing cabinet. Often has to turn people away. |
| Quote | *"I hate telling someone they can't get food today because their form expired. They drove 30 minutes to get here."* |
| Primary Interaction | Check-in screen, guided recertification wizard |

### 4.3 Rosa — Food Bank Client

| Field | Detail |
|---|---|
| Role | Mother of 3, works part-time, receives TEFAP food boxes monthly |
| Demographics | 34 years old, Spanish-speaking (bilingual), limited transportation, smartphone is her primary device. |
| Tech Comfort | Moderate — comfortable with texting and WhatsApp. Will fill out a form on her phone if it's simple. |
| Goals | Keep receiving food assistance without disruption. Complete paperwork as quickly as possible. Not miss work or arrange childcare for administrative visits. |
| Pain Points | Doesn't know when eligibility expires. Found out at pickup she was "expired" and left without food. Forms are in English and confusing. Must come in person during work hours. |
| Quote | *"I went to pick up my food box and they said my papers were expired. I didn't even know. I had to come back another day and I missed work."* |
| Primary Interaction | SMS reminders, mobile recertification form |

---

## 5. User Flows

### 5.1 Client Self-Service Recertification

**Persona:** Rosa (Client)  
**Trigger:** System detects Rosa's certification expires in 30 days.  
**Value:** Clients recertify from their phone in under 5 minutes without visiting the food bank.

#### Steps

1. **Automated Reminder (30 days out)** — Rosa receives an SMS in Spanish: *"Your food bank eligibility expires on [date]. Tap here to renew in 5 minutes: [link]"*
2. **Mobile Recertification Form** — Rosa taps the link and sees a pre-filled form with her existing information (name, address, household size). She only needs to confirm or update what's changed.
3. **Eligibility Self-Declaration** — Rosa confirms her income bracket or selects categorical eligibility ("I receive SNAP benefits"). No documentation upload required for TEFAP self-declaration.
4. **E-Signature & Acknowledgment** — Rosa signs electronically and acknowledges the USDA non-discrimination statement.
5. **Confirmation** — Instant confirmation: *"You're renewed through [new date]! See you at your next pickup."*
6. **Staff Notification** — Maria's dashboard updates in real time. Rosa's status moves from "Expiring Soon" to "Active."

#### Reminder Escalation Schedule

| Timing | Channel | Message Tone |
|---|---|---|
| 30 days before | SMS + Email | Informational: "Your eligibility expires soon" |
| 14 days before | SMS | Nudge: "Renew now to avoid interruption" |
| 3 days before | SMS + Phone option | Urgent: "Final reminder + call us for help" |
| Day of expiration | SMS | Action: "Expired today — you can still renew" |
| 7 days after | Staff outreach | Flagged on dashboard for manual follow-up |

#### Edge Cases

- Client has no phone number on file → Flagged for staff outreach at next visit
- Client's income has changed and no longer qualifies → Form shows ineligibility message with referral to other resources
- Client needs to upload documentation (non-TEFAP programs) → Form includes optional photo upload from phone camera
- Link expires before client completes form → New link sent; old link redirects to "request new link" page

### 5.2 Volunteer-Assisted Walk-In Recertification

**Persona:** James (Volunteer) assisting Rosa (Client)  
**Trigger:** Client arrives for food pickup. System flags their status at check-in.  
**Value:** Client recertifies on the spot in under 3 minutes and receives food without a return visit.

#### Steps

1. **Client Lookup** — James searches by name, phone, or household ID. System shows a clear status badge: **ACTIVE** / **EXPIRING SOON** / **EXPIRED**.
2. **One-Tap Recertify** — If expired or expiring, James taps "Recertify Now" to launch a guided wizard.
3. **Guided Wizard** — Three screens: (1) Confirm/update household info, (2) Verify income or categorical eligibility, (3) Capture e-signature on tablet.
4. **Language Toggle** — James switches the form language with one tap so the client can read what they're signing.
5. **Instant Completion** — Recertification processed immediately. Client proceeds to receive food.
6. **Record Stored** — Digital record auto-saved with timestamp, volunteer ID, and e-signature. No paper filing.

#### Design Requirements for Volunteer Experience

- Large, high-contrast UI elements (WCAG AA minimum)
- Maximum 3 taps to complete a recertification from check-in
- Status badges use color + icon + text (not color alone)
- Offline capability with sync when connectivity returns
- No individual login required — device-based authorization or simple 4-digit PIN
- Tablet-optimized layout (10" screen primary target)

### 5.3 Program Manager Dashboard & Reporting

**Persona:** Maria (Program Manager)  
**Trigger:** Daily workflow — Maria checks the dashboard at the start of each day and generates reports monthly/quarterly.  
**Value:** Real-time visibility into recertification pipeline; one-click compliance reporting.

#### Dashboard Components

- **Status Overview** — At-a-glance metrics: total active clients, expiring in 30/14/7 days, expired in last 30 days, lapsed (60+ days). Donut chart visualization.
- **Expiration Queue** — Sortable/filterable list of clients with upcoming expirations. Filters: program, date range, outreach status (reminder sent, no response, completed).
- **Recertification Funnel** — Conversion metrics: reminders sent → form opened → form completed → renewed. Broken down by channel (self-service vs. walk-in vs. phone).
- **Outreach Tracker** — Per-client view: reminders sent, response status, follow-up needed. One-click to trigger manual outreach (call, additional SMS).

#### Reporting & Compliance

- **TEFAP/CSFP Compliance Report** — Auto-generated, formatted for USDA/state agency submission. Active client count, verification method breakdown, certification dates.
- **Funder Impact Report** — Exportable summary: households served, recertification rate, lapse prevention metrics.
- **Audit Trail** — Every action timestamped and logged: who completed it, when, channel, e-signature. Retained per program-specific requirements (3–5 years).
- **Export Options** — CSV for integration with existing systems; PDF for physical filing or board reports.

### 5.4 New Client Intake & Initial Certification

**Persona:** James (Volunteer) or Maria (Staff)  
**Trigger:** New client arrives at the food bank for the first time.  
**Value:** Digitized intake that automatically sets recertification schedule and enables future self-service renewal.

#### Steps

1. **Client Information** — Name, address (self-declared), household size, preferred language, contact method (phone/SMS/email).
2. **Eligibility Determination** — Income-based: select income bracket against posted guidelines. Or categorical: indicate participation in SNAP, WIC, TANF, Medicaid, or SSI.
3. **Program Enrollment** — Staff selects programs. System auto-calculates next recertification date based on program rules.
4. **USDA Acknowledgments** — Non-discrimination statement and rights/responsibilities presented and acknowledged.
5. **E-Signature** — Client signs on tablet or phone.
6. **Notification Opt-In** — Client provides phone number and opts into SMS reminders. This is the critical step that enables the proactive renewal flow.

#### Data Model for Intake

| Field | Type | Required | Notes |
|---|---|---|---|
| Full Name | Text | Yes | Primary + any aliases |
| Address | Text | No (self-declared residency) | TEFAP: no proof required |
| Household Size | Integer | Yes | Determines income threshold |
| Phone Number | Phone | Strongly encouraged | Required for SMS reminders |
| Preferred Language | Dropdown | Yes | EN, ES (MVP); expandable |
| Income Bracket | Dropdown | If not categorically eligible | Based on FPL guidelines |
| Categorical Program | Multi-select | If not income-based | SNAP, WIC, TANF, etc. |
| Programs Enrolled | Multi-select | Yes | Sets recert schedule |
| Authorized Proxy | Text | No | Person authorized to pick up on behalf |
| E-Signature | Image/Touch | Yes | Stored with timestamp |
| SMS Opt-In | Boolean | No | Enables proactive reminders |

---

## 6. Feature Requirements

### 6.1 Must Have (MVP)

These features define the minimum viable product. Without any one of these, the core value proposition is broken.

- **Client Database with Certification Status** — Store household records with real-time status tracking: Active, Expiring Soon (configurable window), Expired, Lapsed. Status computed automatically from certification dates and program rules.
- **Configurable Program Rules Engine** — Admin can define: recertification frequency, income thresholds by household size, categorical eligibility programs, verification method (self-declaration vs. documentation), and record retention period. Per-program configuration.
- **Automated SMS Reminders** — Scheduled reminders at configurable intervals before expiration (default: 30, 14, 3 days + day-of + 7 days post). Bilingual (EN/ES). Includes unique recertification link.
- **Mobile Client Self-Service Form** — Pre-filled, mobile-optimized recertification form accessible via unique link (no login required). Supports self-declaration of income, categorical eligibility selection, and e-signature. Bilingual EN/ES.
- **E-Signature Capture** — Touch-based signature on mobile and tablet. Stored with timestamp, IP/device info, and associated certification record. Meets TEFAP electronic record requirements.
- **Volunteer Check-In Screen** — Client lookup by name/phone/ID with prominent status badge. One-tap access to recertification wizard. Language toggle. Tablet-optimized.
- **Program Manager Dashboard** — Status overview, expiration queue, completion rate, and outreach tracker as described in Flow 5.3.
- **USDA Non-Discrimination Compliance** — Required statement displayed during intake and recertification. Acknowledgment recorded.

### 6.2 Should Have (V2)

- TEFAP/CSFP formatted compliance reports auto-generated for USDA and state agency submission
- CSV/PDF export for integration with existing tools and physical filing requirements
- Additional language support: Chinese, Vietnamese, Tagalog, Arabic, Haitian Creole (based on food bank network demographics)
- Authorized proxy management — track who is authorized to pick up food and recertify on behalf of a household member
- Offline mode — volunteer check-in and recertification functions work without internet; data syncs when connectivity returns
- Email reminders to supplement SMS for clients who prefer it
- Bulk CSV import to onboard existing client databases from spreadsheets or other systems

### 6.3 Nice to Have (Future)

- API integration — bidirectional sync with Link2Feed, Oasis Insight, PantrySoft, and Pantry Saver
- Feeding America MealConnect integration
- Multi-site management for food banks coordinating across multiple partner pantry agencies
- Benefits screening — surface additional programs the client may qualify for during recertification
- Project DASH delivery integration — sync eligibility status so only certified clients are in the delivery queue
- WhatsApp reminders for populations that prefer WhatsApp over SMS
- Voice call reminders for clients without smartphones

---

## 7. Non-Functional Requirements

### 7.1 Accessibility

- WCAG 2.1 AA compliance minimum
- Large touch targets (48px minimum) for tablet/mobile interfaces
- Screen reader compatible
- Status indicators use color + icon + text (never color alone)
- Simple, plain-language copy at a 6th-grade reading level

### 7.2 Performance

- Client-facing form loads in < 3 seconds on 3G connections
- Volunteer check-in search returns results in < 1 second
- Dashboard loads in < 5 seconds with 10,000+ client records
- SMS delivery within 60 seconds of scheduled send time

### 7.3 Security & Privacy

- All data encrypted at rest (AES-256) and in transit (TLS 1.3)
- PII handled per USDA TEFAP privacy requirements
- Role-based access control: Admin, Staff, Volunteer (limited)
- Audit log immutable and tamper-evident
- Client data deletable upon request (right to erasure)
- No social security numbers or financial account numbers collected

### 7.4 Reliability

- 99.5% uptime SLA (food distributions happen on fixed schedules)
- Offline-capable for critical volunteer flows
- Daily automated backups with 30-day retention

### 7.5 Localization

- MVP: English and Spanish (full UI + client-facing forms + SMS)
- Architecture supports adding languages without code changes (string externalization)
- Date formats and phone number formats locale-aware

---

## 8. Technical Architecture

### 8.1 Recommended Stack

| Layer | Technology | Rationale |
|---|---|---|
| Frontend | React (Next.js or Vite) | Mobile-responsive, PWA-capable for offline support |
| Backend / DB | Supabase (PostgreSQL + Auth + Edge Functions) | Free tier sufficient for MVP; real-time subscriptions for dashboard; Row Level Security for multi-tenant |
| SMS | Twilio | Reliable, affordable, bilingual support, compliance-friendly |
| Hosting | Vercel | Zero-config deployment, edge functions, free tier for MVP |
| E-Signature | Canvas-based (custom) or signature_pad library | Lightweight, no third-party dependency for MVP |
| Auth | Supabase Auth + magic links for clients | No passwords for clients; staff use email/password |

### 8.2 Key Architectural Decisions

- **No client login required for recertification** — Unique, time-limited links sent via SMS. Security through obscurity of the link + phone number verification. Reduces friction for low-tech clients.
- **Multi-tenant from day one** — Each food bank is a tenant. Supabase Row Level Security enforces data isolation. Enables future SaaS model.
- **PWA for offline** — Service worker caches volunteer check-in flow. Queued recertifications sync when online. Critical for rural pantries.
- **Configurable rules engine over hard-coded logic** — Program rules stored as structured data, not code. Admins configure via UI. New program types don't require developer involvement.

---

## 9. Success Metrics

| Metric | Baseline (No ReCert) | Target (With ReCert) |
|---|---|---|
| Recertification completion rate | ~60–70% (estimated) | 90%+ |
| Client lapse rate (eligible but expired) | 15–25% annual lapse | < 5% |
| Staff hours on recert admin per week | 8–12 hours | 2–3 hours |
| Client time to recertify | In-person visit (1+ hours) | < 5 min on phone |
| Self-service recert rate | 0% (doesn't exist) | 50%+ of all recertifications |
| Time to generate compliance report | 2–4 hours manual compilation | < 1 minute (one-click export) |
| Audit readiness | Paper files, manual search | Instant digital retrieval |

---

## 10. Risks & Mitigations

| Risk | Severity | Mitigation |
|---|---|---|
| Low SMS opt-in from clients | High | Train volunteers to emphasize opt-in during intake. Frame as "never be turned away for expired paperwork again." Walk-in recertification still works without SMS. |
| Volunteer resistance to new technology | High | Extremely simple UI (3-tap max). Side-by-side paper fallback during transition. Recruit one "champion volunteer" per site for peer training. |
| State-by-state regulatory variation | Medium | Configurable rules engine handles variation without code changes. Start with one state, expand with validated templates. |
| Integration with existing pantry tools | Medium | MVP uses CSV import/export. API integrations are a V2+ feature. Position ReCert as complementary, not competitive. |
| Client phone number churn | Medium | SMS bounce detection → auto-flag for staff outreach. Prompt phone number verification at each visit. |
| Food bank budget constraints | Medium | Free tier for small pantries (< 500 households). Grant-fundable positioning. ROI story: reduced lapse = more served = better funder metrics. |
| Connectivity at rural pantry sites | Low | PWA offline mode in V2. Paper fallback process documented. |

---

## 11. Go-to-Market Considerations

### 11.1 Distribution Strategy

Food banks are highly networked. Feeding America alone coordinates 200+ member food banks, each serving dozens of partner pantry agencies. The GTM strategy leverages this network effect:

- **Pilot with one food bank** — Ideally a Project DASH partner already using DoorDash delivery. Validate with real users, iterate on feedback.
- **Case study → network spread** — Food bank program managers talk to each other through regional associations and Feeding America convenings. A strong case study from one site creates organic demand.
- **Feeding America partnership** — Long-term goal: become a recommended tool in the Feeding America technology toolkit, similar to MealConnect.
- **Project DASH alignment** — Natural co-marketing opportunity: "Keep clients certified so they can keep receiving deliveries."

### 11.2 Pricing Model

- **Free tier** — Single pantry, up to 500 households, basic features. Lowers adoption barrier.
- **Standard tier** — Multi-program, unlimited households, SMS reminders, compliance reporting. $50–100/month (grant-fundable).
- **Network tier** — Multi-site management for food banks coordinating partner agencies. Custom pricing.

---

## 12. Next Steps

| # | Action | Owner | Timeline |
|---|---|---|---|
| 1 | Share PRD with Project DASH contact for validation | Katie | Week 1 |
| 2 | Conduct 2–3 user interviews with food bank program managers | Katie | Weeks 2–3 |
| 3 | Conduct 2–3 user interviews with pantry volunteers | Katie | Weeks 2–3 |
| 4 | Build clickable prototype of Flow 1 (client self-service recertification) | Katie | Week 4 |
| 5 | Identify one pilot food bank partner willing to test | Katie + Project DASH | Weeks 4–5 |
| 6 | Build MVP (React + Supabase + Twilio) | Katie | Weeks 5–8 |
| 7 | Pilot deployment with real clients at partner food bank | Katie + Pilot Partner | Weeks 9–12 |

---

*This document is a living artifact. It will be updated as user interviews and pilot feedback refine the product definition.*
