# CC Matrimony Reciprocity Specification

Goal: keep MVP enforcement extremely light so Cursor (or any AI co-pilot) can ship Week 5 in a single slice. All richer reciprocity behaviour is documented as Phase 2+ ideas only for future planning.

---

## 1. MVP Scope (Week 5 only)
- Enforce **one rule**: viewers must have **at least one approved photo** to see other members' photos.
- "Approved" = `status === 'approved'` in the `photos` table. MVP auto-approves every upload, so this check passes immediately on upload and fails instantly on delete.
- Zero grace periods, bundles, overrides, or admin toggles.
- No extra tables. Use the existing `photos` table and a single count query.

| Bundle / Field | MVP Behaviour | Notes |
|----------------|---------------|-------|
| Photos | Locked until viewer has at least one approved photo | Prompt: "Upload a photo to view others." |
| All other fields (education, occupation, income, family, horoscope, contact) | **Always visible** | Reciprocity deferred to Phase 2. |

---

## 2. Enforcement Flow
1. User requests another member's photos.
2. API counts the viewer's approved photos (`status = 'approved'`).
3. `count > 0` -> return target photos, respecting owner privacy settings.
4. `count === 0` -> return locked payload with CTA to upload.
5. Deleting a photo triggers the same check on the next view; access is lost immediately.

There is purposely **no grace window** and **no moderation delay** in MVP. Everything happens deterministically per request.

---

## 3. Minimal API Sketch
```typescript
// app/api/profiles/[profileId]/photos/route.ts
export async function GET(_: Request, { params }: { params: { profileId: string } }) {
  const viewer = await requireAuth(); // throws if not authenticated

  const { count, error } = await supabase
    .from('photos')
    .select('id', { count: 'exact', head: true })
    .eq('profile_id', viewer.id)
    .eq('status', 'approved');

  if (error) {
    console.error('photo gate check failed', error);
    throw new Error('Photo gate check failed');
  }

  if (!count || count === 0) {
    return NextResponse.json(
      {
        locked: true,
        message: 'Upload at least one photo to view this profile.',
      },
      { status: 403 },
    );
  }

  const photos = await getVisiblePhotos(params.profileId, viewer.id);
  return NextResponse.json({ locked: false, photos });
}
```

Helper guard for client components:
```typescript
export async function ensureCanViewPhotos(profileId: string) {
  const response = await fetch(`/api/profiles/${profileId}/photos`);
  if (response.status === 403) {
    const payload = await response.json();
    return payload; // { locked: true, message: string }
  }
  return response.json(); // { locked: false, photos: Photo[] }
}
```

---

## 4. UI Copy (MVP)
- Locked card text: "Upload a photo to view others."
- Button CTA: "Upload photo"
- Success toast after upload: "You can now view profile photos."
- This single message is reused on profile cards, modals, and detail pages for consistency.

---

## 5. Testing Checklist
- User with zero photos sees lock state everywhere photos would render.
- Uploading a photo changes the state immediately (same session, refetch required if using React Query).
- Deleting the only photo causes the next photo view attempt to lock again.
- API logs (Sentry/PostHog) confirm when the gate denies access (event name `reciprocity_photo_gate_locked`).

---

## 6. Phase 2+ Backlog (Documentation Only)
These items stay documented for future planning but **must not** creep into MVP implementation:
- Multi-bundle reciprocity (education, occupation, income, family).
- Grace periods, progressive unlocks, mutual-interest relaxations.
- Admin toggles, overrides, dashboards, and cron recalculations.
- Dedicated `reciprocity_state` / `reciprocity_config` tables.
- Partial reveals (blurred photos, count matching), A/B prompt tests.

Keep this section as a reminder, not a requirement for Week 5.

---

**Last updated:** November 12, 2025 (MVP simplification pass)
