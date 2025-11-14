# CC Matrimony - Development Patterns & Quick Reference

> **Purpose**: Common patterns, error handling, and best practices for AI-driven development. Use this as a reference when building features.

---

## 🔐 AUTHENTICATION PATTERNS

### **Get Current User (Server Component)**
```typescript
// app/dashboard/page.tsx
import { createClient } from '@/lib/supabase/server'

export default async function DashboardPage() {
  const supabase = createClient()
  const { data: { user }, error } = await supabase.auth.getUser()
  
  if (error || !user) {
    redirect('/login')
  }
  
  return <div>Welcome {user.email}</div>
}
```

### **Get Current User (Client Component)**
```typescript
// components/ProfileForm.tsx
'use client'
import { useUser } from '@/hooks/useUser'

export function ProfileForm() {
  const { user, loading } = useUser()
  
  if (loading) return <div>Loading...</div>
  if (!user) redirect('/login')
  
  return <form>...</form>
}
```

### **Protected API Route**
```typescript
// app/api/profiles/route.ts
import { createClient } from '@/lib/supabase/server'
import { NextResponse } from 'next/server'

export async function POST(request: Request) {
  const supabase = createClient()
  const { data: { user }, error } = await supabase.auth.getUser()
  
  if (error || !user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  // Your logic here
  return NextResponse.json({ success: true })
}
```

---

## 🗄️ DATABASE PATTERNS

### **Query with Error Handling**
```typescript
const { data, error } = await supabase
  .from('profiles')
  .select('*')
  .eq('user_id', userId)
  .single()

if (error) {
  console.error('Database error:', error)
  return NextResponse.json({ error: 'Failed to fetch profile' }, { status: 500 })
}

if (!data) {
  return NextResponse.json({ error: 'Profile not found' }, { status: 404 })
}

return NextResponse.json({ data })
```

### **Insert with Validation**
```typescript
const { data, error } = await supabase
  .from('profiles')
  .insert({
    user_id: userId,
    full_name: body.name,
    age: body.age,
    gender: body.gender,
    community: body.community
  })
  .select()
  .single()

if (error) {
  // Handle unique constraint violations
  if (error.code === '23505') {
    return NextResponse.json({ error: 'Profile already exists' }, { status: 409 })
  }
  return NextResponse.json({ error: 'Failed to create profile' }, { status: 500 })
}
```

### **Update with Optimistic Locking**
```typescript
const { data, error } = await supabase
  .from('profiles')
  .update({ 
    full_name: body.name,
    updated_at: new Date().toISOString()
  })
  .eq('id', profileId)
  .eq('user_id', userId) // Ensure user owns this profile
  .select()
  .single()

if (error) {
  return NextResponse.json({ error: 'Failed to update' }, { status: 500 })
}
```

---

## 📤 FILE UPLOAD PATTERNS (Cloudinary)

### **Client-Side Upload Flow**
```typescript
// components/PhotoUpload.tsx
'use client'
import { useState } from 'react'
export function PhotoUpload({ onUpload }: { onUpload: (publicId: string) => void }) {
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const handleUpload = async (file: File) => {
    setLoading(true)
    setError(null)

    try {
      const formData = new FormData()
      formData.append('photo', file, file.name)

      const response = await fetch('/api/photos/upload', {
        method: 'POST',
        body: formData
      })

      if (!response.ok) {
        const result = await response.json()
        throw new Error(result.error || 'Upload failed')
      }

      const { public_id } = await response.json()
      onUpload(public_id)
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Upload failed')
    } finally {
      setLoading(false)
    }
  }

  return (
    <div>
      <input
        type="file"
        accept="image/*"
        onChange={(e) => e.target.files?.[0] && handleUpload(e.target.files[0])}
        disabled={loading}
      />
      {error && <div className="text-red-500">{error}</div>}
      {loading && <div>Uploading...</div>}
    </div>
  )
}
```

### **Server-Side Upload Handler**
```typescript
// app/api/photos/upload/route.ts
import { v2 as cloudinary } from 'cloudinary'
import { createClient } from '@/lib/supabase/server'
import { NextResponse } from 'next/server'

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME!,
  api_key: process.env.CLOUDINARY_API_KEY!,
  api_secret: process.env.CLOUDINARY_API_SECRET!,
})

export async function POST(request: Request) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  try {
    const formData = await request.formData()
    const file = formData.get('photo') as File
    
    if (!file) {
      return NextResponse.json({ error: 'No file provided' }, { status: 400 })
    }

    const buffer = Buffer.from(await file.arrayBuffer())

    const uploadResult = await new Promise<cloudinary.UploadApiResponse>((resolve, reject) => {
      const stream = cloudinary.uploader.upload_stream({
        folder: `profiles/${user.id}`,
        overwrite: false,
        resource_type: 'image',
      }, (error, result) => {
        if (error) return reject(error)
        if (!result) return reject(new Error('No result from Cloudinary'))
        resolve(result)
      })
      stream.end(buffer)
    })

    const { public_id, bytes, format, secure_url } = uploadResult

    // Save metadata to database
    const { data: photo, error } = await supabase
      .from('photos')
      .insert({
        profile_id: user.id, // Assuming profile_id = user_id for MVP
        filename: file.name,
        original_name: file.name,
        file_size: bytes,
        mime_type: `image/${format}`,
        public_id,
        is_approved: true, // Auto-approve for MVP
        is_primary: false,
      })
      .select()
      .single()

    if (error) {
      return NextResponse.json({ error: 'Failed to save photo metadata' }, { status: 500 })
    }

    return NextResponse.json({ 
      id: photo.id,
      public_id,
      url: secure_url
    })
  } catch (error) {
    console.error('Upload error:', error)
    return NextResponse.json({ error: 'Upload failed' }, { status: 500 })
  }
}
```

---

## 🔄 STATE MANAGEMENT PATTERNS

### **TanStack Query (Data Fetching)**
```typescript
// hooks/useProfiles.ts
import { useQuery } from '@tanstack/react-query'
import { createClient } from '@/lib/supabase/client'

export function useProfiles(filters: SearchFilters) {
  return useQuery({
    queryKey: ['profiles', filters],
    queryFn: async () => {
      const supabase = createClient()
      const { data, error } = await supabase
        .from('profiles')
        .select('*')
        .eq('religion', filters.religion)
        .eq('community', filters.community)
        .gte('age', filters.minAge)
        .lte('age', filters.maxAge)
      
      if (error) throw error
      return data
    }
  })
}
```

### **Zustand (Client State)**
```typescript
// stores/searchStore.ts
import { create } from 'zustand'

interface SearchState {
  filters: SearchFilters
  setFilters: (filters: Partial<SearchFilters>) => void
  resetFilters: () => void
}

export const useSearchStore = create<SearchState>((set) => ({
  filters: {
    religion: '',
    community: '',
    minAge: 18,
    maxAge: 35
  },
  setFilters: (newFilters) => set((state) => ({
    filters: { ...state.filters, ...newFilters }
  })),
  resetFilters: () => set({ filters: { religion: '', community: '', minAge: 18, maxAge: 35 } })
}))
```

---

## ⚠️ ERROR HANDLING PATTERNS

### **API Route Error Handling**
```typescript
// app/api/profiles/route.ts
import { NextResponse } from 'next/server'

export async function POST(request: Request) {
  try {
    // 1. Validate input
    const body = await request.json()
    if (!body.name || !body.age) {
      return NextResponse.json(
        { error: 'Name and age are required' },
        { status: 400 }
      )
    }

    // 2. Check auth
    const supabase = createClient()
    const { data: { user } } = await supabase.auth.getUser()
    if (!user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
    }

    // 3. Business logic validation
    if (body.age < 18) {
      return NextResponse.json(
        { error: 'Age must be 18 or older' },
        { status: 400 }
      )
    }

    // 4. Database operation
    const { data, error } = await supabase
      .from('profiles')
      .insert({ ...body, user_id: user.id })
      .select()
      .single()

    if (error) {
      console.error('Database error:', error)
      
      // Handle specific errors
      if (error.code === '23505') {
        return NextResponse.json(
          { error: 'Profile already exists' },
          { status: 409 }
        )
      }
      
      return NextResponse.json(
        { error: 'Failed to create profile' },
        { status: 500 }
      )
    }

    return NextResponse.json({ data }, { status: 201 })
  } catch (error) {
    console.error('Unexpected error:', error)
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}
```

### **Component Error Handling**
```typescript
// components/ProfileForm.tsx
'use client'
import { useState } from 'react'
import { Alert, AlertDescription } from '@/components/ui/alert'

export function ProfileForm() {
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const [success, setSuccess] = useState(false)

  const handleSubmit = async (data: ProfileData) => {
    setLoading(true)
    setError(null)
    setSuccess(false)

    try {
      const response = await fetch('/api/profiles', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      })

      const result = await response.json()

      if (!response.ok) {
        throw new Error(result.error || 'Failed to save profile')
      }

      setSuccess(true)
      // Redirect or update UI
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred')
    } finally {
      setLoading(false)
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {error && (
        <Alert variant="destructive">
          <AlertDescription>{error}</AlertDescription>
        </Alert>
      )}
      {success && (
        <Alert>
          <AlertDescription>Profile saved successfully!</AlertDescription>
        </Alert>
      )}
      {/* Form fields */}
      <Button type="submit" disabled={loading}>
        {loading ? 'Saving...' : 'Save Profile'}
      </Button>
    </form>
  )
}
```

---

## 📷 PHOTO REQUEST PATTERN

### **Request Photo from Profile**
```typescript
// lib/photo-requests/request-photo.ts
import { createClient } from '@/lib/supabase/server'

export async function requestPhoto(requesterId: string, profileId: string): Promise<{ success: boolean; error?: string }> {
  const supabase = createClient()
  
  // Check if profile has any photos
  const { data: photos, error: photosError } = await supabase
    .from('photos')
    .select('id')
    .eq('profile_id', profileId)
    .eq('is_approved', true)
    .limit(1)

  if (photosError) {
    console.error('Photo check error:', photosError)
    return { success: false, error: 'Failed to check photos' }
  }

  // If profile already has photos, don't allow request
  if (photos && photos.length > 0) {
    return { success: false, error: 'Profile already has photos' }
  }

  // Check if request already exists
  const { data: existingRequest } = await supabase
    .from('photo_requests')
    .select('id')
    .eq('requester_id', requesterId)
    .eq('profile_id', profileId)
    .single()

  if (existingRequest) {
    return { success: false, error: 'Request already sent' }
  }

  // Create photo request
  const { error } = await supabase
    .from('photo_requests')
    .insert({
      requester_id: requesterId,
      profile_id: profileId
    })

  if (error) {
    console.error('Photo request error:', error)
    return { success: false, error: 'Failed to send request' }
  }

  return { success: true }
}
```

### **Use in API Route**
```typescript
// app/api/profiles/[profileId]/photos/request/route.ts
export async function POST(
  request: Request,
  { params }: { params: { profileId: string } }
) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // Get requester's profile ID
  const { data: requesterProfile } = await supabase
    .from('profiles')
    .select('id')
    .eq('user_id', user.id)
    .single()

  if (!requesterProfile) {
    return NextResponse.json({ error: 'Profile not found' }, { status: 404 })
  }

  const result = await requestPhoto(requesterProfile.id, params.profileId)
  
  if (!result.success) {
    return NextResponse.json({ error: result.error }, { status: 400 })
  }

  // TODO: Send notification to profile owner
  // await sendPhotoRequestNotification(params.profileId, requesterProfile.id)

  return NextResponse.json({ success: true, message: 'Photo request sent' })
}
```

---

## 💳 PAYMENT PATTERNS (Razorpay)

### **Create Order**
```typescript
// app/api/payments/create-order/route.ts
import Razorpay from 'razorpay'
import { NextResponse } from 'next/server'

const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY_ID!,
  key_secret: process.env.RAZORPAY_KEY_SECRET!,
})

export async function POST(request: Request) {
  const { planType, amount } = await request.json()
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  try {
    const order = await razorpay.orders.create({
      amount: amount * 100, // Convert to paise
      currency: 'INR',
      receipt: `order_${user.id}_${Date.now()}`,
      notes: {
        user_id: user.id,
        plan_type: planType
      }
    })

    return NextResponse.json({ orderId: order.id })
  } catch (error) {
    console.error('Razorpay error:', error)
    return NextResponse.json({ error: 'Payment failed' }, { status: 500 })
  }
}
```

### **Verify Webhook**
```typescript
// app/api/payments/webhook/route.ts
import crypto from 'crypto'

export async function POST(request: Request) {
  const body = await request.text()
  const signature = request.headers.get('x-razorpay-signature')

  const expectedSignature = crypto
    .createHmac('sha256', process.env.RAZORPAY_KEY_SECRET!)
    .update(body)
    .digest('hex')

  if (signature !== expectedSignature) {
    return NextResponse.json({ error: 'Invalid signature' }, { status: 401 })
  }

  const event = JSON.parse(body)
  
  if (event.event === 'payment.captured') {
    // Activate subscription
    const { data } = event.payload.payment.entity
    await activateSubscription(data.notes.user_id, data.notes.plan_type)
  }

  return NextResponse.json({ received: true })
}
```

---

## 📧 EMAIL PATTERNS (Resend)
> Auth emails (verification, password reset, magic links) are sent by Supabase's built-in mailer. Do not duplicate these with Resend. Use Resend for all non-auth notifications (interests, reminders, digests, admin alerts).

### **Alternative: Use Supabase generateLink + Resend API for auth emails**

If you prefer Resend for auth emails, do not configure it as SMTP (Resend is API-only). Instead, generate the secure link in Supabase and send via Resend:

```typescript
// app/api/auth/send-email/route.ts
import { createClient } from '@supabase/supabase-js'
import { Resend } from 'resend'

const supabase = createClient(process.env.NEXT_PUBLIC_SUPABASE_URL!, process.env.SUPABASE_SERVICE_ROLE_KEY!)
const resend = new Resend(process.env.RESEND_API_KEY!)

export async function POST(req: Request) {
  const { email, type, redirectTo } = await req.json()

  // 1) Generate link via Supabase
  const { data, error } = await supabase.auth.admin.generateLink({
    type, // 'signup' | 'magiclink' | 'recovery'
    email,
    options: { redirectTo }
  })
  if (error) throw error

  // 2) Send via Resend
  await resend.emails.send({
    from: 'noreply@matri.naveevo.com',
    to: email,
    subject: type === 'recovery' ? 'Reset your password' : 'Verify your email',
    html: `<a href="${data?.redirect_to}">Continue</a>`
  })

  return Response.json({ ok: true })
}
```

Notes:
- Keep auth mail templates minimal; the secure link comes from Supabase.
- Add rate limiting and dedupe on this endpoint to avoid spam.
- Prefer SMTP providers (Postmark/SES) in Supabase if you want higher limits without custom endpoints.

### **Send Email**
```typescript
// lib/email.ts
import { Resend } from 'resend'

const resend = new Resend(process.env.RESEND_API_KEY)

export async function sendWelcomeEmail(email: string, name: string) {
  try {
    await resend.emails.send({
      from: 'CC Matrimony <noreply@matri.naveevo.com>',
      to: email,
      subject: 'Welcome to CC Matrimony!',
      html: `
        <h1>Welcome ${name}!</h1>
        <p>Your account has been created successfully.</p>
        <a href="${process.env.NEXT_PUBLIC_APP_URL}/dashboard">Complete your profile</a>
      `
    })
  } catch (error) {
    console.error('Email error:', error)
    throw error
  }
}
```

---

## 🧪 TESTING PATTERNS

### **Test API Route**
```typescript
// __tests__/api/profiles.test.ts
import { POST } from '@/app/api/profiles/route'
import { createMocks } from 'node-mocks-http'

describe('/api/profiles', () => {
  it('creates profile with valid data', async () => {
    const { req, res } = createMocks({
      method: 'POST',
      body: { name: 'Test User', age: 25, gender: 'Male' }
    })

    await POST(req)
    
    expect(res._getStatusCode()).toBe(201)
  })
})
```

---

## 🎨 UI COMPONENT PATTERNS

### **Loading States**
```typescript
export function ProfileCard({ profileId }: { profileId: string }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['profile', profileId],
    queryFn: () => fetchProfile(profileId)
  })

  if (isLoading) {
    return <ProfileCardSkeleton />
  }

  if (error) {
    return <Alert variant="destructive">Failed to load profile</Alert>
  }

  return <div>{/* Profile content */}</div>
}
```

### **Form Validation (Week 3+ with Zod)**
```typescript
// lib/validations/profile.ts
import { z } from 'zod'

export const profileSchema = z.object({
  full_name: z.string().min(2).max(100),
  age: z.number().min(18).max(100),
  gender: z.enum(['Male', 'Female']),
  community: z.string().min(1)
})

export type ProfileFormData = z.infer<typeof profileSchema>
```

---

**Last Updated**: November 2025  
**Usage**: Reference this document when building features to maintain consistency

### **Auth emails via Supabase + Resend SMTP (recommended for simplicity)**

Configure Resend SMTP in Supabase so built-in flows send auth emails without code changes:

- SMTP host: `smtp.resend.com`
- Ports: `465, 587, 2465, 2587`
- Username: `resend`
- Password: your `RESEND_API_KEY`
- Rate limits: Supabase custom SMTP defaults to ~30 auth emails/hour; adjust via Management API if needed.

Notes:
- Keep non-auth notifications (welcome, interest, digests) on Resend API with React Email templates.
- Ensure domain is verified in Resend (SPF/DKIM/DMARC) for deliverability.

## 🟢 MVP Simplified Mode

Goal: minimize moving parts so a single developer (AI) can ship fast and keep maintenance effortless.

### Principles
- Prefer server-side data fetching (Next.js RSC/SSR) for read-mostly pages.
- Use TanStack Query only for interactive modules with frequent mutations: Interests, Photo Requests, Payments.
- Avoid global state initially; use local state + minimal context. Add Zustand only if shared non-server state becomes painful.
- Validate at boundaries: use Zod for API payloads and critical forms; use native/inline checks for trivial forms.
- Centralize fetch utilities and query keys to avoid ad-hoc code.

### Minimal Data Access Helpers
```ts
// lib/api.ts (reference snippet)
export async function api(path: string, init?: RequestInit) {
  const res = await fetch(`/api${path}` , { ...init, headers: { 'Content-Type': 'application/json', ...(init?.headers || {}) } })
  if (!res.ok) {
    const text = await res.text().catch(() => '')
    throw new Error(`API ${path} failed: ${res.status} ${text}`)
  }
  return res.json()
}

// lib/q.ts (query keys)
export const q = {
  profile: (id: string) => ['profile', id],
  search: (params: Record<string, unknown>) => ['search', params],
  interests: {
    byUser: (userId: string) => ['interests', 'byUser', userId],
  },
  photoRequests: {
    byProfile: (profileId: string) => ['photoRequests', profileId],
  },
  payments: {
    byUser: (userId: string) => ['payments', userId],
  },
}
```

### TanStack Query Usage (targeted)
```ts
// Example: interactive profile view
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { api } from '@/lib/api'
import { q } from '@/lib/q'

function Profile({ id }: { id: string }) {
  const qc = useQueryClient()
  const { data, isLoading } = useQuery({ queryKey: q.profile(id), queryFn: () => api(`/profiles/${id}`) })

  const interest = useMutation({
    mutationFn: () => api(`/interests`, { method: 'POST', body: JSON.stringify({ profileId: id }) }),
    onSuccess: () => qc.invalidateQueries({ queryKey: q.profile(id) })
  })

  if (isLoading) return <div>Loading...</div>
  return <button onClick={() => interest.mutate()}>Send Interest</button>
}
```

### Validation Strategy
- Zod schemas for API routes and critical forms (auth, profile update, payments).
- Keep trivial forms lean: HTML5 validation + simple checks.
- Reuse server-side Zod schemas on client when possible to avoid duplication.

### Email Strategy (Simplified)
- Auth emails: Supabase + Resend SMTP (zero code: verification/reset/magic link).
- Notifications: Resend API with React Email templates.
- Verify domain (SPF/DKIM/DMARC) once; use consistent `from` address.

### Media & Payments (Quick)
- Photos: Cloudinary direct upload (store `public_id`, serve via CDN). Show "Request Photo" when zero approved photos.
- Payments: Razorpay minimal integration; server verifies via webhook, client uses lightweight checkout.

### When to Add Complexity
- Introduce Zustand only if multiple routes need shared, non-cached UI state.
- Expand TanStack Query beyond the targeted modules when manual fetching becomes repetitive.
- Grow Zod coverage as forms become multi-step or require rich validation.

