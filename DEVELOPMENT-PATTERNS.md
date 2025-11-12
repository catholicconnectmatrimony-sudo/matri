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

## 📤 FILE UPLOAD PATTERNS (Cloudflare R2)

### **Client-Side Upload Flow**
```typescript
// components/PhotoUpload.tsx
'use client'
import { useState } from 'react'
import imageCompression from 'browser-image-compression'

export function PhotoUpload({ onUpload }: { onUpload: (url: string) => void }) {
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const handleUpload = async (file: File) => {
    setLoading(true)
    setError(null)

    try {
      // 1. Compress client-side
      const compressed = await imageCompression(file, {
        maxSizeMB: 0.5,
        maxWidthOrHeight: 1200,
        useWebWorker: true
      })

      // 2. Upload via API
      const formData = new FormData()
      formData.append('photo', compressed, file.name)

      const response = await fetch('/api/photos/upload', {
        method: 'POST',
        body: formData
      })

      if (!response.ok) {
        const result = await response.json()
        throw new Error(result.error || 'Upload failed')
      }

      const { url } = await response.json()
      onUpload(url)
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
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3'
import { getSignedUrl } from '@aws-sdk/s3-request-presigner'
import { createClient } from '@/lib/supabase/server'
import { NextResponse } from 'next/server'

const r2Client = new S3Client({
  region: 'auto',
  endpoint: `https://${process.env.R2_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: process.env.R2_ACCESS_KEY_ID!,
    secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!,
  },
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

    // Generate unique filename
    const filename = `${user.id}/${Date.now()}-${file.name}`
    
    // Upload to R2
    const buffer = Buffer.from(await file.arrayBuffer())
    await r2Client.send(new PutObjectCommand({
      Bucket: process.env.R2_BUCKET_NAME!,
      Key: filename,
      Body: buffer,
      ContentType: file.type,
    }))

    // Save metadata to database
    const { data: photo, error } = await supabase
      .from('photos')
      .insert({
        profile_id: user.id, // Assuming profile_id = user_id for MVP
        filename,
        storage_path: filename,
        file_size: file.size,
        mime_type: file.type,
        is_approved: true // Auto-approve for MVP
      })
      .select()
      .single()

    if (error) {
      return NextResponse.json({ error: 'Failed to save metadata' }, { status: 500 })
    }

    // Generate signed URL for viewing
    const signedUrl = await getSignedUrl(
      r2Client,
      new GetObjectCommand({
        Bucket: process.env.R2_BUCKET_NAME!,
        Key: filename,
      }),
      { expiresIn: 3600 } // 1 hour
    )

    return NextResponse.json({ 
      id: photo.id,
      url: signedUrl,
      storage_path: filename
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

## 🔒 RECIPROCITY PATTERN (Simple Photo Gate)

### **Check Photo Reciprocity**
```typescript
// lib/reciprocity/photo-gate.ts
import { createClient } from '@/lib/supabase/server'

export async function canViewPhotos(viewerId: string): Promise<boolean> {
  const supabase = createClient()
  
  // Check if viewer has at least 1 approved photo
  const { data: photos, error } = await supabase
    .from('photos')
    .select('id')
    .eq('profile_id', viewerId)
    .eq('is_approved', true)
    .limit(1)

  if (error) {
    console.error('Reciprocity check error:', error)
    return false
  }

  return (photos?.length ?? 0) > 0
}
```

### **Use in API Route**
```typescript
// app/api/profiles/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // Check photo reciprocity
  const canView = await canViewPhotos(user.id)
  if (!canView) {
    return NextResponse.json({
      error: 'Upload a photo to view other profiles',
      locked: true
    }, { status: 403 })
  }

  // Fetch profile with photos
  const { data: profile } = await supabase
    .from('profiles')
    .select('*, photos(*)')
    .eq('id', params.id)
    .single()

  return NextResponse.json({ profile })
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

