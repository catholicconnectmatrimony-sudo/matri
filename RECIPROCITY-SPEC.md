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

| Plan | Photo Limit | Reciprocity | Features |
|------|-------------|-------------|----------|
| Free | 2 photos | Enforced | Basic search, limited daily views |
| Paid | 5 photos | Enforced | All free features + unlimited views, advanced search |
| VIP | 10 photos | Bypassed | All paid features + priority support, matchmaking |

- **Reciprocity**: Enforced for free/paid, bypassed for VIP
- **Admin Override**: Available for all plans in special cases

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

## 7. Technical Enforcement (Vercel + Supabase)

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
