# AeroGuard Product Requirements Document (PRD)

## 1) Product Overview

**Product name:** AeroGuard

**One-line pitch:** AeroGuard is a pre-flight risk and compliance intelligence platform for commercial drone fleets.

**Core outcome:** Help operators decide whether a mission should fly, recommend safer configurations, and generate audit-ready compliance evidence.

**Business objective:** Build a sellable SaaS product and credible startup story that can win paid pilots, customer case studies, and investor interest.

---

## 2) Problem Statement

Commercial drone operators (especially infrastructure inspection teams) are scaling operations across more pilots, drones, batteries, sites, and regulatory requirements. Their workflow is fragmented across logs, weather tools, airspace tools, spreadsheets, and manual checklists.

This creates two high-cost gaps:

1. **Pre-flight decision gap:** Operators cannot quickly determine mission risk before takeoff.
2. **Compliance evidence gap:** Operators cannot easily prove they performed responsible safety and compliance checks.

---

## 3) Target Customer and Users

## Beachhead customer
- Small-to-mid-sized drone service providers
- 5-50 drones
- Infrastructure-heavy missions (utility, solar, telecom, pipeline, rail, construction, industrial)

## User roles
- **Admin / Owner:** manages org, users, data, and reports
- **Operations Manager:** plans missions, assigns assets, reviews risk
- **Pilot:** executes mission, completes checklist, uploads logs
- **Safety/Compliance Manager:** reviews binders and approvals
- **Viewer/Client (optional):** read-only report access

---

## 4) Product Goals and Non-Goals

## Goals (MVP to V1)
- Ingest flight logs and normalize telemetry
- Manage fleet assets (drones, batteries, pilots, maintenance)
- Plan missions and evaluate weather + airspace risk
- Calculate transparent, explainable mission risk scores
- Recommend safer actions
- Generate professional compliance binder PDFs
- Support human-in-the-loop approval and audit trail

## Non-goals (not now)
- Drone autopilot or flight control
- Detect-and-avoid system
- Full UTM platform
- Hardware-first product
- Insurance underwriting engine
- Full BVLOS certification workflow

---

## 5) Value Proposition

**Primary value:** "Know what can fly today in under 10 seconds."

**Secondary value:**
- Reduce preventable mission risk
- Reduce manual compliance documentation time
- Improve operator confidence and customer trust
- Provide defensible records if incidents or audits occur

---

## 6) Success Metrics

## Product metrics
- Number of flight logs uploaded
- Number of missions risk-scored
- Number of compliance binders generated
- % of missions with actionable recommendations
- Median time from mission creation to report generation
- % of recommendations accepted by operators

## Business metrics
- Free report requests
- Free-to-paid conversion rate
- Number of paid pilots
- MRR and ARPA
- Churn and logo retention

## Validation metrics
- "Did AeroGuard find unknown risks?" (yes/no)
- "Did AeroGuard change mission decisions?" (yes/no)
- "Would customer pay for this?" (yes/no + amount)

---

## 7) MVP Scope

## Must-have features
1. Authentication + organization scoping
2. Fleet asset CRUD (drones, batteries, pilots, maintenance)
3. Mission creation + assignment
4. CSV flight log upload + parser pipeline
5. Telemetry normalization + summary metrics
6. Weather snapshot integration
7. Airspace/compliance snapshot integration
8. Rule-based risk engine with category breakdown
9. Explainable risk reasons + recommendations
10. Pre-flight compliance checklist
11. Post-flight anomaly detection (rule-based)
12. PDF compliance binder generation

## Nice-to-have (after MVP)
- PX4 ULog and ArduPilot parsing
- Batch upload
- Advanced geospatial route risk visualization
- Mobile checklist UX
- More enterprise integrations

---

## 8) Functional Requirements

## FR-1 Authentication and Multi-Tenant Organizations
- Users can sign up/log in
- Users belong to one or more organizations
- Data is organization-scoped
- Admins can invite/manage users

## FR-2 Fleet Management
- Create/edit/archive drones, batteries, pilots
- Store status values (`READY`, `WARNING`, `GROUNDED`, etc.)
- Track maintenance records and status impact

## FR-3 Mission Planning
- Create mission with location, time, route, type, altitude, duration
- Assign drone, battery, pilot
- Store mission statuses (`DRAFT` to `COMPLETED`)

## FR-4 Log Ingestion and Parsing
- Accept CSV uploads first
- Store raw files in object storage
- Parse and normalize into shared telemetry schema
- Compute summary metrics and parser status

## FR-5 Weather and Airspace
- Retrieve weather snapshot for mission time/location
- Check route against airspace constraints and altitude context
- Persist raw API payloads for auditability

## FR-6 Risk Scoring
- Compute category scores and weighted overall score
- Assign level (`LOW`, `MODERATE`, `HIGH`, `CRITICAL`)
- Persist reasons, source, severity, and recommended actions

## FR-7 Recommendations
- Auto-create recommendations for material risk findings
- Track recommendation status lifecycle
- Reflect unresolved recommendations in mission readiness

## FR-8 Compliance Checklist
- Generate checklist items by mission context
- Require explicit human completion and final review
- Include checklist evidence in reports

## FR-9 Post-Flight Anomalies
- Detect baseline anomalies (voltage sag, GPS drop, route deviation, etc.)
- Attach anomalies to flight logs and mission reports

## FR-10 Compliance Binder PDF
- Generate download-ready PDF with mission evidence bundle
- Store generated file and metadata
- Support regeneration after updates

---

## 9) Non-Functional Requirements

- **Security:** org data isolation, role-based access, secure file handling
- **Auditability:** immutable event history for key workflow decisions
- **Explainability:** every significant risk must include plain-language reason
- **Performance:** mission risk calculation should complete in operationally practical time
- **Reliability:** background jobs for parsing/report generation with retry handling
- **Compliance posture:** advisory-only language, explicit human responsibility

---

## 10) Legal and Safety Product Language

Use:
- "risk advisory"
- "recommended review"
- "compliance documentation support"
- "human approval required"

Avoid:
- "guaranteed safe"
- "FAA approved by AeroGuard"
- "crash prevention guaranteed"
- "fully autonomous compliance"

Disclaimer baseline:
"AeroGuard provides mission risk advisory and compliance documentation support. Operators remain responsible for final flight decisions and regulatory adherence."

---

## 11) Suggested Architecture

- **Frontend:** Next.js + React + Tailwind
- **Backend API:** FastAPI
- **Database:** PostgreSQL + PostGIS
- **Jobs/queues:** Redis + Celery/RQ
- **Storage:** S3-compatible object storage
- **Parser/data:** Pandas/Polars (expand to `pyulog` + `pymavlink` later)
- **PDF engine:** WeasyPrint or ReportLab
- **Deployment:** Docker + managed cloud services

---

## 12) Step-by-Step Build Guide (for future coding execution)

This section is written as an implementation playbook you (or a coding agent) can follow later in order.

## Phase 0 - Repo and Delivery Setup
1. Initialize monorepo/app structure for web + API + shared packages.
2. Create environments for local, staging, production.
3. Add CI baseline (lint, test, migration check).
4. Add infra config for database, Redis, object storage.

**Exit criteria:** app boots locally, CI runs, database connects.

## Phase 1 - Data Model and Core Schema
1. Implement core tables: users, organizations, members.
2. Add fleet tables: drones, batteries, pilots, maintenance_records.
3. Add mission tables: missions, routes, checklists.
4. Add ingestion/risk/reporting tables: flight_logs, telemetry_points, anomalies, weather_snapshots, airspace_snapshots, risk_scores, risk_reasons, recommendations, pdf_reports, audit_events.
5. Add migration scripts and seed scaffolding.

**Exit criteria:** schema migrates cleanly and supports all MVP entities.

## Phase 2 - Auth and Organization Boundaries
1. Implement login/session flow.
2. Enforce org-scoped authorization on all API routes.
3. Add role permissions for admin/ops/pilot/compliance/viewer.
4. Build invite and organization switch support.

**Exit criteria:** users can only access their org data.

## Phase 3 - Fleet Asset and Mission CRUD
1. Implement CRUD APIs for drones, batteries, pilots, maintenance.
2. Implement mission creation/editing and asset assignment.
3. Implement route storage as GeoJSON and basic map rendering.
4. Add mission status transitions and validation.

**Exit criteria:** full pre-flight mission object can be created and reviewed.

## Phase 4 - Log Upload and Parsing Pipeline
1. Build secure log upload endpoint and object storage write.
2. Create parsing jobs (start with CSV).
3. Normalize telemetry and compute metrics.
4. Add parser state machine (`PENDING`, `PROCESSING`, `PARSED`, `FAILED`).
5. Build flight log detail view with metrics and basic route plot.

**Exit criteria:** uploaded logs become normalized telemetry with visible summaries.

## Phase 5 - Weather and Airspace Services
1. Implement weather provider adapters and snapshot persistence.
2. Implement airspace check service and snapshot persistence.
3. Add route-level checks (airport proximity, altitude mismatch, controlled airspace).
4. Surface weather/airspace warnings in mission detail panel.

**Exit criteria:** every mission can fetch and display weather/airspace snapshots.

## Phase 6 - Rule-Based Risk Engine
1. Implement category scores: weather, airspace, compliance, battery, drone health, maintenance, pilot readiness, complexity.
2. Implement weighted overall score and risk level assignment.
3. Store risk reasons with clear explanation + recommended action.
4. Add recalculation endpoint and mission risk timeline entry.

**Exit criteria:** each mission gets an explainable, auditable risk output.

## Phase 7 - Recommendation and Readiness Logic
1. Auto-generate recommendations from high-risk findings.
2. Add recommendation actions (accept/dismiss/complete).
3. Tie unresolved high-severity recommendations to mission readiness.

**Exit criteria:** users can act on recommendations and see readiness updates.

## Phase 8 - Checklist and Human Approval
1. Generate default checklist by mission context.
2. Add checklist completion flow with evidence fields.
3. Add human final review / override with mandatory reason.
4. Record all major actions in audit_events.

**Exit criteria:** human-in-loop signoff is required and traceable.

## Phase 9 - Compliance Binder PDF
1. Build report data assembler (mission + assets + snapshots + risk + checklist + recommendations + anomalies).
2. Build template and PDF renderer.
3. Store output file + metadata and expose download route.
4. Add report list page and regeneration action.

**Exit criteria:** users can generate/download professional mission binders.

## Phase 10 - Fleet Readiness Dashboard
1. Create dashboard cards for mission readiness, asset readiness, high-risk findings.
2. Add "what can fly today" prioritized view.
3. Add recent logs, unresolved recommendations, and quick actions.

**Exit criteria:** operations team can triage daily readiness in one screen.

## Phase 11 - Demo Data and Growth Readiness
1. Build `seed_demo_data` command with realistic synthetic/public examples.
2. Include at least 10 risky/anomalous sample flights.
3. Add a scripted demo scenario for sales/investor calls.
4. Prepare first "Free Fleet Risk Report" workflow.

**Exit criteria:** product can be demoed end-to-end without customer data.

## Phase 12 - Pilot Launch and Iteration Loop
1. Onboard 3-10 design partners.
2. Deliver reports, capture outcome feedback, and log missed risks.
3. Prioritize top requested features and data integrations.
4. Convert pilots to paid plans and produce case studies.

**Exit criteria:** paid pilot conversion and evidence of recurring value.

---

## 13) Milestones and Timeline

## 0-1 months: Foundation
- Schema, auth, CRUD, synthetic dataset, initial UI

## 2-3 months: MVP Build
- Log ingestion, risk engine, weather/airspace, PDF reports

## 4-5 months: Customer Pilots
- Free risk reports, design partner onboarding, feedback loop

## Month 6: Paid Pilot
- 1-3 paid customers, refine pricing and must-have workflows

## Months 7-9: Productization
- Better ingestion, stronger recommendations, collaboration features

## Months 10-12: Fundraising/Growth
- Case studies, traction package, investor-ready narrative

---

## 14) Go-To-Market and Pricing

## GTM wedge
- Offer "Free Fleet Risk Report" to infrastructure drone operators
- Use sample upload + risk findings as sales proof
- Move from founder-led service to productized self-serve workflow

## Starter pricing hypothesis
- Starter (1-5 drones): $99-$299/month
- Pro (5-25 drones): $499-$1,500/month
- Enterprise (25+ drones): $2,000-$10,000/month
- Optional one-time paid audits and integration add-ons

---

## 15) Competitive Positioning

Do not position AeroGuard as a generic fleet-management replacement.

Position it as:
- Independent mission-risk intelligence layer
- Compliance evidence and audit binder engine
- Decision-support system for scalable drone operations

---

## 16) Risks and Mitigations

- **Data quality risk:** start with robust parser validation and confidence flags.
- **Liability risk:** maintain advisory language and mandatory human approval.
- **Regulatory change risk:** make rule modules configurable by policy version.
- **Adoption risk:** prove value quickly via free report and immediate findings.
- **Competition risk:** focus on explainability, workflow lock-in, and reports customers share externally.

---

## 17) MVP Acceptance Criteria (Launch Gate)

AeroGuard MVP is launch-ready when:
1. A user can create org, assets, and a mission.
2. A user can upload logs and see parsed telemetry + anomalies.
3. Weather + airspace snapshots are persisted and visible.
4. Risk score includes transparent reasons and recommendations.
5. Checklist + human review are required and audited.
6. Compliance binder PDF is generated and downloadable.
7. Demo data can run end-to-end with a repeatable script.

---

## 18) Investor/Pitch Readiness Checklist

- Clear wedge: infrastructure operators with repeated high-value missions
- Working product demo from mission creation to binder generation
- At least one case study showing discovered unknown risk
- Evidence of behavior change and paid willingness
- Metrics dashboard showing product usage and conversion progress

---

## 19) Final Product Vision

Start narrow: pre-flight mission risk and compliance intelligence.

Win trust by being transparent, practical, and operator-centered.

Expand over time into the safety intelligence layer for scaled commercial and BVLOS-oriented drone operations.
