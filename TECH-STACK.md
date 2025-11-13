# CC Matrimony Tech Stack (Vercel Hobby + Supabase Free)

## 1. Architecture Overview
- **Frontend Hosting**: Vercel Hobby (Next.js SSR/ISR with limited build minutes)
- **Backend & Data Layer**: Supabase Free (PostgreSQL ≤500 MB, Auth, 1 GB Storage, Edge Functions)
- **APIs**: Vercel Serverless Functions + Supabase Edge Functions
- **Mobile Clients**: Responsive web-first design
- **Observability**: Sentry (error tracking) + Supabase logs + PostHog (analytics)
- **System Health**: Real-time monitoring of database, storage, and API performance
- **SEO**: Structured data, dynamic sitemaps, and OpenGraph integration
- **Freemium Model**: Conversion tracking and upgrade prompts

## 2. Frontend (Web)
| Component | Choice | Notes |
|-----------|--------|-------|
| Framework | **Next.js 16.0.1** (React 19.2.0, App Router) | SSR/ISR for SEO-heavy landing pages, SSG for static content |
| Styling | **Tailwind CSS 4.1.16 + Radix UI + shadcn/ui** | Utility-first styling, accessible primitives, component library |
| State/Data | **TanStack Query v5 + Zustand** | Caching, background refetch, offline revalidation, state management (introduce Zustand in Week 3-4 as needed) |
| Validation | **Zod + React Hook Form** | Type-safe schemas, granular form control (introduce Zod in Week 3+) |
| Authentication | **Supabase Auth** | Email/password only for MVP; defer phone OTP/OAuth to Phase 2 |
| Image Handling | **Cloudinary** | Automatic optimization, resizing, format conversion (WebP/AVIF), built-in CDN, watermarking, transformations; mature service with excellent docs (setup in Week 3) |
| Notifications | **Supabase Realtime + Email** | Real-time notifications via Supabase Realtime, email fallback |

## 3. Mobile Apps
- **Primary**: Progressive Web App (PWA) with mobile-first responsive design
- **Features**: Home screen installation, push notifications, offline caching, app-like experience
- **Future**: React Native app when user base exceeds 5,000+ MAU (deferred to Phase 3)
- **Implementation**: Next.js built-in PWA support with service workers

## 4. Backend & Data
| Area | Service | Responsibilities |
|------|---------|------------------|
| Database | **Supabase PostgreSQL** | Normalized profile data, photo requests, plan entitlements, audit logs |
| Auth | **Supabase Auth** | Email/password only (MVP); phone OTP and social login deferred |
| Storage | **Cloudinary** | Photo storage with automatic optimization, resizing, CDN delivery, and built-in transformations (setup in Week 3) |
| Realtime | **Supabase Realtime** | Real-time notifications (optional, for future features) |
| Business Logic | **Supabase Edge Functions + Vercel API Routes** | Photo request handling, plan upgrades, admin actions |
| Background Jobs | **Vercel Cron (≤12 jobs)** | Daily analytics, email digests, cleanup tasks |

### Data Modeling Notes
- Photo requests tracked in dedicated table
- Profile view log with 90-day retention (within 500MB limit)
- Audit tables kept lightweight (essential fields only)
- Full PostgreSQL control for complex queries

## 5. Search Strategy (PHASED APPROACH)
- **Week 1-2 (MVP)**: Simple WHERE clauses only
  ```sql
  WHERE religion = 'hindu' 
    AND primary_community = 'bunt'
    AND age BETWEEN 25 AND 35
    AND gender = 'female'
  ```
- **Week 3-4**: Add basic indexes for performance
  ```sql
  CREATE INDEX idx_profiles_search 
  ON profiles(religion, community, age, gender);
  ```
- **Week 5+**: Add PostgreSQL full-text search for name/bio
  ```sql
  CREATE INDEX idx_profiles_fulltext 
  ON profiles USING GIN(to_tsvector('english', name || ' ' || about_me));
  ```
- **Post-Launch**: Consider Elasticsearch only if search becomes slow (>1000 users)
- **AI Development Note**: Start with simple filters, add complexity incrementally

## 6. Media Handling
- **Upload**: Direct upload to Cloudinary via Vercel API route; PNG/GIF/JPG/JPEG/WebP up to 10 MB (Cloudinary handles optimization automatically)
- **Plan Quotas**: Free (1 profile + 1 album slot); Paid (1 profile + 9 album + 3 family/group slots)
- **Processing (SIMPLIFIED for MVP)**: 
  - **Phase 1 (Week 3)**: Upload original image to Cloudinary via Vercel API; Cloudinary automatically optimizes, resizes, and converts to WebP/AVIF
  - **Phase 2 (Post-launch)**: Add watermarking via Cloudinary transformations (built-in, no extra setup)
  - **No client-side compression needed**: Cloudinary handles all optimization server-side
  - **Responsive variants**: Cloudinary generates multiple sizes automatically; use `srcset` or Cloudinary's responsive breakpoints
- **Display Fit**: Cloudinary provides URL parameters for transformations (`w_400,h_600,c_fill`) or use named transformations
- **Moderation**: Manual admin approval (auto-approve by default)
- **Watermarking**: Built-in via Cloudinary transformations (defer to post-launch, but easy to add)
- **Cloudinary Readiness**: End of Week 2 run a smoke test from a Node.js Vercel API route to upload a test image. Free tier: 25GB storage + 25GB bandwidth/month (plenty for MVP). If fails, temporarily switch `STORAGE_PROVIDER` env to `supabase` for Week 3.

### 6.1 MVP Photo Upload Flow
1. Signup (email/password) → email verification.
2. Complete basic profile (name, age, gender, religion, primary community).
3. Users can browse profiles immediately (no photo required).
4. Other users can request photos from profiles that don't have any photos yet.

#### 6.1.1 Photo Upload UX Copy

**Profile Card** (when profile has no photos):
```
📷 No photos uploaded yet
[Request Photo Button]
```

**After Request Sent**:
```
✅ Photo request sent
The user will be notified to upload photos.
```

## 7. Payments & Monetization
| Capability | Service | Notes |
|------------|---------|-------|
| Online Payments | **Razorpay** | Primary (and only) payment gateway - handles UPI, cards, netbanking, wallets |
| Offline Payments | **Manual Processing** | Bank transfer, cash payments with admin approval |
| Future Gateways | **PhonePe/Paytm** | Add only if Razorpay has issues (deferred to Phase 2) |
| Webhooks | **Vercel API Routes** | Payment verification and subscription management |
| Invoicing | **PostgreSQL + PDF Generation** | Automated invoice generation |

> **Implementation note (Week 6):** Razorpay webhooks must be idempotent. Verify the `x-razorpay-signature` before processing, upsert a `payment_events` record keyed by `razorpay_payment_id`, and respond `200` on duplicates. This prevents early/double notifications from activating subscriptions twice.

## 8. Communications
- **Email**: Resend (transactional templates, interest alerts)
  - Sender: noreply@matri.naveevo.com (subdomain for $0 extra cost)
  - Reply-to: support@matri.naveevo.com
  - DNS: SPF, DKIM, DMARC configured
- **SMS/OTP**: Deferred to Phase 2 (email-only auth for MVP)
- **Push Notifications**: Supabase Realtime notifications

## 9. Admin & Management Tools

> **See [ADMIN-MANAGEMENT-SPEC.md](./ADMIN-MANAGEMENT-SPEC.md) for complete admin panel specifications, features, implementation plan, and future phases.**

**Quick Reference:**
- **Admin Panel**: Next.js app on Vercel, protected via Supabase Auth (Weeks 7-9)
- **Tech Stack**: React Admin + ra-supabase for CRUD, shadcn/ui for custom components
- **MVP Features**: Authentication, User/Profile/Payment Management, Analytics, Settings, Content Management
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

> **Environment note:** MVP operates with two Supabase projects (Dev shared by local + preview, Prod for live traffic). Revisit a dedicated staging project once we exceed free-tier quotas or introduce destructive preview tests.

## 13. Cost Snapshot (Monthly Estimates)
| Item | Est. Cost |
|------|-----------|
| Vercel Hobby | $0 |
| Supabase Free | $0 (until ~800 users) |
| Resend | $40-50 (realistic for 5K emails/month) |
| Monitoring (Sentry + Vercel) | $0 (free tiers) |
| Razorpay fees | Variable (~2% per transaction) |
| **Total MVP (0-500 users)** | **~$60-80/month** |
| **After 800 users** | **~$135-165/month** (Supabase Pro $25 + increased comms) |

> **Revised Estimates**: More realistic cost projections based on actual email/SMS usage patterns. Plan for $70-80/month MVP costs.

### **13.1 Storage Strategy & Migration Plan**
```typescript
STORAGE_CONFIG = {
  mvp: 'Cloudinary (automatic optimization, built-in CDN, transformations; setup in Week 3)',
  abstraction: 'Store only public_id in DB (provider-agnostic)',
  migration_trigger: {
    storage_used: '50GB',
    bandwidth: '10TB/month',
    monthly_cost: '>$50 for storage alone'
  },
  migration_target: 'Remain on Cloudinary; consider R2 only if non-image files needed or cost exceeds $50/month',
  implementation: 'Abstract storage layer from day 1 for easy provider swaps'
}
```

### **13.2 Photo Upload Limits (Revised)**
```typescript
MAX_UPLOAD_SIZES = {
  photo: '15MB',        // Increased from 10MB - client-side compression
  horoscope: '3MB',     // PDF, JPG, PNG - increased from 1MB
  document: '5MB'       // Future use - increased from 2MB
}
```

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
  public_id VARCHAR(255) NOT NULL, -- Cloudinary identifier (provider-agnostic)
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

-- Payment events for webhook idempotency (Week 6)
CREATE TABLE payment_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  razorpay_payment_id VARCHAR(255) UNIQUE NOT NULL,
  razorpay_order_id VARCHAR(255),
  event_type VARCHAR(50) NOT NULL, -- payment.captured, payment.failed, etc.
  payload JSONB NOT NULL,
  processed BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);
CREATE INDEX idx_payment_events_razorpay_id ON payment_events(razorpay_payment_id);

-- Photo requests tracking
CREATE TABLE photo_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  requester_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  profile_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  requested_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(requester_id, profile_id)
);

CREATE INDEX idx_photo_requests_profile ON photo_requests(profile_id);
CREATE INDEX idx_photo_requests_requester ON photo_requests(requester_id);

-- Community-specific tables
CREATE TABLE communities (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  religion VARCHAR(50) NOT NULL,
  slug VARCHAR(100) NOT NULL UNIQUE,
  description TEXT,
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW()
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

- **Media handling**: Store photos and attachments in Cloudinary; persist only metadata and `public_id` in Postgres. Access via Cloudinary URLs (with optional signed URLs).
- **Scheduled work**: Use the **simplified 4-job Vercel Cron schedule** (down from 8 jobs). Group adjacent tasks inside each invocation to stay within Hobby limits and log every run to a `cron_runs` table so the admin "Operations" widget can show last run, duration, and status.
  - **Daily Morning (09:00)**: Match recommendations, birthday nudges, premium expiry alerts, profile verification reminders, "Who viewed me" digest
  - **Daily Evening (18:00)**: Re-engagement emails, inactive user nudges, profile completion reminders
  - **Daily Midnight (00:00)**: Analytics snapshot, sitemap regeneration, database cleanup, backup verification, photo moderation queue
  - **Weekly (Sun 10:00)**: Success story reminders, partner follow-ups, weekly reports
- **Bandwidth planning**: Track Vercel bandwidth weekly; Hobby tier caps at 100 GB/month. Trigger upgrade to Vercel Pro once sustained usage exceeds ~80 GB/month (≈300–500 light MAU or 200–300 average MAU).
- **Storage planning**: Supabase 1 GB free tier supports ~1,000 users with ≤2 compressed photos each (<500 KB). Set alert at 900 MB; plan upgrade to Supabase Pro or external storage when thresholds are hit.
- **Security**: Enforce Row Level Security on all tables. Supabase policies govern photo privacy and admin overrides. Manage secrets via Vercel/Supabase dashboards.

## 16. System Health Monitoring

### 16.1 Core Metrics
- **Database**: Size, active connections, query performance
- **Storage**: Used space, file count, largest files
- **API**: Response times, error rates, request volume
- **Users**: Active sessions, signups, conversions

### 16.2 Implementation (Supabase + Vercel)
- **Database Monitoring**: Supabase dashboard + custom SQL queries
- **Storage Monitoring**: Cloudinary usage via API/dashboard (storage used, bandwidth, transformations)
- **API Monitoring**: Vercel Analytics + custom logging
- **Alerting**: Email notifications for critical issues

### 16.3 Health Check Endpoint
```typescript
// pages/api/health.ts
export default async function handler(req, res) {
  try {
    // Basic database check
    const { data: dbCheck, error: dbError } = await supabase
      .from('health_check')
      .select('*')
      .limit(1);

    if (dbError) throw dbError;

    // Storage check (Cloudinary - fetch stats via Cloudinary Admin API)
    const { storage, bandwidth } = await getCloudinaryStats(); // implement using Cloudinary Admin API

    return res.status(200).json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      database: { status: 'connected', tables: dbCheck?.length ? 'ok' : 'empty' },
      storage: { 
        status: 'connected',
        files,
        size_mb: sizeMb
      },
      system: {
        memory: process.memoryUsage().heapUsed / 1024 / 1024, // MB
        uptime: process.uptime() // seconds
      }
    });
  } catch (error) {
    console.error('Health check failed:', error);
    return res.status(500).json({
      status: 'error',
      error: error.message,
      timestamp: new Date().toISOString()
    });
  }
}
```

### 16.4 Monitoring Setup
1. **Vercel Cron Job** (runs every hour)
   - Calls `/api/health` endpoint
   - Logs results to `health_logs` table
   - Sends email alert if status is not 'healthy'

2. **Health Dashboard** (Admin-only)
   - Simple dashboard showing:
     - Current system status
     - 24-hour trend of key metrics
     - Recent errors or warnings
     - Storage usage with warning at 80% capacity

3. **Alert Thresholds**
   - Storage usage (Cloudinary) – set an early MVP alert around 1 GB (budget-based)
   - Database response time > 500ms
   - API error rate > 5%
   - More than 10 failed login attempts in 5 minutes

### 16.5 Database Schema for Monitoring
```sql
-- Health check logs
CREATE TABLE health_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  status TEXT NOT NULL, -- 'healthy', 'warning', 'error'
  component TEXT, -- 'database', 'storage', 'api', 'auth'
  message TEXT,
  details JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- System metrics (hourly snapshots)
CREATE TABLE system_metrics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  storage_used_mb DECIMAL(10,2),
  storage_files INT,
  db_size_mb DECIMAL(10,2),
  active_sessions INT,
  api_requests_1h INT,
  error_rate_1h DECIMAL(5,2),
  avg_response_time_ms DECIMAL(10,2)
);

-- Index for time-based queries
CREATE INDEX idx_system_metrics_timestamp ON system_metrics(timestamp);

-- Simple health check table
CREATE TABLE health_check (
  id INT PRIMARY KEY DEFAULT 1,
  last_checked TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Insert initial row
INSERT INTO health_check (id) VALUES (1) ON CONFLICT (id) DO NOTHING;
```

### 16.6 Implementation Notes
- Uses existing Supabase free tier features
- No additional cost for monitoring
- Lightweight implementation
- Easy to extend with more metrics
- Simple to maintain

### 16.7 Future Improvements
- Add more detailed metrics as needed
- Implement rate limiting on health endpoint
- Add authentication for health endpoint
- Set up more granular alerts
- Add historical data analysis

### Health Dashboard

> **Admin health monitoring dashboard - see [ADMIN-MANAGEMENT-SPEC.md](./ADMIN-MANAGEMENT-SPEC.md) for admin panel details.**
```typescript
interface SystemHealth {
  database: {
    size_mb: number;
    usage_percent: number; // vs 500 MB limit
    top_tables: Array<{ name: string; size_mb: number }>;
  };
  storage: {
    size_gb: number;
    usage_percent: number; // vs 1 GB limit
    largest_files: Array<{ path: string; size_mb: number }>;
  };
  realtime: {
    active_connections: number;
    usage_percent: number; // vs 500 limit
  };
  api_performance: {
    p50_ms: number;
    p95_ms: number;
    error_rate: number;
  };
}

// Alert thresholds
const ALERTS = {
  database_size: 400,  // MB - Alert at 80%
  storage_size: 0.8,   // GB - Alert at 80%
  connections: 400,    // Alert at 80%
  error_rate: 0.05     // 5% error rate
};
```

## 17. SEO Strategy Implementation

### Structured Data (Schema.org)
- **WebSite Schema**: For community pages and search functionality
- **Person Schema**: For user profiles
- **BreadcrumbList**: For navigation structure
- **FAQPage**: For common questions

### Dynamic Sitemaps
- Generate sitemaps for profiles and communities
- Update sitemap on profile creation/update
- Auto-generate sitemap entries for all active communities

### OpenGraph Implementation
- Dynamic OpenGraph images per community
- Profile-specific metadata for sharing
- Local business schema for regional SEO

## 18. Freemium Conversion System

### Conversion Events Tracking
```sql
CREATE TABLE conversion_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  event_type VARCHAR(50) NOT NULL, -- 'interest_limit', 'contact_locked'
  metadata JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Index for faster lookups
CREATE INDEX idx_conversion_events_user_created ON conversion_events(user_id, created_at);
```

### Upgrade Opportunity Detection
```typescript
const checkConversionOpportunity = async (userId: string) => {
  const { data: events, error } = await supabase
    .from('conversion_events')
    .select('event_type')
    .eq('user_id', userId)
    .gte('created_at', new Date(Date.now() - 86400000).toISOString());

  if (error) {
    console.error('Error checking conversion events:', error);
    return null;
  }

  // Trigger upgrade prompt after 3+ events in 24h
  if (events && events.length >= 3) {
    return {
      showUpgradeModal: true,
      message: "You've hit free limits multiple times today. Upgrade to Silver for unlimited access!"
    };
  }
  return null;
};
```

### Conversion Funnel Points
1. **Interest Limit Reached**
2. **Contact View Blocked**
3. **Profile View Limit**
4. **Search Result Limit**

## 19. Monitoring & Upgrade Triggers
| Metric | Monitor In | Threshold | Action |
|--------|------------|-----------|--------|
| Database size | Supabase usage | ≥400 MB | Start archiving or upgrade to Supabase Pro |
| Realtime connections | Supabase usage | ≥400 concurrent | Refine connection lifecycle or upgrade |
| Storage usage | Cloudinary usage | MVP: alert ~1 GB (budget-based) | Clear stale assets or adjust plan/alerts |
```typescript
const morningTasks = async () => {
  const tasks = [
    { name: 'match-recommendations', run: sendMatchRecommendations },
    { name: 'birthday-notifications', run: sendBirthdayNotifications },
    { name: 'premium-expiry-alerts', run: sendPremiumExpiryAlerts }
  ];

  const results = await Promise.allSettled(tasks.map(t => t.run()));

  await logCronRun({
    job_name: 'morning-tasks',
    tasks_completed: results.map((result, index) => ({
      task: tasks[index].name,
      status: result.status,
      error: result.status === 'rejected' ? String(result.reason) : null
    }))
  });
};
```

### 16.2 Automated cleanup helpers


```typescript
// Weekly Supabase Edge Function invoked by cron to monitor DB size
export const checkDatabaseSize = async () => {
  const { data, error } = await supabase.rpc('get_db_size');
  if (error) throw error;

  if (data.size_mb >= 400) {
    await notifyAdmins({
      channel: 'system',
      title: 'Database approaching free-tier limit',
      body: `Current size: ${data.size_mb} MB`
    });
  }
};
```

### 16.3 Monitoring & alerting

- **Uptime**: BetterUptime free tier (10 monitors) hitting key routes; alerts sent via admin operations panel widgets and email.
- **Error alerts**: Sentry + Vercel function errors routed to admin email list; surfaced in admin panel incident feed for follow-up.
- **Bandwidth & storage**: Weekly cron runs `checkProjectUsage` to notify at 80 GB bandwidth or 900 MB storage.
- **Cron dashboard**: Admin panel surfaces `cron_runs` data with retry option for failed tasks.

### 16.4 Upcoming implementation tickets

- **Watermarking via Cloudinary transformations**: Build `/api/upload/photo` route uploading to Cloudinary; use Cloudinary transformations for watermarking (built-in, easy to add); adjust client flow to call API and add telemetry.
- **Chat idle timeout enforcement**: Add Supabase listener wrapper that auto-disconnects after 10 minutes idle, refreshes tokens every 5 minutes, and logs disconnect reasons for auditing.

## 17. Implementation Roadmap

> Timeline context: These four phases cover the **eight-week build cycle leading into launch (Month −2 to Month 0)**. All communities are treated equally - community pages auto-generate from database.

> **AI Development Note:** This roadmap is optimized for AI-driven development (Cursor/Windsurf). Build incrementally, test after each feature, and avoid generating entire pages in one prompt.

### **Phase 1 – Foundation (Weeks 1-2)** [SIMPLIFIED FOR AI]
**Day 1-2: Project Setup**
- Scaffold Next.js 16 (App Router, TypeScript, Tailwind, shadcn/ui)
- Provision Supabase project; configure Auth providers (email/password + Google only)
- Create `.env.local` with all required keys
- Set up GitHub repo with CI (lint/test/typecheck)

**Day 3-4: Core Schema (Minimal)**
- Create ONLY: `users`, `profiles`, `photos` tables
- Skip RLS policies initially (use service role for testing)
- Generate TypeScript types: `npx supabase gen types typescript > types/database.ts`
- Seed 5-10 test profiles manually

**Day 5-7: Basic Auth + Profile Flow**
- Implement login/register pages (no OTP yet)
- Build simple profile creation form (name, age, gender, community only)
- Test: User can register → create profile → view own profile
- **Checkpoint:** Working auth + basic profile (no photos, no search yet)

### **Phase 2 – Core Platform (Weeks 3-5)** [SIMPLIFIED FOR AI]
**Week 3: Photos + Search**
- Day 8-9: Photo upload (direct upload to Cloudinary via Vercel API; Cloudinary handles optimization automatically; defer watermarking transformations)
- Day 10-11: Basic search page (simple WHERE filters: age, gender, community)
- Day 12-14: Profile detail page + photo gallery
- **Checkpoint:** Users can upload photos, search, and view other profiles

**Week 4: Interests + Subscriptions**
- Day 15-16: Interest system (send/receive/accept/decline)
- Day 17-18: Subscription plans table + basic plan display
- Day 19-21: Razorpay integration (checkout page + webhook)
- **Checkpoint:** Users can send interests and purchase plans

**Week 5: Photo Request Feature**
- Day 22-24: Implement photo request API routes and database table
- Day 25-26: Add "Request Photo" button UI for profiles without photos + notifications
- Day 27-28: Testing and polish
- **Checkpoint:** Photo request feature live; users can request photos from profiles without any photos

### **Phase 3 – Engagement (Week 6)** [SIMPLIFIED FOR AI]
**Week 6: Notifications & Engagement**
- Day 29-31: Email notifications via Resend (templates: welcome, interest received, match)
- Day 32-35: Interest reminders, profile completion nudges, daily/weekly digest emails
- **Checkpoint:** Engagement emails working end-to-end

### **Phase 4 – Admin Panel (Weeks 7-9)** [MVP]

> **See [ADMIN-MANAGEMENT-SPEC.md](./ADMIN-MANAGEMENT-SPEC.md) section 4 for detailed week-by-week implementation plan.**

**Week 7: Admin Foundation**
- Admin authentication + user management (list, search, suspend, activate)
- User detail views + subscription management

**Week 8: Profile & Payment Management**
- Profile approval queue (approve/reject with reasons)
- Photo moderation queue (review flagged photos)
- Payment management (view payments, process refunds)

**Week 9: Analytics & Settings**
- Analytics dashboard (user stats, profile stats, payment stats)
- System settings (feature toggles, email/SMS config, payment gateway)
- Content management (success stories, testimonials)

### **Phase 5 – Launch Prep (Week 10)** [SIMPLIFIED FOR AI]
**Week 10: Polish + Launch**
- Day 57-58: Add RLS policies (now that features work)
- Day 59-60: Mobile responsive fixes + PWA manifest
- Day 61-62: SEO (meta tags, sitemap, robots.txt)
- Day 63-70: Final testing, bug fixes, staging deployment + smoke tests
- **Checkpoint:** Platform ready for launch with complete admin panel

## 18. Environment Variables (.env.local)

```bash
# Supabase (Required Day 1)
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Cloudinary (Add Week 3)
CLOUDINARY_CLOUD_NAME=xxxxxxxxxxxxxxxxxxxx
CLOUDINARY_API_KEY=xxxxxxxxxxxxxxxxxxxx
CLOUDINARY_API_SECRET=xxxxxxxxxxxxxxxxxxxx  # Keep secret!

# Payments (Required Week 4)
RAZORPAY_KEY_ID=rzp_test_xxxxx
RAZORPAY_KEY_SECRET=xxxxx

# Communications (Required Week 6)
RESEND_API_KEY=re_xxxxx
# FAST2SMS_API_KEY (Deferred to Phase 2)

# Monitoring (Optional, add Week 8)
NEXT_PUBLIC_SENTRY_DSN=https://xxxxx@sentry.io/xxxxx
NEXT_PUBLIC_POSTHOG_KEY=phc_xxxxx
NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com

# Admin (Required Weeks 7-9)
ADMIN_EMAIL=admin@matri.naveevo.com
ADMIN_PASSWORD_HASH=xxxxx
```

## 19. API Routes Structure

### **Authentication APIs**
```
POST   /api/auth/register          - Register new user
POST   /api/auth/login             - Login user
POST   /api/auth/logout            - Logout user
POST   /api/auth/reset-password    - Request password reset
GET    /api/auth/session           - Get current session
```

### **Profile APIs**
```
GET    /api/profiles               - List profiles (with filters)
POST   /api/profiles               - Create profile
GET    /api/profiles/[id]          - Get single profile
PATCH  /api/profiles/[id]          - Update profile
DELETE /api/profiles/[id]          - Delete profile
GET    /api/profiles/me            - Get current user's profile
```

### **Photo APIs**
```
POST   /api/photos                 - Upload photo
DELETE /api/photos/[id]            - Delete photo
PATCH  /api/photos/[id]/primary    - Set as primary photo
```

### **Interest APIs**
```
GET    /api/interests              - List sent/received interests
POST   /api/interests              - Send interest
PATCH  /api/interests/[id]         - Accept/decline interest
DELETE /api/interests/[id]         - Withdraw interest
```

### **Search APIs**
```
GET    /api/search                 - Search profiles
GET    /api/search/suggestions     - Get search suggestions
```

### **Subscription APIs**
```
GET    /api/subscriptions          - Get user subscriptions
POST   /api/subscriptions/checkout - Create Razorpay order
POST   /api/subscriptions/verify   - Verify payment webhook
```

### **Admin APIs**

> **See [ADMIN-MANAGEMENT-SPEC.md](./ADMIN-MANAGEMENT-SPEC.md) section 5 for complete admin API routes reference.**

**Core Admin Endpoints:**
```
GET    /api/admin/users            - List all users
PATCH  /api/admin/users/[id]       - Update user status
GET    /api/admin/profiles/pending - Get pending approvals
PATCH  /api/admin/profiles/[id]    - Approve/reject profile
GET    /api/admin/stats            - Get dashboard stats
```

## 20. Component Architecture

### **Layout Components**
```
app/
├── (auth)/
│   ├── layout.tsx              - Auth layout (centered, no nav)
│   ├── login/page.tsx
│   └── register/page.tsx
├── (dashboard)/
│   ├── layout.tsx              - Main layout (header, sidebar, footer)
│   ├── dashboard/page.tsx
│   ├── search/page.tsx
│   ├── profile/[id]/page.tsx
│   └── interests/page.tsx
└── (admin)/
    ├── layout.tsx              - Admin layout (admin nav)
    └── admin/
        ├── page.tsx
        ├── users/page.tsx
        └── profiles/page.tsx
```

### **Shared Components**
```
components/
├── ui/                         - shadcn/ui components
│   ├── button.tsx
│   ├── card.tsx
│   ├── input.tsx
│   └── dialog.tsx
├── shared/
│   ├── Header.tsx
│   ├── Footer.tsx
│   ├── Sidebar.tsx
│   └── ProfileCard.tsx
└── features/
    ├── auth/
    │   ├── LoginForm.tsx
    │   └── RegisterForm.tsx
    ├── profile/
    │   ├── ProfileForm.tsx
    │   ├── PhotoUpload.tsx
    │   └── ProfileGallery.tsx
    ├── search/
    │   ├── SearchFilters.tsx
    │   └── SearchResults.tsx
    └── interests/
        ├── InterestButton.tsx
        └── InterestList.tsx
```

## 21. Database Schema (Phased Approach)

### **Phase 1 (Week 1-2): Core Tables**
```sql
-- users table (managed by Supabase Auth)
-- profiles table (basic fields only)
CREATE TABLE profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  name VARCHAR(100) NOT NULL,
  age INTEGER NOT NULL,
  gender VARCHAR(10) NOT NULL,
  community VARCHAR(50) NOT NULL,
  religion VARCHAR(20) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- photos table
CREATE TABLE photos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  profile_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  storage_path VARCHAR(255) NOT NULL,
  is_primary BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### **Phase 2 (Week 3-5): Feature Tables**
```sql
-- interests table
CREATE TABLE interests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id UUID REFERENCES profiles(id),
  receiver_id UUID REFERENCES profiles(id),
  status VARCHAR(20) DEFAULT 'pending',
  message TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- subscriptions table
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id),
  plan_type VARCHAR(20) NOT NULL,
  status VARCHAR(20) DEFAULT 'active',
  start_date TIMESTAMP DEFAULT NOW(),
  end_date TIMESTAMP,
  payment_id VARCHAR(255)
);

-- photo_requests table
CREATE TABLE photo_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  requester_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  profile_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  requested_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(requester_id, profile_id)
);

CREATE INDEX idx_photo_requests_profile ON photo_requests(profile_id);
CREATE INDEX idx_photo_requests_requester ON photo_requests(requester_id);
```


## 22. Supabase RLS Policies (Add Week 10)

```sql
-- Profiles: Users can view approved profiles
CREATE POLICY "Profiles are viewable by authenticated users"
ON profiles FOR SELECT
TO authenticated
USING (is_approved = true);

-- Profiles: Users can edit their own profile
CREATE POLICY "Users can edit own profile"
ON profiles FOR UPDATE
TO authenticated
USING (user_id = auth.uid());

-- Photos: Users can view photos of approved profiles
CREATE POLICY "Photos are viewable by authenticated users"
ON photos FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles 
    WHERE profiles.id = photos.profile_id 
    AND profiles.is_approved = true
  )
);

-- Interests: Users can view their own interests
CREATE POLICY "Users can view own interests"
ON interests FOR SELECT
TO authenticated
USING (sender_id = auth.uid() OR receiver_id = auth.uid());

-- Messages: Users can view their own messages
CREATE POLICY "Users can view own messages"
ON messages FOR SELECT
TO authenticated
USING (sender_id = auth.uid() OR receiver_id = auth.uid());
```

## 23. AI Development Best Practices

### **For Cursor/Windsurf:**

1. **Incremental Prompts**
   - ❌ "Build the entire profile page with all features"
   - ✅ "Create a basic profile form with name, age, gender fields"

2. **Test After Each Feature**
   - Build auth → test login/register
   - Build profile creation → test form submission
   - Build search → test filtering

3. **Use Templates**
   - Create one API route correctly, then replicate pattern
   - Build one form component, then reuse for other forms

4. **Type Safety First**
   - Generate database types before building components
   - Use Zod schemas for form validation

5. **Rollback Strategy**
   - Git commit after each working feature
   - Tag major milestones: `git tag week-1-complete`

## 24. Next Steps
1. Confirm Supabase and Vercel projects plus GitHub repo are provisioned.
2. Create `.env.local` with all required keys (see section 18).
3. Generate TypeScript types: `npx supabase gen types typescript > types/database.ts`
4. Follow day-by-day roadmap (section 17), testing after each checkpoint.
5. Configure monitoring alerts before opening beta access.

---

**Last Updated:** November 6, 2025  
**Document Version:** 5.0 (AI-Optimized for Implementation)  
**Implementation Start:** November 7, 2025

---

## 25. Frontend-First Planning (reference only; no new files)

### 25.1 Mock Data (Week 1-2; TypeScript constants)
- Create small constant datasets (5 users, 20 profiles) mirroring `profiles` and `photos` table shapes.
- Import directly into pages/components for initial UI without backend coupling.
- Store alongside feature UIs (e.g., `features/search/__mocks__/profiles.ts`) and remove once APIs are wired.

### 25.2 Component Build Order (vertical slices with backend)
- Week 1: Layout shell (Header, Footer), Auth UI, Dashboard shell, Profile form UI.
- Week 2: Search filters + results UI, Profile card, Profile detail UI.
- Week 3: Photo upload wired (Vercel API → Cloudinary), introduce Zustand where cross-feature state emerges.
- Week 4: Interests UI + backend, subscriptions UI + backend.

### 25.3 UI Patterns
- Use Tailwind defaults for spacing/typography/colors; extract tokens only after repeated use.
- Use shadcn/ui directly; extract shared components after ≥3 reuse cases.
- Sentry from Week 1; introduce Zod schemas in Week 3+ when forms become complex.

---

## 26. Reference Documents

### 26.1 Pre-Launch Checklist
- **File**: `PRE-LAUNCH-CHECKLIST.md`
- **Purpose**: Complete checklist of infrastructure setup, accounts, legal pages, testing, and deployment steps.
- **Use**: Review before Day 1, and reference during Week 10 deployment.

### 26.2 Development Patterns
- **File**: `DEVELOPMENT-PATTERNS.md`
- **Purpose**: Common code patterns, error handling, authentication, database queries, file uploads, payments, and email.
- **Use**: Reference when building features to maintain consistency and avoid reinventing patterns.

### 26.3 Feature Reference
- **File**: `FEATURE-REFERENCE.md`
- **Purpose**: Quick reference for feature verification, priority levels, and implementation order.
- **Use**: Verify feature completeness and check priority during development.

### 26.4 Profile Fields
- **File**: `PROFILE-FIELDS.md`
- **Purpose**: Single source of truth for all profile field definitions, validation rules, and phasing.
- **Use**: Reference when building profile forms and validation.

### 26.5 Photo Request Specification
- **File**: `PHOTO-REQUEST-SPEC.md`
- **Purpose**: Detailed specification of the photo request feature (request users to upload photos when they have none).
- **Use**: Reference when implementing profile visibility and photo access rules.

### 26.6 Admin Management (Phase 2)
- **File**: `ADMIN-MANAGEMENT-SPEC.md`
- **Purpose**: Admin panel specification (deferred to Phase 2).
- **Use**: Reference when building admin features post-MVP.


---

## 27. URL Structure & Community Routing (Technical Implementation)

### 27.1 URL Patterns
```
/ - Home
/[religion] - Religion page (e.g., /hindu, /christian)
/[religion]/[community] - Community page (auto-generated from database, e.g., /hindu/bunt, /christian/latin-catholic)
```

### 27.2 Next.js Dynamic Routing
```typescript
// File structure (App Router)
app/
  [religion]/
    [community]/
      page.tsx         // Community page
      [subcommunity]/
        page.tsx       // Sub-community page (optional)
    page.tsx           // Religion landing page
```

### 27.3 URL Redirects & 404 Handling
- **Database Tables**: `url_redirects`, `not_found_logs` (see database schema)
- **Middleware**: Log 404s, check redirects, suggest similar communities
- **Implementation**: Next.js middleware with Supabase queries

### 27.4 SEO Implementation
- **Structured Data**: JSON-LD for community pages
- **Breadcrumbs**: Home > Religion > Community > Sub-community
- **Sitemap**: Dynamic generation for all active communities
- **OpenGraph**: Community-specific meta tags

> **Note**: Full implementation details, database schema, and middleware code will be added during Week 10 (SEO & Polish phase). Community pages are auto-generated from database - all communities use the same template. Community options are defined in [PROFILE-FIELDS.md](./PROFILE-FIELDS.md).