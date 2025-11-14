# CC Matrimony - Pre-Launch Checklist & Manual Operations Guide

> **Purpose**: Ensure all infrastructure, accounts, and processes are ready before Day 1 of development. This document also serves as a reference for manual operations during MVP (admin panel will be built in Weeks 7-9).

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
  - [ ] If fails or rate-limited: Configure SMTP (Postmark/SES) for auth emails, or use the Resend API workaround (see below)
  - [ ] Recommended: Configure Resend SMTP for Supabase auth emails (host `smtp.resend.com`, username `resend`, password = `RESEND_API_KEY`). Use Resend API for non-auth notifications. Reference: https://resend.com/supabase

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
 - **Supabase email issues**: Optionally switch to a Resend-driven auth email flow (see workaround below)
  - Set up a minimal Edge Function to send emails via Resend using your `RESEND_API_KEY`.
  - Keep templates consistent across password reset and verification flows.
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
- [ ] **⚠️ Email Rate Limit**: Supabase built-in mailer enforces a low, project-wide limit on auth emails (historically ~2–4 emails/hour across signup/verification/reset). If you expect bursts above this, consider a custom SMTP or the Resend workaround (see "Supabase Auth Email Configuration").

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
RESEND_GUIDE=https://resend.com/supabase  # Reference for integration with Supabase

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
  - [ ] Variables: `{{sender_name}}`, `{{profile_link}}`

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

**Critical**: Supabase built-in mailer enforces a low, project-wide limit on auth emails (historically ~2–4 emails/hour across signup/verification/reset).

**Decision Point**: For low auth volume you can trial the built-in mailer. For sustained or burst traffic, configure Resend SMTP in Supabase for auth emails (or any SMTP like Postmark/SES). Alternatively, use the API-based flow with `generateLink` + Resend.

**Week 1-2 Workaround** (if >4 signups/hour expected):

1. Option A (Recommended): Configure Resend SMTP in Supabase for auth emails.
   - SMTP host: `smtp.resend.com`
   - Ports: `465, 587, 2465, 2587`
   - Username: `resend`
   - Password: your `RESEND_API_KEY`
   - Rate limit: default ≈ 30 auth emails/hour (configurable via Supabase Management API)
2. Option B: Keep Supabase link generation and send via Resend API. Implement custom verification flow:

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

---


## 🧪 TESTING CHECKLIST

### **Week 1-2: Auth & Profile**

- [ ] Can register with email/password
- [ ] Email verification link works
- [ ] Can login with verified email
- [ ] Can create basic profile (name, age, gender, community)
- [ ] Profile completeness calculates correctly
- [ ] Cannot browse without profile

### **Week 3: Photos & Search**

- [ ] Can upload profile photo
- [ ] Photo uploads to Cloudinary (Cloudinary handles optimization automatically)
- [ ] Photo displays correctly
- [ ] Can edit profile
- [ ] Photo gallery works
- [ ] Can search by age, gender, community
- [ ] Search results paginate correctly
- [ ] Can view other profiles
- [ ] Profile view tracking works

### **Week 4: Interests & Payments**

- [ ] Can send interest
- [ ] Daily limit enforced (3 for free)
- [ ] Can receive interests
- [ ] Can accept/decline interest
- [ ] Mutual match alert works
- [ ] Interest counter resets daily
- [ ] Pricing page displays correctly
- [ ] Can initiate Razorpay checkout
- [ ] Payment webhook receives events
- [ ] Subscription activates after payment
- [ ] Premium badge displays
- [ ] Limits increase for premium users
- [ ] Test payment with Razorpay test card

### **Week 5: Photo Request Feature**

- [ ] Photo request API routes work
- [ ] "Request Photo" button shows for profiles without photos
- [ ] Can send photo request
- [ ] Duplicate requests prevented
- [ ] Notification sent to profile owner

### **Week 6: Notifications**

- [ ] Welcome email sends on registration
- [ ] Interest received email sends
- [ ] Interest accepted email sends
- [ ] Password reset email works
- [ ] In-app notifications display

### **Week 7-9: Admin Panel**

> **See [ADMIN-MANAGEMENT-SPEC.md](./ADMIN-MANAGEMENT-SPEC.md) section 4 for complete implementation checklist and section 2 for feature details.**

**Week 7: Admin Foundation**
- [ ] Admin login page works
- [ ] Admin authentication protects routes
- [ ] User management (list, search, filters, actions) works

**Week 8: Profile & Payments**
- [ ] Profile approval queue and workflow work
- [ ] Photo moderation queue and actions work
- [ ] Payment management and refund processing work

**Week 9: Analytics & Settings**
- [ ] Analytics dashboard displays all statistics
- [ ] System settings and feature toggles work
- [ ] Content management pages work

### **Week 10: Polish & Launch**

- [ ] RLS policies implemented and tested
- [ ] Mobile responsive on all pages
- [ ] PWA manifest works
- [ ] SEO meta tags added
- [ ] Sitemap generated
- [ ] robots.txt configured
- [ ] All tests pass
- [ ] Staging deployment successful
- [ ] Smoke tests pass

### **Pre-Launch Smoke Tests**

- [ ] Register new account → Create profile → Upload photo → Search → Send interest → Complete payment
- [ ] Test on mobile (iOS Safari, Android Chrome)
- [ ] Test on desktop (Chrome, Firefox, Safari)
- [ ] Test email delivery (all templates)
- [ ] Test payment flow (test mode)
- [ ] Test error scenarios (invalid email, failed payment, etc.)

---

## 🔧 Emergency Manual Operations (Fallback Only)

> **⚠️ Use Only When Admin Panel is Unavailable**  
> Admin panel is available from Week 9 onwards. For normal operations, use the admin panel (see [ADMIN-MANAGEMENT-SPEC.md](./ADMIN-MANAGEMENT-SPEC.md)).  
> These SQL queries are for **emergency situations only** when the admin panel is down or inaccessible.

### **Critical Emergency Operations**

#### **1. Suspend User (Urgent Safety Issue)**
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

#### **2. Hide Profile (Urgent Moderation)**
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

#### **3. Remove Suspicious Photo (Urgent)**
```sql
-- Delete photo from database
DELETE FROM photos WHERE id = 'photo-uuid-here';
-- Note: Also delete from Cloudinary manually via dashboard or Admin API
```

### **Legal Compliance (GDPR/DPDPA)**

#### **4. Export User Data (Legal Request)**
```sql
-- Export all user data as JSON (use admin panel when available)
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

> **Note**: All other operations (user search, profile management, payment processing, analytics) should be done via the admin panel. These SQL queries are emergency fallbacks only.

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

