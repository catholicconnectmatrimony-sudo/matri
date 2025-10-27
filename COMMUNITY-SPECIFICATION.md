# CC Matrimony Community Specification - Tulunadu Focus

## 1. Purpose & Scope
- Focus on Coastal Karnataka/Tulunadu region communities
- Primary focus on Bunt community for initial launch
- Support for local languages and cultural elements
- Community-specific features and traditions
- Platform: CC Matrimony (matri.naveevo.com)

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

## 3. Community-Specific Features

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

## 12. Future Expansion

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

**Last Updated:** December 2024  
**Document Version:** 1.0  
**Next Review:** January 2025
