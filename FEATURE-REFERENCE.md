# CC Matrimony Portal - Feature Reference

> **Quick Reference for Feature Verification**

## 📑 Table of Contents

- [How to Use This File](#-how-to-use-this-file)
- [Authentication & Security](#-authentication--security)
- [Profile Management](#-profile-management)
- [Search & Discovery](#-search--discovery)
- [Interest & Matching System](#-interest--matching-system)
- [Photo & Video Management](#-photo--video-management)
- [Privacy & Safety](#-privacy--safety)
- [Premium Features & Payments](#-premium-features--payments)
- [VIP & Matchmaking Services](#-vip--matchmaking-services)
- [Analytics & Insights](#-analytics--insights)
- [Notifications & Alerts](#-notifications--alerts)
- [Community & Portals](#-community--portals)
- [Mobile Features](#-mobile-features)
- [Horoscope & Compatibility](#-horoscope--compatibility)
- [Verification & Safety](#-verification--safety)
- [Admin & Management](#-admin--management)
- [Legal & Compliance](#-legal--compliance)
- [User Experience](#-user-experience)
- [Content & Stories](#-content--stories)
- [Infrastructure & Technical](#-infrastructure--technical)
- [Features We Don't Have](#-features-we-dont-have-intentionally-excluded)

## 📋 How to Use This File

When you see a feature on any matrimony portal, use **Ctrl+F** to search for it here and check where it sits on our roadmap:
- ✅ **MVP Scope (Planned)** – Committed for initial launch but not yet implemented
- 🛠️ **In Progress** – Currently being built/testing
- 🔄 **Future Phase** – Planned after MVP
- ❌ **Not Included** – Intentionally excluded unless priorities change

### **Priority Levels:**
- **P0** - Critical for MVP/launch (must-have)
- **P1** - Important for user adoption (should-have)
- **P2** - Nice-to-have for competitive advantage (could-have)

### **Implementation Timeline**
> **See [TECH-STACK.md](./TECH-STACK.md) section 17 for detailed day-by-day implementation roadmap (10 weeks).**

**High-Level Overview:**
- **Weeks 1-2**: Foundation (Auth, Basic Profile)
- **Week 3**: Photos + Search
- **Week 4**: Interests + Payments
- **Week 5**: Photo Request Feature
- **Week 6**: Notifications
- **Weeks 7-9**: Admin Panel
- **Week 10**: Polish + Launch

### 🚀 MVP Snapshot
- **Timeline:** 10 weeks (extended from 8 weeks to include admin panel)
- **Planned Categories:** 18 across auth, profiles, search, matching, privacy, payments, and admin.
- **Core References:** See [Photo Request Feature](#-photo--video-management) for requesting users to upload photos.
- **Deferred Items:** Organized in [🚫 Features We Don't Have](#-features-we-dont-have-intentionally-excluded).
- **Hosting Plan:** Runs entirely on Vercel Hobby + Supabase Free (limited storage, Vercel Cron ≤12 jobs).

### 🧱 Tech Stack
> **See [TECH-STACK.md](./TECH-STACK.md) for complete technical specifications, architecture, database schema, API routes, and implementation details.**

---

## 🔐 AUTHENTICATION & SECURITY

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Email Registration** | ✅ | P0 | Register with email address |
| **Phone Registration** | 🔄 | P2 (Phase 2) | Register with phone number |
| **Email Verification** | ✅ | P0 | Email verification link (24hr expiry) |
| **Phone OTP Verification** | 🔄 | P2 (Phase 2) | Deferred from MVP |
| **Social Login - Google** | 🔄 | P2 (Phase 2) | Deferred from MVP |
| **Social Login - Facebook** | 🔄 | P2 (Phase 2) | Deferred from MVP |
| **Password Reset** | ✅ | P0 | Email reset link flow |
| **Password Strength Meter** | ✅ | P1 | Real-time validation with requirements display |
| **Two-Factor Authentication** | ❌ | P2 | Not included (OTP is sufficient) |
| **Account Lockout** | ✅ | P1 | After 5 failed login attempts (15min lockout) |
| **Session Management** | ✅ | P0 | JWT authentication with 30-day expiry |
| **Device Fingerprinting** | ✅ | P1 | Limit 3 accounts per device (IP + browser fingerprint) |
| **Profile Uniqueness Check** | ✅ | P0 | Prevent duplicate accounts (phone/email validation) |
| **Login Activity Logging** | ✅ | P1 | Track login attempts, IP addresses, device info |
| **Account Recovery Flow** | ✅ | P0 | Multi-step recovery via email/phone with security questions |
| **Guest Browsing** | ❌ | P2 | Not included (registration required for all features) |

---

## 👤 PROFILE MANAGEMENT
| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Basic Profile Fields** | ✅ | P0 | Name, age, gender, location, education, occupation |
| **Photo Upload** | ✅ | P0 | Free: 1 profile + 1 album; Paid: 1 profile + 9 album + 3 family/group (Max 15MB per photo) |
| **Photo Formats & Size** | ✅ | P0 | PNG/GIF/JPG/JPEG/WebP up to 15 MB per photo (client-side compression to <500KB before upload) |
| **Photo Watermarking** | ✅ | P1 | Small watermark on all photos (visible in-app and downloads) for branding/security |
| **Profile Completeness** | ✅ | P1 | Real-time score (0-100%) with progress bar; excludes intentionally skipped fields |
| **Profile ID Generation** | ✅ | P0 | Auto-generated CCM001234 format |
| **Profile Approval Workflow** | ✅ | P0 | Admin approval queue for all new profiles |
| **Profile Preview Mode** | ✅ | P1 | View as other users see it |
| **Profile Deactivation** | ✅ | P1 | Temporarily hide (data retained) |
| **Profile Export PDF** | ❌ | P2 | Removed from MVP - complex implementation, users can screenshot |
| **Profile Export JPG** | ❌ | P2 | Removed from MVP - complex implementation, users can screenshot |
| **Profile Strength Analyzer** | ✅ | P1 | Advanced scoring with improvement tips |
| **Profile Last Updated** | ✅ | P0 | Timestamp of last edit |
| **Multi-step Onboarding** | ✅ | P0 | Flexible: Quick setup (2-min minimal) + Gradual completion prompts |
| **Profile Created By** | ✅ | P1 | Self, Parent, Sibling, Relative, Friend |
| **Age Validation** | ✅ | P0 | Hard block: Male ≥21, Female ≥18 |
| **Profile Uniqueness Check** | ✅ | P0 | Prevent duplicate accounts |
| **Profile Completion Prompts** | ✅ | P1 | Progressive nudges with contextual copy (dashboard + email) when key sections are missing |
| **Profile Verification Status** | ✅ | P0 | Pending/Approved/Rejected with admin notes |
| **Profile Edit History** | ✅ | P1 | Track all profile changes with timestamps |
| **Profile Backup/Restore** | ❌ | P2 | Deferred: Complex implementation not needed for MVP |

Profile fields specification
- For the authoritative list of fields, validation, and phasing, see PROFILE-FIELDS.md.
- Summary:
  - Week 1-2 (MVP core): minimal required set (name, gender, DOB→age, phone OTP, email, password, marital_status, created_by).
  - Week 3+: education/occupation/income/family/photos wired; Cloudinary for media (15MB, signed URLs).
  - Address-heavy PII deferred to later phase; preferences optional at creation.

Note: Detailed field lists, community-specific variants, and phased P0/P1/P2 breakdowns have been consolidated into PROFILE-FIELDS.md.

---

## 🔍 SEARCH & DISCOVERY

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Basic Search** | ✅ | P0 | Search by age, religion, community, location |
| **Advanced Search** | ✅ | P1 | 15+ filters (height, education, income, etc.) |
| **Gender Filter** | ✅ | P0 | Looking for: Male/Female |
| **Age Range Filter** | ✅ | P0 | Min-max age slider |
| **Religion Filter** | ✅ | P0 | Hindu, Christian, Muslim |
| **Community Filter** | ✅ | P0 | Based on selected religion |
| **Location Filter** | ✅ | P0 | City, State, Country |
| **Marital Status Filter** | ✅ | P0 | Never married, Divorced, Widowed |
| **Height Range Filter** | ✅ | P1 | Min-max height |
| **Education Level Filter** | ✅ | P1 | By education level (High School, Diploma, Bachelor's, Master's, PhD) |
| **Education Field Filter** | ✅ | P1 | By education field (Engineering, Business, Medicine, etc.) |
| **Occupation Sector Filter** | ✅ | P1 | By occupation sector (IT, Healthcare, Education, etc.) |
| **Job Title Filter** | ✅ | P2 | By specific job title/role |
| **Income Range Filter** | ✅ | P1 | By annual income bracket |
| **Children Status Filter** | ✅ | P1 | With/without children |
| **Employed In Filter** | ✅ | P1 | Private/Government/Business/etc. |
| **Recently Active Filter** | ✅ | P1 | Last 24h/7d/30d |
| **Has Photo Filter** | ✅ | P0 | Only profiles with photos |
| **Verified Only Filter** | ✅ | P1 | Only verified profiles |
| **Paid Member Filter** | ✅ | P1 | Only premium/paid members |
| **Keyword Search** | ✅ | P0 | Search by name, city, occupation, bio |
| **Search Suggestions** | ✅ | P1 | Auto-suggest locations, occupations |
| **Search by Profile ID** | ✅ | P0 | Direct lookup (CCM001234) |
| **Search History** | ❌ | P2 | Removed from MVP - not essential, adds complexity |
| **Shareable Search URLs** | 🔄 | P1 | Expose query params for bookmarking/sharing search results |
| **Search Alerts** | ❌ | P2 | Planned post-MVP |
| **Saved Searches** | ❌ | P2 | Removed from MVP - defer to Phase 2 |
| **Quick Match Preview** | ❌ | P1 | Planned post-MVP |
| **Search Result Sorting** | ✅ | P1 | Sort by: Recently Active, Profile Completeness, Mutual Interests, Distance |
| **Search Result Pagination** | ✅ | P0 | 20 results per page with infinite scroll option |
| **Search Filters Reset** | ✅ | P1 | Clear all filters button |
| **Search Result Export** | ❌ | P2 | Deferred: Not essential for core functionality |

### **Tiered Search Strategy**
| Search Type | User Access | Features | Response Time |
|-------------|-------------|----------|---------------|
| **Basic Search** | Free Users | Age, Religion, Community, Location, Marital Status | <200ms |
| **Advanced Search** | Premium Users | All 15+ filters available | <500ms |

---

## 💌 INTEREST & MATCHING SYSTEM

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Send Interest** | ✅ | P0 | Express interest with optional message |
| **Receive Interest** | ✅ | P0 | View incoming interests with sender's profile |
| **Accept/Decline Interest** | ✅ | P0 | Respond with optional message |
| **Interest Send Limits** | ✅ | P0 | Free: 3/day, Premium: 10-50/day |
| **Mutual Match Alert** | ✅ | P0 | "It's a Match!" notification |
| **Interest Counter Dashboard** | ✅ | P1 | Shows remaining interests for the day |
| **Interest Withdraw** | ✅ | P1 | Withdraw before acceptance |
| **Interest with Message** | ✅ | P1 | Personalized 200-char message |
| **Interest Expiry** | ✅ | P1 | Auto-archive after 30 days |
| **Mutual Matches Filter** | ✅ | P1 | Show 2-way preference matches |
| **Recommended Matches** | ✅ | P0 | Rule-based compatibility (20+ parameters) |
| **Compatibility Score** | ✅ | P0 | Percentage match based on preferences |
| **Smart Suggestions** | ✅ | P1 | "You may also like" section |
| **Auto-Match Notifications** | ✅ | P1 | Daily email: 5 new matches |
| **Interest Limit Reset** | ✅ | P1 | Daily reset at midnight local time with countdown indicator |
| **Interest Analytics** | ❌ | P2 | Removed from MVP - defer to Phase 2 |
| **Interest Reminder System** | ❌ | P2 | Deferred: Email notifications sufficient |
| **Interest Bulk Actions** | ❌ | P2 | Deferred: Not essential for core functionality |
| **Interest Priority System** | ❌ | P2 | Deferred: Adds complexity without clear value |

---


## 📸 PHOTO & VIDEO MANAGEMENT

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Photo Upload** | ✅ | P0 | Drag/drop uploader with crop preview and reorder |
| **Photo Slots by Plan** | ✅ | P0 | Free: 1 profile + 1 album; Paid: 1 profile + 9 album + 3 family/group |
| **Family / Group Album** | ✅ | P0 | Dedicated slot bucket (3 photos) for group shots |
| **Photo Upload Requirement** | ✅ | P0 | Cannot browse without ≥1 photo |
| **Photo Auto-Approval** | ✅ | P0 | Photos publish instantly; admins post-moderate flagged items |
| **Flag Review Queue** | ✅ | P1 | Queue of flagged/reported photos for manual review |
| **Rejection Reasons** | ✅ | P1 | Blurry, Inappropriate, No Face, Fake |
| **User Notification on Rejection** | ✅ | P1 | Email user with reason, allow re-upload |
| **Profile Visibility Lock** | ✅ | P0 | Profile not visible until ≥1 photo uploaded |
| **Photo Privacy Settings** | ✅ | P1 | Everyone/Premium/Sent/Accepted/Request |
| **Per-Photo Privacy** | ✅ | P1 | Different levels per photo |
| **Private Photos Album** | ✅ | P1 | Password/access controlled |
| **Request Photo Feature** | ✅ | P0 | Show "Request Photo" button when profile has no photos |
| **Photo Request Notifications** | ✅ | P0 | Notify users when someone requests them to upload photos |
| **Photo Compression** | ✅ | P1 | Automatic compression to optimize storage and loading |
| **Photo Metadata Stripping** | ✅ | P1 | Remove EXIF data for privacy protection |
| **Photo Backup System** | ❌ | P2 | Deferred: Not needed with Supabase reliability |
| **Photo Analytics** | ❌ | P2 | Deferred: Analytics not critical for MVP |

---

## 🔐 PRIVACY & SAFETY

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Contact Privacy Settings** | ✅ | P0 | All premium/Accepted/Mutual/Hidden |
| **Photo Privacy Settings** | ✅ | P0 | Everyone/Premium/Sent/Accepted/Request |
| **Privacy Settings Dashboard** | ✅ | P1 | Central hub to manage all privacy preferences |
| **3-Tier Field Privacy** | ✅ | P1 | Per field: Public/Premium Only/Hidden |
| **Profile Visibility Settings** | ✅ | P1 | Hide from: Search/Recently Active/New Matches |
| **Search Appearance Control** | ✅ | P2 | Don't show to: Viewed profiles/Declined profiles |
| **Privacy Request System** | ✅ | P1 | Request access to hidden fields |
| **Access Management Dashboard** | ✅ | P1 | View requests, approve/deny, revoke |
| **Request Limits** | ✅ | P1 | Free: 5/day, Premium: unlimited |
| **Unified Privacy Dashboard** | ✅ | P1 | Central hub for all privacy settings |
| **Block Users** | ✅ | P0 | Block unwanted users |
| **Report Abuse** | ✅ | P0 | Report with category (fake, harassment) |
| **Basic Profanity Filter** | ✅ | P1 | Auto-filter bad words in bio |
| **Verification Badges System** | ✅ | P0 | Email verified ✓, Phone verified ✓ |
| **Income Verification Badge** | ✅ | P1 | Salary slip/ITR review with badge display |
| **Education Verification Badge** | ✅ | P1 | Degree certificate review with badge display |
| **Profile View Limits** | ✅ | P1 | Free users: 50 views/day |
| **Device Fingerprinting** | ✅ | P1 | Limit 3 accounts per device |
| **Religion-Based Visibility** | ✅ | P1 | Default: same religion, opt-in: other religions (user-controlled) |
| **Caste-Based Visibility** | ✅ | P1 | Default: same caste, option: other castes (user-controlled) |
| **Community-Specific Search** | ✅ | P1 | Search results filtered by religion and caste |
| **Privacy Audit Trail** | ✅ | P1 | Track all privacy setting changes with timestamps |
| **Privacy Compliance Dashboard** | ✅ | P1 | Admin view of privacy settings across all users |


---

## 💎 PREMIUM FEATURES & PAYMENTS

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **FREE Plan** | ✅ | P0 | Manual admin approval, 3 interests/day, 1 month free trial |
| **Silver Plan** | ✅ | P0 | ₹399/month: 20 contacts, 5 interests/day |
| **Silver Plus Plan** | ✅ | P1 | ₹799/3M: 50 contacts, 10 interests/day |
| **Gold Plan** | ✅ | P1 | ₹1,499/6M: 150 contacts, 30 interests/day |
| **Platinum Plan** | ✅ | P1 | ₹2,499/12M: 500 contacts, 50 interests/day |
| **VIP Assisted** | ✅ | P1 | ₹24,999/6M: Dedicated matchmaker |
| **Premium Tier Badges** | ✅ | P1 | Visual badges on profiles |
| **Contact Viewing System** | ✅ | P0 | One-time permanent unlock per profile |
| **Contact Limit Tracking** | ✅ | P1 | Dashboard showing total + daily usage |
| **Premium Expiry Alerts** | ✅ | P1 | 7d, 3d, 1d reminders |


### **Payment Gateways**
| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Razorpay Integration** | ✅ | P0 | Primary payment gateway |
| **PhonePe Integration** | ✅ | P1 | UPI payment option (primary backup) |
| **Paytm Integration** | ✅ | P2 | UPI payment option (secondary backup) |
| **Cashfree Integration** | ✅ | P2 | UPI payment option (tertiary backup) |
| **Payment Gateway Management** | ✅ | P0 | Admin enable/disable gateways |
| **Gateway Priority Settings** | ✅ | P0 | Set primary and fallback order |
| **Payment Verification Queue** | ✅ | P1 | Approve/reject offline payments |
| **Offline Payment System** | ✅ | P1 | Bank transfer/cash → manual activation |
| **Payment Webhooks** | ✅ | P0 | Handle success/failure/refund callbacks |
| **Manual Refund Button** | ✅ | P1 | Process refunds with reason dropdown |
| **Payment History View** | ✅ | P1 | Table with filters (user, date, status) |
| **Payment Analytics Dashboard** | ❌ | P2 | Removed from MVP - basic payment table sufficient |
| **Payment Failure Recovery** | ❌ | P2 | Deferred: Manual handling sufficient for MVP |
| **Payment Refund Automation** | ❌ | P2 | Deferred: Manual refund processing for MVP |

### **Coupon System**
| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Coupon Management** | ✅ | P1 | Create, edit, and track promotional codes (percentage or fixed amount) |

---

## 🎯 VIP & MATCHMAKING SERVICES

> Deferred to post-MVP roadmap. Refer to [🚫 Features We Don't Have](#-features-we-dont-have-intentionally-excluded) for coverage.

---

## 📊 ANALYTICS & INSIGHTS

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Who Viewed My Profile** | ✅ | P1 | Dashboard module listing viewers with timestamps; daily email digest (batch once per day) |
| **Recently Viewed Me** | ✅ | P1 | Dashboard card summarizing recent visitors (no external alerts) |
| **Profile Visit Alerts** | ✅ | P2 | Highlighted banner in dashboard when priority profiles revisit |
| **New View Indicator** | ✅ | P1 | Bell icon badge increments when fresh views arrive (Supabase Realtime) |
| **Churn Prediction Signals** | ✅ | P2 | Flag users with 14+ days inactivity or stalled onboarding; feed telecaller outreach list |

---

## 🔔 NOTIFICATIONS & ALERTS

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Email Notifications** | ✅ | P0 | Interests, acceptances, premium expiry, and profile reminders |
| **Email Template System** | ✅ | P1 | Reusable templates with variables |
| **Notification Preferences** | ✅ | P1 | User controls for instant, daily, or weekly cadence |
| **In-app Notifications** | ✅ | P0 | Bell icon with dropdown and read/unread states |
| **SMS Alerts** | ✅ | P0 | OTP and critical alerts with provider fallback |

---

## 🌐 COMMUNITY & PORTALS

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Main Platform** | ✅ | P0 | ccmatrimony.com with all communities |
| **Religion Pages** | ✅ | P0 | /hindu, /christian, /muslim with scoped browsing |
| **Community Pages** | ✅ | P0 | Auto-generated from database (/religion/community), all communities use same template |
| **Auto-Subfolder Filtering** | ✅ | P0 | Detect subfolder, auto-filter profiles by religion |
| **Cross-listing Logic** | ✅ | P0 | Display profiles on all relevant pages |
| **Religion Visibility Rules** | ✅ | P0 | Default: same religion, optional opt-in to others (user-controlled) |
| **Community Visibility Rules** | ✅ | P1 | Default: same community, option: other communities (user-controlled) |
| **Advanced SEO** | ✅ | P1 | Schema.org, social previews, dynamic meta tags per community |
| **Local Language Support** | ❌ | P2 | UI remains English-only; mother-tongue field powers matching |
| **Community Success Stories** | ✅ | P1 | Local success stories and testimonials |

---

## 📱 MOBILE FEATURES

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Progressive Web App (PWA)** | ✅ | P0 | Home screen installation, push notifications, offline caching, app-like experience |
| **Mobile-First Responsive UI** | ✅ | P0 | Touch-friendly, mobile-optimized design |
| **Profile QR Code Sharing** | ✅ | P1 | Generate QR code for easy profile sharing |
| **Image Lazy Loading** | ✅ | P1 | Load on scroll for faster mobile |
| **Responsive Tables** | ✅ | P1 | Mobile-friendly profile details |
| **React Native App (Android)** | ❌ | P2 | Deferred to Phase 3 - PWA provides app-like experience for MVP |
| **React Native App (iOS)** | ❌ | P2 | Deferred to Phase 3 - focus on Android after PWA validation |

---

## 🔬 HOROSCOPE & COMPATIBILITY

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Horoscope Upload** | ✅ | P1 | Upload PDF/Image (optional) |
| **Rashi/Nakshatra Fields** | ✅ | P1 | Birth star, moon sign, gothra (Hindu) |

---

## 🛡️ VERIFICATION & SAFETY

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Email Verification** | ✅ | P0 | Email verification link (24hr expiry) |
| **Phone OTP Verification** | 🔄 | P2 (Phase 2) | Deferred from MVP - email/password only for MVP |
| **Verified Badges** | ✅ | P0 | Email verified ✓, Phone verified ✓ |
| **Manual Profile Verification** | ✅ | P1 | Review and approve verification badge |
| **Document Verification Badges** | ❌ | P2 | Deferred: storage footprint too large for Supabase Free |

---

## ⚙️ ADMIN & MANAGEMENT

> **Complete admin panel specifications, MVP features (Weeks 7-9), and future phases (Phase 2+) are documented in [ADMIN-MANAGEMENT-SPEC.md](./ADMIN-MANAGEMENT-SPEC.md).**

**Quick Summary:**
- **MVP/Phase 1 (Weeks 7-9)**: Admin Authentication, User Management, Profile Approval, Photo Moderation, Payment Management, Analytics Dashboard, System Settings, Content Management
- **Phase 2**: Advanced Roles, Enhanced Bulk Operations, Enhanced Analytics, Email/SMS Broadcasting, Enhanced Export Capabilities, Enhanced Audit Trail, Coupon Code Management
- **Phase 3**: Red Flag Dashboard, Churn Signals, Advanced Reporting
- **Phase 4**: Automated Workflows, Advanced Permissions
- **Phase 5**: Multi-tenant Support, Advanced Integration
- **Phase 6+**: Partner & Agency Features (deferred)

---

## 📋 LEGAL & COMPLIANCE

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Download My Data - JSON** | ✅ | P0 | Export all user data as JSON (DPDPA requirement) |
| **Delete My Account** | ✅ | P0 | Soft delete with 30-day grace period |
| **Cookie Consent Banner** | ✅ | P0 | First-visit popup with accept/decline |
| **Privacy Policy Acceptance** | ✅ | P0 | Mandatory checkbox during signup |
| **Legal Pages** | ✅ | P0 | Terms of Service, Privacy Policy, Refund Policy |
| **Safety Tips Page** | ✅ | P1 | Educational content on safe practices |
| **DPDPA Compliance** | ✅ | P0 | India's Data Protection Act compliance |
| **GDPR Compliance** | ✅ | P0 | EU data protection compliance |

---

## 🎨 USER EXPERIENCE

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Shortlist/Favorites** | ✅ | P0 | Save profiles with private notes (unlimited) |
| **Recently Viewed** | ✅ | P0 | History of last 50 profiles viewed |
| **Not Interested/Hide** | ✅ | P0 | Mark profiles as not interested |
| **Accessibility Features** | ✅ | P0 | Screen reader, keyboard nav, ARIA labels |
| **Relationship Status Update** | ✅ | P1 | Users can mark engaged/married to pause visibility |
| **Match Closure Flow** | ✅ | P1 | "Found a match" wizard: choose status (Dating/Engaged/Married), auto-hide profile, capture testimonial |
| **User Onboarding Flow** | ✅ | P0 | Guided tour for new users with feature highlights |
| **Help & Support Center** | ✅ | P1 | FAQ, video tutorials, and contact support |
| **WhatsApp Support Chat** | ❌ | P2 | Removed from MVP - email + phone support sufficient |
| **User Feedback System** | ❌ | P2 | Deferred: Contact form sufficient for MVP |
| **Accessibility Compliance** | ✅ | P0 | WCAG 2.1 AA compliance for screen readers and keyboard navigation |

---

## 📊 CONTENT & STORIES

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **Marketing Content CMS** | ✅ | P1 | Manage success stories, testimonials, FAQ, and blog content |
| **Contact Form** | ✅ | P0 | Static contact page (sends to admin email) |
| **WhatsApp Integration** | ✅ | P1 | Floating support button + profile sharing |
| **WhatsApp Share Tracking** | ✅ | P1 | Share links include campaign tracking + short URLs for analytics |

---

## 🔧 INFRASTRUCTURE & TECHNICAL

| Feature | Status | Priority | Description |
|---------|--------|----------|-------------|
| **API Rate Limiting** | ✅ | P0 | 100 req/min per user, 1000/min per IP |
| **Error Logging** | ✅ | P0 | Sentry integration for error tracking |
| **Health Check Endpoint** | ✅ | P0 | /api/health for monitoring |
| **Database Seeding** | ✅ | P1 | Sample data for testing (50 fake profiles) |
| **Sitemap Generation** | ✅ | P1 | Auto-generate XML sitemap (SEO) |
| **Robots.txt Management** | ✅ | P1 | Configure crawler access |
| **Graceful Shutdown** | ✅ | P0 | Handle ongoing requests during deployment |
| **Operational Configuration Controls** | ✅ | P1 | Admin tools for SMS providers, plan limits, and religion/community master data |
| **Scheduled Jobs (Vercel Cron)** | ✅ | P1 | Consolidated ≤8 job schedule with admin-visible `cron_runs` log (includes weekly incomplete-profile reminder emails)
| **Database Optimization** | ✅ | P1 | Index optimization, query performance monitoring |
| **Chat Idle Timeout** | ✅ | P1 | Auto-disconnect chat sessions after 10 minutes idle; connection pooling to avoid exceeding Supabase limits |
| **CDN Integration** | ❌ | P2 | Deferred: Supabase built-in CDN sufficient for MVP |
| **Backup & Recovery** | ❌ | P2 | Deferred: Supabase handles basic backups |
| **Performance Monitoring** | ❌ | P2 | Deferred: Basic monitoring sufficient for MVP |
| **Security Headers** | ✅ | P1 | Implement security headers (CSP, HSTS, etc.) |

---


## 🚫 FEATURES WE DON'T HAVE (Intentionally Excluded)

### Communications & Engagement
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **Chat System** | ❌ | Removed - users connect directly via "View Contact" option |
| **Daily Match Digest** | ❌ | Scheduled marketing email deferred |

### UX Experiments & Gamification
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **Profile Badge System** | ❌ | Dating app feature, not suitable for matrimony |
| **Completeness Leaderboard** | ❌ | Gamification not suitable for matrimony context |
| **Daily Login Streak** | ❌ | Creates FOMO, not appropriate for matrimony |
| **Profile Completeness Incentive** | ❌ | Gamification deferred |
| **Common Background Highlighter** | ❌ | Nice-to-have UX highlight deferred |
| **Onboarding Tutorial** | ❌ | Guided tour deferred |
| **Dark Mode Toggle** | ❌ | Visual polish deferred |
| **Custom Theme Colors** | ❌ | Admin theming deferred |

### Discovery & Search Extensions
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **Profile Highlight Package** | ❌ | Too many boost options create confusion |
| **Per-Profile Filter Bypass** | ❌ | Edge case, not needed for core functionality |
| **Proximity Search** | ❌ | Privacy concerns, not relevant for matrimony |
| **Swipe Interface** | ❌ | Dating app UX, not matrimony-appropriate |
| **Smart Defaults (IP/DOB)** | ❌ | Privacy concerns, accuracy issues |
| **Comparison Tool** | ❌ | Creates shopping mentality, not respectful |
| **Body Type Filter** | ❌ | Lifestyle filter deferred to keep search focused |
| **Boolean Search** | ❌ | Complex query syntax deferred for simplicity |
| **Diet Filter** | ❌ | Lifestyle filter deferred to keep search focused |
| **Drinking Filter** | ❌ | Lifestyle filter deferred to keep search focused |
| **Manglik Filter** | ❌ | Religious compatibility filter deferred for MVP |
| **Gulf Experience** | ❌ | Not relevant for general matrimony, specific to certain communities only |
| **Search Result Export** | ❌ | Not essential for core functionality |

### UX & Profile Features (Removed from MVP)
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **Profile Export PDF** | ❌ | Complex implementation for MVP, users can screenshot profiles |
| **Profile Export JPG** | ❌ | Complex implementation for MVP, users can screenshot profiles |
| **Profile Backup/Restore** | ❌ | Database backups handle this, unnecessary complexity |
| **Search History** | ❌ | Not essential for core experience, adds complexity |
| **Saved Searches** | ❌ | Advanced feature deferred to Phase 2 |
| **Match Closure Testimonial Wizard** | ❌ | Simplified to status update only, manual testimonial collection via email |

### Media & Verification
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **Video Testimonials** | ❌ | Resource-intensive, bandwidth concerns |
| **Automated Photo Moderation (AI)** | ❌ | Requires third-party vision AI (e.g., Rekognition) beyond Vercel + Supabase |
| **Video Profile Upload** | ❌ | Rich media backlog item; requires dedicated moderation |
| **Video Moderation Queue** | ❌ | Not needed without video uploads |
| **Video Storage** | ❌ | Not needed without video uploads |
| **Video Player** | ❌ | Not needed without video uploads |
| **Video Profile Mobile** | ❌ | Rich media backlog item |
| **In-app Camera** | ❌ | Adds mobile complexity beyond MVP |
| **Government ID Verification** | ❌ | Bundled under document badge workflow |
| **Income Verification** | ❌ | Bundled under document badge workflow |
| **Education Verification** | ❌ | Bundled under document badge workflow |
| **Document Verification Badges** | ❌ | Supabase Free storage limits |
| **Photo Backup System** | ❌ | Storage constraints on Supabase Free |
| **Photo Analytics** | ❌ | Analytics not critical for MVP |
| **Profile Backup/Restore** | ❌ | Complex implementation not needed for MVP |

### Premium & Pricing Experiments
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **Profile Boost Add-on** | ❌ | Premium upsell deferred to simplify offerings |
| **Featured Listing** | ❌ | Premium upsell deferred to simplify offerings |
| **Boost Renewal Reminder** | ❌ | Only required with boost feature |
| **App-Only Pricing** | ❌ | Pricing experiments deferred |
| **Auto-Apply Launch Discount** | ❌ | Promotional automation deferred |

### Data & Analytics
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **User Impersonation** | ❌ | Debug tool deferred |
| **Interest Analytics** | ❌ | Complex analytics not needed for MVP |
| **Interest Reminder System** | ❌ | Email notifications sufficient |
| **Interest Bulk Actions** | ❌ | Not essential for core functionality |
| **Interest Priority System** | ❌ | Adds complexity without clear value |
| **Preference Learning** | ❌ | Machine-learning driven personalization deferred |
| **Bulk Operations** | ❌ | High-volume actions exceed Supabase Free limits |
| **Payment Analytics Dashboard** | ❌ | Manual tracking sufficient for MVP |
| **Payment Failure Recovery** | ❌ | Manual handling sufficient for MVP |
| **Payment Refund Automation** | ❌ | Manual refund processing for MVP |
| **Admin Dashboard Analytics** | ❌ | Basic metrics sufficient for MVP |
| **Admin Notification System** | ❌ | Manual monitoring sufficient for MVP |
| **User Feedback System** | ❌ | Contact form sufficient for MVP |

### Partner Programs
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **Partner & Agency Management (Basic)** | 🔄 Phase 2 | Deferred from MVP - not required for initial launch |
| **Partner Portal** | 🔄 Phase 2 | Separate partner login and dashboard - requires additional infrastructure |
| **Partner Commission System** | 🔄 Phase 2 | Commission tracking and payment system |
| **Partner Marketplace** | 🔄 Phase 3 | Service listings and booking system |
| **Partner Network** | 🔄 Phase 5 | Full partner ecosystem with directory and collaboration |

### Mobile & Platform Extensions
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **React Native App (Android)** | ❌ | PWA provides sufficient app-like experience for MVP, deferred to Phase 3 |
| **React Native App (iOS)** | ❌ | Deferred to Phase 3, focus on web + PWA first |
| **Home Screen Widget** | ❌ | Requires deeper mobile platform integration |
| **Simultaneous Mobile Launch** | ❌ | iOS launch deferred |

### Legal & Data Ops
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **Background Check** | ❌ | Expensive, privacy concerns, legal liability |
| **Auto-fill from LinkedIn** | ❌ | Integration complexity, privacy concerns |
| **Download My Data - PDF** | ❌ | Optional export deferred |
| **Download My Data - CSV** | ❌ | Optional export deferred |
| **Bulk Profile Import** | ❌ | Migration tooling deferred |
| **Data Retention Policy Automation** | ❌ | Requires legal review and automation |
| **Marketing Blog/Articles Section** | ❌ | Content marketing backlog |

### Infrastructure & Technical (Deferred)
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **CDN Integration** | ❌ | Supabase built-in CDN sufficient for MVP |
| **Backup & Recovery** | ❌ | Supabase handles basic backups |
| **Performance Monitoring** | ❌ | Basic monitoring sufficient for MVP |

### Horoscope & Community Extras
| Feature | Status | Our Reason |
|---------|--------|-----------|
| **Astrologer Consultation** | ❌ | Partnership complexity, liability issues |
| **Horoscope Display** | ❌ | Advanced horoscope comparison deferred |
| **Guna Matching** | ❌ | Requires external API integration |
| **Compatibility Report** | ❌ | Requires advanced astrology tooling |
| **Mangal Dosha Calculator** | ❌ | Requires advanced astrology tooling |

---

## 🔍 HOW TO USE THIS REFERENCE


---

## 🗺️ IMPLEMENTATION ROADMAP

> **See [TECH-STACK.md](./TECH-STACK.md) section 17 for detailed implementation roadmap with day-by-day breakdown.**

**MVP (Weeks 1-10):** Authentication, Profiles, Photos, Search, Interests, Payments, Photo Requests, Notifications, Admin Panel

**Post-MVP:** Advanced Search, Enhanced Privacy, Premium Features, SEO, Analytics, Mobile Apps

---

## 📊 SUCCESS METRICS & KPIs

### **User Engagement**
- Profile completion rate (target: >80%)
- Daily active users (target: 30% of registered users)
- Session duration (target: >5 minutes)
- Profile views per user (target: >20 per session)

### **Conversion Metrics**
- Free to paid conversion rate (target: 15%)
- Interest to match conversion rate (target: 25%)
- Profile approval rate (target: >90%)
- User retention rate (target: 60% after 30 days)

### **Business Metrics**
- Monthly recurring revenue (MRR)
- Customer acquisition cost (CAC)
- Lifetime value (LTV)
- Churn rate (target: <5% monthly)

### **Quality Metrics**
- User-reported issues (target: <1% of users)
- Admin moderation time (target: <24 hours)
- System uptime (target: 99.9%)
- Page load time (target: <2 seconds)

---

## 🔧 TECHNICAL DEBT & OPTIMIZATION

### **Immediate Optimizations**
- Implement photo compression and metadata stripping
- Add comprehensive error handling and logging
- Optimize database queries and indexing
- Implement proper caching strategies

### **Medium-term Improvements**
- Add CDN for faster image loading
- Implement advanced search with Elasticsearch
- Add real-time analytics and monitoring
- Optimize mobile performance and PWA features

### **Long-term Considerations**
- Plan migration to Supabase Pro for advanced features
- Consider microservices architecture for scalability
- Implement advanced ML/AI features for matching
- Add multi-language support for regional expansion

---

## 🤖 AI DEVELOPMENT GUIDELINES

> **See [TECH-STACK.md](./TECH-STACK.md) section 23 for complete AI development best practices, incremental development strategies, and common pitfalls to avoid.**

---

**Last Updated:** November 6, 2025  
**Document Version:** 4.0 (AI-Optimized for Implementation)  
**Implementation Start:** November 7, 2025  
**Features Removed from MVP:** 11  
**Next Review:** Post-launch (January 2026)
