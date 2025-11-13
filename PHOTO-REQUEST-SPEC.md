# CC Matrimony Photo Request Specification

## 1. Purpose

Simple feature to allow users to request photos from profiles that don't have any photos uploaded yet. This encourages profile completion and improves user engagement.

---

## 2. MVP Scope

- Show "Request Photo" button when viewing a profile that has **zero approved photos**
- When clicked, send a photo request notification to the profile owner
- Track photo requests in database (prevent duplicate requests)
- Simple notification system (email + in-app)

---

## 3. User Flow

### **3.1 Requesting Photos**

1. User A views User B's profile
2. System checks if User B has any approved photos
3. If **zero photos** → Show "Request Photo" button
4. If **has photos** → Show photos (respecting privacy settings)
5. User A clicks "Request Photo"
6. System checks if request already exists
7. If new request → Create record + send notification
8. If duplicate → Show "Request already sent" message

### **3.2 Receiving Requests**

1. User B receives notification (email + in-app)
2. Notification shows: "X requested you to upload photos"
3. User B can click notification → Go to profile upload page
4. Once User B uploads first photo → All pending requests are considered fulfilled

---

## 4. Database Schema

```sql
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

**Notes:**
- `UNIQUE(requester_id, profile_id)` prevents duplicate requests
- Indexes optimize queries for "get requests for profile" and "get requests by requester"
- Cascade delete ensures cleanup when profiles are deleted

---

## 5. API Endpoints

> **See [TECH-STACK.md](./TECH-STACK.md) section 19 for complete API routes reference.**

**Photo Request Endpoints:**
- `POST /api/profiles/[profileId]/photos/request` - Send photo request
- `GET /api/profiles/[profileId]/photos/request-status` - Check if user has requested
- `GET /api/photos/requests` - Get all photo requests (for profile owner)

**Request/Response formats:**
- Success: `{ success: true, message: "Photo request sent" }`
- Error: `{ error: "Profile already has photos" | "Request already sent" }`
- Status: `{ hasRequested: boolean, requestedAt: string | null }`

---

## 6. UI Components

### **6.1 Profile View - No Photos State**

```tsx
{profile.photos.length === 0 && (
  <div className="photo-request-section">
    <div className="empty-state">
      <PhotoIcon />
      <p>No photos uploaded yet</p>
      {!hasRequested ? (
        <Button onClick={handleRequestPhoto}>
          Request Photo
        </Button>
      ) : (
        <div className="request-sent">
          <CheckIcon />
          <span>Photo request sent</span>
        </div>
      )}
    </div>
  </div>
)}
```

### **6.2 Notification Badge**

Show badge count on notifications icon:
```tsx
<Badge count={photoRequestCount}>
  <BellIcon />
</Badge>
```

### **6.3 Notification Item**

```tsx
<div className="notification-item">
  <Avatar src={requester.photo} />
  <div>
    <p>
      <strong>{requester.name}</strong> requested you to upload photos
    </p>
    <span className="time">{formatTime(requestedAt)}</span>
  </div>
  <Button onClick={goToUploadPhotos}>Upload Photos</Button>
</div>
```

---

## 7. Notification System

### **7.1 Email Notification**

**Subject:** "Someone requested you to upload photos"

**Template:**
```
Hi [Name],

[Requester Name] (Profile ID: [CCM001234]) has requested you to upload photos to your profile.

Uploading photos helps you:
- Get more profile views
- Receive more interests
- Increase your match potential

[Upload Photos Button]

Best regards,
CC Matrimony Team
```

### **7.2 In-App Notification**

- Show in notifications dropdown
- Link to photo upload page
- Mark as read when user uploads first photo

---

## 8. Business Rules

1. **Request Eligibility:**
   - Profile must have **zero approved photos**
   - Requester cannot be the profile owner
   - One request per requester-profile pair (enforced by UNIQUE constraint)

2. **Request Visibility:**
   - Only profile owner can see who requested photos
   - Requesters cannot see other requesters

3. **Request Fulfillment:**
   - When profile owner uploads first approved photo → All requests are considered fulfilled
   - No need to track "fulfilled" status (can be inferred from photo count)

4. **Request Limits:**
   - **No limits** in MVP (users can request from unlimited profiles)
   - Future: Consider daily limits if spam becomes an issue

---

## 9. Implementation Checklist

### **Backend:**
- [ ] Create `photo_requests` table migration
- [ ] Implement `POST /api/profiles/[profileId]/photos/request` endpoint
- [ ] Implement `GET /api/profiles/[profileId]/photos/request-status` endpoint
- [ ] Implement `GET /api/photos/requests` endpoint (for profile owner)
- [ ] Add email notification on request
- [ ] Add in-app notification on request

### **Frontend:**
- [ ] Add "Request Photo" button component
- [ ] Show button only when profile has zero photos
- [ ] Handle request state (pending/sent)
- [ ] Add notification badge for photo requests
- [ ] Add notification item in notifications dropdown
- [ ] Link notification to photo upload page

### **Testing:**
- [ ] User can request photo from profile without photos
- [ ] Duplicate requests are prevented
- [ ] Request button disappears after sending request
- [ ] Profile owner receives email notification
- [ ] Profile owner sees request in notifications
- [ ] Request is no longer shown after profile uploads first photo

---

## 10. Future Enhancements (Phase 2+)

- **Request Analytics:** Track how many requests lead to photo uploads
- **Bulk Requests:** Allow users to request photos from multiple profiles at once
- **Request Reminders:** Send follow-up email if no photo uploaded after 7 days
- **Request Limits:** Add daily limits for free users (e.g., 10 requests/day)
- **Request History:** Show history of all photo requests sent/received
- **Auto-Request:** Option to automatically request photos when viewing profiles without photos

---

**Last Updated:** January 2025  
**Status:** MVP Specification - Ready for Implementation

