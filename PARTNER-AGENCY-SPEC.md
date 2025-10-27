# Partner & Agency Portal Specification *(Deferred Until Infrastructure Upgrade)*

> **Status:** Deferred while the platform operates on Vercel Hobby + Supabase Free. Feature set requires higher quotas, scheduled jobs, and expanded auditing available on Pro tiers.

## 1. Purpose & Scope
- Empower approved Catholic agencies, parish offices, and professional matchmakers to generate and manage leads for CCMatrimony.
- Support referral tracking, assisted registrations, and commission payouts while preserving user privacy and compliance.
- Provide admins with full oversight, auditability, and control over partner activity.

## 2. Roles & Access Model
| Role | Permissions | Notes |
|------|-------------|-------|
| **Partner / Agency / Agent** | Limited | Access only to leads created via referral link or assigned by admin; can onboard profiles, update lead status, request payouts.
| **Admin** | Full | Manage partners, commissions, payouts, and audit logs; assign leads and override partner actions.
| **Telecaller** | Existing | No change; remains separate from partner portal with standard CRM capabilities.

### 2.1 Authentication & Onboarding
- Partners are vetted offline (in-person meeting, document review handled by admin outside the system).
- Admin records partner details directly in the backend and toggles "Approved" status to grant access.
- System generates Partner ID, credentials, referral link, and QR code once admin creates the record.
- Optional lightweight interest form can capture leads for follow-up, but no sensitive docs are uploaded online.
- Partner login is isolated from admin/telecaller (separate subdomain or route namespace).

## 3. Core Workflows

### 3.1 Lead Generation & Assignment
- **Referral Capture:** Users registering via partner link/QR are auto-tagged with Partner ID.
- **Manual Assignment:** Admin can assign existing leads to a partner (filters: region, parish, community).
- **Lead States:** New → Contacted → Verified → Converted → Closed. Partners can update status; transitions logged with timestamp and actor.

### 3.2 Partner-Assisted Profile Creation
- Partner can create/edit profiles on behalf of assigned leads (photo upload, education, occupation, income).
- Mark profile as “Verified by Partner” with notes capturing offline verification details (no document upload required).
- Data scope restricted: partner sees only fields necessary for completion.

### 3.3 Commission Tracking & Payouts
- Configurable commission triggers per partner (profile completion, paid plan, match success).
- Earnings dashboard shows: Total Earned, Pending Approval, Paid, Upcoming Payout.
- Partner can request payout once threshold met; admin reviews → approve/decline and status updates are reflected in the dashboard (no email/SMS alerts).
- Payout history includes invoice reference, date, amount, status.

### 3.4 Partner Analytics & Leaderboards
- Metrics: Leads generated (MTD/YTD), completed registrations, paid conversions, matches, commission earned.
- Optional leaderboard highlighting top partners (configurable time window).
- Export to CSV for partner’s internal reporting.
- Telecaller coordination: partner and telecaller notes share the same profile timeline; callers review activity history before outreach to avoid duplicate contact.

## 4. Admin Controls & Oversight
- Partner CRUD: add, edit details, suspend, reactivate, delete.
- Commission configuration per partner or per event type (flat rate or percentage).
- Lead assignment screen with filters and bulk assignment.
- Audit log for partner actions (field edits, status changes, payout requests) with timestamp and IP.
- Anomaly detection: flag heavy activity bursts (e.g., >20 profile edits/hour) and auto-notify admins for review.
- Rate limits: cap partner API usage (e.g., 60 requests/min) to prevent abuse.
- Dashboard cards: Active Partners, Leads This Month, Conversion Rate, Commission Liability.

## 5. Security & Compliance
- Role-based access enforced via Supabase RLS / API middleware (partners scoped to their entities).
- Mask sensitive fields (contact info) until verification stage to prevent misuse.
- Partner terms & privacy agreement stored with audit trail (version, acceptance timestamp).
- All downloads (CSV exports) logged; optional watermark with partner ID.
- GDPR/Indian Data Protection compliance: consent stored for partner-assisted registrations.
- Partner usage monitoring: weekly report highlighting anomalies, inactive partners, and escalations resolved.

## 6. Technical Architecture
- **Frontend:** Next.js partner portal route (`/partner/*`) with protected layouts.
- **Backend:** Supabase Postgres tables (`partners`, `partner_leads`, `partner_commissions`, `partner_audit_log`).
- **APIs:** Vercel serverless / Supabase Edge Functions for partner endpoints with JWT auth.
- **Background Jobs:** Scheduled task to reconcile commissions, send leaderboard summaries, refresh analytics materialized view.
- **Integrations:** None required initially (dashboard surfaces all status changes; no email/SMS notifications).

## 7. API Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/partner/register` | Submit partner application (public). |
| POST | `/api/partner/login` | Partner authentication (password + optional OTP). |
| GET | `/api/partner/dashboard` | Summary metrics for logged-in partner. |
| GET | `/api/partner/leads` | List assigned leads with filters/pagination. |
| PATCH | `/api/partner/leads/:id` | Update lead status, add notes, mark verification. |
| POST | `/api/partner/profiles` | Create profile for partner lead. |
| GET | `/api/partner/commission` | Earnings summary and transaction history. |
| POST | `/api/partner/payout-request` | Request commission payout. |
| GET | `/api/admin/partners` | Admin view of all partners with filters. |
| PATCH | `/api/admin/partners/:id` | Approve, suspend, or update partner settings. |
| POST | `/api/admin/partner-assign` | Bulk assign leads to partner. |
| POST | `/api/admin/partner-payout` | Approve payout and trigger downstream processes. |

## 8. UI/UX Blueprint
- **Partner Dashboard:** KPI cards (Leads This Month, Conversions, Commission Earned), quick actions (Add Lead, Request Payout).
- **Leads Page:** Table with status filters, search by name/contact, inline status updates, activity sidebar.
- **Profile Wizard:** Guided steps for photo upload, personal info, education, occupation, family, with progress tracker.
- **Earnings Page:** Graph of commissions over time, payout request CTA, payout history table.
- **Admin Partner View:** Tabs for Partners, Assignments, Commissions, Audit Log.
- Responsive design prioritizing desktop/tablet; future mobile experience in roadmap.

- **Operational Runbook Notes:**
  - If misuse or anomaly detected → auto-flag partner → admin reviews log → suspend or warn → document outcome in audit log.
  - Payout dispute: mark disputed status, freeze further payouts, notify finance, resolve, then unlock.
  - Inactive partner (>60 days) → send re-engagement email → consider suspension if no response.

## 9. Roadmap & Phasing
| Phase | Deliverables |
|-------|--------------|
| **P3 (MVP)** | Partner onboarding, referral tracking, basic lead dashboard, profile assistance, simple commission payouts. |
| **P4 (Enhancements)** | Automated regional assignment, advanced commission rules, leaderboard, payout integrations. |
| **P5 (Deferred)** | External partner API, marketplace listing, SLA tracking, mobile partner app. |

## 10. Success Metrics
- Active partners onboarded per quarter.
- Lead-to-registration conversion rate for partner referrals.
- Paid conversions attributable to partners.
- Commission payout cycle time and volume.
- Regional penetration attributed to partner network.

## 11. Open Questions
- Do partners require multilingual support or localized pricing?
- Should partner payouts integrate with external accounting/payroll systems?
- Is there a need for partner training/certification modules within the portal?
- What SLA targets should partners meet (response time, verification accuracy)?

## 12. Data Model Preview (Draft)
| Table | Key Fields | Notes |
|-------|------------|-------|
| `partners` | `id`, `name`, `region`, `status`, `commission_config`, `created_at`, `approved_by` | Stores partner account and configuration. |
| `partner_users` | `id`, `partner_id`, `auth_user_id`, `role`, `last_login` | Portal users tied to partner; supports multi-user agencies. |
| `partner_leads` | `id`, `partner_id`, `user_id`, `source`, `status`, `notes`, `last_action_at` | Tracks lead lifecycle and shared notes for partner/telecaller. |
| `partner_commissions` | `id`, `partner_id`, `event_type`, `amount`, `status`, `payout_id`, `earned_at` | Commission events and payout linkage. |
| `partner_audit_log` | `id`, `partner_id`, `actor`, `action`, `payload`, `created_at`, `ip_address` | Immutable log of partner/admin actions. |
