# CC Matrimony Admin Management Specification

> **Status**: Admin panel is **NOW part of MVP** (Weeks 4-6).  
> **This is the single source of truth for all admin panel features, implementation, and future phases.**  
> **Goal**: Build the most advanced admin panel possible with current tech stack (Next.js, Supabase, Vercel, React Admin, shadcn/ui, Recharts, Resend, Sentry).

> **Note**: **MVP is considered Phase 1** (detailed planning). Future phases (Phase 2+) are kept brief and will be detailed when starting each phase.

## 1. Purpose & Scope

- **MVP Timeline**: Weeks 4-6 (3 weeks for admin panel implementation)
- **Goal**: Provide comprehensive, advanced admin interface leveraging all capabilities of the tech stack
- **Tech Stack**: 
  - **Frontend**: React Admin + ra-supabase + shadcn/ui + Recharts
  - **Backend**: Supabase (PostgreSQL, Auth, Edge Functions, Realtime) + Vercel API Routes + Vercel Cron
  - **Services**: Resend (Email), Razorpay (Payments), Cloudinary (Media), Sentry (Monitoring), PostHog (Analytics)
- **Platform**: CC Matrimony (matri.naveevo.com)
- **Documentation**: All admin-related information is consolidated in this file. Other files reference this document.

## 2. MVP Admin Features (Weeks 4-6) - Advanced Implementation

> **Note**: This admin panel leverages ALL capabilities of the tech stack to provide maximum functionality within the 3-week timeline.

### **Week 4: Admin Foundation**

#### **4.1 Admin Authentication** (P0)
- Secure admin login page (`/admin/login`)
- Role-based access control (Super Admin only for MVP)
- Session management (30-day expiry)
- Protected admin routes (redirect to login if not authenticated)

#### **4.2 Advanced User Management** (P0)
- **User List & Search** (PostgreSQL Full-text Search):
  - View all users in advanced table (React Admin)
  - Full-text search by email, phone, name, profile ID (PostgreSQL GIN index)
  - Advanced filters:
    - Status (active, suspended, pending, deleted)
    - Community/religion (multi-select)
    - Subscription status (free, active, expired)
    - Registration date range
    - Last active date range
    - Profile completeness range
    - Has photos (yes/no)
    - Email verified (yes/no)
  - Sort by: registration date, last active, profile completeness, subscription value
  - Bulk selection for batch operations
  - Export filtered results (CSV, JSON)
- **User Detail View** (Comprehensive):
  - Complete profile information
  - Profile photos gallery
  - Activity timeline (Supabase Realtime events)
  - Interests sent/received
  - Profile views (who viewed this profile)
  - Payment history (with Razorpay links)
  - Subscription history
  - Login history (IP addresses, devices)
  - Email notification history
  - Profile edit history (audit trail)
- **User Actions**:
  - Activate user account
  - Suspend user account (with duration: 1 day, 7 days, 30 days, permanent)
  - Delete user account (soft delete with 30-day grace period)
  - View/edit user subscription status
  - Manually activate premium plan
  - Send email to user (via Resend)
  - Impersonate user (for support - with audit log)
  - Export user data (GDPR/DPDPA compliance)
  - View user's photo requests sent/received

### **Week 5: Profile & Payment Management**

#### **5.1 Advanced Profile Approval Queue** (P0)
- **Pending Profiles List** (Real-time Updates):
  - View all pending profiles (auto-refresh via Supabase Realtime)
  - Advanced filters:
    - Registration date range
    - Community/religion (multi-select)
    - Profile completeness range
    - Has photos (yes/no)
    - Age range
  - Sort by: newest first, oldest first, completeness score
  - Bulk approve/reject (select multiple)
  - Quick preview cards (with photo thumbnails)
- **Profile Review** (Enhanced):
  - Full profile details view (all fields)
  - Photo gallery with zoom
  - Profile completeness breakdown
  - Related profiles (same community/religion)
  - Approve profile (with optional notes, auto-email user)
  - Reject profile (with required reason dropdown + custom notes, auto-email user)
  - Request changes (send email with specific requirements)
  - Edit profile directly (inline editing)
  - View profile edit history
  - Approve/reject individual photos separately
- **Profile Management** (Advanced):
  - View all profiles (approved/rejected/pending) with filters
  - Full-text search (PostgreSQL) by name, profile ID, email, bio
  - Advanced search with multiple criteria
  - Edit any profile field (with change tracking)
  - View profile completeness score (with improvement suggestions)
  - Profile analytics (views, interests received, conversion rate)
  - Duplicate profile detection
  - Profile comparison tool (side-by-side)
  - Bulk profile operations (approve, reject, edit, export)
  - Policy: Profiles are visible even without photos; "Request Photo" prompts appear in user UI when a profile has zero approved photos

#### **5.2 Advanced Photo Moderation Queue** (P0)
- **Flagged Photos List** (Real-time):
  - View all flagged/reported photos (auto-refresh)
  - Filter by:
    - Reason (inappropriate, fake, no face, blurry, wrong person)
    - Report date range
    - Profile owner
    - Reporter
    - Status (pending, approved, rejected)
  - Sort by: report date, priority (multiple reports)
  - Photo thumbnails with quick preview
  - Bulk approve/reject
- **Photo Review** (Enhanced):
  - View photo in full size (with zoom)
  - Photo metadata (upload date, size, dimensions, Cloudinary URL)
  - Reporter details (with history of reports)
  - Profile owner details
  - Related photos from same profile
  - Approve photo (keep, notify user)
  - Reject photo (with reason dropdown + notes, auto-email user, auto-delete from Cloudinary)
  - Request replacement (email user to upload new photo)
  - Delete photo permanently (with Cloudinary cleanup)
  - View photo analytics (views, downloads if applicable)

#### **5.3 Advanced Payment Management** (P0)
- **Payment Overview** (Advanced Table):
  - View all payments (with real-time updates)
  - Advanced filters:
    - Status (success, failed, pending, refunded, cancelled)
    - Date range (with presets: today, this week, this month, custom)
    - Plan type (free, silver, gold, platinum, VIP)
    - Payment method (Razorpay, manual, offline)
    - Amount range
    - User search
  - Sort by: date, amount, status
  - Export payments (CSV, Excel for accounting)
  - Payment statistics summary (total, success rate, refund rate)
- **Payment Detail View**:
  - Complete payment information
  - Razorpay transaction details (with link to Razorpay dashboard)
  - User details
  - Subscription details
  - Payment timeline (created, processed, refunded if applicable)
  - Webhook events log
  - Invoice download (PDF generation)
- **Payment Actions**:
  - Process manual refund (with reason dropdown, auto-email user, Razorpay API integration)
  - Partial refund support
  - View subscription details (with renewal dates)
  - Manually activate premium plan (with payment record creation)
  - Cancel subscription (with prorated refund option)
  - Extend subscription (add days/months)
  - View payment analytics:
    - Revenue trends (line chart)
    - Revenue by plan (pie chart)
    - Revenue by date (bar chart)
    - Conversion funnel (registrations → payments)
    - Refund rate analysis
    - Average transaction value
    - Revenue forecasting

### **Week 6: Analytics & Settings**

#### **6.1 Advanced Analytics Dashboard** (P1)
- **Real-time Metrics** (Supabase Realtime):
  - Live user count
  - Active sessions (last 5 minutes)
  - Real-time revenue tracking
  - Current pending approvals
- **User Statistics** (Interactive Charts - Recharts):
  - Total users (with growth trend)
  - Active users (last 7/30/90 days) - line chart
  - New registrations (daily/weekly/monthly) - bar chart
  - Premium users count and conversion rate
  - User retention cohort analysis
  - User engagement heatmap
- **Profile Statistics**:
  - Total profiles (with approval rate)
  - Pending approvals count (real-time)
  - Approved/rejected profiles (pie chart)
  - Profile completion rate distribution
  - Average profile completeness score
- **Payment Statistics** (Advanced):
  - Total revenue (with trend line)
  - Revenue by plan type (stacked bar chart)
  - Revenue by date range (time series)
  - Active subscriptions (with churn rate)
  - Pending payments
  - Revenue forecasting (30/60/90 days)
  - Average revenue per user (ARPU)
  - Lifetime value (LTV) estimates
- **Community Statistics** (Interactive):
  - User distribution by religion (pie chart)
  - User distribution by community (bar chart)
  - Top communities by user count
  - Community growth trends
  - Geographic distribution (if location data available)
- **Conversion Funnel** (Visual):
  - Registration → Profile Creation → Photo Upload → First Interest → Premium
  - Drop-off analysis at each stage
  - Conversion rate by source
- **Engagement Metrics**:
  - Daily active users (DAU)
  - Weekly active users (WAU)
  - Monthly active users (MAU)
  - Average session duration
  - Interests sent/received ratio
  - Profile views per user

#### **6.2 Advanced System Settings** (P1)
- **Feature Flags** (Database-driven):
  - Photo Request Feature (ON/OFF)
  - Photo Approval Workflow (ON/OFF)
  - Community Features (ON/OFF)
  - Premium Features (ON/OFF)
  - Advanced Search (ON/OFF)
  - Email Notifications (ON/OFF)
  - SMS Notifications (ON/OFF)
  - Real-time Notifications (ON/OFF)
  - A/B Testing Features (ON/OFF)
- **Email Configuration** (Resend Integration):
  - Configure email templates (visual editor)
  - Email template variables preview
  - Test email sending (to admin email)
  - Email delivery status tracking
  - Bounce/complaint handling
  - Email analytics (open rates, click rates)
- **SMS Configuration** (Future):
  - Configure SMS provider
  - SMS templates
  - Test SMS sending
  - Delivery tracking
- **Payment Gateway Configuration**:
  - Enable/disable Razorpay
  - Configure Razorpay keys (test/live toggle)
  - View payment gateway status
  - Webhook endpoint configuration
  - Payment retry settings
- **Platform Settings**:
  - Site name, description, logo
  - Contact email, support email
  - Maintenance mode toggle (with custom message)
  - Site-wide announcements
  - Cookie consent settings
  - Privacy policy/Terms links
- **Rate Limiting**:
  - API rate limits per user
  - Admin rate limits
  - IP-based rate limiting
- **Cron Jobs Management**:
  - View scheduled cron jobs (Vercel Cron)
  - View cron execution logs
  - Manual trigger for cron jobs
  - Cron job status monitoring

#### **6.3 Advanced Content Management** (P1)
- **Success Stories** (Full CMS):
  - Add/edit/delete success stories (rich text editor)
  - Upload multiple photos (via Cloudinary)
  - Photo gallery management
  - Publish/unpublish stories (with scheduled publishing)
  - Story categories/tags
  - Featured stories (pin to top)
  - Story analytics (views, shares)
  - SEO settings (meta title, description, keywords)
  - Preview before publishing
- **Testimonials**:
  - Manage testimonials (CRUD)
  - Approve user-submitted testimonials
  - Featured testimonials
  - Testimonial moderation queue
  - Display order management
- **Community Content**:
  - Manage community-specific content (rich text editor)
  - Update community descriptions
  - Community-specific landing pages
  - Community statistics display
- **FAQ Management**:
  - Add/edit/delete FAQ items
  - Categorize FAQs
  - Search functionality
  - Display order
- **Blog/Articles** (If applicable):
  - Rich text editor for articles
  - Categories and tags
  - Featured images
  - SEO optimization
  - Publishing workflow

## 3. Additional Advanced MVP Features (Weeks 4-6)

### **3.1 Monitoring & Observability Dashboard** (P1)
- **System Health** (Real-time):
  - Database size and usage (Supabase quota monitoring)
  - Storage usage (Cloudinary quota)
  - API response times
  - Error rate (Sentry integration)
  - Active connections
- **Error Log Viewer** (Sentry Integration):
  - View all errors from Sentry
  - Filter by severity, date, user
  - Error details with stack traces
  - Error trends and patterns
  - Resolve/ignore errors
- **Cron Job Monitor**:
  - View all Vercel Cron jobs
  - Execution history and logs
  - Success/failure rates
  - Manual trigger capability
  - Scheduled run times
- **API Usage Analytics**:
  - API call volume
  - Response time distribution
  - Error rate by endpoint
  - Rate limit hits
  - Most used endpoints

### **3.2 Email Campaign Management** (P1)
- **Campaign Builder** (Resend API):
  - Create email campaigns
  - User segmentation (by plan, community, activity, registration date)
  - Email template editor (with variables)
  - Schedule campaigns (future date/time)
  - A/B testing (subject lines, content)
  - Track delivery, opens, clicks (Resend analytics)
  - Campaign performance dashboard
- **Email Templates**:
  - Manage all email templates
  - Preview templates with sample data
  - Test email sending
  - Template versioning
- **Bulk Email**:
  - Send email to selected users
  - Send to filtered user groups
  - Email history per user

### **3.3 Advanced Data Operations** (P1)
- **Data Export**:
  - Export users (CSV, JSON, Excel)
  - Export profiles (with filters)
  - Export payments (accounting format)
  - Export analytics reports (PDF, Excel)
  - Scheduled exports (email delivery)
- **Data Import**:
  - Import users from CSV
  - Bulk profile updates from CSV
  - Data validation before import
  - Import history and logs
- **Data Cleanup**:
  - Find duplicate profiles
  - Find inactive users (configurable threshold)
  - Bulk delete operations (with confirmation)
  - Archive old data

### **3.4 Audit Trail & Activity Logs** (P1)
- **Admin Activity Log**:
  - All admin actions logged (who, what, when, IP)
  - Filter by admin user, action type, date
  - Export audit logs
  - Search audit trail
- **User Activity Tracking**:
  - User login history
  - Profile edit history
  - Payment activity
  - Interest activity
  - View user activity timeline
- **System Change Log**:
  - Settings changes
  - Feature flag changes
  - Configuration changes

### **3.5 Advanced Search & Filtering** (P1)
- **Global Search** (PostgreSQL Full-text):
  - Search across users, profiles, payments
  - Advanced search operators
  - Search history
  - Saved searches
- **Smart Filters**:
  - Save filter combinations
  - Share filter URLs
  - Filter presets (common queries)

### **3.6 User Support Tools** (P1)
- **User Impersonation**:
  - Impersonate user (for support)
  - Full audit logging
  - Auto-logout after session
- **Direct Communication**:
  - Send email to user (from admin panel)
  - View email history per user
  - Email templates for common issues
- **Support Tickets** (If implemented):
  - View user support requests
  - Respond to tickets
  - Ticket status management

### **3.7 Advanced Reporting** (P1)
- **Custom Reports**:
  - Build custom reports (drag-and-drop metrics)
  - Date range selection
  - Export reports (PDF, Excel, CSV)
  - Schedule report generation (email delivery)
- **Pre-built Reports**:
  - Daily/weekly/monthly summaries
  - Revenue reports
  - User growth reports
  - Conversion reports
  - Engagement reports

## 4. Admin Roles & Permissions (MVP)

### **4.1 Super Admin (Platform Owner)** - MVP Only
- Full access to all features
- System configuration
- User and profile management
- Payment management
- Content management
- Audit log access
- System settings modification

### **4.2 Permission System** (Database-driven)
- Permissions stored in `admin_permissions` table
- Role-based access control (RBAC) via Supabase RLS
- Feature-level permissions (can be extended later)
- IP whitelist for admin access (optional)
- Two-factor authentication (2FA) support (future)

> **Note**: Additional roles (Admin, Telecaller) and granular permissions deferred to Phase 2, but infrastructure is ready

## 5. Implementation Plan

### **Week 7 Implementation**
- [ ] Set up admin authentication (Supabase Auth with admin role)
- [ ] Create admin layout and navigation (sidebar, header, breadcrumbs)
- [ ] Build advanced user management page:
  - [ ] Full-text search implementation (PostgreSQL GIN index)
  - [ ] Advanced filters (multi-select, date ranges, status)
  - [ ] Bulk selection and operations
  - [ ] Export functionality (CSV, JSON, Excel)
  - [ ] User detail view (comprehensive with activity timeline)
  - [ ] User impersonation (with audit log)
  - [ ] Real-time updates (Supabase Realtime)
- [ ] Implement audit trail logging (PostgreSQL triggers)
- [ ] Set up error logging (Sentry integration)

### **Week 5 Implementation**
- [ ] Build advanced profile approval queue:
  - [ ] Real-time pending profiles list (Supabase Realtime)
  - [ ] Bulk approve/reject functionality
  - [ ] Profile comparison tool
  - [ ] Duplicate profile detection
  - [ ] Profile analytics display
- [ ] Implement advanced photo moderation:
  - [ ] Real-time flagged photos (Supabase Realtime)
  - [ ] Cloudinary API integration (delete photos)
  - [ ] Photo metadata display
  - [ ] Bulk photo operations
- [ ] Build advanced payment management:
  - [ ] Payment analytics (Recharts - line, pie, bar charts)
  - [ ] Razorpay API integration (refunds, partial refunds)
  - [ ] Invoice generation (PDF)
  - [ ] Payment export (Excel for accounting)
  - [ ] Revenue forecasting calculations
- [ ] Implement bulk operations API endpoints
- [ ] Add data export/import functionality

### **Week 6 Implementation**
- [ ] Build advanced analytics dashboard:
  - [ ] Real-time metrics (Supabase Realtime)
  - [ ] Interactive charts (Recharts - all chart types)
  - [ ] Conversion funnel visualization
  - [ ] Revenue forecasting
  - [ ] Engagement metrics (DAU, WAU, MAU)
  - [ ] User retention cohort analysis
- [ ] Implement monitoring dashboard:
  - [ ] Sentry error log viewer (with filters)
  - [ ] Cron job monitor (Vercel Cron logs)
  - [ ] System health metrics (database, storage, API)
  - [ ] API usage analytics
- [ ] Build email campaign management:
  - [ ] Campaign builder (Resend API)
  - [ ] User segmentation
  - [ ] Email template editor
  - [ ] Campaign scheduling
  - [ ] A/B testing setup
  - [ ] Campaign analytics (open/click rates)
- [ ] Implement advanced system settings:
  - [ ] Feature flags (database-driven)
  - [ ] Email configuration (Resend integration)
  - [ ] Cron job management UI
  - [ ] Rate limiting settings
  - [ ] Platform settings (maintenance mode, announcements)
- [ ] Build advanced content management:
  - [ ] Rich text editor (for stories, articles)
  - [ ] SEO settings
  - [ ] Scheduled publishing
  - [ ] FAQ management
- [ ] Add advanced reporting:
  - [ ] Custom report builder
  - [ ] Pre-built reports
  - [ ] Report scheduling
- [ ] Testing and polish

## 6. Tech Stack & Implementation

> **See [TECH-STACK.md](./TECH-STACK.md) for complete tech stack details and [DEVELOPMENT-PATTERNS.md](./DEVELOPMENT-PATTERNS.md) for code patterns.**

**Admin Framework:**
- React Admin + ra-supabase for standard CRUD
- shadcn/ui for custom components
- Recharts for analytics charts

**Database:**
- Add `admin_users` table (see TECH-STACK.md section 14 for schema)
- Use existing Supabase tables
- Partner tables will be added in Phase 6+ (see section 10)

**API Routes:**
- **Core Admin Endpoints:**
  - `GET /api/admin/users` - List all users (with search, filters, pagination)
  - `PATCH /api/admin/users/[id]` - Update user status (activate, suspend, delete)
  - `GET /api/admin/profiles/pending` - Get pending profile approvals
  - `PATCH /api/admin/profiles/[id]` - Approve/reject profile (with reason/notes)
  - `GET /api/admin/photos/flagged` - Get flagged photos for moderation
  - `PATCH /api/admin/photos/[id]` - Approve/reject photo (with reason)
  - `GET /api/admin/payments` - List all payments (with filters)
  - `POST /api/admin/payments/[id]/refund` - Process manual refund
  - `GET /api/admin/stats` - Get dashboard statistics (users, profiles, payments, communities)
  - `GET /api/admin/settings` - Get system settings
  - `PATCH /api/admin/settings` - Update system settings
  - `GET /api/admin/content/stories` - Manage success stories
  - `POST /api/admin/content/stories` - Create success story
  - `PATCH /api/admin/content/stories/[id]` - Update success story
  - `DELETE /api/admin/content/stories/[id]` - Delete success story

## 7. Key Features Summary (Advanced MVP)

| Feature Category | MVP Status | Week | Priority | Tech Used |
|----------------|------------|------|----------|-----------|
| Admin Authentication | ✅ MVP | 4 | P0 | Supabase Auth |
| Advanced User Management | ✅ MVP | 4 | P0 | React Admin + PostgreSQL Full-text |
| Advanced Profile Approval | ✅ MVP | 5 | P0 | Real-time + Bulk Operations |
| Advanced Photo Moderation | ✅ MVP | 5 | P0 | Real-time + Cloudinary API |
| Advanced Payment Management | ✅ MVP | 5 | P0 | Razorpay API + Analytics |
| Advanced Analytics Dashboard | ✅ MVP | 6 | P1 | Recharts + Supabase Realtime |
| Advanced System Settings | ✅ MVP | 6 | P1 | Feature Flags + Resend API |
| Advanced Content Management | ✅ MVP | 6 | P1 | Rich Text Editor + Cloudinary |
| Bulk Operations | ✅ MVP | 4-6 | P1 | Batch API Endpoints |
| Data Export/Import | ✅ MVP | 4-6 | P1 | CSV/JSON/Excel + Scheduled Exports |
| Audit Trail | ✅ MVP | 4-6 | P1 | PostgreSQL Triggers + Activity Logs |
| Real-time Updates | ✅ MVP | 4-6 | P1 | Supabase Realtime |
| Error Log Viewer | ✅ MVP | 6 | P1 | Sentry Integration |
| Cron Job Management | ✅ MVP | 6 | P1 | Vercel Cron Logs |
| Email Campaigns | ✅ MVP | 6 | P1 | Resend API + Segmentation + A/B Testing |
| Global Search | ✅ MVP | 4-6 | P1 | PostgreSQL Full-text |
| Data Visualization | ✅ MVP | 6 | P1 | Recharts |
| User Activity Tracking | ✅ MVP | 4-6 | P1 | Supabase Events |
| Conversion Funnel | ✅ MVP | 6 | P1 | Analytics Queries |
| Monitoring Dashboard | ✅ MVP | 6 | P1 | Sentry + System Metrics + Cron Logs |
| User Impersonation | ✅ MVP | 4 | P1 | Support Tool + Audit Logging |
| Advanced Reporting | ✅ MVP | 6 | P1 | Custom Reports + Scheduled Delivery |
| Data Cleanup Tools | ✅ MVP | 5-6 | P1 | Duplicate Detection + Archive |
| Invoice Generation | ✅ MVP | 5 | P1 | PDF Generation |
| Profile Comparison | ✅ MVP | 5 | P1 | Side-by-side View |
| Revenue Forecasting | ✅ MVP | 6 | P1 | Analytics Calculations |
| Advanced Roles | 🔄 Phase 2 | - | P2 | Extended RBAC |
| Partner & Agency Management | 🔄 Phase 6+ | - | P2 | Basic CRUD + User Assignment |
| SMS Broadcasting | 🔄 Phase 2 | - | P2 | SMS Provider API |
| Red Flag Dashboard | 🔄 Phase 3 | - | P2 | ML/Analytics |
| Churn Signals | 🔄 Phase 3 | - | P2 | Analytics + Alerts |
| Automated Workflows | 🔄 Phase 4 | - | P2 | Vercel Cron + Edge Functions |
| Advanced Permissions | 🔄 Phase 4 | - | P2 | Granular RBAC |
| AI-Powered Insights | 🔄 Phase 2 | - | P2 | ML Models |

## 8. UI/UX Guidelines

- **Design**: Consistent with main app (shadcn/ui components)
- **Navigation**: Sidebar navigation with collapsible sections
- **Tables**: Sortable, filterable, paginated
- **Actions**: Clear buttons with confirmation modals for destructive actions
- **Feedback**: Toast notifications for all actions
- **Loading**: Skeleton loaders for data fetching

## 9. Security Considerations

- **Authentication**: Secure admin login with strong password requirements
- **Authorization**: Role-based access (Super Admin only for MVP)
- **Audit Trail**: Log all admin actions (who, what, when)
- **Rate Limiting**: Prevent brute force attacks on admin login
- **Session Management**: Secure session handling, auto-logout after inactivity

---

## 10. Future Phases - Brief Overview

> **Note**: Future phases are kept brief here. Detailed planning will be done when starting each phase, similar to how MVP is detailed above.

### **Phase 2: Enhanced Admin Capabilities**
- **Advanced Roles & Permissions**: Admin (Community Manager) and Telecaller roles with granular permissions
- **Enhanced Bulk Operations**: Enhanced bulk operations beyond MVP (bulk email/SMS, advanced bulk workflows)
- **Enhanced Analytics**: Advanced analytics beyond MVP (detailed custom reports, scheduled reports, advanced dashboards)
- **Email & SMS Broadcasting**: Campaign builder, segmentation, A/B testing, delivery tracking
- **Enhanced Export Capabilities**: Enhanced export features beyond MVP (data migration tools, advanced import)
- **Enhanced Audit Trail**: Enhanced audit features beyond MVP (compliance reporting, advanced filtering)
- **Coupon Code Management**: Create, edit, track promotional codes (percentage or fixed amount), usage analytics

### **Phase 3: Advanced Monitoring & Safety**
- **Red Flag Dashboard**: Suspicious activity detection, automated alerts, investigation tools
- **Churn Signals**: Churn prediction, retention actions, win-back campaigns
- **Advanced Reporting**: Business intelligence, revenue forecasting, custom dashboards

### **Phase 4: Automation & Efficiency**
- **Automated Workflows**: Workflow builder, automated approval chains, conditional actions
- **Advanced Permissions System**: Fine-grained access control, permission templates

### **Phase 5: Enterprise Features**
- **Multi-tenant Support**: Multiple organization support, white-label options
- **Advanced Integration**: API management, CRM integration, marketing automation

### **Phase 6+: Partner & Agency Features** (Deferred)
- **Partner & Agency Management**: Basic CRUD, user assignment
- **Partner Portal**: Separate login, dashboard, commission tracking
- **Partner Marketplace**: Service listings, booking system, reviews & ratings
- **Partner Automation**: Auto-assignment, workflows, reporting
- **Partner Ecosystem**: Partner directory, collaboration, white-label portal

---

## 11. Phase Summary

| Phase | Focus | Key Features |
|-------|-------|--------------|
| **MVP (Weeks 4-6)** | Advanced Admin Panel | Authentication, Advanced User/Profile/Payment Management, Real-time Analytics, Monitoring Dashboard, Email Campaigns, Export/Import, Audit Trail, Bulk Operations, Data Visualization, Error Logs, Cron Management, User Impersonation, Advanced Reporting |
| **Phase 2** | Enhanced Capabilities | Advanced Roles, Enhanced Bulk Operations (beyond MVP), Enhanced Analytics (beyond MVP), Email/SMS Broadcasting, Enhanced Export Capabilities, Enhanced Audit Trail, Coupon Code Management |
| **Phase 3** | Advanced Monitoring & Safety | Red Flag Dashboard, Churn Signals, Advanced Reporting |
| **Phase 4** | Automation | Automated Workflows, Advanced Permissions |
| **Phase 5** | Enterprise Features | Multi-tenant Support, Advanced Integrations |
| **Phase 6+** | Partner & Agency Features | Partner Management, Partner Portal, Partner Marketplace, Partner Automation, Partner Ecosystem |

---

**Last Updated**: January 2025  
**Status**: MVP (Phase 1) - Weeks 4-6 Implementation  
**Future Phases**: See section 10 for brief Phase 2+ overview. Detailed planning will be done when starting each phase.
