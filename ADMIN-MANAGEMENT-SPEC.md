# CC Matrimony Admin Management Specification

## 1. Purpose & Scope
- Provide easy-to-use admin interface for non-technical users
- Enable efficient management of users, profiles, and platform operations
- Support community-focused management for Tulunadu region
- Ensure platform safety and quality through effective moderation
- Platform: CC Matrimony (matri.naveevo.com)
- Tech Stack: Vercel Hobby + Supabase Free

## 2. Admin Roles & Permissions

> **MVP Implementation**: Single Super Admin role only. Admin and Telecaller roles deferred to Week 6+ when hiring additional staff.

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
  categories: {
    'Bunt Community': SuccessStory[];
    'Mangalorean Christian': SuccessStory[];
    'Regional Stories': SuccessStory[];
    'Traditional Marriages': SuccessStory[];
  };
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
  communities: {
    'Bunt': CommunityStats;
    'Billava': CommunityStats;
    'Mangalorean Christian': CommunityStats;
    'Devadiga': CommunityStats;
  };
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

**Last Updated:** November 6, 2025  
**Document Version:** 3.0 (Finalized - Simplified Roles)  
**MVP Roles:** Super Admin only  
**Future Roles:** Admin, Telecaller (Week 6+ when hiring)  
**Next Review:** Post-launch (January 2026)
