# Database Schema - GuruNirman Platform

## Database: MongoDB

MongoDB is chosen for its flexibility with schema evolution, ability to handle nested documents, and excellent support for horizontal scaling.

---

## Collections

### 1. users

**Description**: Stores all user accounts (teachers, admins, coordinators)

**Schema**:
```javascript
{
  _id: ObjectId,
  userId: String,              // Unique identifier (e.g., "T12345", "A001")
  email: String,               // Unique, required
  passwordHash: String,        // Bcrypt hashed password
  role: String,                // TEACHER | ADMIN | COORDINATOR
  name: String,
  phone: String,
  
  // Role-specific fields
  clusterId: String,           // For TEACHER and COORDINATOR
  subject: [String],           // For TEACHER
  grade: [String],             // For TEACHER (e.g., ["6", "7", "8"])
  challenges: [String],        // Current challenges (for TEACHER)
  language: String,            // Preferred language
  
  // Authentication
  refreshTokens: [{
    token: String,
    expiresAt: Date,
    createdAt: Date
  }],
  lastLogin: Date,
  
  // Profile
  profilePicture: String,      // URL
  bio: String,
  
  // Metadata
  isActive: Boolean,
  isVerified: Boolean,
  verificationToken: String,
  resetPasswordToken: String,
  resetPasswordExpires: Date,
  
  createdAt: Date,
  updatedAt: Date
}
```

**Indexes**:
```javascript
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ userId: 1 }, { unique: true });
db.users.createIndex({ clusterId: 1, role: 1 });
db.users.createIndex({ role: 1, isActive: 1 });
```

---

### 2. clusters

**Description**: Geographical/organizational grouping of schools and teachers

**Schema**:
```javascript
{
  _id: ObjectId,
  clusterId: String,           // Unique identifier (e.g., "C001")
  name: String,                // "Delhi North Cluster"
  region: String,              // "Delhi", "Maharashtra", etc.
  district: String,
  state: String,
  
  // Characteristics
  language: String,            // Primary language
  secondaryLanguages: [String],
  resourceLevel: String,       // LOW | MEDIUM | HIGH
  connectivity: String,        // POOR | MODERATE | GOOD
  
  // Geographic
  location: {
    type: String,              // "Point"
    coordinates: [Number]      // [longitude, latitude]
  },
  
  // Metadata
  teacherCount: Number,
  schoolCount: Number,
  
  // Management
  coordinatorId: String,       // Reference to users collection
  dietCenter: String,          // Name of DIET
  
  // Statistics (cached aggregates)
  stats: {
    avgModuleCompletionRate: Number,
    avgSatisfactionScore: Number,
    totalFeedbackSubmitted: Number,
    lastUpdated: Date
  },
  
  createdAt: Date,
  updatedAt: Date
}
```

**Indexes**:
```javascript
db.clusters.createIndex({ clusterId: 1 }, { unique: true });
db.clusters.createIndex({ region: 1, state: 1 });
db.clusters.createIndex({ location: "2dsphere" });  // For geospatial queries
```

---

### 3. trainingModules

**Description**: Base training content and metadata

**Schema**:
```javascript
{
  _id: ObjectId,
  moduleId: String,            // Unique identifier (e.g., "M001")
  
  // Base Content
  title: String,
  description: String,
  baseContent: String,         // Original content (markdown/HTML)
  contentType: String,         // TEXT | VIDEO | INTERACTIVE
  
  // Classification
  subject: String,             // Mathematics, Science, Language, etc.
  grade: [String],            // Target grades
  tags: [String],             // Keywords for search/matching
  difficulty: String,          // BEGINNER | INTERMEDIATE | ADVANCED
  
  // Duration
  estimatedDuration: Number,   // Minutes
  
  // Structure
  sections: [{
    sectionId: String,
    title: String,
    content: String,
    duration: Number,
    order: Number
  }],
  
  keyTakeaways: [String],
  
  // Assessment
  assessmentQuestions: [{
    questionId: String,
    question: String,
    type: String,              // MCQ | TRUE_FALSE | SHORT_ANSWER
    options: [String],         // For MCQ
    correctAnswer: String,
    explanation: String
  }],
  
  // Versioning
  version: String,             // "1.0", "1.1", etc.
  parentModuleId: String,      // For versioned modules
  versionHistory: [{
    version: String,
    changes: String,
    updatedBy: String,
    updatedAt: Date
  }],
  
  // Cluster Adaptations (references to adapted versions)
  adaptedVersions: [{
    clusterId: String,
    versionId: String,
    language: String,
    adaptationDate: Date
  }],
  
  // Media
  thumbnail: String,           // URL
  media: [{
    type: String,              // IMAGE | VIDEO | PDF
    url: String,
    caption: String
  }],
  
  // Metadata
  status: String,              // DRAFT | REVIEW | PUBLISHED | ARCHIVED
  createdBy: String,           // User ID
  reviewedBy: String,
  publishedAt: Date,
  
  // Statistics (cached)
  stats: {
    viewCount: Number,
    completionCount: Number,
    avgRating: Number,
    avgCompletionTime: Number,
    helpfulCount: Number,
    notHelpfulCount: Number,
    lastUpdated: Date
  },
  
  createdAt: Date,
  updatedAt: Date
}
```

**Indexes**:
```javascript
db.trainingModules.createIndex({ moduleId: 1 }, { unique: true });
db.trainingModules.createIndex({ subject: 1, grade: 1, status: 1 });
db.trainingModules.createIndex({ tags: 1 });
db.trainingModules.createIndex({ status: 1, publishedAt: -1 });
db.trainingModules.createIndex({ 
  title: "text", 
  description: "text", 
  tags: "text" 
});  // Full-text search
```

---

### 4. adaptedModules

**Description**: Cluster-specific adapted versions of base modules

**Schema**:
```javascript
{
  _id: ObjectId,
  adaptedModuleId: String,     // Unique ID for this adaptation
  baseModuleId: String,        // Reference to trainingModules
  clusterId: String,           // Target cluster
  
  // Adapted Content
  title: String,               // May be translated
  description: String,
  adaptedContent: String,      // Localized/adapted content
  
  // Adaptation Details
  language: String,
  adaptations: {
    languageTranslated: Boolean,
    examplesLocalized: Boolean,
    resourcesAdjusted: Boolean,
    culturallyAdapted: Boolean
  },
  
  // Adapted sections (mirroring base module structure)
  sections: [{
    sectionId: String,
    title: String,
    content: String,
    examples: [{
      title: String,
      description: String,
      materials: [String],       // Adjusted for resource level
      localContext: String       // Cultural/regional context
    }],
    duration: Number
  }],
  
  keyTakeaways: [String],
  assessmentQuestions: [{        // Translated/adapted
    questionId: String,
    question: String,
    options: [String],
    correctAnswer: String
  }],
  
  // AI Generation Metadata
  generatedBy: String,           // "AI" or user ID
  aiModel: String,               // "gpt-4-turbo", "claude-3", etc.
  generationPrompt: String,      // Prompt used for generation
  humanReviewed: Boolean,
  reviewedBy: String,
  reviewNotes: String,
  
  // Status
  status: String,                // GENERATED | UNDER_REVIEW | APPROVED | PUBLISHED
  publishedAt: Date,
  
  createdAt: Date,
  updatedAt: Date
}
```

**Indexes**:
```javascript
db.adaptedModules.createIndex({ adaptedModuleId: 1 }, { unique: true });
db.adaptedModules.createIndex({ baseModuleId: 1, clusterId: 1 });
db.adaptedModules.createIndex({ clusterId: 1, status: 1 });
```

---

### 5. feedback

**Description**: Teacher feedback submissions

**Schema**:
```javascript
{
  _id: ObjectId,
  feedbackId: String,          // Unique identifier
  
  // Who
  teacherId: String,           // Reference to users
  clusterId: String,
  
  // What
  subject: String,
  grade: String,
  topic: String,               // Specific lesson topic
  challenges: [String],        // Selected challenge categories
  description: String,         // Free-text description
  
  // Severity
  severity: String,            // LOW | MEDIUM | HIGH | CRITICAL
  urgencyScore: Number,        // 0-1, calculated by AI
  
  // Attachments
  attachments: [{
    filename: String,
    url: String,
    size: Number,
    mimeType: String
  }],
  
  // Status & Processing
  status: String,              // SUBMITTED | PROCESSING | PROCESSED | RESOLVED
  processedAt: Date,
  resolvedAt: Date,
  
  // AI Analysis Results
  aiAnalysis: {
    processedAt: Date,
    keywords: [String],
    sentiment: String,         // POSITIVE | NEUTRAL | NEGATIVE
    sentimentScore: Number,    // -1 to 1
    topics: [{
      topic: String,
      confidence: Number
    }],
    suggestedModules: [String], // Module IDs
    similarFeedbackCount: Number,
    urgencyFactors: [String],
    actionableInsights: [String]
  },
  
  // Response/Resolution
  response: {
    respondedBy: String,       // Admin user ID
    message: String,
    recommendedActions: [String],
    assignedModules: [String], // Module IDs
    respondedAt: Date
  },
  
  // Follow-up
  followUpRequired: Boolean,
  followUpDate: Date,
  followUpNotes: String,
  
  createdAt: Date,
  updatedAt: Date
}
```

**Indexes**:
```javascript
db.feedback.createIndex({ feedbackId: 1 }, { unique: true });
db.feedback.createIndex({ teacherId: 1, createdAt: -1 });
db.feedback.createIndex({ clusterId: 1, status: 1 });
db.feedback.createIndex({ status: 1, severity: 1 });
db.feedback.createIndex({ challenges: 1, subject: 1 });
db.feedback.createIndex({ createdAt: -1 });
```

---

### 6. moduleProgress

**Description**: Tracks teacher progress through modules

**Schema**:
```javascript
{
  _id: ObjectId,
  teacherId: String,
  moduleId: String,            // Can be base or adapted module ID
  
  // Progress
  status: String,              // NOT_STARTED | IN_PROGRESS | COMPLETED
  progressPercentage: Number,  // 0-100
  
  // Section-level tracking
  sectionProgress: [{
    sectionId: String,
    completed: Boolean,
    timeSpent: Number,         // Minutes
    completedAt: Date
  }],
  
  // Time tracking
  totalTimeSpent: Number,      // Minutes
  lastAccessedAt: Date,
  startedAt: Date,
  completedAt: Date,
  
  // Assessment
  quizAttempts: [{
    attemptNumber: Number,
    score: Number,
    totalQuestions: Number,
    answers: [{
      questionId: String,
      selectedAnswer: String,
      isCorrect: Boolean
    }],
    attemptedAt: Date
  }],
  bestQuizScore: Number,
  
  // Rating & Feedback
  rating: Number,              // 1-5
  isHelpful: Boolean,
  comment: String,
  ratedAt: Date,
  
  // Implementation tracking
  implementationStatus: String, // NOT_STARTED | TRYING | IMPLEMENTED
  implementationNotes: String,
  impact: String,              // LOW | MEDIUM | HIGH
  
  createdAt: Date,
  updatedAt: Date
}
```

**Indexes**:
```javascript
db.moduleProgress.createIndex({ teacherId: 1, moduleId: 1 }, { unique: true });
db.moduleProgress.createIndex({ teacherId: 1, status: 1 });
db.moduleProgress.createIndex({ moduleId: 1, status: 1 });
db.moduleProgress.createIndex({ completedAt: -1 });
```

---

### 7. cohorts

**Description**: AI-generated teacher groups with similar characteristics

**Schema**:
```javascript
{
  _id: ObjectId,
  cohortId: String,
  clusterId: String,
  
  // Cohort Definition
  name: String,
  description: String,
  
  // Grouping Criteria
  criteria: {
    subject: String,
    gradeRange: [String],
    primaryChallenge: String,
    secondaryChallenges: [String],
    resourceLevel: String
  },
  
  // Members
  teacherIds: [String],
  teacherCount: Number,
  
  // AI Generation
  generatedBy: String,         // "AI" or user ID
  generationMethod: String,    // "K_MEANS" | "HIERARCHICAL" | "MANUAL"
  similarityScore: Number,     // 0-1, avg similarity within cohort
  
  // Recommended Content
  recommendedModules: [{
    moduleId: String,
    relevanceScore: Number,
    assignedAt: Date
  }],
  
  // Performance
  stats: {
    avgCompletionRate: Number,
    avgSatisfactionScore: Number,
    totalFeedback: Number,
    commonOutcomes: [String]
  },
  
  // Status
  isActive: Boolean,
  
  createdAt: Date,
  updatedAt: Date
}
```

**Indexes**:
```javascript
db.cohorts.createIndex({ cohortId: 1 }, { unique: true });
db.cohorts.createIndex({ clusterId: 1, isActive: 1 });
db.cohorts.createIndex({ "criteria.subject": 1, "criteria.primaryChallenge": 1 });
```

---

### 8. recommendations

**Description**: Personalized module recommendations for teachers

**Schema**:
```javascript
{
  _id: ObjectId,
  teacherId: String,
  
  // Recommendations
  recommendations: [{
    moduleId: String,
    rank: Number,              // 1, 2, 3... (priority order)
    matchScore: Number,        // 0-1, confidence score
    
    // Reasoning
    reasons: [String],         // Why this module was recommended
    basedOn: {
      challenges: [String],
      recentFeedback: [String],  // Feedback IDs
      completedModules: [String],
      cohortSimilarity: Boolean
    },
    
    estimatedImpact: String,   // LOW | MEDIUM | HIGH
    
    // Status
    viewed: Boolean,
    viewedAt: Date,
    dismissed: Boolean,
    dismissedAt: Date
  }],
  
  // Generation metadata
  generatedAt: Date,
  expiresAt: Date,             // Recommendations refresh periodically
  algorithmVersion: String,
  
  createdAt: Date,
  updatedAt: Date
}
```

**Indexes**:
```javascript
db.recommendations.createIndex({ teacherId: 1, generatedAt: -1 });
db.recommendations.createIndex({ expiresAt: 1 }, { expireAfterSeconds: 0 }); // TTL index
```

---

### 9. analytics

**Description**: Pre-aggregated analytics data for performance

**Schema**:
```javascript
{
  _id: ObjectId,
  
  // Dimension
  dimension: String,           // CLUSTER | SUBJECT | GRADE | MODULE | SYSTEM
  dimensionValue: String,      // e.g., "C001", "Mathematics", "M001"
  
  // Time period
  periodType: String,          // DAILY | WEEKLY | MONTHLY
  periodStart: Date,
  periodEnd: Date,
  
  // Metrics
  metrics: {
    // Engagement
    activeTeachers: Number,
    totalTeachers: Number,
    engagementRate: Number,
    
    // Module metrics
    modulesViewed: Number,
    modulesStarted: Number,
    modulesCompleted: Number,
    avgCompletionRate: Number,
    avgTimePerModule: Number,
    
    // Feedback metrics
    feedbackSubmitted: Number,
    avgResponseTime: Number,   // Minutes
    feedbackByChallenge: [{
      challenge: String,
      count: Number
    }],
    
    // Satisfaction
    avgRating: Number,
    helpfulPercentage: Number,
    npsScore: Number,
    
    // Implementation
    implementationRate: Number,
    avgImpactScore: Number
  },
  
  // Trends
  trends: {
    engagementChange: Number,  // % change from previous period
    completionChange: Number,
    satisfactionChange: Number
  },
  
  computedAt: Date,
  createdAt: Date
}
```

**Indexes**:
```javascript
db.analytics.createIndex({ dimension: 1, dimensionValue: 1, periodStart: -1 });
db.analytics.createIndex({ periodType: 1, periodStart: -1 });
```

---

### 10. uploads

**Description**: Tracks content uploads and AI processing jobs

**Schema**:
```javascript
{
  _id: ObjectId,
  uploadId: String,
  
  // File info
  filename: String,
  originalFilename: String,
  mimeType: String,
  size: Number,
  url: String,                 // Storage location
  
  // Uploader
  uploadedBy: String,          // User ID
  
  // Metadata (provided during upload)
  metadata: {
    title: String,
    subject: String,
    grade: [String],
    tags: [String],
    language: String,
    estimatedDuration: Number
  },
  
  // Processing
  processingStatus: String,    // PENDING | PROCESSING | COMPLETED | FAILED
  processingStage: String,     // EXTRACTION | CHUNKING | ADAPTATION | PUBLISHING
  
  // AI Job tracking
  aiJobs: [{
    jobId: String,
    jobType: String,           // CONTENT_CHUNKING | ADAPTATION | TRANSLATION
    status: String,
    startedAt: Date,
    completedAt: Date,
    error: String
  }],
  
  // Results
  generatedModules: [String],  // Module IDs created from this upload
  
  // Error handling
  error: {
    code: String,
    message: String,
    details: Object
  },
  
  createdAt: Date,
  updatedAt: Date
}
```

**Indexes**:
```javascript
db.uploads.createIndex({ uploadId: 1 }, { unique: true });
db.uploads.createIndex({ uploadedBy: 1, createdAt: -1 });
db.uploads.createIndex({ processingStatus: 1 });
```

---

## Relationships

### Entity Relationship Diagram (ERD)

```
users (1) ──────────── (N) feedback
users (1) ──────────── (N) moduleProgress
users (1) ──────────── (N) uploads

clusters (1) ────────── (N) users
clusters (1) ────────── (N) feedback
clusters (1) ────────── (N) cohorts
clusters (1) ────────── (N) adaptedModules

trainingModules (1) ── (N) adaptedModules
trainingModules (1) ── (N) moduleProgress
trainingModules (1) ── (N) recommendations

cohorts (N) ──────────── (N) users (via teacherIds array)
```

---

## Data Aggregation Pipelines

### Example: Cluster Challenge Summary

```javascript
db.feedback.aggregate([
  // Match feedback for specific cluster
  { $match: { clusterId: "C001", status: "PROCESSED" } },
  
  // Unwind challenges array
  { $unwind: "$challenges" },
  
  // Group by challenge and count
  { $group: {
    _id: "$challenges",
    count: { $sum: 1 },
    teachers: { $addToSet: "$teacherId" }
  }},
  
  // Sort by count
  { $sort: { count: -1 } },
  
  // Calculate percentages
  { $group: {
    _id: null,
    total: { $sum: "$count" },
    challenges: { $push: { challenge: "$_id", count: "$count", teachers: "$teachers" } }
  }},
  
  { $unwind: "$challenges" },
  
  { $project: {
    challenge: "$challenges.challenge",
    count: "$challenges.count",
    percentage: { $multiply: [{ $divide: ["$challenges.count", "$total"] }, 100] },
    uniqueTeachers: { $size: "$challenges.teachers" }
  }}
]);
```

---

## Data Retention Policy

| Collection | Retention | Archive Strategy |
|------------|-----------|------------------|
| users | Indefinite | Soft delete (isActive flag) |
| feedback | 2 years | Move to cold storage after 2 years |
| moduleProgress | Indefinite | Important for long-term analytics |
| analytics | 1 year active, 3 years archived | Summarize daily → weekly → monthly |
| uploads | 6 months | Delete after successful processing |
| recommendations | 30 days | TTL index auto-deletion |

---

## Backup Strategy

**Frequency**: 
- Full backup: Daily at 2 AM IST
- Incremental backup: Every 6 hours

**Retention**:
- Daily backups: 7 days
- Weekly backups: 4 weeks
- Monthly backups: 12 months

**Recovery Testing**: Monthly disaster recovery drills

---

## Database Optimization

### 1. Caching Strategy (Redis)
```javascript
// Cache frequently accessed data
CACHE_KEYS = {
  USER_PROFILE: `user:${userId}`,
  MODULE_DETAILS: `module:${moduleId}`,
  CLUSTER_STATS: `cluster:${clusterId}:stats`,
  RECOMMENDATIONS: `recommendations:${teacherId}`
}

// TTL (Time To Live)
USER_PROFILE_TTL = 3600;      // 1 hour
MODULE_DETAILS_TTL = 1800;    // 30 minutes
CLUSTER_STATS_TTL = 900;      // 15 minutes
```

### 2. Read/Write Splitting
- Primary node: All writes
- Secondary nodes: Read replicas for analytics and reports
- Read preference: `secondaryPreferred`

### 3. Sharding Strategy
```javascript
// Shard keys for horizontal scaling
{
  users: { clusterId: 1, _id: 1 },
  feedback: { clusterId: 1, createdAt: 1 },
  trainingModules: { subject: 1, _id: 1 },
  moduleProgress: { teacherId: 1 }
}
```

---

**Schema Version**: 1.0  
**Last Updated**: January 3, 2026  
**Maintained By**: GuruNirman Database Team
