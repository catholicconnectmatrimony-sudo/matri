# CC Matrimony Reciprocity Engine Specification

## 1. Purpose & Scope
- Enforce reciprocal visibility so members can only view sensitive profile fields after sharing equivalent details.
- Apply consistent rules across web and mobile experiences.
- Support plan-based overrides, grace periods, and admin-configurable modes.
- Key differentiator for CC Matrimony platform.

## 2. Field Coverage
| Bundle / Field | Reciprocity Required? | Notes |
|----------------|-----------------------|-------|
| Photos | Yes (count-based) | Users see up to the number of photos they upload. Admin can cap photos per plan. |
| Education | Yes | Text fields plus dropdown selections count as provided once completed. |
| Occupation | Yes | Job title + sector combination must be filled. |
| Income | Yes | Annual income bracket selection required. |
| Family Details | Yes (bundle) | Requires Father + Mother + Siblings info. Partial completion shows a prompt. |
| Horoscope | Optional | Only enforced for members who enable horoscope sharing. |
| Contact Details | No | Managed separately via plan limits and privacy controls. |
| Basic Info & Bio | No | Always visible. |

## 3. Reciprocity Mode

### Single Gradual Mode
- **Enforcement Behavior**: Grace period applies, then strict visibility until reciprocity is met.
- **Relaxation Trigger**: Mutual interest or premium upgrade unlocks additional fields.
- **Grace Period**: 24 hours after first login **and** at least 5 profile views (whichever occurs later).
- **Admin Override**: Available for support cases with audit logging.

## 4. Plan Structure

| Plan | Photo Slots | Reciprocity | Features |
|------|-------------|-------------|----------|
| Free | 1 profile + 1 album | Enforced | Basic search, 10 daily views |
| Paid | 1 profile + 9 album + 3 family/group | Admin-configurable (default OFF) | All free + unlimited views, advanced search |

- **Reciprocity**: Enforced for Free by default. Paid tiers start with reciprocity disabled but can be enabled per plan from the admin dashboard.
- **Admin Override**: Available for all plans in special cases (single member or entire tier).

### Photo Slot Details
- **Profile Photo**: Required primary image shown in listings (1 slot all plans)
- **Album Photos**: Showcase gallery (Free: 1, Paid: 9)
- **Family / Group Photos**: Separate bucket for family or group pictures (Paid only: 3 slots)
- **Formats & Size**: PNG, GIF, JPG, JPEG, WebP up to 30 MB per upload
- **Compression**: Client-side compression from Week 3; optional server-side Sharp pipeline converts to WebP (fallback JPEG 85) and can cap dimensions at 1920×1920 to save bandwidth.

## 5. UX Flows
### 5.1 Locked State Indicators
- Show lock icon with tooltip: "Add your [field] to unlock others' [field]."
- Include count reminders for photos: "Upload 2 more photos to view 2 more."

### 5.2 Prompts & Alerts
- **Attempted view without reciprocity** → "Share your occupation to view theirs." (CTA: Edit profile)
- **Grace period expiring (1 hour left)** → "Reciprocity kicks in soon—complete your details to keep full access." (CTA: Open completion checklist)
- **Post mutual interest (gradual mode)** → "You've matched! You now get limited access to hidden photos." (Informational toast)

### 5.3 Privacy Dashboard
- Display per-field status: Shared / Locked / Premium Only.
- Provide quick edit links and reciprocity progress bar.
- Photo deletions immediately re-check reciprocity; if approved photo count drops below matched users' photo count, viewing locks until restored. Restoring the minimum photo count re-opens access instantly.
- When the grace period ends, show a banner toast (“Reciprocity now active”) and enforce restrictions on the next navigation to avoid jarring interrupts.
- When another member has more photos than the viewer, expose photos in chronological upload order up to the viewer's contribution count.
- Photo privacy tiers (e.g., family-only album) remain respected alongside reciprocity; conservative users can limit visibility without bypassing reciprocity checks.

## 6. Admin Controls
- Global reciprocity toggle (ON/OFF) with confirmation modal; OFF state logs admin ID + reason and bypasses enforcement for all users.
- Toggle mode (Lenient / Strict / Gradual) globally or per plan.
- Configure grace period duration (hours) and view threshold.
- Override reciprocity for specific members (e.g., customer support cases) with audit log entry.
- Trigger on-demand eligibility recalculation for a member via admin action when discrepancies are reported.
- Admin dashboard module lists the 20 most recent global toggles and recalculations (timestamp, admin, note) for quick oversight.

## 7. Technical Enforcement (Vercel + Supabase + Cloudflare R2)

### **7.1 Database Schema**
```sql
-- Reciprocity state tracking
CREATE TABLE reciprocity_state (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  field_bundle VARCHAR(50) NOT NULL, -- photos, education, occupation, income, family
  is_eligible BOOLEAN DEFAULT FALSE,
  last_calculated TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, field_bundle)
);

-- Reciprocity configuration
CREATE TABLE reciprocity_config (
  id SERIAL PRIMARY KEY,
  plan_type VARCHAR(20) NOT NULL,
  field_bundle VARCHAR(50) NOT NULL,
  is_enabled BOOLEAN DEFAULT TRUE,
  grace_period_days INTEGER DEFAULT 7,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### **7.2 Implementation Details**
- **Database**: Supabase PostgreSQL with full control and 500MB limit
- **Middleware**: Supabase Edge Functions check reciprocity before returning protected fields
- **Real-time Updates**: Supabase triggers automatically recompute reciprocity state on profile updates
- **Caching**: In-memory caching for reciprocity states to improve performance
- **Background Jobs**: Vercel Cron Jobs (≤12) for daily reciprocity recalculation and cleanup
- **Admin Tools**: Full admin panel with reciprocity management and override capabilities
- **Media Storage**: Photos stored in Cloudflare R2 (private bucket). Access is via time-limited signed URLs generated by a Vercel API route. Database stores only `storage_key` for provider-agnostic access.

### **7.3 API Enforcement Example**

```typescript
export async function GET(request: Request) {
  const viewerId = await getUserIdOrThrow(request);
  const profileId = new URL(request.url).searchParams.get('id');
  if (!profileId) {
    return NextResponse.json({ error: 'Missing profile id' }, { status: 400 });
  }

  const canView = await checkReciprocity({
    viewerId,
    profileId,
    fieldBundle: 'education'
  });

  if (!canView) {
    return NextResponse.json({
      locked: true,
      message: 'Add your education details to unlock theirs.'
    }, { status: 403 });
  }

  const profile = await getProfileWithEducation(profileId);
  return NextResponse.json(profile);
}
```

## 8. Audit & Logging
- Log every reciprocity denial with user ID, bundle, timestamp, and reason.
- Admin overrides recorded with admin ID and justification.
- On-demand reporting: admin can export recent denials via SQL/dashboard when investigating UX issues.

## 9. Future Enhancements
- A/B test prompt copy and timing to boost reciprocity completion.
- Prototype partial reveals (blurred photos, summary income band) post-mutual interest.

---

## 10. AI Implementation Guide (Week 5)

### **Implementation Order for AI Development:**

**Day 22-23: Database Setup**
```sql
-- Step 1: Create reciprocity_state table
CREATE TABLE reciprocity_state (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  field_bundle VARCHAR(50) NOT NULL,
  is_eligible BOOLEAN DEFAULT FALSE,
  last_calculated TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, field_bundle)
);

-- Step 2: Create helper function
CREATE OR REPLACE FUNCTION check_reciprocity(
  viewer_id UUID,
  target_id UUID,
  bundle VARCHAR(50)
) RETURNS BOOLEAN AS $$
BEGIN
  -- Check if viewer has shared the field
  RETURN EXISTS (
    SELECT 1 FROM reciprocity_state
    WHERE user_id = viewer_id
    AND field_bundle = bundle
    AND is_eligible = true
  );
END;
$$ LANGUAGE plpgsql;
```

**Day 24: Build Calculation Functions**
```typescript
// lib/reciprocity/calculator.ts
export function hasEducation(profile: Profile): boolean {
  return !!(profile.education_level && profile.education_field);
}

export function hasOccupation(profile: Profile): boolean {
  return !!(profile.occupation_sector && profile.job_title);
}

export function hasIncome(profile: Profile): boolean {
  return !!profile.annual_income;
}

export function hasFamilyDetails(profile: Profile): boolean {
  return !!(
    profile.father_name && 
    profile.mother_name && 
    profile.siblings_count !== null
  );
}

export async function updateReciprocityState(userId: string) {
  const profile = await getProfile(userId);
  
  await supabase.from('reciprocity_state').upsert([
    { user_id: userId, field_bundle: 'education', is_eligible: hasEducation(profile) },
    { user_id: userId, field_bundle: 'occupation', is_eligible: hasOccupation(profile) },
    { user_id: userId, field_bundle: 'income', is_eligible: hasIncome(profile) },
    { user_id: userId, field_bundle: 'family', is_eligible: hasFamilyDetails(profile) }
  ]);
}
```

**Day 25: Build UI Components**
```typescript
// components/profile/LockedField.tsx
interface LockedFieldProps {
  fieldName: string;
  canView: boolean;
  value?: string;
  onUnlock: () => void;
}

export function LockedField({ fieldName, canView, value, onUnlock }: LockedFieldProps) {
  if (canView) {
    return <div className="text-gray-900">{value}</div>;
  }
  
  return (
    <div className="flex items-center gap-2 text-gray-400">
      <LockIcon className="w-4 h-4" />
      <span>Add your {fieldName} to unlock</span>
      <Button onClick={onUnlock} variant="link">Edit Profile</Button>
    </div>
  );
}
```

**Day 26-28: Grace Period + Testing**
```typescript
// lib/reciprocity/grace-period.ts
export async function isInGracePeriod(userId: string): Promise<boolean> {
  const user = await getUser(userId);
  const hoursSinceCreation = (Date.now() - user.created_at.getTime()) / (1000 * 60 * 60);
  
  if (hoursSinceCreation < 24) return true;
  
  const viewCount = await getProfileViewCount(userId);
  if (viewCount < 5) return true;
  
  return false;
}

export async function canViewField(
  viewerId: string,
  targetId: string,
  fieldBundle: string
): Promise<boolean> {
  // Check grace period
  if (await isInGracePeriod(viewerId)) return true;
  
  // Check premium status
  const viewer = await getUser(viewerId);
  if (viewer.plan_type !== 'free') return true;
  
  // Check reciprocity
  const { data } = await supabase.rpc('check_reciprocity', {
    viewer_id: viewerId,
    target_id: targetId,
    bundle: fieldBundle
  });
  
  return data;
}
```

### **Testing Checklist:**
- [ ] User in grace period can view all fields
- [ ] User after grace period sees locked fields
- [ ] Adding education unlocks education on other profiles
- [ ] Premium users bypass reciprocity
- [ ] Admin can override reciprocity for specific users
- [ ] Reciprocity state updates when profile is edited

### **Common AI Mistakes to Avoid:**
1. ❌ Don't implement all field bundles at once → Start with education only
2. ❌ Don't build complex grace period logic first → Start with simple time-based check
3. ❌ Don't create elaborate UI → Start with simple lock icon + tooltip
4. ❌ Don't optimize prematurely → Make it work, then optimize

### **Defer to Post-Launch:**
- Photo count-based reciprocity (complex logic)
- Horoscope reciprocity (optional feature)
- Advanced grace period rules
- A/B testing of prompts
- Partial field reveals (blurred content)
