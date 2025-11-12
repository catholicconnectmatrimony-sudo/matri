# CC Matrimony Admin Management Specification (Deferred to Phase 2)

## 1. Purpose & Scope
- Status: Admin panel is not part of the MVP. This specification outlines the Phase 2+ Admin features.
- Provide easy-to-use admin interface for non-technical users
- Enable efficient management of users, profiles, and platform operations
- **Recommended Tech Stack (Phase 2)**: **Hybrid Approach** - React Admin + ra-supabase for standard CRUD (users, profiles, subscriptions, interests, payments) + shadcn/ui for custom features (reciprocity dashboard, photo moderation queue, community management). This balances speed (2-3 days for 80% of admin) with flexibility (custom features where needed).
- Ensure platform safety and quality through effective moderation
- Platform: CC Matrimony (matri.naveevo.com)
- Tech Stack: Vercel Hobby + Supabase Free

## 2. Admin Roles & Permissions

> **MVP Implementation**: Admin panel deferred. All roles and tools in this document are Phase 2+ deliverables.

### **2.1 Super Admin (Platform Owner)** - IMPLEMENTED FOR MVP
| Permission | Access Level | Description |
|------------|--------------|-------------|
| **User Management** | Full | Create, edit, delete, suspend users |
| **Profile Management** | Full | Approve, reject, edit all profiles |
| **Payment Management** | Full | View payments, process refunds, manage subscriptions |
| **Content Management** | Full | Manage all content, success stories, community pages |
| **Analytics Access** | Full | View all analytics and reports |
| **System Settings** | Full | Configure platform settings, features, and policies |
| **Support Channels**: Configure support email, hotline, WhatsApp Business number (future), and escalation rules. |
| **Community Management** | Full | Manage community-specific features and content |
| **Red Flag Dashboard**: View and manage red-flagged users, profiles, and content |
| **Churn Signals**: View and manage churn signals for users and profiles |

### **2.2 Admin (Community Manager)** - DEFERRED (Post-MVP when hiring community managers)
| Permission | Access Level | Description |
|------------|--------------|-------------|
| **User Management** | Limited | View users, basic user operations |
| **Profile Management** | Full | Approve, reject, edit profiles |
| **Payment Management** | View Only | View payments and subscriptions |
| **Content Management** | Limited | Manage community-specific content |
| **Analytics Access** | Limited | View basic analytics |
| **System Settings** | None | Cannot modify system settings |
| **Community Management** | Full | Manage assigned community features |

### **2.3 Telecaller (Support Staff)** - DEFERRED (Post-MVP when hiring support staff)
| Permission | Access Level | Description |
|------------|--------------|-------------|
| **User Management** | View Only | View user profiles and basic info |
| **Profile Management** | Limited | Basic profile operations |
| **Payment Management** | None | No payment access |
| **Content Management** | None | No content management |
| **Analytics Access** | None | No analytics access |
| **System Settings** | None | No system settings access |
| **Community Management** | None | No community management |

## 3. Admin Dashboard Overview

### **3.1 Main Dashboard**
```typescript
interface AdminDashboard {
  overview: {
    totalUsers: number;
    activeUsers: number;
    premiumUsers: number;
    pendingApprovals: number;
    recentActivity: Activity[];
  };
  quickActions: {
    approveProfiles: () => void;
    viewPayments: () => void;
    manageUsers: () => void;
    viewAnalytics: () => void;
  };
  alerts: {
    pendingApprovals: number;
    paymentIssues: number;
    userReports: number;
    systemAlerts: number;
  };
}
```

### **3.2 Dashboard Widgets**
- **User Statistics**: Total users, new registrations, active users
- **Profile Statistics**: Total profiles, pending approvals, completed profiles
- **Payment Statistics**: Revenue, subscriptions, pending payments
- **Community Statistics**: Community-wise user distribution
- **Recent Activity**: Latest user actions, profile updates, payments
- **Quick Actions**: One-click access to common tasks

## 4. User Management

### **4.1 User List & Search**
```typescript
interface UserManagement {
  search: {
    byEmail: string;
    byPhone: string;
    byName: string;
    byProfileId: string;
    byCommunity: string;
    byStatus: 'active' | 'suspended' | 'pending';
  };
  filters: {
    registrationDate: DateRange;
    lastActive: DateRange;
    subscriptionStatus: string[];
    community: string[];
    verificationStatus: string[];
  };
  actions: {
    viewProfile: (userId: string) => void;
    editUser: (userId: string) => void;
    suspendUser: (userId: string) => void;
    activateUser: (userId: string) => void;
    deleteUser: (userId: string) => void;
    sendMessage: (userId: string) => void;
  };
}
```

### **4.2 User Profile Management**
- **Profile View**: Complete user profile with all details
- **Profile Edit**: Edit user information and preferences
- **Profile Status**: Activate, suspend, or delete profiles
- **Verification Status**: Mark profiles as verified
- **Subscription Management**: View and manage user subscriptions
- **Activity History**: View user activity and interactions

### **4.3 Bulk Operations**
- **Bulk Approve**: Approve multiple profiles at once
- **Bulk Suspend**: Suspend multiple users at once
- **Bulk Email**: Send emails to multiple users
- **Bulk Export**: Export user data in CSV/Excel format
- **Bulk Status Change**: Change status of multiple users
- **Incomplete Profile Outreach**: Filter profiles below completion threshold, assign follow-ups to telecallers, and trigger reminder campaigns.

## 5. Community & URL Management

### **5.1 Community Management**
| Feature | Description | Access Level |
|---------|-------------|--------------|
| **Community Creation** | Add new communities with religion association | Super Admin |
| **URL Structure** | Manage URL slugs for all communities | Super Admin |
| **Community Metadata** | Edit display names, descriptions, and images | Admin+ |
| **Sub-communities** | Manage hierarchical community structures | Admin+ |
| **Bulk Import** | Import communities via CSV with religion mapping | Super Admin |
| **URL Redirects** | Manage 301 redirects for URL changes | Super Admin |
| **Canonical URLs** | Set canonical URLs for community pages | Admin+ |

### **5.2 URL Management**
```typescript
interface URLManagement {
  currentURL: string;
  redirectsTo?: string;
  isCanonical: boolean;
  lastModified: Date;
  modifiedBy: string;
  status: 'active' | 'redirect' | 'archived';
}
```

### **5.3 Community Schema**
```typescript
interface Community {
  id: string;
  name: string;
  slug: string;
  religion: {
    id: string;
    name: string;
    slug: string;
  };
  description: string;
  seoTitle: string;
  metaDescription: string;
  featuredImage: string;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
  updatedBy: string;
}
```

## 6. Profile Management

### **5.1 Profile Approval Queue**
```typescript
interface ProfileApproval {
  pendingProfiles: Profile[];
  approvalActions: {
    approve: (profileId: string) => void;
    reject: (profileId: string, reason: string) => void;
    requestChanges: (profileId: string, changes: string[]) => void;
    bulkApprove: (profileIds: string[]) => void;
  };
  rejectionReasons: {
    'Incomplete Profile': string;
    'Inappropriate Content': string;
    'Fake Profile': string;
    'Poor Quality Photos': string;
    'Missing Information': string;
  };
}
```

### **5.2 Photo Management**
- **Post-Upload Review**: Photos auto-approve for instant profile visibility; admins skim recently uploaded/flagged photos and revoke if necessary.
- **Flag Handling**: Queue shows photos flagged by users or automated heuristics for further review.
- **Photo Quality Check**: Review photo quality and appropriateness during post-moderation.
- **Photo Replacement**: Request users to replace poor quality or policy-violating photos.
- **Photo Analytics**: Track photo revocation rate and flagged-photo resolution time.

### **5.3 Profile Quality Management**
- **Profile Completeness**: Track profile completion rates
- **Profile Quality Score**: Automated quality scoring
- **Profile Improvement Suggestions**: Suggest improvements to users
- **Profile Verification**: Mark profiles as verified
- **Profile Analytics**: Track profile performance and engagement

## 6. Payment Management

### **6.1 Payment Overview**
```typescript
interface PaymentManagement {
  overview: {
    totalRevenue: number;
    monthlyRevenue: number;
    activeSubscriptions: number;
    pendingPayments: number;
    failedPayments: number;
  };
  subscriptions: {
    active: Subscription[];
    expired: Subscription[];
    cancelled: Subscription[];
    pending: Subscription[];
  };
  actions: {
    processRefund: (paymentId: string) => void;
    extendSubscription: (userId: string, days: number) => void;
    cancelSubscription: (subscriptionId: string) => void;
    viewPaymentHistory: (userId: string) => void;
  };
}
```

### **6.2 Subscription Management**
- **Active Subscriptions**: View all active subscriptions
- **Expired Subscriptions**: View expired subscriptions
- **Cancelled Subscriptions**: View cancelled subscriptions
- **Subscription Analytics**: Track subscription trends and patterns
- **Manual Subscription Management**: Manually extend or cancel subscriptions

### **6.3 Payment Processing**
- **Payment History**: View all payment transactions
- **Refund Processing**: Process refunds for cancelled subscriptions
- **Payment Analytics**: Track payment success rates and trends
- **Offline Payment Management**: Manage offline payments and bank transfers
- **Payment Gateway Management**: Configure and manage payment gateways

## 7. Content Management

### **7.1 Success Stories Management**
```typescript
interface SuccessStoryManagement {
  stories: SuccessStory[];
  actions: {
    approve: (storyId: string) => void;
    reject: (storyId: string, reason: string) => void;
    edit: (storyId: string) => void;
    publish: (storyId: string) => void;
    unpublish: (storyId: string) => void;
  };
  categories: Record<string, SuccessStory[]>; // Dynamic categories based on communities in database
}
```

### **7.2 Community Content Management**
- **Community Pages**: Manage community-specific landing pages
- **Local Content**: Manage local language content
- **Festival Content**: Manage festival and tradition content
- **Community News**: Manage community news and updates
- **Content Analytics**: Track content performance and engagement

### **7.3 User-Generated Content**
- **User Reviews**: Manage user reviews and testimonials
- **User Feedback**: Manage user feedback and suggestions
- **Content Moderation**: Moderate user-generated content
- **Content Guidelines**: Manage content guidelines and policies

## 8. Analytics & Reporting

### **8.1 User Analytics**
```typescript
interface UserAnalytics {
  registration: {
    daily: number[];
    weekly: number[];
    monthly: number[];
    byCommunity: Record<string, number>;
    bySource: Record<string, number>;
  };
  engagement: {
    activeUsers: number;
    sessionDuration: number;
    pageViews: number;
    profileViews: number;
    interestsSent: number;
  };
  conversion: {
    freeToPaid: number;
    interestToMatch: number;
    profileCompletion: number;
    subscriptionRetention: number;
  };
}
```

### **8.2 Business Analytics**
- **Revenue Analytics**: Track revenue trends and patterns
- **Subscription Analytics**: Track subscription trends and churn
- **Payment Analytics**: Track payment success rates and trends
- **User Acquisition Analytics**: Track user acquisition costs and sources
- **Community Analytics**: Track community-specific metrics

### **8.3 Platform Analytics**
- **Performance Analytics**: Track platform performance and uptime
- **Feature Usage Analytics**: Track feature usage and adoption
- **Search Analytics**: Track search patterns and popular searches
- **Mobile Analytics**: Track mobile usage and performance
- **Scheduled Job Health**: View consolidated cron run log (status, duration, last run) sourced from `cron_runs` table; manual retry for failed jobs.
- **Churn Signals Panel**: Highlight members inactive for 14+ days or stuck below 50% completeness; export to telecaller outreach list.

## 9. Community Management

### **9.1 Community-Specific Management**
```typescript
interface CommunityManagement {
  communities: Record<string, CommunityStats>; // Dynamic communities from database
  actions: {
    manageCommunityContent: (community: string) => void;
    viewCommunityAnalytics: (community: string) => void;
    manageCommunityLeaders: (community: string) => void;
    updateCommunitySettings: (community: string) => void;
  };
}
```

### **9.2 Community Leader Management**
- **Community Leaders**: Manage community leader accounts
- **Leader Permissions**: Set permissions for community leaders
- **Leader Analytics**: Track community leader performance
- **Leader Communication**: Communicate with community leaders

### **9.3 Community Content Management**
- **Community Pages**: Manage community-specific pages
- **Community Events**: Manage community events and announcements
- **Community Success Stories**: Manage community-specific success stories
- **Community Guidelines**: Manage community-specific guidelines

## 10. Safety & Moderation

### **10.1 User Reports Management**
```typescript
interface ReportManagement {
  reports: UserReport[];
  categories: {
    'Fake Profile': UserReport[];
    'Inappropriate Content': UserReport[];
    'Harassment': UserReport[];
    'Spam': UserReport[];
    'Other': UserReport[];
  };
  actions: {
    investigate: (reportId: string) => void;
    resolve: (reportId: string, action: string) => void;
    escalate: (reportId: string) => void;
    blockUser: (userId: string) => void;
  };
}
```

### **10.2 Content Moderation**
- **Profile Moderation**: Moderate user profiles and content
- **Photo Moderation**: Moderate uploaded photos
- **Message Moderation**: Moderate user messages and communications
- **Content Flagging**: Handle flagged content and user reports

### **10.3 Safety Measures**
- **User Blocking**: Block users who violate terms
- **Content Filtering**: Filter inappropriate content
- **Spam Prevention**: Prevent spam and fake profiles
- **Safety Guidelines**: Manage safety guidelines and policies
- **Red Flag Dashboard**: Detect duplicate accounts (device/email overlap), rapid interest spikes, or unusual activity; surface investigation queue with audit notes.

## 11. System Configuration

### **11.1 Platform Settings**
```typescript
interface PlatformSettings {
  general: {
    siteName: string;
    siteDescription: string;
    contactEmail: string;
    supportEmail: string;
    maintenanceMode: boolean;
  };
  features: {
    reciprocityEngine: boolean;
    photoApproval: boolean;
    communityFeatures: boolean;
    premiumFeatures: boolean;
  };
  payments: {
    razorpayEnabled: boolean;
    phonepeEnabled: boolean;
    offlinePayments: boolean;
    freeTrialDays: number;
  };
}
```

### **11.2 Feature Toggles**
- **Reciprocity Engine**: Enable/disable reciprocity features
- **Photo Approval**: Enable/disable photo approval workflow
- **Community Features**: Enable/disable community-specific features
- **Premium Features**: Enable/disable premium features
- **Maintenance Mode**: Enable/disable maintenance mode

### **11.3 Email & SMS Settings**
- **Email Templates**: Manage email templates and content
- **SMS Templates**: Manage SMS templates and content
- **Notification Settings**: Configure notification preferences
- **Communication Settings**: Configure communication channels

## 12. SEO & URL Settings

### **12.1 URL Configuration**
| Setting | Description | Access Level |
|---------|-------------|--------------|
| **Base Domain** | Primary domain configuration | Super Admin |
| **URL Structure** | Pattern: `/{religion}/{community}/{sub-community?}` | Super Admin |
| **Trailing Slash** | Enable/disable trailing slashes | Super Admin |
| **URL Case Sensitivity** | Configure case sensitivity | Super Admin |
| **Canonical URL** | Global canonical URL settings | Super Admin |

### **12.2 SEO Management**
- **Meta Tags**: Manage default meta tags for all pages
- **Structured Data**: Configure JSON-LD templates
- **Sitemap**: Control sitemap generation settings
- **Robots.txt**: Customize robot directives
- **Hreflang**: Configure language/regional targeting

### **12.3 Redirect Management**
| Feature | Description | Access Level |
|---------|-------------|--------------|
| **301 Redirects** | Manage permanent redirects | Admin+ |
| **404 Monitoring** | Track and fix broken links | Admin+ |
| **Bulk Redirects** | Import/export redirects | Super Admin |
| **Regex Support** | Advanced pattern matching | Super Admin |

## 13. Admin User Experience

### **13.1 Easy-to-Use Interface**
- **Intuitive Navigation**: Simple and clear navigation
- **Quick Actions**: One-click access to common tasks
- **Bulk Operations**: Efficient bulk operations for multiple items
- **Search & Filters**: Powerful search and filtering capabilities
- **Responsive Design**: Works well on desktop and mobile

### **13.2 Non-Technical User Support**
- **Help Documentation**: Comprehensive help documentation
- **Video Tutorials**: Video tutorials for common tasks
- **Tooltips & Help**: Contextual help and tooltips
- **Support Contact**: Easy access to technical support
- **Training Materials**: Training materials for admin users

### **13.3 Efficiency Features**
- **Keyboard Shortcuts**: Keyboard shortcuts for power users
- **Customizable Dashboard**: Customizable dashboard widgets
- **Saved Searches**: Save frequently used searches
- **Quick Filters**: Quick access to common filters
- **Export Functions**: Easy data export and reporting

## 14. Mobile Admin Access

### **14.1 Mobile Admin Panel**
- **Responsive Design**: Mobile-friendly admin interface
- **Mobile-Specific Features**: Features optimized for mobile use
- **Touch-Friendly Interface**: Touch-friendly interface design
- **Offline Capabilities**: Basic offline capabilities
- **Mobile Notifications**: Mobile notifications for important events

### **14.2 Mobile Admin Features**
- **Quick Approvals**: Quick profile and photo approvals
- **User Management**: Basic user management on mobile
- **Payment Monitoring**: Monitor payments on mobile
- **Analytics View**: View basic analytics on mobile
- **Emergency Actions**: Emergency actions like user suspension

## 15. System Health Monitoring

### **15.1 Critical Metrics**
```typescript
// Essential metrics for Vercel Hobby + Supabase Free
interface SystemHealth {
  database: {
    size_mb: number;      // Current database size
    percent_used: number; // % of 500MB limit
    row_count: number;    // Approx. rows
    connections: number;  // Active connections
  };
  storage: {
    used_gb: number;     // Used storage
    percent_used: number; // % of 1GB limit
    file_count: number;   // Total files
  };
  updated_at: string;    // Last metrics update
}

// Alert thresholds (conservative for free tier)
const ALERTS = {
  database: {
    size_mb: 400,       // 80% of 500MB
    connections: 40     // 80% of 50
  },
  storage: {
    gb: 0.8,           // 80% of 1GB
    files: 5000        // Arbitrary high file count
  }
};
```

### **15.2 Implementation**
1. **SQL Function** (Run in Supabase SQL Editor):
   ```sql
   create or replace function get_health_metrics()
   returns json as $$
     select json_build_object(
       'size_mb', pg_database_size(current_database()) / (1024 * 1024.0),
       'row_count', (
         select sum(reltuples)::bigint
         from pg_class
         where relkind = 'r' and relname not like 'pg_%' and relname not like 'sql_%'
       ),
       'connections', (
         select count(*) from pg_stat_activity where pid <> pg_backend_pid()
       )
     );
   $$ language sql stable;
   ```

2. **Metrics Collection** (Add to existing API route):
   ```typescript
   // Example: Add to /api/admin/health
   export async function GET() {
     const { data: metrics } = await supabase.rpc('get_health_metrics');
     return Response.json({
       database: {
         size_mb: Math.round(metrics.size_mb * 100) / 100,
         percent_used: Math.min(100, Math.round((metrics.size_mb / 500) * 100)),
         row_count: metrics.row_count,
         connections: metrics.connections
       },
       storage: {
         // Implement storage metrics collection
         used_gb: 0,
         percent_used: 0,
         file_count: 0
       },
       updated_at: new Date().toISOString()
     });
   }
   ```

### **15.3 Monitoring**
- **Auto-refresh**: Every 5 minutes
- **Access Control**: Admin users only
- **Storage**: 
  - Monitor via Supabase dashboard
  - Manual checks for large files
- **API Usage**: Track in Vercel dashboard

---

## 16. Partner & Agency Portal *(Deferred Until Infrastructure Upgrade)*

> **Status:** Deferred while the platform operates on Vercel Hobby + Supabase Free. Feature set requires higher quotas, scheduled jobs, and expanded auditing available on Pro tiers.

### 16.1 Purpose & Scope
- Empower approved Catholic agencies, parish offices, and professional matchmakers to generate and manage leads for CCMatrimony.
- Support referral tracking, assisted registrations, and commission payouts while preserving user privacy and compliance.
- Provide admins with full oversight, auditability, and control over partner activity.

### 16.2 Roles & Access Model
| Role | Permissions | Notes |
|------|-------------|-------|
| **Partner / Agency / Agent** | Limited | Access only to leads created via referral link or assigned by admin; can onboard profiles, update lead status, request payouts. |
| **Admin** | Full | Manage partners, commissions, payouts, and audit logs; assign leads and override partner actions. |
| **Telecaller** | Existing | No change; remains separate from partner portal with standard CRM capabilities. |

**Authentication & Onboarding:**
- Partners are vetted offline (in-person meeting, document review handled by admin outside the system).
- Admin records partner details directly in the backend and toggles "Approved" status to grant access.
- System generates Partner ID, credentials, referral link, and QR code once admin creates the record.
- Optional lightweight interest form can capture leads for follow-up, but no sensitive docs are uploaded online.
- Partner login is isolated from admin/telecaller (separate subdomain or route namespace).

### 16.3 Core Workflows

**Lead Generation & Assignment:**
- **Referral Capture:** Users registering via partner link/QR are auto-tagged with Partner ID.
- **Manual Assignment:** Admin can assign existing leads to a partner (filters: region, parish, community).
- **Lead States:** New → Contacted → Verified → Converted → Closed. Partners can update status; transitions logged with timestamp and actor.

**Partner-Assisted Profile Creation:**
- Partner can create/edit profiles on behalf of assigned leads (photo upload, education, occupation, income).
- Mark profile as "Verified by Partner" with notes capturing offline verification details (no document upload required).
- Data scope restricted: partner sees only fields necessary for completion.

**Commission Tracking & Payouts:**
- Configurable commission triggers per partner (profile completion, paid plan, match success).
- Earnings dashboard shows: Total Earned, Pending Approval, Paid, Upcoming Payout.
- Partner can request payout once threshold met; admin reviews → approve/decline and status updates are reflected in the dashboard (no email/SMS alerts).
- Payout history includes invoice reference, date, amount, status.

**Partner Analytics & Leaderboards:**
- Metrics: Leads generated (MTD/YTD), completed registrations, paid conversions, matches, commission earned.
- Optional leaderboard highlighting top partners (configurable time window).
- Export to CSV for partner's internal reporting.
- Telecaller coordination: partner and telecaller notes share the same profile timeline; callers review activity history before outreach to avoid duplicate contact.

### 16.4 Admin Controls & Oversight
- Partner CRUD: add, edit details, suspend, reactivate, delete.
- Commission configuration per partner or per event type (flat rate or percentage).
- Lead assignment screen with filters and bulk assignment.
- Audit log for partner actions (field edits, status changes, payout requests) with timestamp and IP.
- Anomaly detection: flag heavy activity bursts (e.g., >20 profile edits/hour) and auto-notify admins for review.
- Rate limits: cap partner API usage (e.g., 60 requests/min) to prevent abuse.
- Dashboard cards: Active Partners, Leads This Month, Conversion Rate, Commission Liability.

### 16.5 Security & Compliance
- Role-based access enforced via Supabase RLS / API middleware (partners scoped to their entities).
- Mask sensitive fields (contact info) until verification stage to prevent misuse.
- Partner terms & privacy agreement stored with audit trail (version, acceptance timestamp).
- All downloads (CSV exports) logged; optional watermark with partner ID.
- GDPR/Indian Data Protection compliance: consent stored for partner-assisted registrations.
- Partner usage monitoring: weekly report highlighting anomalies, inactive partners, and escalations resolved.

### 16.6 Technical Architecture
- **Frontend:** Next.js partner portal route (`/partner/*`) with protected layouts.
- **Backend:** Supabase Postgres tables (`partners`, `partner_leads`, `partner_commissions`, `partner_audit_log`).
- **APIs:** Vercel serverless / Supabase Edge Functions for partner endpoints with JWT auth.
- **Background Jobs:** Scheduled task to reconcile commissions, send leaderboard summaries, refresh analytics materialized view.
- **Integrations:** None required initially (dashboard surfaces all status changes; no email/SMS notifications).

### 16.7 API Endpoints
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

### 16.8 UI/UX Blueprint
- **Partner Dashboard:** KPI cards (Leads This Month, Conversions, Commission Earned), quick actions (Add Lead, Request Payout).
- **Leads Page:** Table with status filters, search by name/contact, inline status updates, activity sidebar.
- **Profile Wizard:** Guided steps for photo upload, personal info, education, occupation, family, with progress tracker.
- **Earnings Page:** Graph of commissions over time, payout request CTA, payout history table.
- **Admin Partner View:** Tabs for Partners, Assignments, Commissions, Audit Log.
- Responsive design prioritizing desktop/tablet; future mobile experience in roadmap.

**Operational Runbook Notes:**
- If misuse or anomaly detected → auto-flag partner → admin reviews log → suspend or warn → document outcome in audit log.
- Payout dispute: mark disputed status, freeze further payouts, notify finance, resolve, then unlock.
- Inactive partner (>60 days) → send re-engagement email → consider suspension if no response.

### 16.9 Roadmap & Phasing
| Phase | Deliverables |
|-------|--------------|
| **P3 (Post-MVP)** | Partner onboarding, referral tracking, basic lead dashboard, profile assistance, simple commission payouts. |
| **P4 (Enhancements)** | Automated regional assignment, advanced commission rules, leaderboard, payout integrations. |
| **P5 (Deferred)** | External partner API, marketplace listing, SLA tracking, mobile partner app. |

### 16.10 Success Metrics
- Active partners onboarded per quarter.
- Lead-to-registration conversion rate for partner referrals.
- Paid conversions attributable to partners.
- Commission payout cycle time and volume.
- Regional penetration attributed to partner network.

### 16.11 Data Model Preview (Draft)
| Table | Key Fields | Notes |
|-------|------------|-------|
| `partners` | `id`, `name`, `region`, `status`, `commission_config`, `created_at`, `approved_by` | Stores partner account and configuration. |
| `partner_users` | `id`, `partner_id`, `auth_user_id`, `role`, `last_login` | Portal users tied to partner; supports multi-user agencies. |
| `partner_leads` | `id`, `partner_id`, `user_id`, `source`, `status`, `notes`, `last_action_at` | Tracks lead lifecycle and shared notes for partner/telecaller. |
| `partner_commissions` | `id`, `partner_id`, `event_type`, `amount`, `status`, `payout_id`, `earned_at` | Commission events and payout linkage. |
| `partner_audit_log` | `id`, `partner_id`, `actor`, `action`, `payload`, `created_at`, `ip_address` | Immutable log of partner/admin actions. |

### 16.12 Open Questions
- Do partners require multilingual support or localized pricing?
- Should partner payouts integrate with external accounting/payroll systems?
- Is there a need for partner training/certification modules within the portal?
- What SLA targets should partners meet (response time, verification accuracy)?

---

**Last Updated:** November 6, 2025  
**Document Version:** 3.1 (Merged Partner/Agency Spec)  
**MVP Roles:** Super Admin only  
**Future Roles:** Admin, Telecaller (Week 6+ when hiring)  
**Partner Portal:** Deferred until infrastructure upgrade (Pro tiers)  
**Next Review:** Post-launch (January 2026)
