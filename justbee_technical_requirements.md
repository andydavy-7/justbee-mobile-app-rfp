# Just Bee Platform - Technical Requirements Specification (TRS)

## Document Control

| Field | Value |
|-------|-------|
| Document Version | 1.0 |
| Date | December 2024 |
| Status | For Review |
| Author | Paradox Requirements Engineering |
| Document Type | Technical Specification |
| Compliance | WCAG 2.2 AAA, GDPR, SOC 2 |

## 1. System Architecture Overview

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                              │
├────────────────┬────────────────┬────────────────┬──────────────┤
│   iOS App      │   Android App   │   Web App      │   PWA        │
│  (React Native)│  (React Native) │    (React)     │  (React)     │
└────────┬───────┴────────┬────────┴────────┬───────┴──────┬──────┘
         │                 │                 │               │
┌────────▼─────────────────▼─────────────────▼───────────────▼──────┐
│                      API GATEWAY (Kong/Apollo)                     │
│                   Rate Limiting | Auth | Routing                   │
└────────┬──────────────────────────────────────────────────┬───────┘
         │                                                   │
┌────────▼───────────────────────────────────────────────────▼───────┐
│                     MICROSERVICES LAYER                            │
├─────────────┬──────────────┬──────────────┬──────────────────────┤
│   User      │   Places     │   Works      │   Love              │
│   Service   │   Service    │   Service    │   Service           │
├─────────────┼──────────────┼──────────────┼──────────────────────┤
│   Connect   │   Events     │   Matching   │   Notification      │
│   Service   │   Service    │   Engine     │   Service           │
├─────────────┼──────────────┼──────────────┼──────────────────────┤
│   Safety    │   Payment    │   Analytics  │   Media             │
│   Service   │   Service    │   Service    │   Service           │
└─────────────┴──────────────┴──────────────┴──────────────────────┘
                                │
┌───────────────────────────────▼────────────────────────────────────┐
│                         DATA LAYER                                 │
├──────────────┬──────────────────┬──────────────┬─────────────────┤
│  PostgreSQL  │   Redis Cache    │  Elasticsearch│  S3 Storage    │
│  (Primary)   │   (Sessions)     │   (Search)    │  (Media)       │
└──────────────┴──────────────────┴──────────────┴─────────────────┘
                                │
┌───────────────────────────────▼────────────────────────────────────┐
│                    INFRASTRUCTURE LAYER                            │
├──────────────┬──────────────────┬──────────────┬─────────────────┤
│     AWS      │   Kubernetes     │   Terraform  │   Monitoring    │
│   (Cloud)    │  (Orchestration) │   (IaC)      │  (Datadog)      │
└──────────────┴──────────────────┴──────────────┴─────────────────┘
```

### 1.2 Technology Stack

| Layer | Technology | Justification |
|-------|------------|---------------|
| **Frontend** | React Native 0.72+ | Cross-platform efficiency, accessibility support |
| **Web** | React 18 + Next.js 14 | SSR for SEO, performance, accessibility |
| **Backend** | Node.js 20 LTS + TypeScript | Type safety, ecosystem, team expertise |
| **API** | GraphQL + REST | Flexible queries + standard integrations |
| **Database** | PostgreSQL 15 | ACID compliance, JSON support, reliability |
| **Cache** | Redis 7 | Session management, real-time features |
| **Search** | Elasticsearch 8 | Advanced search, geo-queries, analytics |
| **Queue** | RabbitMQ | Reliable message processing, scalability |
| **AI/ML** | TensorFlow.js + Python services | Client-side + server-side ML capabilities |
| **Monitoring** | Datadog + Sentry | Full-stack observability, error tracking |

## 2. Functional Requirements

### 2.1 Authentication & Authorization

#### 2.1.1 Authentication Methods

**Multi-Factor Authentication (MFA)**:
```typescript
interface AuthenticationMethods {
  primary: {
    email: string;
    password: string; // Min 12 chars, complexity requirements
  };
  mfa: {
    totp: boolean; // Time-based one-time password
    sms: boolean; // SMS verification
    biometric: boolean; // Face ID, Touch ID
    securityKey: boolean; // FIDO2/WebAuthn
  };
  social: {
    google: boolean;
    apple: boolean;
    microsoft: boolean; // For accessibility tools integration
  };
}
```

**Session Management**:
- JWT tokens with 15-minute access token lifetime
- Refresh tokens with 30-day lifetime
- Device fingerprinting for security
- Concurrent session limits (3 devices max)

#### 2.1.2 Authorization Model (RBAC + ABAC)

```typescript
enum UserRole {
  USER = 'user',
  CAREGIVER = 'caregiver',
  BUSINESS_OWNER = 'business_owner',
  BUSINESS_STAFF = 'business_staff',
  MODERATOR = 'moderator',
  ADMIN = 'admin',
  SUPER_ADMIN = 'super_admin'
}

interface Permission {
  resource: string; // e.g., 'profile', 'business', 'moderation'
  action: string; // 'create', 'read', 'update', 'delete'
  conditions?: { // Attribute-based conditions
    ownership?: boolean;
    caregiverApproved?: boolean;
    ageRestriction?: number;
    certificationLevel?: string;
  };
}
```

### 2.2 User Service

#### 2.2.1 User Profile Management

**Data Model**:
```typescript
interface UserProfile {
  // Core Identity
  id: UUID;
  email: string;
  phoneNumber?: string;
  dateOfBirth: Date;
  
  // Neurodivergent Profile (Optional)
  neurotype?: {
    disclosed: boolean;
    types: NeurotypeDivergence[]; // autism, ADHD, dyslexia, etc.
    supportNeeds: 'low' | 'medium' | 'high';
  };
  
  // Communication Preferences
  communication: {
    preferred: 'text' | 'voice' | 'video' | 'symbols';
    responseTime: 'immediate' | 'hours' | 'days';
    directness: 'very_direct' | 'direct' | 'gentle';
    initiationComfort: number; // 1-10 scale
  };
  
  // Sensory Preferences
  sensory: {
    noiseLevel: 'silent' | 'quiet' | 'moderate' | 'loud';
    lightLevel: 'dim' | 'medium' | 'bright';
    crowdComfort: 'empty' | 'sparse' | 'moderate' | 'busy';
    textureAvoidance: string[];
    scentSensitivity: boolean;
  };
  
  // Interests & Special Topics
  interests: {
    primary: string[]; // Up to 5 special interests
    secondary: string[]; // General interests
    discussionDepth: 'surface' | 'moderate' | 'deep_dive';
  };
  
  // Safety & Privacy
  safety: {
    caregiverId?: UUID;
    supervisionLevel: 'none' | 'alerts' | 'monitoring' | 'full';
    blockedUsers: UUID[];
    trustedContacts: UUID[];
  };
}
```

**API Endpoints**:
```typescript
// User Profile CRUD
POST   /api/v1/users/register
GET    /api/v1/users/profile
PUT    /api/v1/users/profile
DELETE /api/v1/users/account

// Preference Management
PUT    /api/v1/users/preferences/communication
PUT    /api/v1/users/preferences/sensory
PUT    /api/v1/users/preferences/interests

// Safety Features
POST   /api/v1/users/safety/caregiver
PUT    /api/v1/users/safety/supervision
POST   /api/v1/users/safety/block/{userId}
POST   /api/v1/users/safety/report
```

### 2.3 Places Service

#### 2.3.1 Business Data Model

```typescript
interface Business {
  id: UUID;
  name: string;
  category: BusinessCategory;
  location: {
    address: Address;
    coordinates: GeoJSON;
    accessibility: {
      wheelchairAccess: boolean;
      elevatorAvailable: boolean;
      accessibleParking: number;
      accessibleRestrooms: boolean;
    };
  };
  
  certification: {
    level: 'none' | 'bronze' | 'silver' | 'gold';
    validUntil: Date;
    trainingCompleted: string[];
    auditResults?: AuditReport;
  };
  
  sensoryEnvironment: {
    typicalNoiseLevel: 1-10;
    lightingOptions: LightingType[];
    quietSpaces: boolean;
    sensoryRoom?: {
      available: boolean;
      equipment: string[];
      bookingRequired: boolean;
    };
    fragranceFree: boolean;
  };
  
  accommodations: {
    communicationSupports: string[];
    visualSupports: boolean;
    flexibleServiceOptions: boolean;
    staffTraining: {
      neurodiversityAware: boolean;
      crisisSupport: boolean;
      communicationStrategies: boolean;
    };
  };
  
  realTimeData?: {
    currentNoiseLevel: number;
    currentCrowdLevel: number;
    waitTime: number;
    lastUpdated: Date;
  };
}
```

#### 2.3.2 Search & Discovery

**Elasticsearch Mapping**:
```json
{
  "mappings": {
    "properties": {
      "name": { "type": "text", "analyzer": "autocomplete" },
      "category": { "type": "keyword" },
      "location": { "type": "geo_point" },
      "certificationLevel": { "type": "keyword" },
      "sensoryScore": { "type": "float" },
      "accommodations": { "type": "nested" },
      "ratings": {
        "properties": {
          "overall": { "type": "float" },
          "sensory": { "type": "float" },
          "staff": { "type": "float" },
          "accessibility": { "type": "float" }
        }
      }
    }
  }
}
```

**Search Algorithm**:
```typescript
function searchBusinesses(params: SearchParams): BusinessResult[] {
  const query = {
    bool: {
      must: [
        { match: { category: params.category } },
        { range: { sensoryScore: { gte: params.minSensoryScore } } }
      ],
      filter: [
        { geo_distance: {
          distance: params.radius,
          location: params.userLocation
        }},
        { terms: { certificationLevel: params.certificationLevels } }
      ],
      should: [
        { match: { accommodations: params.requiredAccommodations } }
      ]
    }
  };
  
  // Apply personalization based on user preferences
  return personalizeResults(elasticsearchQuery(query), user);
}
```

### 2.4 Works Service (Employment)

#### 2.4.1 Job Matching Engine

**Matching Algorithm**:
```python
class NeurodiverseJobMatcher:
    def __init__(self):
        self.weights = {
            'skills_match': 0.35,
            'accommodation_fit': 0.25,
            'communication_style': 0.20,
            'environment_match': 0.15,
            'interest_alignment': 0.05
        }
    
    def calculate_match_score(self, user: User, job: Job) -> float:
        scores = {
            'skills_match': self._skills_similarity(user.skills, job.requirements),
            'accommodation_fit': self._accommodation_compatibility(
                user.needs, job.accommodations_offered
            ),
            'communication_style': self._communication_match(
                user.communication_pref, job.communication_requirements
            ),
            'environment_match': self._environmental_fit(
                user.sensory_prefs, job.work_environment
            ),
            'interest_alignment': self._interest_overlap(
                user.interests, job.domain
            )
        }
        
        return sum(score * self.weights[key] 
                  for key, score in scores.items())
    
    def get_explanations(self, score_breakdown: dict) -> list:
        """Generate human-readable explanations for match"""
        explanations = []
        if score_breakdown['accommodation_fit'] > 0.8:
            explanations.append("Excellent accommodation support available")
        if score_breakdown['environment_match'] > 0.7:
            explanations.append("Work environment matches your sensory preferences")
        return explanations
```

#### 2.4.2 Alternative Assessment Framework

```typescript
interface AlternativeAssessment {
  type: 'work_sample' | 'project' | 'trial_period' | 'skills_demo';
  
  workSample?: {
    task: string;
    duration: number; // hours allowed
    realWorldScenario: boolean;
    evaluationCriteria: string[];
  };
  
  project?: {
    description: string;
    timeline: number; // days
    milestones: Milestone[];
    collaborationType: 'solo' | 'pair' | 'team';
  };
  
  trialPeriod?: {
    duration: number; // days
    paid: boolean;
    mentorAssigned: boolean;
    adjustmentPeriod: number; // additional days for settling
  };
}
```

### 2.5 Love Service (Dating)

#### 2.5.1 Matching Algorithm

**Compatibility Scoring**:
```python
class NeurodiverseDatingMatcher:
    def __init__(self):
        self.model = self._load_ml_model('compatibility_v2.h5')
        
    def calculate_compatibility(self, user_a: User, user_b: User) -> dict:
        # Core compatibility factors
        factors = {
            'communication_style': self._communication_compatibility(
                user_a.communication, user_b.communication
            ),
            'sensory_alignment': self._sensory_compatibility(
                user_a.sensory, user_b.sensory
            ),
            'interest_overlap': self._interest_similarity(
                user_a.interests, user_b.interests
            ),
            'relationship_pace': self._pace_compatibility(
                user_a.relationship_pace, user_b.relationship_pace
            ),
            'values_alignment': self._values_match(
                user_a.values, user_b.values
            )
        }
        
        # ML prediction for nuanced compatibility
        ml_score = self.model.predict([
            self._vectorize_profiles(user_a, user_b)
        ])[0]
        
        return {
            'overall_score': ml_score,
            'factors': factors,
            'confidence': self._calculate_confidence(factors)
        }
```

#### 2.5.2 Safety Protocols

```typescript
interface SafetyProtocol {
  // Identity Verification Levels
  verification: {
    level1: { // Basic
      email: boolean;
      phone: boolean;
    };
    level2: { // Standard
      photoId: boolean;
      selfieVerification: boolean;
    };
    level3: { // Enhanced
      backgroundCheck: boolean;
      references: boolean;
    };
  };
  
  // Interaction Safety
  messaging: {
    contentModeration: 'ai' | 'human' | 'both';
    escalationDetection: boolean;
    templateSuggestions: boolean;
    paceControl: {
      minMessageInterval: number; // minutes
      dailyMessageLimit?: number;
    };
  };
  
  // Meeting Safety
  inPerson: {
    publicPlaceRequired: boolean;
    locationSharing: boolean;
    checkInProtocol: {
      enabled: boolean;
      intervals: number[]; // minutes
      emergencyContact: Contact;
    };
    panicButton: {
      enabled: boolean;
      action: 'call_911' | 'alert_contact' | 'both';
    };
  };
}
```

### 2.6 AI/ML Components

#### 2.6.1 Matching Engine Architecture

```python
class UniversalMatchingEngine:
    """Central matching engine for all modules"""
    
    def __init__(self):
        self.embeddings_model = self._load_embeddings_model()
        self.ranking_model = self._load_ranking_model()
        
    def generate_embeddings(self, entity: Union[User, Business, Job, Event]):
        """Generate vector embeddings for any entity"""
        features = self._extract_features(entity)
        return self.embeddings_model.encode(features)
    
    def find_matches(self, 
                    source: Entity, 
                    candidates: List[Entity], 
                    match_type: str) -> List[Match]:
        """Find best matches using vector similarity"""
        source_embedding = self.generate_embeddings(source)
        candidate_embeddings = [self.generate_embeddings(c) for c in candidates]
        
        # Calculate similarities
        similarities = cosine_similarity(source_embedding, candidate_embeddings)
        
        # Apply domain-specific ranking
        ranked = self.ranking_model.rank(
            similarities, 
            match_type=match_type,
            user_preferences=source.preferences
        )
        
        return ranked[:MAX_RESULTS]
```

#### 2.6.2 Safety AI Module

```python
class SafetyAI:
    """AI-powered safety monitoring"""
    
    def __init__(self):
        self.text_classifier = self._load_text_safety_model()
        self.pattern_detector = self._load_pattern_model()
        self.image_screener = self._load_image_model()
        
    def analyze_conversation(self, messages: List[Message]) -> SafetyAnalysis:
        """Detect concerning patterns in conversations"""
        
        risks = {
            'harassment': self._detect_harassment(messages),
            'grooming': self._detect_grooming_patterns(messages),
            'exploitation': self._detect_exploitation(messages),
            'crisis': self._detect_crisis_language(messages)
        }
        
        return SafetyAnalysis(
            risk_level=self._calculate_risk_level(risks),
            specific_concerns=risks,
            recommended_action=self._recommend_action(risks)
        )
    
    def screen_image(self, image: bytes) -> ImageSafetyResult:
        """Screen images for inappropriate content"""
        return self.image_screener.classify(image)
```

## 3. Non-Functional Requirements

### 3.1 Performance Requirements

| Component | Metric | Requirement | Measurement |
|-----------|--------|-------------|-------------|
| **API Response** | Latency | <200ms p95 | Datadog APM |
| **Search** | Query time | <100ms p99 | Elasticsearch metrics |
| **Matching** | Calculation time | <500ms per match | Custom metrics |
| **Real-time updates** | Delivery | <50ms | WebSocket monitoring |
| **Page load** | Time to Interactive | <2s on 3G | Lighthouse |
| **Mobile app** | Cold start | <3s | Firebase Performance |
| **Database** | Query performance | <50ms p95 | PG stats |
| **Throughput** | Requests/sec | 10,000 RPS | Load testing |

### 3.2 Scalability Requirements

```yaml
scaling_parameters:
  horizontal_scaling:
    min_instances: 3
    max_instances: 100
    target_cpu: 70%
    target_memory: 80%
    
  database_scaling:
    read_replicas: 5
    connection_pool: 200
    sharding_strategy: "user_id_hash"
    
  cache_scaling:
    redis_cluster: 6 nodes
    memcached: 10GB per node
    
  expected_load:
    year_1: 100K users
    year_2: 300K users
    year_3: 500K users
    peak_multiplier: 5x
```

### 3.3 Security Requirements

#### 3.3.1 Data Encryption

```yaml
encryption:
  at_rest:
    database: AES-256-GCM
    file_storage: AES-256-GCM
    backups: AES-256-GCM with HSM keys
    
  in_transit:
    external: TLS 1.3 minimum
    internal: mTLS between services
    websocket: WSS only
    
  sensitive_data:
    pii: Field-level encryption
    payment: PCI DSS compliant tokenization
    health_data: HIPAA compliant encryption
```

#### 3.3.2 Security Controls

| Control | Implementation | Compliance |
|---------|---------------|------------|
| **Authentication** | MFA, biometrics, FIDO2 | NIST 800-63B |
| **Authorization** | RBAC + ABAC | OWASP |
| **Input validation** | Schema validation, sanitization | OWASP Top 10 |
| **SQL injection** | Parameterized queries, ORMs | CWE-89 |
| **XSS prevention** | CSP headers, output encoding | CWE-79 |
| **CSRF protection** | Tokens, SameSite cookies | CWE-352 |
| **Rate limiting** | Per-user, per-IP, per-endpoint | DDoS mitigation |
| **Audit logging** | Immutable logs, SIEM integration | SOC 2 Type II |

### 3.4 Accessibility Requirements (WCAG 2.2 AAA)

```typescript
interface AccessibilityRequirements {
  // Perceivable
  perceivable: {
    textAlternatives: 'all_non_text_content';
    captions: 'all_video_content';
    audioDescriptions: 'all_visual_content';
    contrastRatio: {
      normalText: 7.0, // AAA
      largeText: 4.5,  // AAA
      nonText: 3.0      // AAA
    };
    textSpacing: {
      lineHeight: 1.5;
      paragraphSpacing: '2em';
      letterSpacing: '0.12em';
      wordSpacing: '0.16em';
    };
  };
  
  // Operable
  operable: {
    keyboardAccessible: 'all_functionality';
    timeAdjustable: true;
    seizurePrevention: {
      flashThreshold: '3_per_second';
      animationControl: true;
    };
    consistentNavigation: true;
    focusVisible: 'enhanced';
  };
  
  // Understandable
  understandable: {
    readingLevel: 'lower_secondary_education';
    predictableNavigation: true;
    inputAssistance: true;
    errorSuggestions: true;
    contextSensitiveHelp: true;
  };
  
  // Robust
  robust: {
    parsingValid: true;
    nameRolValue: 'all_ui_components';
    statusMessages: true;
  };
  
  // Neurodivergent-Specific
  neurodivergentEnhancements: {
    simplifiedMode: true;
    reducedMotion: true;
    plainLanguage: true;
    visualSchedules: true;
    predictableLayout: true;
    sensoryControls: true;
  };
}
```

## 4. System Integrations

### 4.1 Third-Party Services

| Service | Purpose | Integration Method | Fallback |
|---------|---------|-------------------|----------|
| **Twilio** | SMS, voice calls | REST API | SendGrid SMS |
| **Stripe** | Payments | SDK + webhooks | PayPal |
| **Auth0** | SSO, enterprise auth | OAuth 2.0 | Internal auth |
| **Mapbox** | Mapping, geocoding | JavaScript SDK | Google Maps |
| **AWS Rekognition** | Image moderation | REST API | Google Vision |
| **SendGrid** | Email delivery | REST API | AWS SES |
| **Pusher** | Real-time messaging | WebSocket | Firebase |
| **Algolia** | Search (backup) | REST API | Elasticsearch |

### 4.2 API Specifications

#### 4.2.1 GraphQL Schema (Excerpt)

```graphql
type User {
  id: ID!
  profile: UserProfile!
  preferences: UserPreferences!
  connections: [Connection!]!
  matches(type: MatchType, limit: Int): [Match!]!
  safety: SafetySettings!
}

type Business {
  id: ID!
  name: String!
  certification: Certification
  accessibility: AccessibilityInfo!
  sensoryEnvironment: SensoryEnvironment!
  ratings: BusinessRatings!
  jobs: [Job!]!
  events: [Event!]!
}

type Match {
  id: ID!
  user: User!
  matchedEntity: MatchEntity!
  score: Float!
  explanation: [String!]!
  createdAt: DateTime!
}

union MatchEntity = User | Job | Business | Event

type Query {
  me: User!
  searchBusinesses(filters: BusinessFilters!, location: Location!): [Business!]!
  getMatches(type: MatchType!, limit: Int): [Match!]!
}

type Mutation {
  updateProfile(input: ProfileInput!): User!
  reportSafetyConcern(input: SafetyReportInput!): SafetyReport!
  initiateMatch(targetId: ID!, type: MatchType!): Match
}

type Subscription {
  messageReceived(chatId: ID!): Message!
  matchUpdate(userId: ID!): Match!
  safetyAlert(userId: ID!): SafetyAlert!
}
```

#### 4.2.2 REST API Standards

```yaml
api_standards:
  versioning: URI versioning (/api/v1/)
  authentication: Bearer token (JWT)
  
  response_format:
    success:
      status: 200|201|204
      body:
        data: object|array
        meta:
          pagination: object
          timestamp: ISO8601
    
    error:
      status: 400|401|403|404|500
      body:
        error:
          code: string
          message: string
          details: array
          timestamp: ISO8601
  
  rate_limits:
    anonymous: 100/hour
    authenticated: 1000/hour
    premium: 5000/hour
    business: 10000/hour
```

## 5. Database Design

### 5.1 Core Schema

```sql
-- Users table with neurodivergent-specific fields
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    phone_number VARCHAR(20),
    date_of_birth DATE NOT NULL,
    neurotype JSONB, -- Flexible neurotype information
    communication_preferences JSONB NOT NULL,
    sensory_preferences JSONB NOT NULL,
    interests JSONB,
    safety_settings JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP, -- Soft delete
    CONSTRAINT age_check CHECK (date_of_birth <= CURRENT_DATE - INTERVAL '13 years')
);

-- Businesses table with certification tracking
CREATE TABLE businesses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    category VARCHAR(100) NOT NULL,
    location GEOGRAPHY(POINT, 4326), -- PostGIS for geo queries
    address JSONB NOT NULL,
    certification_level VARCHAR(20) DEFAULT 'none',
    certification_valid_until DATE,
    sensory_environment JSONB NOT NULL,
    accommodations JSONB,
    real_time_data JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Jobs table with accommodation transparency
CREATE TABLE jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    business_id UUID REFERENCES businesses(id),
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    requirements JSONB,
    accommodations_offered JSONB,
    work_environment JSONB,
    salary_range JSONB,
    assessment_options JSONB,
    status VARCHAR(20) DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP
);

-- Matches table (polymorphic for all match types)
CREATE TABLE matches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    matched_entity_type VARCHAR(20) NOT NULL, -- 'user', 'job', 'event'
    matched_entity_id UUID NOT NULL,
    match_score DECIMAL(3,2),
    explanation JSONB,
    status VARCHAR(20) DEFAULT 'pending',
    user_action VARCHAR(20), -- 'liked', 'passed', 'blocked'
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Safety reports table
CREATE TABLE safety_reports (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    reporter_id UUID REFERENCES users(id),
    reported_entity_type VARCHAR(20) NOT NULL,
    reported_entity_id UUID NOT NULL,
    reason VARCHAR(100) NOT NULL,
    details TEXT,
    evidence JSONB, -- Screenshots, message IDs, etc.
    status VARCHAR(20) DEFAULT 'pending',
    moderator_id UUID REFERENCES users(id),
    resolution JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    resolved_at TIMESTAMP
);

-- Create indexes for performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_neurotype ON users USING GIN(neurotype);
CREATE INDEX idx_businesses_location ON businesses USING GIST(location);
CREATE INDEX idx_businesses_certification ON businesses(certification_level);
CREATE INDEX idx_jobs_business ON jobs(business_id);
CREATE INDEX idx_jobs_status ON jobs(status);
CREATE INDEX idx_matches_user ON matches(user_id);
CREATE INDEX idx_matches_entity ON matches(matched_entity_type, matched_entity_id);
CREATE INDEX idx_safety_reports_status ON safety_reports(status);