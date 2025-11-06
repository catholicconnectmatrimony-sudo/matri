# CC Matrimony Community Specification - Tulunadu Focus

## 1. Purpose & Scope
- Focus on Coastal Karnataka/Tulunadu region communities
- **MVP Launch (Week 8)**: Bunt community ONLY for focused product-market fit validation
- **Week 4 Expansion**: Add Billava community
- **Month 2 Expansion**: All Hindu, Christian, and Muslim communities
- Support for local languages and cultural elements (matching preferences only, English UI)
- Community-specific features and traditions
- Platform: CC Matrimony (matri.naveevo.com)

## 1.1. Launch Phase Strategy

### **Phase 1: Bunt-Only Launch (Weeks 1-4)**
- Single community focus: Hindu/Bunt
- URL: `/hindu/bunt` (other community URLs return 404 or "Coming Soon")
- Database supports all communities (ready for expansion)
- Faster launch, clearer marketing message
- Easier to validate product-market fit
- 200+ profile target

### **Phase 2: Expansion (Week 4+)**
- Add Hindu/Billava community
- URL: `/hindu/billava` goes live
- Parallel community operations

### **Phase 3: Full Launch (Month 2+)**
- All religions: Hindu, Christian, Muslim
- All planned communities active
- Multi-community marketing

## 2. Primary Communities (Launch Priority)

### **Phase 1: Bunt Community (Primary Focus)**
| Sub-community | Population | Digital Adoption | Priority |
|---------------|------------|------------------|----------|
| **Shetty** | Largest | High | P0 |
| **Hegde** | Large | High | P0 |
| **Poojary** | Medium | Medium | P1 |
| **Kotian** | Medium | Medium | P1 |
| **Karkera** | Small | High | P2 |

### **Phase 2: Other Hindu Communities**
| Community | Population | Digital Adoption | Priority |
|-----------|------------|------------------|----------|
| **Billava** | Large | Medium | P1 |
| **Devadiga** | Medium | Medium | P1 |
| **Mogaveera** | Medium | High | P1 |
| **Brahmin** | Small | High | P2 |

### **Phase 3: Christian Communities**
| Community | Population | Digital Adoption | Priority |
|-----------|------------|------------------|----------|
| **Mangalorean Christian** | Large | High | P1 |
| **Konkani Christian** | Medium | Medium | P2 |
| **Syro-Malabar** | Small | High | P2 |

## 3. URL Structure & Routing

### 3.1 URL Patterns
```
/ - Home
/[religion] - Religion page (e.g., /hindu, /christian)
/[religion]/[community] - Community page (e.g., /hindu/bunt, /christian/mangalorean)
```

### 3.2 404 Tracking & Redirects

#### Database Schema
```sql
-- Add to existing database schema
CREATE TABLE url_redirects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  from_path VARCHAR(255) NOT NULL,
  to_path VARCHAR(255) NOT NULL,
  status_code INTEGER DEFAULT 301,
  is_active BOOLEAN DEFAULT TRUE,
  hit_count INTEGER DEFAULT 0,
  last_hit_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  created_by UUID REFERENCES users(id),
  UNIQUE(from_path)
);

-- Index for performance
CREATE INDEX idx_redirects_from_path ON url_redirects(from_path) WHERE is_active = TRUE;

-- 404 Logs
CREATE TABLE not_found_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  path VARCHAR(500) NOT NULL,
  referrer VARCHAR(500),
  user_agent TEXT,
  ip_address INET,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Index for analysis
CREATE INDEX idx_not_found_path ON not_found_logs(path);
CREATE INDEX idx_not_found_created ON not_found_logs(created_at);
```

#### Middleware Implementation
```typescript
// middleware.ts
export async function middleware(req: NextRequest) {
  const path = req.nextUrl.pathname.toLowerCase();
  
  // Skip API routes and static files
  if (path.startsWith('/api') || path.match(/\.[a-z0-9]+$/i)) {
    return NextResponse.next();
  }

  // Check for redirects
  const { data: redirect } = await supabase
    .from('url_redirects')
    .select('to_path, status_code')
    .eq('from_path', path)
    .eq('is_active', true)
    .single();

  if (redirect) {
    // Update hit counter (async)
    supabase.rpc('increment_redirect_hits', { redirect_id: redirect.id });
    
    return NextResponse.redirect(
      new URL(redirect.to_path, req.url),
      { status: redirect.status_code || 301 }
    );
  }

  // Check if this is a valid community URL
  const pathParts = path.split('/').filter(Boolean);
  if (pathParts.length === 2) {
    const [religion, community] = pathParts;
    const { data: communityExists } = await supabase
      .from('communities')
      .select('id')
      .eq('slug', community)
      .eq('religion.slug', religion)
      .single();

    if (!communityExists) {
      // Log 404
      await supabase.from('not_found_logs').insert({
        path,
        referrer: req.referrer,
        user_agent: req.headers.get('user-agent'),
        ip_address: req.ip
      });

      // Check for similar communities
      const { data: similar } = await supabase
        .from('communities')
        .select('slug, religion:religions(slug)')
        .ilike('slug', `%${community}%`)
        .limit(3);

      if (similar?.length) {
        // Redirect to search results for similar communities
        const searchParams = new URLSearchParams({
          q: community,
          type: 'community',
          did_you_mean: 'true'
        });
        return NextResponse.redirect(
          new URL(`/search?${searchParams}`, req.url)
        );
      }
    }
  }

  return NextResponse.next();
}
```

#### Database Function for Hit Counting
```sql
-- Add this to your database functions
CREATE OR REPLACE FUNCTION increment_redirect_hits(redirect_id UUID)
RETURNS void AS $$
BEGIN
  UPDATE url_redirects 
  SET 
    hit_count = hit_count + 1,
    last_hit_at = NOW(),
    updated_at = NOW()
  WHERE id = redirect_id;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

#### Admin Interface for Managing Redirects
```typescript
// Example component for the admin dashboard
function RedirectManager() {
  const [redirects, setRedirects] = useState([]);
  const [newRedirect, setNewRedirect] = useState({ from: '', to: '', statusCode: 301 });
  
  // Fetch existing redirects
  useEffect(() => {
    const loadRedirects = async () => {
      const { data } = await supabase
        .from('url_redirects')
        .select('*')
        .order('created_at', { ascending: false });
      setRedirects(data || []);
    };
    loadRedirects();
  }, []);

  // Add new redirect
  const addRedirect = async (e) => {
    e.preventDefault();
    const { data, error } = await supabase
      .from('url_redirects')
      .insert([{
        from_path: newRedirect.from,
        to_path: newRedirect.to,
        status_code: newRedirect.statusCode,
        created_by: user.id
      }])
      .select();
    
    if (!error && data?.[0]) {
      setRedirects([data[0], ...redirects]);
      setNewRedirect({ from: '', to: '', statusCode: 301 });
    }
  };

  // Toggle redirect status
  const toggleRedirect = async (id, isActive) => {
    const { error } = await supabase
      .from('url_redirects')
      .update({ is_active: !isActive, updated_at: new Date() })
      .eq('id', id);
    
    if (!error) {
      setRedirects(redirects.map(r => 
        r.id === id ? { ...r, is_active: !isActive } : r
      ));
    }
  };

  return (
    <div className="space-y-6">
      <form onSubmit={addRedirect} className="space-y-4 p-4 bg-gray-50 rounded-lg">
        <h3 className="text-lg font-medium">Add New Redirect</h3>
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <div>
            <label className="block text-sm font-medium text-gray-700">From Path</label>
            <input
              type="text"
              className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"
              value={newRedirect.from}
              onChange={(e) => setNewRedirect({...newRedirect, from: e.target.value})}
              placeholder="/old-path"
              required
            />
          </div>
          <div>
            <label className="block text-sm font-medium text-gray-700">To Path</label>
            <input
              type="text"
              className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"
              value={newRedirect.to}
              onChange={(e) => setNewRedirect({...newRedirect, to: e.target.value})}
              placeholder="/new-path"
              required
            />
          </div>
          <div>
            <label className="block text-sm font-medium text-gray-700">Status Code</label>
            <select
              className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500"
              value={newRedirect.statusCode}
              onChange={(e) => setNewRedirect({...newRedirect, statusCode: parseInt(e.target.value)})}
            >
              <option value={301}>301 (Permanent)</option>
              <option value={302}>302 (Temporary)</option>
            </select>
          </div>
        </div>
        <div className="flex justify-end">
          <button
            type="submit"
            className="inline-flex justify-center py-2 px-4 border border-transparent shadow-sm text-sm font-medium rounded-md text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
          >
            Add Redirect
          </button>
        </div>
      </form>

      <div className="bg-white shadow overflow-hidden sm:rounded-md">
        <ul className="divide-y divide-gray-200">
          {redirects.map((redirect) => (
            <li key={redirect.id}>
              <div className="px-4 py-4 sm:px-6">
                <div className="flex items-center justify-between">
                  <div className="flex-1 min-w-0">
                    <div className="flex items-center">
                      <p className="text-sm font-medium text-blue-600 truncate">
                        {redirect.from_path}
                      </p>
                      <p className="ml-2 flex-shrink-0 text-sm text-gray-500">
                        → {redirect.to_path} ({redirect.status_code})
                      </p>
                    </div>
                    <div className="mt-2 flex">
                      <p className="flex items-center text-sm text-gray-500">
                        <span>Hits: {redirect.hit_count}</span>
                        <span className="mx-1">•</span>
                        <span>Last hit: {redirect.last_hit_at ? new Date(redirect.last_hit_at).toLocaleDateString() : 'Never'}</span>
                      </p>
                    </div>
                  </div>
                  <div className="ml-4 flex-shrink-0">
                    <button
                      onClick={() => toggleRedirect(redirect.id, redirect.is_active)}
                      className={`inline-flex items-center px-3 py-1 border border-transparent text-sm leading-5 font-medium rounded-md ${
                        redirect.is_active 
                          ? 'bg-green-100 text-green-800 hover:bg-green-200' 
                          : 'bg-gray-100 text-gray-800 hover:bg-gray-200'
                      }`}
                    >
                      {redirect.is_active ? 'Active' : 'Inactive'}
                    </button>
                  </div>
                </div>
              </div>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

## 4. Community-Specific Features

### **3.1 Bunt Community Features**
```typescript
interface BuntProfile {
  subCommunity: 'Shetty' | 'Hegde' | 'Poojary' | 'Kotian' | 'Karkera';
  familyTitle: string; // e.g., "Shetty", "Hegde"
  ancestralVillage: string;
  familyBusiness: string;
  communityEvents: string[];
  traditionalPractices: string[];
}
```

### **3.2 Community-Specific Search Filters**
- **Sub-community matching** (Shetty with Shetty, etc.)
- **Ancestral village proximity**
- **Family business compatibility**
- **Traditional practice alignment**
- **Community event participation**

### **3.3 Language Preferences**
| Language | Priority | Usage |
|----------|----------|-------|
| **English** | P0 | Primary interface language (universal) |
| **Kannada** | P1 | Mother tongue preference for matching |
| **Tulu** | P1 | Mother tongue preference for matching |
| **Konkani** | P1 | Mother tongue preference for matching |

> **Language note:** Platform interface remains English-only for MVP; local languages capture preferences and power matching logic without UI translation.

## 4. Geographic Focus

### **4.1 Primary Cities**
| City | Population | Bunt Population | Priority |
|------|------------|-----------------|----------|
| **Mangalore** | 500K+ | 50K+ | P0 |
| **Udupi** | 200K+ | 20K+ | P0 |
| **Kundapura** | 100K+ | 10K+ | P1 |
| **Karkala** | 50K+ | 5K+ | P1 |
| **Bantwal** | 30K+ | 3K+ | P2 |

### **4.2 Regional Coverage**
- **Coastal Karnataka**: Primary focus
- **Karnataka**: Secondary focus
- **Other States**: Tertiary focus (Mumbai, Bangalore, etc.)

## 5. Cultural Elements

### **5.1 Festivals & Traditions**
| Festival | Community | Importance | Integration |
|----------|-----------|------------|-------------|
| **Kambala** | Bunt | High | Event calendar, community posts |
| **Paryaya** | Udupi | High | Local event integration |
| **Monti Fest** | Christian | High | Community celebration |
| **Dasara** | Hindu | Medium | Regional festival |
| **Deepavali** | Hindu | Medium | Regional festival |

### **5.2 Traditional Practices**
- **Family Structure**: Joint vs Nuclear family preferences
- **Marriage Customs**: Traditional vs modern ceremony preferences
- **Food Habits**: Vegetarian vs Non-vegetarian preferences
- **Religious Practices**: Temple visits, church attendance
- **Community Involvement**: Participation in community events

## 6. Community-Specific Content

### **6.1 Success Stories**
- **Bunt Community**: Focus on successful Bunt marriages
- **Local Stories**: Mangalore, Udupi success stories
- **Traditional Marriages**: Stories highlighting local customs
- **Modern Marriages**: Stories of successful modern Bunt couples

### **6.2 Community Pages**
- **Bunt Matrimony**: Dedicated Bunt community page
- **Tulunadu Matrimony**: Regional focus page
- **Mangalorean Matrimony**: City-specific page
- **Local Traditions**: Information about local customs

### **6.3 URL Structure & Navigation**

#### **6.3.1 URL Pattern**
```
https://matri.naveevo.com/{religion}/{community}/{sub-community?}
```

#### **6.3.2 URL Examples**
- `https://matri.naveevo.com/hindu/bunt`
- `https://matri.naveevo.com/christian/mangalorean`
- `https://matri.naveevo.com/muslim/sunni`
- `https://matri.naveevo.com/hindu/bunt/shetty` (for sub-communities)

#### **6.3.3 URL Rules**
- All lowercase letters
- Hyphens for word separation
- No trailing slashes
- Canonical URLs for all pages
- Redirects for common misspellings

#### **6.3.4 Breadcrumb Navigation**
```
Home > {Religion} Matrimony > {Community} > {Sub-community}
```

Example for Bunt community:
```
Home > Hindu Matrimony > Bunt Community > Shetty
```

### **6.4 SEO & Structured Data**
```typescript
// Example for Bunt community page
export const BuntCommunitySchema = {
  '@context': 'https://schema.org',
  '@type': 'WebPage',
  'name': 'Bunt Matrimony - CC Matrimony',
  'description': 'Find your perfect Bunt community life partner. Connect with Shetty, Hegde, Poojary, Kotian, and Karkera sub-communities.',
  'url': 'https://matri.naveevo.com/${community.religion.toLowerCase()}/${community.slug}',
  'mainEntityOfPage': {
    '@type': 'WebPage',
    '@id': 'https://matri.naveevo.com/${community.religion.toLowerCase()}/${community.slug}'
  },
  'publisher': {
    '@type': 'Organization',
    'name': 'CC Matrimony',
    'logo': {
      '@type': 'ImageObject',
      'url': 'https://matri.naveevo.com/logo.png'
    }
  },
  'breadcrumb': {
    '@type': 'BreadcrumbList',
    'itemListElement': [
      {
        '@type': 'ListItem',
        'position': 1,
        'name': 'Home',
        'item': 'https://matri.naveevo.com/'
      },
      {
        '@type': 'ListItem',
        'position': 2,
        'name': '${community.religion} Matrimony',
        'item': 'https://matri.naveevo.com/${community.religion.toLowerCase()}'
      },
      {
        '@type': 'ListItem',
        'position': 3,
        'name': '${community.name} Matrimony'
      }
    ]
  },
  'image': 'https://matri.naveevo.com/images/communities/bunt-og.jpg',
  'datePublished': '2025-01-01',
  'dateModified': '2025-11-06'
};
```

## 7. Marketing & Outreach

### **7.1 Community Leaders**
- **Bunt Community Leaders**: Partner with local Bunt leaders
- **Religious Leaders**: Temple and church leaders
- **Social Organizations**: Bunt community organizations
- **Local Influencers**: Community influencers and social media personalities

### **7.2 Community Events**
- **Kambala Events**: Sponsor and participate in Kambala
- **Community Gatherings**: Attend local community events
- **Religious Festivals**: Participate in temple and church festivals
- **Social Functions**: Community weddings and celebrations

## 8. Technical Implementation

### **8.0 Route Handling**

#### **8.0.1 Next.js Dynamic Routing**
```typescript
// File structure
/pages
  /[religion]
    /[community]
      /index.tsx         // Community page
      /[subcommunity].tsx // Sub-community page
    /index.tsx           // Religion landing page
```

#### **8.0.2 Route Handler**
```typescript
// pages/[religion]/[community]/index.tsx
import { GetStaticProps, GetStaticPaths } from 'next';

type CommunityPageProps = {
  religion: string;
  community: Community;
  breadcrumbs: Array<{ name: string; href: string }>;
};

export default function CommunityPage({ religion, community, breadcrumbs }: CommunityPageProps) {
  // Page implementation
}

export const getStaticPaths: GetStaticPaths = async () => {
  // Generate paths for all active communities
  const { data: communities } = await supabase
    .from('communities')
    .select('slug, religion:religions(slug)')
    .eq('is_active', true);

  const paths = communities.map(community => ({
    params: {
      religion: community.religion.slug,
      community: community.slug
    }
  }));

  return { paths, fallback: 'blocking' };
};

export const getStaticProps: GetStaticProps = async ({ params }) => {
  const { religion: religionSlug, community: communitySlug } = params as {
    religion: string;
    community: string;
  };

  // Fetch community data with religion and sub-communities
  const { data: community } = await supabase
    .from('communities')
    .select(`
      *,
      religion:religions(*),
      sub_communities(*)
    `)
    .eq('slug', communitySlug)
    .eq('religion.slug', religionSlug)
    .single();

  if (!community) {
    return { notFound: true };
  }

  const breadcrumbs = [
    { name: 'Home', href: '/' },
    { 
      name: `${community.religion.name} Matrimony`, 
      href: `/${community.religion.slug}` 
    },
    { 
      name: `${community.name} Community`,
      href: `/${community.religion.slug}/${community.slug}`,
      current: true
    }
  ];

  return {
    props: {
      religion: community.religion,
      community,
      breadcrumbs
    },
    revalidate: 3600 // Regenerate page every hour
  };
};
```

#### **8.0.3 Sitemap Generation**
```typescript
// pages/sitemap.xml.tsx
export async function getServerSideProps({ res }) {
  const { data: communities } = await supabase
    .from('communities')
    .select(`
      slug,
      updated_at,
      religion:religions(slug)
    `)
    .eq('is_active', true);

  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      ${communities.map(community => `
        <url>
          <loc>https://matri.naveevo.com/${community.religion.slug}/${community.slug}</loc>
          <lastmod>${new Date(community.updated_at).toISOString()}</lastmod>
          <changefreq>weekly</changefreq>
          <priority>0.8</priority>
        </url>
      `).join('')}
    </urlset>`;

  res.setHeader('Content-Type', 'text/xml');
  res.write(sitemap);
  res.end();

  return { props: {};
}
```

#### **8.0.4 Next.js Configuration**
```javascript
// next.config.js
module.exports = {
  async redirects() {
    return [
      // Redirect /hindu/ to /hindu
      {
        source: '/:religion/',
        destination: '/:religion',
        permanent: true,
      },
      // Handle common misspellings
      {
        source: '/hindhu/:path*',
        destination: '/hindu/:path*',
        permanent: true,
      },
    ];
  },
};
```

### **8.0 Community Page SEO Implementation**

#### **8.0.1 Dynamic Sitemap Generation**
```typescript
// pages/sitemap.xml.tsx
export async function getServerSideProps({ res }) {
  const { data: communities } = await supabase
    .from('communities')
    .select('slug, updated_at, religion')
    .eq('is_active', true);

  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
    <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      ${communities.map(community => `
        <url>
          <loc>https://matri.naveevo.com/${community.religion.toLowerCase()}/${community.slug}</loc>
          <lastmod>${new Date(community.updated_at).toISOString()}</lastmod>
          <changefreq>weekly</changefreq>
          <priority>0.8</priority>
        </url>
      `).join('')}
    </urlset>`;

  res.setHeader('Content-Type', 'text/xml');
  res.write(sitemap);
  res.end();

  return { props: {} };
}
```

#### **8.0.2 OpenGraph Implementation**
```tsx
// components/community/CommunityHead.tsx
interface CommunityHeadProps {
  community: {
    name: string;
    description: string;
    slug: string;
    imageUrl: string;
    religion: string;
  };
}

export default function CommunityHead({ community }: CommunityHeadProps) {
  return (
    <Head>
      <title>{community.name} Matrimony - CC Matrimony</title>
      <meta name="description" content={community.description} />
      
      {/* Open Graph / Facebook */}
      <meta property="og:type" content="website" />
      <meta property="og:url" content={`https://matri.naveevo.com/${community.religion.toLowerCase()}/${community.slug}`} />
      <meta property="og:title" content={`${community.name} Matrimony - Find Your Life Partner`} />
      <meta property="og:description" content={community.description} />
      <meta property="og:image" content={community.imageUrl || 'https://matri.naveevo.com/og-default.jpg'} />

      {/* Twitter */}
      <meta property="twitter:card" content="summary_large_image" />
      <meta property="twitter:url" content={`https://matri.naveevo.com/${community.religion.toLowerCase()}/${community.slug}`} />
      <meta property="twitter:title" content={`${community.name} Matrimony - CC Matrimony`} />
      <meta property="twitter:description" content={community.description} />
      <meta property="twitter:image" content={community.imageUrl || 'https://matri.naveevo.com/og-default.jpg'} />
    </Head>
  );
}
```

### **8.1 Community-Specific Database Schema**
```sql
-- Community-specific tables
CREATE TABLE communities (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  religion VARCHAR(50) NOT NULL,
  region VARCHAR(100) NOT NULL,
  priority INTEGER DEFAULT 0
);

CREATE TABLE sub_communities (
  id SERIAL PRIMARY KEY,
  community_id INTEGER REFERENCES communities(id),
  name VARCHAR(100) NOT NULL,
  population_estimate INTEGER,
  digital_adoption_score INTEGER
);

CREATE TABLE user_community_details (
  user_id UUID REFERENCES auth.users(id),
  community_id INTEGER REFERENCES communities(id),
  sub_community_id INTEGER REFERENCES sub_communities(id),
  family_title VARCHAR(100),
  ancestral_village VARCHAR(100),
  family_business VARCHAR(200),
  traditional_practices TEXT[],
  community_events TEXT[]
);
```

### **8.2 Community-Specific Search Logic**
```typescript
interface CommunitySearchFilters {
  community: string;
  subCommunity: string[];
  ancestralVillage: string[];
  familyBusiness: string[];
  traditionalPractices: string[];
  communityEvents: string[];
  language: string[];
  region: string[];
}
```

## 9. Launch Strategy

### **9.1 Phase 1: Bunt Community Launch**
- **Target**: 200+ Bunt profiles in first 3 months
- **Focus**: Mangalore and Udupi
- **Strategy**: Community leader partnerships
- **Content**: Bunt-specific success stories and content

### **9.2 Phase 2: Regional Expansion**
- **Target**: 500+ profiles across all communities
- **Focus**: Coastal Karnataka region
- **Strategy**: Local event participation
- **Content**: Regional success stories

### **9.3 Phase 3: State-wide Expansion**
- **Target**: 1000+ profiles across Karnataka
- **Focus**: Bangalore, Mysore, Hubli
- **Strategy**: Digital marketing and referrals
- **Content**: State-wide success stories

## 10. Success Metrics

### **10.1 Community-Specific KPIs**
- **Bunt Community Adoption**: 200+ profiles in 3 months
- **Local Language Usage**: 60%+ profiles in Kannada/Tulu
- **Geographic Coverage**: 80%+ profiles from Coastal Karnataka
- **Community Engagement**: 40%+ active participation in community features

### **10.2 Business Metrics**
- **User Acquisition**: 500+ users in first 6 months
- **Premium Conversion**: 25%+ (higher due to community focus)
- **Match Success Rate**: 35%+ (community-specific matching)
- **User Retention**: 70%+ after 30 days

## 11. Content Strategy

### **11.1 Community-Specific Content**
- **Bunt Traditions**: Articles about Bunt customs and traditions
- **Local Festivals**: Information about Kambala, Paryaya, etc.
- **Success Stories**: Real Bunt marriage success stories
- **Community News**: Local community updates and events

### **11.2 Content Strategy**
- **English Content**: Primary content in English (universal)
- **Community-Specific Content**: Content tailored for each community
- **Local Cultural Content**: Information about local traditions and festivals
- **Success Stories**: Success stories in English with community context
- **Platform Branding**: CC Matrimony branding throughout all content

## 12. Technical SEO Considerations

### **12.1 Canonical URLs**
- Automatically add canonical tags to all pages
- Handle URL parameters for search and filtering
- Implement hreflang for regional variations

### **12.2 Performance Optimization**
- Implement ISR (Incremental Static Regeneration)
- Optimize images with next/image
- Implement proper caching headers
- Use CDN for static assets

### **12.3 Structured Data**
- Implement JSON-LD for all pages
- Add FAQ schema for common questions
- Include local business schema for regional offices
- Implement breadcrumb schema

## 13. Future Expansion

### **12.1 Additional Communities**
- **Billava Community**: Second priority after Bunt
- **Mangalorean Christian**: Third priority
- **Other Coastal Communities**: Gradual expansion

### **12.2 Regional Expansion**
- **Karnataka**: State-wide expansion
- **Maharashtra**: Mumbai, Pune Bunt communities
- **Goa**: Konkani communities
- **Kerala**: Malayalam-speaking communities

---

## 14. Monitoring & Maintenance

### **14.1 SEO Audits**
- Monthly technical SEO audits
- Monitor crawl errors in Google Search Console
- Track keyword rankings and impressions
- Monitor Core Web Vitals

### **14.2 URL Management**
- 301 redirects for all URL changes
- Monitor 404 errors
- Regular sitemap submission to search engines
- Canonical URL validation

### **14.3 Performance Monitoring**
- Page load speed tracking
- Mobile usability testing
- Server response times
- API response times

---

**Last Updated:** November 6, 2025  
**Document Version:** 3.0 (Finalized - Bunt-Only Launch Strategy)  
**MVP Launch:** Bunt community only (January 5, 2026)  
**Expansion:** Billava (Month 4), All communities (Month 6)  
**Next Review:** Post-launch (January 2026)
