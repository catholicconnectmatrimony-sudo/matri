# CC Matrimony Tech Stack (Vercel Hobby + Supabase Free)

## 1. Architecture Overview
- **Frontend Hosting**: Vercel Hobby (Next.js SSR/ISR with limited build minutes)
- **Backend & Data Layer**: Supabase Free (PostgreSQL ≤500 MB, Auth, 1 GB Storage, Edge Functions)
- **APIs**: Vercel Serverless Functions + Supabase Edge Functions
- **Mobile Clients**: Responsive web-first design
- **Observability**: Sentry (error tracking) + Supabase logs + PostHog (analytics)

## 2. Frontend (Web)
| Component | Choice | Notes |
|-----------|--------|-------|
| Framework | **Next.js 16** (React 19.2, App Router) | SSR/ISR for SEO-heavy landing pages, SSG for static content |
| Styling | **Tailwind CSS 4.0 + Radix UI + shadcn/ui** | Utility-first styling, accessible primitives, component library |
| State/Data | **TanStack Query v5 + Zustand** | Caching, background refetch, offline revalidation, state management |
| Validation | **Zod + React Hook Form** | Type-safe schemas, granular form control |
| Authentication | **Supabase Auth** | Session handling via JWT, email/phone verification |
| Image Handling | **Supabase Storage** | Photo storage with CDN (1GB free, then $0.125/GB) |
| Notifications | **Supabase Realtime + Email** | Real-time notifications via Supabase Realtime, email fallback |

## 3. Mobile Apps
- **Primary**: Responsive web design (mobile-first)
- **Future**: React Native app when user base grows
- **PWA**: Progressive Web App features for mobile experience

## 4. Backend & Data
| Area | Service | Responsibilities |
|------|---------|------------------|
| Database | **Supabase PostgreSQL** | Normalized profile data, reciprocity states, plan entitlements, audit logs |
| Auth | **Supabase Auth** | Email/password, phone OTP, Google login, session management |
| Storage | **Supabase Storage** | Photo uploads, document storage (1GB free, then $0.125/GB) |
| Realtime | **Supabase Realtime** | Real-time messaging, presence indicators |
| Business Logic | **Supabase Edge Functions + Vercel API Routes** | Reciprocity checks, plan upgrades, admin actions |
| Background Jobs | **Vercel Cron (≤12 jobs)** | Daily analytics, email digests, cleanup tasks |

### Data Modeling Notes
- Reciprocity calculated on save via Supabase functions
- Profile view log with 90-day retention (within 500MB limit)
- Audit tables kept lightweight (essential fields only)
- Full PostgreSQL control for complex queries

## 5. Search Strategy
- **Primary**: PostgreSQL indexed filters and full-text search
- **Indexes**: B-Tree + GIN indexes for name/bio search
- **Performance**: Optimized queries with proper indexing
- **Future**: Elasticsearch when search volume grows

## 6. Media Handling
- **Upload**: Supabase Storage with signed URLs
- **Processing**: Client-side compression and resizing
- **CDN**: Supabase built-in CDN
- **Moderation**: Manual admin approval (auto-approve by default)
- **Watermarking**: Client-side watermarking for security

## 7. Payments & Monetization
| Capability | Service | Notes |
|------------|---------|-------|
| Online Payments | **Razorpay** | Primary payment gateway, subscription management |
| UPI Backup | **PhonePe** | Secondary UPI payment option |
| Offline Payments | **Manual Processing** | Bank transfer, cash payments with admin approval |
| Webhooks | **Vercel API Routes** | Payment verification and subscription management |
| Invoicing | **PostgreSQL + PDF Generation** | Automated invoice generation |

## 8. Communications
- **Email**: Resend (transactional templates, interest alerts)
- **SMS/OTP**: Fast2SMS (primary OTP and critical alerts)
- **Push Notifications**: Supabase Realtime notifications
- **Chat**: Supabase Realtime-based messaging with file sharing

## 9. Admin & Management Tools
- **Admin Panel**: Next.js app on Vercel, protected via Supabase Auth
- **Capabilities**: User management, profile approvals, payment tracking, analytics
- **Analytics**: Real-time dashboards with Supabase queries
- **Bulk Operations**: Efficient bulk management tools

## 10. Observability & Quality
| Area | Tool | Usage |
|------|------|-------|
| Error Monitoring | **Sentry** (frontend + backend) | Capture exceptions, performance traces |
| Logging | **Vercel / Supabase Logs** (exportable) | API audit, debugging, real-time monitoring |
| Analytics | **PostHog** (free tier) | User behavior, conversion tracking |
| Performance | **Vercel Analytics** | Response times, page performance |
| Testing | **Jest + React Testing Library** | Unit tests, integration tests |

## 11. DevOps & CI/CD
- **Repository**: GitHub (web + infrastructure scripts)
- **CI/CD**: GitHub Actions (lint/test → deploy to Vercel)
- **Database Migrations**: Supabase migrations managed via `supabase db diff`
- **Infrastructure**: Supabase CLI + Vercel project config tracked in repo
- **Environment Secrets**: Vercel/Supabase dashboards

## 12. Environments
| Environment | Setup | Notes |
|-------------|-------|-------|
| **Local** | Next.js dev server + Supabase **Dev** project (cloud) | `.env.local` points to isolated dev keys; no Docker required |
| **Preview** | Vercel preview deploys + Supabase **Dev** project | Auto-deploy on PRs, uses shared dev data and secrets |
| **Production** | Vercel + Supabase **Prod** project | Live traffic; enable feature flags/maintenance modes before releases |

## 13. Cost Snapshot (Monthly Estimates)
| Item | Est. Cost |
|------|-----------|
| Vercel Hobby | $0 |
| Supabase Free | $0 |
| Razorpay fees | Variable (per transaction) |
| Resend + Fast2SMS | $20-30 |
| Sentry + PostHog | $0 (free tiers) |
| Total Core Spend | **~$20-30/month** |

> Costs scale with usage; start FREE, pay only when you exceed limits

## 14. Database Schema (PostgreSQL)
```sql
-- Core tables
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  phone VARCHAR(15) UNIQUE,
  password_hash VARCHAR(255),
  email_verified BOOLEAN DEFAULT FALSE,
  phone_verified BOOLEAN DEFAULT FALSE,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  profile_id VARCHAR(20) UNIQUE NOT NULL, -- CCM001234 format
  name VARCHAR(100) NOT NULL,
  age INTEGER NOT NULL,
  gender VARCHAR(10) NOT NULL,
  community VARCHAR(50) NOT NULL,
  sub_community VARCHAR(50),
  religion VARCHAR(20) NOT NULL,
  mother_tongue VARCHAR(50),
  native_place VARCHAR(100),
  current_location VARCHAR(100),
  height INTEGER, -- in cm
  education_level VARCHAR(50),
  education_field VARCHAR(100),
  occupation_sector VARCHAR(50),
  job_title VARCHAR(100),
  annual_income VARCHAR(50),
  about_me TEXT,
  family_type VARCHAR(20),
  marital_status VARCHAR(20) DEFAULT 'Never Married',
  children_status VARCHAR(20),
  diet VARCHAR(20),
  smoking VARCHAR(20),
  drinking VARCHAR(20),
  profile_completeness INTEGER DEFAULT 0,
  is_approved BOOLEAN DEFAULT TRUE, -- Auto-approve as requested
  is_verified BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE photos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  profile_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  filename VARCHAR(255) NOT NULL,
  original_name VARCHAR(255) NOT NULL,
  file_size INTEGER NOT NULL,
  mime_type VARCHAR(100) NOT NULL,
  storage_path VARCHAR(255) NOT NULL, -- Supabase Storage object key
  is_approved BOOLEAN DEFAULT TRUE, -- Auto-approve as requested
  is_primary BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE interests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  receiver_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  message TEXT,
  status VARCHAR(20) DEFAULT 'pending', -- pending, accepted, declined, withdrawn
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(sender_id, receiver_id)
);

CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  receiver_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  message_type VARCHAR(20) DEFAULT 'text', -- text, image, document
  file_url VARCHAR(500), -- Supabase Storage signed URL
  file_name VARCHAR(255), -- Original filename
  file_size INTEGER, -- File size in bytes
  is_read BOOLEAN DEFAULT FALSE,
  read_at TIMESTAMP,
  is_deleted BOOLEAN DEFAULT FALSE,
  deleted_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  plan_type VARCHAR(20) NOT NULL, -- free, silver, gold, platinum, vip
  status VARCHAR(20) DEFAULT 'active', -- active, expired, cancelled
  start_date TIMESTAMP DEFAULT NOW(),
  end_date TIMESTAMP,
  payment_id VARCHAR(255),
  amount DECIMAL(10,2),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Reciprocity tracking
CREATE TABLE reciprocity_state (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  field_bundle VARCHAR(50) NOT NULL, -- photos, education, occupation, income, family
  is_eligible BOOLEAN DEFAULT FALSE,
  last_calculated TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, field_bundle)
);

-- Community-specific tables
CREATE TABLE communities (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  religion VARCHAR(50) NOT NULL,
  region VARCHAR(100) NOT NULL,
  priority INTEGER DEFAULT 0,
  is_active BOOLEAN DEFAULT TRUE
);

CREATE TABLE sub_communities (
  id SERIAL PRIMARY KEY,
  community_id INTEGER REFERENCES communities(id),
  name VARCHAR(100) NOT NULL,
  population_estimate INTEGER,
  digital_adoption_score INTEGER,
  is_active BOOLEAN DEFAULT TRUE
);
```

## 15. Operational Practices (Vercel Hobby + Supabase Free)

- **Lazy realtime connections**: Only subscribe to Supabase Realtime channels when a conversation is open; disconnect after 5–10 minutes of inactivity to stay well under the free-tier 500 concurrent cap.
- **Message retention**: Keep 6–12 months of chat history in the primary `messages` table. Export older data to Supabase Storage (CSV/JSON) if database size approaches 400 MB.
- **Media handling**: Store photos and attachments in Supabase Storage; persist only metadata and `storage_path` keys in Postgres. Use signed URLs for access.
- **Scheduled work**: Use Vercel Cron (≤12 jobs) for digests and cleanup. For heavier processing, leverage Supabase Edge Functions triggered by webhooks or scheduled via Supabase.
- **Security**: Enforce Row Level Security on all tables. Supabase policies govern reciprocity visibility and admin overrides. Manage secrets via Vercel/Supabase dashboards.

## 16. Monitoring & Upgrade Triggers

| Metric | Monitor In | Threshold | Action |
|--------|------------|-----------|--------|
| Database size | Supabase usage | ≥400 MB | Start archiving or upgrade to Supabase Pro |
| Realtime connections | Supabase usage | ≥400 concurrent | Refine connection lifecycle or upgrade |
| Storage usage | Supabase usage | ≥900 MB | Clear stale assets or purchase add-on |
| Bandwidth/runtime | Vercel analytics | ≥80 GB/mo or sustained >1 req/s | Upgrade to Vercel Pro |
| Error rate | Sentry alerts | Above baseline | Triage and fix |

Automate weekly metric snapshots (Supabase Edge Function or GitHub Action) so trends are visible before thresholds are hit.

## 17. Implementation Roadmap

### **Phase 1 – Foundation (Weeks 1-2)**
- Scaffold Next.js 16 (App Router, TypeScript, Tailwind, shadcn/ui).
- Provision Supabase project; configure Auth providers, Storage buckets, policies.
- Define schema with Supabase migrations/SQL; enable RLS and seed baseline data.
- Integrate Supabase Auth on web (client + server helpers) and set up CI (lint/test/typecheck).

### **Phase 2 – Core Platform (Weeks 3-5)**
- Build profile onboarding, verification, reciprocity enforcement flows.
- Implement search, filters, and interest/matching logic backed by Supabase.
- Integrate Razorpay (web checkout + webhook Edge Function) and PhonePe fallback.
- Set up plan management, entitlements, and dashboard widgets.

### **Phase 3 – Communication & Admin (Weeks 6-7)**
- Deliver realtime chat with lazy connections, read receipts, and file sharing.
- Handle media uploads via Supabase Storage with watermarking pipeline.
- Build admin panel (user moderation, payment tracking, privacy controls).
- Configure notifications (Resend template set, Fast2SMS OTP flow).

### **Phase 4 – Launch Prep (Week 8)**
- Add analytics dashboards (PostHog) and performance monitoring (Vercel Analytics).
- Populate staging data, run E2E and load tests, finalize security hardening.
- Document runbooks, set alerts for monitoring thresholds, point domain to Vercel.

## 18. Next Steps
1. Confirm Supabase and Vercel projects plus GitHub repo are provisioned.
2. Establish migration workflow (Supabase SQL or db diff) and CI checks.
3. Implement MVP slices following the roadmap, validating against FEATURE-REFERENCE.md.
4. Configure monitoring alerts before opening beta access.

---

**Last Updated:** October 2025  
**Document Version:** 3.0