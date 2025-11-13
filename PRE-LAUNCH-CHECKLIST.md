# CC Matrimony - Pre-Launch Checklist & Manual Operations Guide

> **Purpose**: Ensure all infrastructure, accounts, and processes are ready before Day 1 of development. This document also serves as a reference for manual operations during MVP (since admin panel is deferred to Phase 2).

---

## 🔥 Week 0.5: De-Risking (4 hours, Thursday before Week 1)

**Purpose**: Validate all external services before Day 1 to avoid Week 1 blockers.

### Checklist

- [ ] **Cloudinary Smoke Test** (1 hour): Upload test image from Node.js Vercel API route
  - [ ] Create test route: `app/api/test-cloudinary/route.ts`
  - [ ] Upload 100KB test image to Cloudinary
  - [ ] Verify image URL works and optimization is applied
  - [ ] If fails: Set `STORAGE_PROVIDER=supabase` in `.env.local`, defer Cloudinary to Week 4

- [ ] **Supabase Auth Email Test** (30 min): Send 5 verification emails, confirm delivery
  - [ ] Create 5 test accounts via Supabase dashboard
  - [ ] Verify emails arrive in inbox (check spam)
  - [ ] If fails or rate-limited: Use Resend for verification from Day 1 (see workaround below)

- [ ] **Razorpay Test Payment** (1 hour): Complete test payment, verify webhook
  - [ ] Create test payment via Razorpay test keys
  - [ ] Verify webhook arrives at `https://your-vercel-app.vercel.app/api/payments/webhook`
  - [ ] Check webhook signature validation works
  - [ ] If fails: Defer payments to Week 5, focus on profiles first

- [ ] **Community Data Seeding** (1 hour): Seed 20 communities from PROFILE-FIELDS.md
  - [ ] Export community arrays from `PROFILE-FIELDS.md`
  - [ ] Create SQL seed script or insert via Supabase dashboard
  - [ ] Verify all communities appear in dropdowns
  - [ ] If missing: Use hardcoded arrays in Week 1, seed Week 2

- [ ] **Create .env.local** (30 min): All keys documented in TECH-STACK.md
  - [ ] Copy template from this document
  - [ ] Fill in all Supabase keys (Dev project)
  - [ ] Add Cloudinary keys (if smoke test passed)
  - [ ] Add Razorpay test keys
  - [ ] Verify `.env.local` is in `.gitignore`

### If Any Test Fails

- **Cloudinary fails**: Set `STORAGE_PROVIDER=supabase`, defer Cloudinary to Week 4
- **Supabase email fails**: Use Resend for verification emails from Day 1 (see workaround below)
- **Razorpay fails**: Defer payments to Week 5, focus on profiles
- **Community data missing**: Use hardcoded arrays in Week 1, seed Week 2

---

## 📋 DAY 0: Infrastructure & Accounts Setup

### **Required Accounts (Create Before Day 1)**

- [ ] **Supabase**
  - [ ] Dev project created: `cc-matrimony-dev`
  - [ ] Prod project created: `cc-matrimony-prod`
  - [ ] Note down: `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` for both projects
  - [ ] Note down: `SUPABASE_SERVICE_ROLE_KEY` for both projects (keep secret!)
  - [ ] Enable Email Auth provider in both projects
  - [ ] Configure email templates (password reset, email verification)
  - [ ] **⚠️ Email Rate Limit**: Supabase Free tier limits auth emails to **4/hour**. If you expect >4 signups/hour in MVP, use Resend workaround (see "Supabase Auth Email Configuration" section below)

- [ ] **Vercel**
  - [ ] Account created and connected to GitHub
  - [ ] Project created: `cc-matrimony` (or your preferred name)
  - [ ] Connect GitHub repository
  - [ ] Note: Auto-deploys on push to main branch

- [ ] **Cloudinary**
  - [ ] Account created
  - [ ] Free tier enabled (25GB storage + 25GB bandwidth/month)
  - [ ] Note down: `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`
  - [ ] Configure upload presets (optional, for signed uploads)

- [ ] **Razorpay**
  - [ ] Merchant account created
  - [ ] Test keys obtained: `RAZORPAY_KEY_ID` (rzp_test_xxx), `RAZORPAY_KEY_SECRET`
  - [ ] Live keys obtained (for production): `RAZORPAY_KEY_ID` (rzp_live_xxx), `RAZORPAY_KEY_SECRET`
  - [ ] Webhook URL configured: `https://matri.naveevo.com/api/payments/webhook`
  - [ ] Test payment flow verified

- [ ] **Resend**
  - [ ] Account created
  - [ ] Domain added: `matri.naveevo.com`
  - [ ] DNS records configured:
    - [ ] SPF record: `v=spf1 include:_spf.resend.com ~all`
    - [ ] DKIM record (provided by Resend)
    - [ ] DMARC record: `v=DMARC1; p=none; rua=mailto:dmarc@matri.naveevo.com`
  - [ ] Domain verified (can take 24-48 hours)
  - [ ] API key obtained: `RESEND_API_KEY`
  - [ ] Test email sent successfully

- [ ] **GitHub**
  - [ ] Repository created: `cc-matrimony` (or your preferred name)
  - [ ] `.gitignore` configured (exclude `.env.local`, `node_modules`, etc.)
  - [ ] Initial commit structure ready

- [ ] **Sentry** (Optional but recommended)
  - [ ] Account created
  - [ ] Project created for Next.js
  - [ ] DSN obtained: `NEXT_PUBLIC_SENTRY_DSN`

- [ ] **PostHog** (Optional for analytics)
  - [ ] Account created
  - [ ] Project key obtained: `NEXT_PUBLIC_POSTHOG_KEY`

### **Domain & DNS Setup**

- [ ] **Primary Domain**: `matri.naveevo.com`
  - [ ] Domain registered/configured
  - [ ] DNS A record pointing to Vercel (or CNAME to Vercel domain)
  - [ ] SSL certificate (automatic via Vercel)
  - [ ] Domain verified in Vercel

- [ ] **Email Domain**: `matri.naveevo.com` (for Resend)
  - [ ] SPF record configured
  - [ ] DKIM record configured
  - [ ] DMARC record configured
  - [ ] Domain verified in Resend dashboard

### **Environment Variables Template**

Create `.env.local` with all keys (DO NOT commit this file):

```bash
# Supabase (Required Day 1)
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Cloudinary (Add Week 3)
CLOUDINARY_CLOUD_NAME=xxxxxxxxxxxxxxxxxxxx
CLOUDINARY_API_KEY=xxxxxxxxxxxxxxxxxxxx
CLOUDINARY_API_SECRET=xxxxxxxxxxxxxxxxxxxx  # Keep secret!
STORAGE_PROVIDER=cloudinary  # fallback to 'supabase' if Cloudinary smoke test fails

# Razorpay (Required Week 4)
RAZORPAY_KEY_ID=rzp_test_xxxxx  # Use rzp_live_xxx for production
RAZORPAY_KEY_SECRET=xxxxx

# Resend (Required Week 6)
RESEND_API_KEY=re_xxxxx
RESEND_FROM_EMAIL=noreply@matri.naveevo.com
RESEND_REPLY_TO=support@matri.naveevo.com

# Monitoring (Optional, add Week 7)
NEXT_PUBLIC_SENTRY_DSN=https://xxxxx@sentry.io/xxxxx
NEXT_PUBLIC_POSTHOG_KEY=phc_xxxxx
NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com

# App Config
NEXT_PUBLIC_APP_URL=https://matri.naveevo.com
```

---

## 📝 CONTENT & LEGAL PREPARATION

### **Legal Pages (Required Before Launch)**

- [ ] **Terms of Service**
  - [ ] Drafted and reviewed
  - [ ] Includes user responsibilities, platform rules, payment terms
  - [ ] Legal review completed (if applicable)

- [ ] **Privacy Policy**
  - [ ] Drafted and reviewed
  - [ ] DPDPA compliant (India)
  - [ ] GDPR compliant (if serving EU users)
  - [ ] Data collection, storage, sharing clearly explained

- [ ] **Refund Policy**
  - [ ] Drafted and reviewed
  - [ ] Clear refund conditions and process
  - [ ] Timeline for refunds specified

- [ ] **Safety Tips Page**
  - [ ] Educational content on safe practices
  - [ ] How to report abuse
  - [ ] Contact information

### **Email Templates (Required Week 6)**

- [ ] **Welcome Email**
  - [ ] Template designed
  - [ ] Content written
  - [ ] Variables defined: `{{name}}`, `{{profile_link}}`

- [ ] **Interest Received Email**
  - [ ] Template designed
  - [ ] Content written
  - [ ] Variables: `{{sender_name}}`, `{{profile_link}}`, `{{message}}`

- [ ] **Interest Accepted Email**
  - [ ] Template designed
  - [ ] Content written
  - [ ] Variables: `{{receiver_name}}`, `{{profile_link}}`

- [ ] **Password Reset Email**
  - [ ] Template designed (can use Resend default)
  - [ ] Reset link format verified

- [ ] **Email Verification Email**
  - [ ] Template designed (can use Resend default)
  - [ ] Verification link format verified

### **Seed Data Preparation**

- [ ] **Test Profiles** (20-30 profiles for launch)
  - [ ] Profile data prepared (names, ages, communities, etc.)
  - [ ] Test photos collected (appropriate, diverse)
  - [ ] SQL seeding script ready (or manual entry plan)

- [ ] **Master Data**
  - [ ] Religion list (Christian, Hindu, Muslim, Other)
  - [ ] Primary Community list per religion (use arrays from `PROFILE-FIELDS.md`)
    - [ ] Hindu: Bunt, Billava, Devadiga, Mogaveera, Iyengar, Nair, Vokkaliga, Brahmin, Other
    - [ ] Christian: Latin Catholic, Syrian Catholic, CSI, Marthoma, Pentecostal, Protestant, Orthodox, Other
    - [ ] Muslim: Sunni, Shia, Shafi, Hanafi, Other
  - [ ] Sub-community list per primary community (e.g., `Bunt → Shetty, Hegde, Poojary, Nayak, Rai, Other` as defined in `PROFILE-FIELDS.md`)
  - [ ] Education qualifications list
  - [ ] Occupation categories list
  - [ ] Countries/states/cities list

---

## 🗄️ DATABASE SETUP (Day 1-2)

### **Schema Creation**

- [ ] Run core schema SQL in Supabase SQL Editor:
  - [ ] `users` table (or use Supabase Auth)
  - [ ] `profiles` table
  - [ ] `photos` table
  - [ ] `interests` table
  - [ ] `subscriptions` table
  - [ ] `payment_events` table (for Week 6 webhook idempotency)
  - [ ] `verification_tokens` table (only if using Resend workaround for email)
  - [ ] `messages` table (for Phase 2 chat)
  - [ ] `communities` table
  - [ ] `sub_communities` table

- [ ] Generate TypeScript types:
  ```bash
  npx supabase gen types typescript --project-id YOUR_PROJECT_ID > types/database.ts
  ```

- [ ] Seed master data:
  - [ ] Communities
  - [ ] Sub-communities
  - [ ] Education/occupation lists

### **Supabase Auth Email Configuration**

**Critical**: Supabase Free tier limits auth emails to **4/hour**.

**Decision Point**: If you expect <4 signups/hour in MVP, use default Supabase email verification (simpler). If >4 signups/hour expected, implement Resend workaround below.

**Week 1-2 Workaround** (if >4 signups/hour expected):

1. Disable Supabase email verification in auth settings
2. Use Resend for all auth emails (no rate limit)
3. Implement custom verification flow:

```typescript
// app/api/auth/register/route.ts
import { createClient } from '@supabase/supabase-js';
import { Resend } from 'resend';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);
const resend = new Resend(process.env.RESEND_API_KEY!);

export async function POST(req: Request) {
  const { email, password } = await req.json();

  // 1. Create user with email_verified: false
  const { data: user, error } = await supabase.auth.admin.createUser({
    email,
    password,
    email_confirm: false, // Disable Supabase email
  });

  if (error) throw error;

  // 2. Generate custom verification token
  const token = crypto.randomUUID();
  await supabase
    .from('verification_tokens')
    .insert({ user_id: user.user.id, token, expires_at: new Date(Date.now() + 24 * 60 * 60 * 1000) });

  // 3. Send via Resend (no rate limit)
  await resend.emails.send({
    from: 'noreply@matri.naveevo.com',
    to: email,
    subject: 'Verify your email',
    html: `<a href="${process.env.NEXT_PUBLIC_APP_URL}/verify?token=${token}">Verify Email</a>`,
  });

  return Response.json({ success: true });
}
```

**Required table** (add to schema):
```sql
CREATE TABLE verification_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  token VARCHAR(255) UNIQUE NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### **RLS Policies (Week 7)**

- [ ] Profiles: Users can view approved profiles
- [ ] Profiles: Users can edit own profile
- [ ] Photos: Users can view photos of approved profiles
- [ ] Interests: Users can view own interests
- [ ] Messages: Users can view own messages (Phase 2)

---

## 🧰 Admin Operations (MVP Weeks 1-8)
- **Profile approvals**: Auto-approved (`is_approved = true`). If a profile must be hidden, run `UPDATE profiles SET is_approved = false WHERE id = 'uuid';` in Supabase SQL.
- **Photo moderation**: Photos auto-approved. Remove bad uploads with `DELETE FROM photos WHERE id = 'uuid';` and delete the image from Cloudinary using the Cloudinary dashboard or Admin API.
- **User suspensions**: Suspend via Supabase Auth (`UPDATE auth.users SET banned_until = NOW() + INTERVAL '7 days' WHERE id = 'uuid';`). Volume expectation: <5 suspensions/week @ 100 users.
- **Escalation rule**: If manual operations exceed 2 hours in any week, log the pain point and revisit the Phase 2 admin UI scope.

---

## 🧪 TESTING CHECKLIST

### **Week 1-2: Auth & Profile**

- [ ] Can register with email/password
- [ ] Email verification link works
- [ ] Can login with verified email
- [ ] Can create basic profile (name, age, gender, community)
- [ ] Profile completeness calculates correctly
- [ ] Cannot browse without profile

### **Week 3: Photos**

- [ ] Can upload profile photo
- [ ] Photo uploads to Cloudinary (Cloudinary handles optimization automatically)
- [ ] Photo displays correctly
- [ ] Can edit profile
- [ ] Photo gallery works

### **Week 4: Search**

- [ ] Can search by age, gender, community
- [ ] Search results paginate correctly
- [ ] Can view other profiles
- [ ] Profile view tracking works
- [ ] Cannot browse without photo (reciprocity)

### **Week 5: Interests**

- [ ] Can send interest
- [ ] Daily limit enforced (3 for free)
- [ ] Can receive interests
- [ ] Can accept/decline interest
- [ ] Mutual match alert works
- [ ] Interest counter resets daily

### **Week 6: Payments**

- [ ] Pricing page displays correctly
- [ ] Can initiate Razorpay checkout
- [ ] Payment webhook receives events
- [ ] Subscription activates after payment
- [ ] Premium badge displays
- [ ] Limits increase for premium users
- [ ] Test payment with Razorpay test card

### **Week 7: Notifications & Polish**

- [ ] Welcome email sends on registration
- [ ] Interest received email sends
- [ ] Interest accepted email sends
- [ ] Password reset email works
- [ ] In-app notifications display
- [ ] Mobile responsive on all pages
- [ ] PWA installable
- [ ] SEO meta tags present
- [ ] Sitemap generates

### **Pre-Launch Smoke Tests**

- [ ] Register new account → Create profile → Upload photo → Search → Send interest → Complete payment
- [ ] Test on mobile (iOS Safari, Android Chrome)
- [ ] Test on desktop (Chrome, Firefox, Safari)
- [ ] Test email delivery (all templates)
- [ ] Test payment flow (test mode)
- [ ] Test error scenarios (invalid email, failed payment, etc.)

---

## 🔧 MANUAL OPERATIONS (MVP - No Admin Panel)

Since admin panel is deferred to Phase 2, owner will use Supabase dashboard for manual operations.

### **Daily Operations**

#### **Morning Check (5 minutes)**
```sql
-- New users today
SELECT 
  p.profile_id,
  p.full_name,
  p.email,
  p.community,
  p.created_at
FROM profiles p
WHERE DATE(p.created_at) = CURRENT_DATE
ORDER BY p.created_at DESC;

-- New payments today
SELECT 
  p.full_name,
  s.plan_type,
  s.amount,
  s.status,
  s.created_at
FROM subscriptions s
JOIN profiles p ON s.user_id = p.user_id
WHERE DATE(s.created_at) = CURRENT_DATE
ORDER BY s.created_at DESC;
```

#### **Weekly Stats (Monday Morning)**
```sql
-- Week overview
SELECT 
  COUNT(*) FILTER (WHERE created_at >= date_trunc('week', CURRENT_DATE)) as new_users_this_week,
  COUNT(*) FILTER (WHERE last_active >= date_trunc('week', CURRENT_DATE)) as active_users_this_week,
  (SELECT COUNT(*) FROM subscriptions 
   WHERE created_at >= date_trunc('week', CURRENT_DATE)) as new_subscriptions_this_week,
  (SELECT SUM(amount) FROM subscriptions 
   WHERE created_at >= date_trunc('week', CURRENT_DATE) 
   AND status = 'active') as revenue_this_week
FROM profiles;
```

### **User Support Operations**

#### **Find User**
```sql
-- Search by email
SELECT * FROM profiles WHERE email = 'user@example.com';

-- Search by phone
SELECT * FROM profiles WHERE phone = '+91XXXXXXXXXX';

-- Search by profile ID
SELECT * FROM profiles WHERE profile_id = 'CCM001234';

-- Search by name
SELECT * FROM profiles WHERE full_name ILIKE '%John%';
```

#### **Suspend User**
```sql
-- Suspend user account (7 days)
UPDATE auth.users 
SET banned_until = NOW() + INTERVAL '7 days'
WHERE email = 'spam@example.com';

-- Permanently ban
UPDATE auth.users 
SET banned_until = '2099-12-31'
WHERE email = 'abusive@example.com';

-- Check suspension status
SELECT email, banned_until 
FROM auth.users 
WHERE email = 'user@example.com';
```

#### **Activate User**
```sql
-- Remove suspension
UPDATE auth.users 
SET banned_until = NULL
WHERE email = 'user@example.com';
```

#### **Manually Activate Premium**
```sql
-- Activate Silver plan manually
INSERT INTO subscriptions (
  user_id, 
  plan_type, 
  status, 
  amount, 
  start_date, 
  end_date,
  payment_id
)
VALUES (
  (SELECT user_id FROM profiles WHERE email = 'user@example.com'),
  'silver',
  'active',
  399,
  NOW(),
  NOW() + INTERVAL '1 month',
  'manual-activation-' || NOW()::text
);
```

#### **Refund Subscription**
```sql
-- Cancel and refund
UPDATE subscriptions 
SET 
  status = 'cancelled',
  end_date = NOW()
WHERE user_id = (SELECT user_id FROM profiles WHERE email = 'user@example.com')
AND status = 'active';

-- Note: Process actual refund via Razorpay dashboard
```

### **Content Moderation**

#### **Review Reported Profiles**
```sql
-- Find profiles with reports (if reports table exists)
SELECT 
  p.profile_id,
  p.full_name,
  p.email,
  COUNT(r.id) as report_count
FROM profiles p
LEFT JOIN reports r ON p.id = r.reported_profile_id
GROUP BY p.id, p.profile_id, p.full_name, p.email
HAVING COUNT(r.id) > 0
ORDER BY report_count DESC;
```

#### **Deactivate Profile**
```sql
-- Hide profile from search
UPDATE profiles 
SET is_approved = false
WHERE profile_id = 'CCM001234';

-- Reactivate
UPDATE profiles 
SET is_approved = true
WHERE profile_id = 'CCM001234';
```

#### **Delete Photo**
```sql
-- Find user's photos
SELECT * FROM photos 
WHERE profile_id = (SELECT id FROM profiles WHERE email = 'user@example.com');

-- Delete specific photo
DELETE FROM photos 
WHERE id = 'photo-uuid-here';
-- Note: Also delete from Cloudinary manually via dashboard or Admin API
```

### **Data Export (GDPR/DPDPA)**

#### **Export User Data**
```sql
-- Export all user data as JSON
SELECT 
  json_build_object(
    'profile', row_to_json(p.*),
    'photos', (SELECT json_agg(row_to_json(ph.*)) FROM photos ph WHERE ph.profile_id = p.id),
    'interests_sent', (SELECT json_agg(row_to_json(i.*)) FROM interests i WHERE i.sender_id = p.id),
    'interests_received', (SELECT json_agg(row_to_json(i.*)) FROM interests i WHERE i.receiver_id = p.id),
    'subscriptions', (SELECT json_agg(row_to_json(s.*)) FROM subscriptions s WHERE s.user_id = p.user_id)
  ) as user_data
FROM profiles p
WHERE p.email = 'user@example.com';
```

#### **Delete User Account (Soft Delete)**
```sql
-- Soft delete profile
UPDATE profiles 
SET 
  is_active = false,
  deleted_at = NOW()
WHERE email = 'user@example.com';

-- Hard delete (after 30-day grace period)
-- WARNING: This is permanent!
DELETE FROM profiles WHERE email = 'user@example.com' AND deleted_at < NOW() - INTERVAL '30 days';
```

---

## 🚀 DEPLOYMENT CHECKLIST

### **Pre-Deployment (Week 7)**

- [ ] All environment variables set in Vercel dashboard
- [ ] Supabase Prod project configured
- [ ] Cloudinary account created and configured
- [ ] Razorpay live keys configured (not test keys)
- [ ] Resend domain verified and sending emails
- [ ] All tests passing
- [ ] No console errors
- [ ] Mobile responsive verified
- [ ] RLS policies enabled
- [ ] Database seeded with 20-30 test profiles

### **Deployment Steps**

1. [ ] Push code to `main` branch
2. [ ] Vercel auto-deploys
3. [ ] Verify production site loads: `https://matri.naveevo.com`
4. [ ] Test critical path:
   - [ ] Register new account
   - [ ] Verify email
   - [ ] Create profile
   - [ ] Upload photo
   - [ ] Search profiles
   - [ ] Send interest
   - [ ] Complete test payment
   - [ ] Receive email notifications

### **Post-Deployment**

- [ ] Monitor Sentry for errors (first 24 hours)
- [ ] Check Vercel Analytics for traffic
- [ ] Verify email delivery (check Resend dashboard)
- [ ] Test payment with real card (small amount)
- [ ] Create 20-30 seed profiles manually
- [ ] Share with beta testers (if applicable)

---

## 📊 MONITORING SETUP

### **Week 7: Enable Monitoring**

- [ ] Sentry error tracking configured
- [ ] PostHog analytics configured (optional)
- [ ] Vercel Analytics enabled
- [ ] Health check endpoint working: `/api/health`
- [ ] Set up email alerts for:
  - [ ] Critical errors (Sentry)
  - [ ] Payment failures
  - [ ] Email delivery issues

### **Daily Monitoring (Owner)**

- [ ] Check Sentry dashboard for errors
- [ ] Check Vercel Analytics for traffic
- [ ] Check Resend dashboard for email delivery
- [ ] Check Razorpay dashboard for payments
- [ ] Review Supabase logs for unusual activity

---

## ⚠️ CRITICAL REMINDERS

1. **Never commit `.env.local`** - Use Vercel environment variables
2. **Use test keys in development** - Switch to live keys only in production
3. **Test payments in test mode first** - Verify webhook before going live
4. **Backup database regularly** - Supabase handles this, but verify
5. **Monitor storage usage** - Cloudinary (25GB storage + 25GB bandwidth/month free) and Supabase have limits
6. **Keep service role keys secret** - Never expose in client code
7. **Test email deliverability** - Verify SPF/DKIM/DMARC before launch

---

## 📞 SUPPORT CONTACTS

- **Supabase Support**: [support@supabase.com](mailto:support@supabase.com)
- **Vercel Support**: [support@vercel.com](mailto:support@vercel.com)
- **Cloudflare Support**: [Cloudflare Dashboard](https://dash.cloudflare.com)
- **Razorpay Support**: [Razorpay Dashboard](https://dashboard.razorpay.com)
- **Resend Support**: [Resend Dashboard](https://resend.com)

---

**Last Updated**: November 2025  
**Next Review**: Before Day 1 of development  
**Status**: Pre-development preparation

