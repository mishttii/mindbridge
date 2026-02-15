# Design Document: MindfulSpace Mental Wellness Ecosystem

## Overview

MindfulSpace is a full-stack mental wellness application built with a modern web architecture. The system implements a structured support flow (Trigger → Logging → AI Intervention → Resolution) to provide users with timely emotional support. The architecture separates concerns into three main layers: presentation (React/Next.js frontend), business logic (Node.js backend), and data persistence (MongoDB).

The system prioritizes user privacy through secure authentication and anonymous community features, while leveraging AI-powered analysis to provide personalized mental health resources. The design emphasizes scalability, maintainability, and a calming user experience appropriate for mental wellness applications.

## Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Client Layer (Browser)                   │
│  ┌────────────────────────────────────────────────────────┐ │
│  │         React/Next.js Frontend Application             │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐ │ │
│  │  │   Mood   │  │Community │  │   Resource           │ │ │
│  │  │ Tracking │  │  Feed    │  │ Recommendations      │ │ │
│  │  │   UI     │  │   UI     │  │      UI              │ │ │
│  │  └──────────┘  └──────────┘  └──────────────────────┘ │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            │
                    HTTPS/REST API
                            │
┌─────────────────────────────────────────────────────────────┐
│                   Application Layer (Node.js)                │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              Express.js API Server                     │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │ │
│  │  │     Auth     │  │     Mood     │  │  Community  │ │ │
│  │  │   Service    │  │   Service    │  │   Service   │ │ │
│  │  └──────────────┘  └──────────────┘  └─────────────┘ │ │
│  │  ┌──────────────┐  ┌──────────────┐                  │ │
│  │  │  AI Analysis │  │   Resource   │                  │ │
│  │  │   Service    │  │   Service    │                  │ │
│  │  └──────────────┘  └──────────────┘                  │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            │
                      MongoDB Driver
                            │
┌─────────────────────────────────────────────────────────────┐
│                    Data Layer (MongoDB)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │    Users     │  │  Mood Logs   │  │ Community Posts │  │
│  │  Collection  │  │  Collection  │  │   Collection    │  │
│  └──────────────┘  └──────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Process Flow Architecture

The system implements a four-stage support flow:

1. **Trigger Identification**: User recognizes an emotional trigger
2. **Logging**: User records mood, trigger category, and intensity
3. **AI Intervention**: System analyzes entry and provides recommendations
4. **Resolution**: User engages with resources and logs outcome

This flow is implemented through coordinated service interactions:

```
User Action → Mood Service → AI Analysis Service → Resource Service → User
```


## Components and Interfaces

### Frontend Components

#### 1. Authentication Module

**Responsibilities:**
- User registration and login forms
- Session management
- Token storage and refresh

**Key Components:**
- `LoginForm`: Handles user authentication
- `RegisterForm`: Handles new user registration
- `AuthContext`: Manages authentication state across the application

**Interface:**
```typescript
interface AuthService {
  login(email: string, password: string): Promise<AuthToken>
  register(email: string, password: string, name: string): Promise<User>
  logout(): Promise<void>
  refreshToken(): Promise<AuthToken>
  getCurrentUser(): User | null
}
```

#### 2. Mood Tracking Module

**Responsibilities:**
- Mood entry form with trigger selection and intensity slider
- Mood history display
- 7-Day Summary visualization

**Key Components:**
- `MoodEntryForm`: Input form for logging moods
- `MoodHistory`: List view of past mood logs
- `SevenDaySummary`: Chart visualization of emotional baseline
- `IntensitySlider`: 0-10 scale input component

**Interface:**
```typescript
interface MoodLog {
  id: string
  userId: string
  mood: string
  trigger: 'Work' | 'Family' | 'Health'
  intensity: number // 0-10
  timestamp: Date
  notes?: string
}

interface MoodTrackingService {
  createMoodLog(log: Omit<MoodLog, 'id' | 'timestamp'>): Promise<MoodLog>
  getMoodLogs(userId: string, days?: number): Promise<MoodLog[]>
  getSevenDaySummary(userId: string): Promise<SummaryData>
}
```

#### 3. Community Module

**Responsibilities:**
- Anonymous post creation
- Community feed display
- Post reporting mechanism

**Key Components:**
- `AnonymousPostForm`: Form for creating anonymous posts
- `CommunityFeed`: Scrollable list of approved posts
- `PostCard`: Individual post display component
- `ReportButton`: Mechanism for flagging inappropriate content

**Interface:**
```typescript
interface AnonymousPost {
  id: string
  content: string
  timestamp: Date
  moderationStatus: 'pending' | 'approved' | 'rejected'
  reportCount: number
}

interface CommunityService {
  createPost(content: string, sessionId: string): Promise<AnonymousPost>
  getApprovedPosts(limit?: number, offset?: number): Promise<AnonymousPost[]>
  reportPost(postId: string): Promise<void>
}
```

#### 4. Resource Recommendation Module

**Responsibilities:**
- Display grounding exercises
- Show professional resources
- Track exercise completion

**Key Components:**
- `ResourceCard`: Display individual resource recommendations
- `GroundingExerciseDetail`: Step-by-step exercise instructions
- `ProfessionalResourceList`: Crisis contacts and therapist directories
- `CompletionTracker`: Log exercise completion and effectiveness

**Interface:**
```typescript
interface GroundingExercise {
  id: string
  title: string
  description: string
  steps: string[]
  duration: number // minutes
  triggerCategories: string[]
}

interface ProfessionalResource {
  id: string
  name: string
  type: 'helpline' | 'crisis' | 'therapist'
  contact: string
  description: string
  availability: string
}

interface ResourceService {
  getRecommendedExercises(trigger: string, intensity: number): Promise<GroundingExercise[]>
  getProfessionalResources(urgency: 'low' | 'medium' | 'high'): Promise<ProfessionalResource[]>
  logExerciseCompletion(exerciseId: string, userId: string, effectiveness: number): Promise<void>
}
```

### Backend Services

#### 1. Authentication Service

**Responsibilities:**
- User registration with password hashing
- Login authentication with JWT token generation
- Token validation and refresh
- Password reset functionality

**Implementation Details:**
- Uses bcrypt for password hashing (minimum 10 salt rounds)
- JWT tokens with 24-hour expiration
- Refresh tokens with 7-day expiration
- Secure HTTP-only cookies for token storage

**API Endpoints:**
```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/logout
POST /api/auth/refresh
POST /api/auth/reset-password
```

#### 2. Mood Service

**Responsibilities:**
- Create and store mood logs
- Retrieve mood history
- Generate 7-day summary data
- Validate mood log inputs

**Implementation Details:**
- Validates intensity values (0-10 range)
- Validates trigger categories (Work, Family, Health)
- Associates logs with authenticated user
- Implements pagination for mood history

**API Endpoints:**
```
POST /api/moods
GET /api/moods/:userId
GET /api/moods/:userId/summary
GET /api/moods/:userId/history?days=7
```

#### 3. AI Analysis Service

**Responsibilities:**
- Analyze mood patterns
- Identify frequent triggers
- Calculate average intensity by trigger
- Detect high-intensity patterns
- Trigger resource recommendations

**Implementation Details:**
- Analyzes logs from past 7 days for pattern detection
- Flags consecutive high-intensity logs (>7) within 48 hours
- Calculates trigger frequency and average intensity
- Determines urgency level for resource allocation

**Analysis Algorithm:**
```
function analyzeMoodPattern(logs: MoodLog[]): AnalysisResult {
  // Calculate trigger frequencies
  triggerCounts = countByTrigger(logs)
  mostFrequentTrigger = max(triggerCounts)
  
  // Calculate average intensities
  avgIntensities = averageIntensityByTrigger(logs)
  
  // Detect high-intensity patterns
  highIntensityLogs = filter(logs, log => log.intensity > 7)
  consecutiveHighIntensity = detectConsecutiveWithin48Hours(highIntensityLogs)
  
  // Determine urgency
  urgency = calculateUrgency(avgIntensities, consecutiveHighIntensity)
  
  return {
    mostFrequentTrigger,
    avgIntensities,
    highIntensityCount: highIntensityLogs.length,
    urgency
  }
}
```

#### 4. Community Service

**Responsibilities:**
- Create anonymous posts with session-based tracking
- Retrieve approved posts for feed
- Implement moderation queue
- Handle post reporting
- Maintain anonymity guarantees

**Implementation Details:**
- Generates temporary session IDs for moderation (not linked to user accounts)
- Stores posts with moderation status (pending/approved/rejected)
- Implements keyword filtering for automatic flagging
- Maintains separate moderation logs
- Ensures no user-identifying information in post records

**API Endpoints:**
```
POST /api/community/posts
GET /api/community/posts?status=approved&limit=20&offset=0
POST /api/community/posts/:postId/report
GET /api/community/moderation/queue (admin only)
PUT /api/community/moderation/:postId/approve (admin only)
PUT /api/community/moderation/:postId/reject (admin only)
```

#### 5. Resource Service

**Responsibilities:**
- Recommend grounding exercises based on trigger and intensity
- Provide professional resources based on urgency
- Store and retrieve exercise library
- Log exercise completions
- Maintain India-specific resource directory

**Implementation Details:**
- Exercise recommendation algorithm considers trigger category and intensity
- Professional resources prioritized by urgency level
- Maintains curated library of grounding exercises
- Stores India-specific helplines and crisis contacts

**Recommendation Algorithm:**
```
function recommendExercises(trigger: string, intensity: number): GroundingExercise[] {
  // Filter exercises by trigger category
  relevantExercises = filterByTrigger(exerciseLibrary, trigger)
  
  // Sort by appropriateness for intensity level
  if (intensity >= 7) {
    // High intensity: prioritize immediate calming techniques
    return sortBy(relevantExercises, 'immediateEffectiveness')
  } else if (intensity >= 4) {
    // Medium intensity: balanced approach
    return sortBy(relevantExercises, 'versatility')
  } else {
    // Low intensity: preventive techniques
    return sortBy(relevantExercises, 'preventive')
  }
}

function recommendProfessionalResources(urgency: string): ProfessionalResource[] {
  if (urgency === 'high') {
    // Crisis resources first
    return filterByType(resourceDirectory, ['crisis', 'helpline'])
  } else if (urgency === 'medium') {
    // Mix of helplines and therapist directories
    return resourceDirectory
  } else {
    // General therapist directories
    return filterByType(resourceDirectory, ['therapist'])
  }
}
```

**API Endpoints:**
```
GET /api/resources/exercises?trigger=Work&intensity=8
GET /api/resources/professional?urgency=high
POST /api/resources/exercises/:exerciseId/complete
GET /api/resources/exercises/:exerciseId
```


## Data Models

### User Model

```typescript
interface User {
  _id: ObjectId
  email: string // unique, indexed
  passwordHash: string
  name: string
  createdAt: Date
  lastLogin: Date
  preferences: {
    notifications: boolean
    dataSharing: boolean
  }
}
```

**MongoDB Schema:**
```javascript
{
  email: { type: String, required: true, unique: true, index: true },
  passwordHash: { type: String, required: true },
  name: { type: String, required: true },
  createdAt: { type: Date, default: Date.now },
  lastLogin: { type: Date },
  preferences: {
    notifications: { type: Boolean, default: true },
    dataSharing: { type: Boolean, default: false }
  }
}
```

### MoodLog Model

```typescript
interface MoodLog {
  _id: ObjectId
  userId: ObjectId // indexed, references User
  mood: string
  trigger: 'Work' | 'Family' | 'Health'
  intensity: number // 0-10
  timestamp: Date // indexed
  notes?: string
  aiAnalysis?: {
    recommendedExercises: ObjectId[]
    urgencyLevel: string
    analyzedAt: Date
  }
}
```

**MongoDB Schema:**
```javascript
{
  userId: { type: ObjectId, required: true, ref: 'User', index: true },
  mood: { type: String, required: true },
  trigger: { 
    type: String, 
    required: true, 
    enum: ['Work', 'Family', 'Health'] 
  },
  intensity: { 
    type: Number, 
    required: true, 
    min: 0, 
    max: 10 
  },
  timestamp: { type: Date, default: Date.now, index: true },
  notes: { type: String },
  aiAnalysis: {
    recommendedExercises: [{ type: ObjectId, ref: 'GroundingExercise' }],
    urgencyLevel: { type: String, enum: ['low', 'medium', 'high'] },
    analyzedAt: { type: Date }
  }
}
```

**Indexes:**
- Compound index on `(userId, timestamp)` for efficient history queries
- Index on `timestamp` for 7-day summary queries

### AnonymousPost Model

```typescript
interface AnonymousPost {
  _id: ObjectId
  content: string
  sessionId: string // temporary, for moderation only
  timestamp: Date // indexed
  moderationStatus: 'pending' | 'approved' | 'rejected'
  moderatedAt?: Date
  moderatedBy?: ObjectId // references admin user
  reportCount: number
  flaggedKeywords: string[]
}
```

**MongoDB Schema:**
```javascript
{
  content: { type: String, required: true },
  sessionId: { type: String, required: true }, // NOT linked to userId
  timestamp: { type: Date, default: Date.now, index: true },
  moderationStatus: { 
    type: String, 
    required: true, 
    enum: ['pending', 'approved', 'rejected'],
    default: 'pending',
    index: true
  },
  moderatedAt: { type: Date },
  moderatedBy: { type: ObjectId, ref: 'User' },
  reportCount: { type: Number, default: 0 },
  flaggedKeywords: [{ type: String }]
}
```

**Indexes:**
- Compound index on `(moderationStatus, timestamp)` for feed queries
- Index on `moderationStatus` for moderation queue

**Anonymity Guarantee:**
- No direct or indirect reference to User._id
- sessionId is generated per-session and not stored with user records
- Moderation logs stored separately without user correlation

### GroundingExercise Model

```typescript
interface GroundingExercise {
  _id: ObjectId
  title: string
  description: string
  steps: string[]
  duration: number // minutes
  triggerCategories: string[]
  intensityRange: {
    min: number
    max: number
  }
  effectiveness: {
    immediate: number // 0-10 rating
    preventive: number // 0-10 rating
    versatility: number // 0-10 rating
  }
  createdAt: Date
}
```

### ProfessionalResource Model

```typescript
interface ProfessionalResource {
  _id: ObjectId
  name: string
  type: 'helpline' | 'crisis' | 'therapist'
  contact: string
  description: string
  availability: string
  region: string // India-specific regions
  language: string[]
  urgencyLevel: string[]
  verified: boolean
  lastVerified: Date
}
```

### ExerciseCompletion Model

```typescript
interface ExerciseCompletion {
  _id: ObjectId
  userId: ObjectId // indexed
  exerciseId: ObjectId
  completedAt: Date
  effectiveness: number // 0-10 user rating
  notes?: string
}
```


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Authentication and Security Properties

**Property 1: Registration and login round-trip**
*For any* valid user credentials (email, password, name), registering a new account and then logging in with those credentials should successfully authenticate the user and grant platform access.
**Validates: Requirements 1.1, 1.2**

**Property 2: Invalid credentials rejection**
*For any* invalid credentials (wrong password, non-existent email, malformed input), authentication attempts should be rejected with an appropriate error message.
**Validates: Requirements 1.3**

**Property 3: Password encryption invariant**
*For any* user account in the database, the stored passwordHash should never equal the plaintext password provided during registration.
**Validates: Requirements 1.4**

### Mood Tracking Properties

**Property 4: Mood log creation and persistence**
*For any* authenticated user and valid mood entry (trigger in {Work, Family, Health}, intensity in [0, 10]), creating a mood log should result in a persisted record associated with that user's account, retrievable in chronological order.
**Validates: Requirements 2.1, 2.5, 2.6**

**Property 5: Trigger category validation**
*For any* trigger value submitted in a mood entry, the system should accept it if and only if it is one of the valid categories: Work, Family, or Health.
**Validates: Requirements 2.2**

**Property 6: Intensity range validation**
*For any* intensity value submitted in a mood entry, the system should accept it if and only if the value is a number between 0 and 10 inclusive.
**Validates: Requirements 2.3, 2.4**

**Property 7: Seven-day summary correctness**
*For any* user with mood logs, the Seven_Day_Summary should include exactly those logs with timestamps within the past 7 days, visualize all trigger categories and intensity levels from those logs, and immediately reflect any newly created mood log.
**Validates: Requirements 3.1, 3.2, 3.5**

### AI Analysis Properties

**Property 8: Most frequent trigger identification**
*For any* user with at least three mood logs, the AI analysis should correctly identify the trigger category that appears most frequently in their logs.
**Validates: Requirements 4.1**

**Property 9: Average intensity calculation**
*For any* set of mood logs grouped by trigger category, the calculated average intensity for each trigger should equal the mathematical mean of all intensity values for that trigger.
**Validates: Requirements 4.2**

**Property 10: High-intensity flagging**
*For any* mood log with intensity level greater than 7, the system should flag the entry as high-intensity.
**Validates: Requirements 4.3**

**Property 11: Consecutive high-intensity pattern detection**
*For any* sequence of mood logs containing multiple entries with intensity > 7 where the time difference between consecutive high-intensity logs is less than 48 hours, the system should detect this as an elevated distress pattern and trigger resource recommendations.
**Validates: Requirements 4.4, 4.5**

### Community and Anonymity Properties

**Property 12: Anonymity guarantee**
*For any* anonymous post created by a user, the post record in the database should not contain the user's ID, and any query or display of the post should not reveal user-identifying information. The post should have a sessionId that is distinct from any userId.
**Validates: Requirements 5.1, 5.2, 5.3, 5.4**

**Property 13: Community feed ordering**
*For any* set of approved anonymous posts, the community feed should display them in reverse chronological order (newest first).
**Validates: Requirements 5.5**

**Property 14: Moderation initial state**
*For any* newly submitted anonymous post, its initial moderationStatus should be 'pending'.
**Validates: Requirements 6.1**

**Property 15: Moderation status visibility**
*For any* anonymous post, it should appear in the public community feed if and only if its moderationStatus is 'approved'. Posts with status 'pending' or 'rejected' should not be visible to regular users.
**Validates: Requirements 6.2, 6.3**

**Property 16: Harmful keyword flagging**
*For any* post containing keywords from the harmful keywords list, the system should flag the post for priority moderation review.
**Validates: Requirements 6.4**

**Property 17: Post reporting escalation**
*For any* post that receives a report, the system should increment its reportCount and ensure it is flagged for moderation review.
**Validates: Requirements 6.5**

### Resource Allocation Properties

**Property 18: Exercise recommendation completeness**
*For any* mood log with intensity level greater than 5, the Resource_Allocator should recommend at least one grounding exercise that matches the logged trigger category and includes clear step-by-step instructions.
**Validates: Requirements 7.1, 7.2, 7.3**

**Property 19: Exercise completion logging**
*For any* grounding exercise that a user completes, the system should create and persist an ExerciseCompletion record with the user's effectiveness rating.
**Validates: Requirements 7.5**

**Property 20: Professional resource recommendation trigger**
*For any* user who logs consecutive mood entries with intensity > 7 within a 48-hour period, the Resource_Allocator should recommend professional resources including helpline numbers, crisis support contacts, and therapist directories.
**Validates: Requirements 8.1, 8.2**

**Property 21: India-specific resource prioritization**
*For any* user located in Bharat (India), the recommended professional resources should prioritize India-specific mental health resources with appropriate regional information and language support.
**Validates: Requirements 8.3**

**Property 22: Professional resource completeness**
*For any* professional resource recommendation, each resource should include a clear description, contact method, and availability information. Crisis support resources should be immediately accessible.
**Validates: Requirements 8.4, 8.5**

### Data Persistence Properties

**Property 23: Cross-session data persistence**
*For any* user account, all mood logs, preferences, and exercise completions should persist across sessions and be retrievable when the user logs in from any device.
**Validates: Requirements 9.1, 9.2, 9.3**

**Property 24: Process flow orchestration**
*For any* mood log creation, the system should immediately invoke the AI analysis service, and if the analysis indicates need for support, the Resource_Allocator should be triggered to provide appropriate recommendations.
**Validates: Requirements 11.2, 11.3**

**Property 25: Activity completion persistence**
*For any* recommended activity (grounding exercise or professional resource engagement) that a user completes, the system should accept and persist the completion record with resolution status.
**Validates: Requirements 11.4**

### Accessibility Properties

**Property 26: Interactive element labeling**
*For any* interactive UI element (buttons, inputs, links), the element should have appropriate labels and ARIA attributes for accessibility.
**Validates: Requirements 10.3**

**Property 27: Loading state indication**
*For any* asynchronous data operation, the UI should display a loading indicator while the operation is in progress.
**Validates: Requirements 10.6**


## Error Handling

### Authentication Errors

**Invalid Credentials:**
- Return 401 Unauthorized with message: "Invalid email or password"
- Do not reveal whether email exists (security best practice)

**Duplicate Registration:**
- Return 409 Conflict with message: "An account with this email already exists"

**Token Expiration:**
- Return 401 Unauthorized with message: "Session expired, please log in again"
- Frontend should automatically redirect to login page

**Password Reset:**
- Always return success message even if email doesn't exist (security best practice)
- Send reset email only if account exists

### Mood Tracking Errors

**Invalid Trigger Category:**
- Return 400 Bad Request with message: "Trigger must be one of: Work, Family, Health"

**Invalid Intensity:**
- Return 400 Bad Request with message: "Intensity must be a number between 0 and 10"

**Missing Required Fields:**
- Return 400 Bad Request with message: "Missing required field: [field_name]"

**Unauthorized Access:**
- Return 403 Forbidden when user attempts to access another user's mood logs

### Community Errors

**Empty Post Content:**
- Return 400 Bad Request with message: "Post content cannot be empty"

**Post Too Long:**
- Return 400 Bad Request with message: "Post content exceeds maximum length of 2000 characters"

**Post Not Found:**
- Return 404 Not Found with message: "Post not found or has been removed"

**Unauthorized Moderation:**
- Return 403 Forbidden when non-admin attempts moderation actions

### Resource Allocation Errors

**No Recommendations Available:**
- Return empty array with message: "No exercises currently available for this trigger"
- Log warning for admin review

**Exercise Not Found:**
- Return 404 Not Found with message: "Exercise not found"

**Invalid Effectiveness Rating:**
- Return 400 Bad Request with message: "Effectiveness rating must be between 0 and 10"

### Database Errors

**Connection Failure:**
- Return 503 Service Unavailable with message: "Service temporarily unavailable, please try again"
- Log error with full stack trace for debugging
- Implement retry logic with exponential backoff

**Validation Errors:**
- Return 400 Bad Request with specific validation error messages
- Include field name and validation rule that failed

**Duplicate Key Errors:**
- Return 409 Conflict with message indicating which unique constraint was violated

### General Error Handling Strategy

**Error Response Format:**
```typescript
interface ErrorResponse {
  error: {
    code: string // Machine-readable error code
    message: string // Human-readable error message
    details?: any // Optional additional context
    timestamp: Date
  }
}
```

**Logging:**
- Log all errors with severity levels (ERROR, WARN, INFO)
- Include request ID for tracing
- Sanitize sensitive information (passwords, tokens) from logs

**User-Facing Messages:**
- Keep messages clear and actionable
- Avoid technical jargon
- Provide guidance on how to resolve the issue when possible

**Graceful Degradation:**
- If AI analysis fails, still allow mood logging
- If resource recommendations fail, show cached/default resources
- If community feed fails, show cached posts with staleness indicator

## Testing Strategy

### Overview

The testing strategy employs a dual approach combining unit tests for specific scenarios and property-based tests for comprehensive coverage. This ensures both concrete examples work correctly and universal properties hold across all inputs.

### Property-Based Testing

**Framework:** fast-check (for JavaScript/TypeScript)

**Configuration:**
- Minimum 100 iterations per property test
- Each test tagged with feature name and property number
- Tag format: `Feature: mindful-space-wellness-ecosystem, Property N: [property description]`

**Property Test Coverage:**

1. **Authentication Properties (1-3):**
   - Generate random valid credentials for registration/login round-trip
   - Generate invalid credentials (malformed emails, short passwords, SQL injection attempts)
   - Verify password hashing by comparing stored hash to plaintext

2. **Mood Tracking Properties (4-7):**
   - Generate random mood logs with valid triggers and intensities
   - Generate invalid triggers and intensities to test validation
   - Generate sequences of mood logs to test chronological ordering
   - Generate logs across different time periods to test 7-day summary filtering

3. **AI Analysis Properties (8-11):**
   - Generate random sets of mood logs to test trigger frequency calculation
   - Generate logs with known averages to verify intensity calculations
   - Generate high-intensity patterns to test detection algorithms
   - Generate edge cases: all same trigger, all different triggers, exactly 3 logs

4. **Community Properties (12-17):**
   - Generate random post content to verify anonymity guarantees
   - Generate posts with and without harmful keywords
   - Generate moderation state transitions to verify visibility rules
   - Generate report sequences to test escalation

5. **Resource Allocation Properties (18-22):**
   - Generate mood logs with varying intensities to test recommendation thresholds
   - Generate different trigger categories to test exercise matching
   - Generate high-intensity patterns to test professional resource triggers
   - Generate completion records to test persistence

6. **Data Persistence Properties (23-25):**
   - Generate user sessions across multiple "devices" to test cross-session persistence
   - Generate mood log sequences to test process flow orchestration
   - Generate activity completions to test resolution tracking

7. **UI Properties (26-27):**
   - Generate random UI components to verify accessibility attributes
   - Generate async operations to verify loading states

**Example Property Test:**
```typescript
// Feature: mindful-space-wellness-ecosystem, Property 4: Mood log creation and persistence
test('Property 4: Mood log creation and persistence', async () => {
  await fc.assert(
    fc.asyncProperty(
      fc.record({
        userId: fc.uuid(),
        mood: fc.string({ minLength: 1, maxLength: 50 }),
        trigger: fc.constantFrom('Work', 'Family', 'Health'),
        intensity: fc.integer({ min: 0, max: 10 }),
        notes: fc.option(fc.string({ maxLength: 500 }))
      }),
      async (moodData) => {
        // Create mood log
        const createdLog = await moodService.createMoodLog(moodData)
        
        // Retrieve user's logs
        const userLogs = await moodService.getMoodLogs(moodData.userId)
        
        // Verify log exists and is associated with user
        expect(userLogs).toContainEqual(expect.objectContaining({
          userId: moodData.userId,
          mood: moodData.mood,
          trigger: moodData.trigger,
          intensity: moodData.intensity
        }))
        
        // Verify chronological ordering
        for (let i = 1; i < userLogs.length; i++) {
          expect(userLogs[i].timestamp >= userLogs[i-1].timestamp).toBe(true)
        }
      }
    ),
    { numRuns: 100 }
  )
})
```

### Unit Testing

**Framework:** Jest (for JavaScript/TypeScript)

**Unit Test Focus:**
- Specific edge cases not easily covered by property tests
- Integration points between services
- Error handling scenarios
- UI component rendering

**Unit Test Coverage:**

1. **Authentication:**
   - Empty state: no users registered
   - Password reset flow with valid/invalid emails
   - Token refresh mechanism
   - Session timeout handling

2. **Mood Tracking:**
   - Empty state: user with no mood logs
   - Exactly 7 days of data boundary
   - Timezone handling for 7-day summary
   - Concurrent mood log creation

3. **AI Analysis:**
   - Exactly 3 logs (minimum threshold)
   - Tie-breaking for equal trigger frequencies
   - Exactly 48-hour boundary for consecutive detection
   - Empty analysis when no logs exist

4. **Community:**
   - Empty feed: no approved posts
   - Post with exactly maximum character limit
   - Simultaneous moderation actions
   - Report threshold triggering

5. **Resource Allocation:**
   - Intensity exactly at threshold (5, 7)
   - No exercises available for trigger
   - Multiple simultaneous recommendations
   - Resource unavailability fallback

6. **Integration Tests:**
   - Complete flow: register → login → log mood → receive recommendations
   - Complete flow: create post → moderate → display in feed
   - Error recovery: database failure → retry → success
   - Session management: login → expire → refresh → continue

**Example Unit Test:**
```typescript
describe('Mood Tracking - Edge Cases', () => {
  test('should handle user with no mood logs', async () => {
    const userId = 'new-user-123'
    const summary = await moodService.getSevenDaySummary(userId)
    
    expect(summary.logs).toEqual([])
    expect(summary.isEmpty).toBe(true)
    expect(summary.message).toBe('No mood logs yet. Start tracking your emotional well-being today!')
  })
  
  test('should handle exactly 7 days boundary', async () => {
    const userId = 'test-user-456'
    const now = new Date()
    
    // Create logs at exactly 7 days ago and 7 days + 1 second ago
    await moodService.createMoodLog({
      userId,
      mood: 'anxious',
      trigger: 'Work',
      intensity: 6,
      timestamp: new Date(now.getTime() - 7 * 24 * 60 * 60 * 1000)
    })
    
    await moodService.createMoodLog({
      userId,
      mood: 'stressed',
      trigger: 'Family',
      intensity: 7,
      timestamp: new Date(now.getTime() - (7 * 24 * 60 * 60 * 1000 + 1000))
    })
    
    const summary = await moodService.getSevenDaySummary(userId)
    
    // Should include the 7-day log but not the 7-day + 1 second log
    expect(summary.logs).toHaveLength(1)
    expect(summary.logs[0].trigger).toBe('Work')
  })
})
```

### Test Data Management

**Generators:**
- Create reusable generators for common data types (users, mood logs, posts)
- Use factories for complex object creation
- Maintain seed data for consistent test scenarios

**Database:**
- Use in-memory MongoDB for fast test execution
- Reset database between test suites
- Use transactions for test isolation

**Mocking:**
- Mock external services (email, SMS for crisis resources)
- Mock time for testing time-dependent features
- Mock AI analysis for deterministic testing

### Continuous Integration

**Pre-commit:**
- Run linting and formatting checks
- Run fast unit tests (<5 seconds)

**Pull Request:**
- Run full unit test suite
- Run property-based tests with 100 iterations
- Check code coverage (target: >80%)
- Run integration tests

**Main Branch:**
- Run extended property-based tests with 1000 iterations
- Run end-to-end tests
- Generate coverage reports
- Deploy to staging environment

### Coverage Goals

- **Line Coverage:** >80%
- **Branch Coverage:** >75%
- **Function Coverage:** >85%
- **Property Coverage:** 100% of defined correctness properties

### Performance Testing

**Load Testing:**
- Simulate 1000 concurrent users
- Test mood log creation under load
- Test community feed with 10,000+ posts
- Measure response times (target: <200ms for API calls)

**Stress Testing:**
- Test database connection pool exhaustion
- Test memory usage with large datasets
- Test recovery from service failures

**Benchmarking:**
- AI analysis performance with varying log counts
- 7-day summary generation time
- Community feed pagination performance
