# ReCert
## Food Bank Eligibility Recertification App
### User Personas, Flows & Product Specification

*February 2026 — Katie Silorio*

---

## The Problem

Food banks across the U.S. serve millions of households, but people regularly lose access to food assistance not because they are ineligible, but because the recertification process fails them.

The recertification landscape is fragmented and burdensome for both staff and clients. Different programs (TEFAP, CSFP, SNAP, and food bank-specific programs) each have different renewal cadences — ranging from every distribution visit to every three years. Staff track these deadlines manually, often across paper forms, spreadsheets, and disconnected systems. Clients miss windows because they never received a reminder, couldn't get to the food bank during business hours, or didn't understand what was needed.

The result: eligible families fall through the cracks, food banks underreport their service numbers, and funder compliance becomes a constant fire drill.

---

## Competitive Landscape

Several tools exist in the food bank software space, but none focus specifically on the recertification workflow as a standalone, purpose-built solution.

| Tool | Focus | Recert Support | Gap |
|---|---|---|---|
| Link2Feed | Client intake, reporting, case mgmt | Basic — tracks eligibility status | No proactive reminders or client self-service |
| PantrySoft | Check-in, ordering, inventory | Minimal — flags at check-in | Reactive only; no outreach workflow |
| Oasis Insight | Distribution tracking, reporting | Tracks expiration dates | No automated reminders; UI challenges |
| Pantry Saver | Intake, kiosk check-in, scale tracking | Interview scheduling | Not focused on renewal lifecycle |
| **ReCert** | **Recertification lifecycle mgmt** | **Purpose-built** | **This is the gap we fill** |

ReCert is not trying to replace these tools — it fills a specific gap in the recertification lifecycle that none of them adequately address.

---

## Key Personas

### Persona 1: Maria — Program Manager

| Field | Detail |
|---|---|
| Role | Program Manager at a mid-size regional food bank |
| Demographics | 42 years old, manages 3 staff + 20 volunteers, oversees TEFAP + CSFP + local pantry program |
| Tech Comfort | Moderate — uses Excel, email, and basic database tools. Not afraid of new software but has limited time to learn. |
| Goals | Keep clients enrolled without gaps; produce accurate reports for USDA and funders; reduce staff time on administrative tasks; increase program participation rates. |
| Pain Points | Tracks recertification deadlines across programs in a spreadsheet. No visibility into which clients are expiring until it's too late. Spends hours pulling reports manually. Worries about audit compliance. |
| Quote | *"We lose people every month who still qualify. They just didn't know they needed to re-sign paperwork. Then I have to explain to our funder why participation dropped."* |

### Persona 2: James — Intake Volunteer

| Field | Detail |
|---|---|
| Role | Weekly volunteer at a food pantry partner agency |
| Demographics | 67 years old, retired teacher, volunteers 2 shifts per week. Handles client check-in and intake paperwork. |
| Tech Comfort | Low to moderate — comfortable with a tablet or phone, but gets frustrated with complex interfaces. Needs large text and clear instructions. |
| Goals | Get clients checked in quickly and correctly. Know at a glance if someone needs to recertify. Avoid making mistakes that create compliance problems. |
| Pain Points | Paper forms are confusing and inconsistent. Can't tell if a client's certification is current without flipping through a filing cabinet. Often has to turn people away and ask them to come back with documentation. |
| Quote | *"I hate telling someone they can't get food today because their form expired. They drove 30 minutes to get here. There has to be a better way."* |

### Persona 3: Rosa — Food Bank Client

| Field | Detail |
|---|---|
| Role | Mother of 3, works part-time, receives TEFAP food boxes monthly |
| Demographics | 34 years old, Spanish-speaking (bilingual), limited transportation, uses a smartphone as her primary device. |
| Tech Comfort | Moderate — comfortable with texting and WhatsApp. Uses phone for everything. Will fill out a form on her phone if it's simple. |
| Goals | Keep receiving food assistance for her family without disruption. Complete any required paperwork as quickly and easily as possible. Not have to take time off work or arrange childcare to visit the food bank for administrative tasks. |
| Pain Points | Doesn't know when her eligibility expires. Found out at pickup that she was "expired" and had to leave without food. The recertification form is in English and confusing. Has to come in person during work hours. |
| Quote | *"I went to pick up my food box and they said my papers were expired. I didn't even know. I had to come back another day and I missed work."* |

---

## User Flows

### Flow 1: Client Self-Service Recertification (Rosa)

This is the primary value-driving flow — enabling clients to recertify from their phone without visiting the food bank.

**Trigger:** System detects Rosa's certification expires in 30 days.

#### Steps

1. **Automated Reminder (30 days before expiration)** — Rosa receives an SMS in Spanish: *"Your food bank eligibility expires on [date]. Tap here to renew in 5 minutes: [link]"*
2. **Mobile-Friendly Recertification Form** — Rosa taps the link and sees a pre-filled form with her existing information (name, address, household size). She only needs to confirm or update what's changed.
3. **Eligibility Self-Declaration** — Rosa confirms her income bracket or selects a categorical eligibility option (e.g., "I receive SNAP benefits"). No documentation upload required for TEFAP self-declaration.
4. **E-Signature & Acknowledgment** — Rosa signs electronically and acknowledges the USDA non-discrimination statement.
5. **Confirmation** — Rosa gets instant confirmation: *"You're renewed through [new date]! See you at your next pickup."*
6. **Staff Notification** — Maria's dashboard updates in real time. Rosa's status moves from "Expiring Soon" to "Active."

#### Reminder Escalation Schedule

- **30 days before:** First SMS/email reminder
- **14 days before:** Second reminder with "renew now" urgency
- **3 days before:** Final reminder + option to call for help
- **Day of expiration:** "Your eligibility expired today. You can still renew — tap here."
- **7 days after:** Grace period outreach from staff (flagged on dashboard)

---

### Flow 2: Volunteer-Assisted Walk-In Recertification (James)

For clients who come in person or prefer assistance, James can complete the recertification on their behalf.

**Trigger:** Client arrives for food pickup. System flags their status at check-in.

#### Steps

1. **Client Check-In** — James looks up Rosa by name, phone number, or household ID. The system shows a clear status badge: **EXPIRED** (red), **EXPIRING SOON** (yellow), or **ACTIVE** (green).
2. **Guided Recertification** — If expired or expiring, James taps "Recertify Now" and walks through a simple wizard: confirm household info → verify income/categorical eligibility → capture e-signature on tablet.
3. **Language Toggle** — James can switch the form to Spanish (or other languages) with one tap so Rosa can read what she's signing.
4. **Instant Completion** — Recertification is processed immediately. Rosa can proceed to receive her food box without a return visit.
5. **Record Stored** — Digital record is auto-saved with timestamp, volunteer name, and e-signature. No paper filing required.

#### Key Design Requirements for James

- Large, high-contrast UI elements (accessible for older volunteers)
- Maximum 3 taps to complete a recertification
- Clear visual status badges with color + text (not color alone)
- Offline capability for pantries with unreliable internet
- No login required for volunteers — device-based auth or simple PIN

---

### Flow 3: Program Manager Dashboard & Reporting (Maria)

Maria needs a bird's-eye view of her entire client base's certification status, plus the ability to generate audit-ready reports.

**Trigger:** Daily workflow — Maria checks the dashboard at the start of each day and generates reports monthly/quarterly.

#### Dashboard Views

- **Status Overview** — At-a-glance metrics: total active clients, expiring in 30 days, expired in last 30 days, lapsed (expired 60+ days). Visualized as a status funnel or donut chart.
- **Expiration Queue** — Sortable list of all clients with upcoming expirations. Filterable by program (TEFAP, CSFP, pantry-specific), date range, and outreach status (reminder sent, no response, completed).
- **Recertification Completion Rate** — Tracks what percentage of expiring clients successfully recertify before their deadline. Broken down by channel: self-service vs. walk-in vs. phone.
- **Outreach Tracker** — Shows which clients have been contacted, how many reminders they've received, and their response status. Enables manual follow-up for non-responders.

#### Reporting & Compliance

- **TEFAP/CSFP Compliance Report** — Auto-generated report showing active client count, eligibility verification method (self-declaration vs. categorical), and certification dates. Formatted for USDA/state agency submission.
- **Funder Impact Report** — Exportable summary showing total households served, recertification completion rate, lapse prevention metrics.
- **Audit Trail** — Every recertification action is timestamped and logged: who completed it, when, through what channel, with e-signature attached. Records retained per the 5-year requirement.
- **Export Options** — CSV export for integration with existing systems (Link2Feed, Oasis Insight, etc.). PDF export for physical filing or board reports.

---

### Flow 4: New Client Intake & Initial Certification

Before recertification can happen, clients must be registered. ReCert can serve as the intake tool or integrate with existing systems.

#### Steps

1. **Client Information** — Name, address (self-declared residency), household size, preferred language, contact method (phone/SMS/email).
2. **Eligibility Determination** — Income-based: household selects income bracket against posted guidelines. Or categorical: client indicates participation in SNAP, WIC, TANF, Medicaid, or SSI.
3. **Program Enrollment** — Staff selects which programs the client is enrolling in. System auto-calculates next recertification date based on program rules.
4. **USDA Acknowledgments** — Non-discrimination statement presented and acknowledged. Rights and responsibilities form presented.
5. **E-Signature** — Client signs on tablet/phone.
6. **Notification Opt-In** — Client provides phone number and opts into SMS reminders. This is the critical step that enables the proactive renewal flow.

---

## Configurable Program Rules

Every food bank operates differently. ReCert must be flexible enough to support varying program requirements without custom development.

| Configuration | TEFAP | CSFP | Local Pantry |
|---|---|---|---|
| Recertification Frequency | Every visit / annually / every 3 years (varies by state) | Every 6 months | 6 months or 12 months |
| Income Threshold | State-specific (e.g., 185% or 225% FPL) | 130% FPL | Up to 200% FPL |
| Categorical Eligibility | SNAP, WIC, TANF, Medicaid, SSI | SNAP, TANF, SSI | Configurable |
| Verification Method | Self-declaration allowed | Documentation may be required | Varies by org |
| ID Required | No (federal rule) | Varies | Varies by org |
| Record Retention | 5 years | 3 years | Varies |

---

## MVP Feature Scope

The initial build should focus on the highest-impact, lowest-complexity features that demonstrate the core value proposition: keeping clients enrolled.

### Must Have (MVP)

- Client database with certification status tracking (active, expiring, expired, lapsed)
- Configurable recertification rules per program (frequency, income thresholds, categorical eligibility)
- Automated SMS reminders with escalation schedule
- Mobile-friendly client self-service recertification form (bilingual English/Spanish)
- E-signature capture
- Staff/volunteer check-in screen with status badges
- Basic dashboard with expiration queue and completion rate

### Should Have (V2)

- USDA-formatted compliance reporting (TEFAP, CSFP)
- CSV/PDF export for integration with existing tools
- Additional language support beyond English/Spanish
- Authorized proxy management (family member picking up on behalf of client)
- Offline mode for pantries with unreliable internet

### Nice to Have (Future)

- API integration with Link2Feed, Oasis Insight, and other pantry management tools
- Feeding America MealConnect integration
- Multi-site management for food banks with partner agencies
- Benefits screening (help clients discover additional programs they qualify for)
- Project DASH delivery integration for home-delivered food box clients

---

## Success Metrics

| Metric | Baseline (Without ReCert) | Target (With ReCert) |
|---|---|---|
| Recertification completion rate | ~60–70% (estimated) | 90%+ |
| Client lapse rate | 15–25% of clients lapse annually | < 5% |
| Staff time on recert admin | 8–12 hours/week | 2–3 hours/week |
| Time to complete recert (client) | Requires in-person visit (1+ hours) | < 5 minutes on phone |
| Audit readiness | Manual report compilation | One-click export |

---

## Next Steps

1. **Validate with Project DASH contact** — Share this document and get feedback on accuracy of pain points, program rules, and user flows.
2. **User interviews** — Talk to 2–3 food bank program managers and 2–3 pantry volunteers to validate personas and prioritize features.
3. **Prototype the client flow** — Build a clickable prototype of Flow 1 (client self-service recertification) as the "wow moment" demo.
4. **Identify pilot partner** — Find one food bank willing to test with a subset of clients.
5. **Tech stack decision** — Choose build approach: lightweight web app (React + Supabase) vs. Salesforce-native (leveraging nonprofit free licenses).
