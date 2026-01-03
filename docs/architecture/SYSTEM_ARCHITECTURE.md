# System Architecture - GuruNirman

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [System Components](#system-components)
3. [Data Flow](#data-flow)
4. [Component Interaction](#component-interaction)
5. [Scalability Considerations](#scalability-considerations)
6. [Security Architecture](#security-architecture)

---

## Architecture Overview

GuruNirman follows a **microservices-inspired architecture** with clear separation of concerns, enabling independent scaling and maintenance of different system components.

### Architecture Layers

```
┌────────────────────────────────────────────────────────────────────┐
│                         PRESENTATION LAYER                         │
│                    (React Frontend Application)                    │
├────────────────────────────────────────────────────────────────────┤
│  Teacher Portal  │  Admin Dashboard  │  Analytics Dashboard        │
│  - Feedback Form │  - Content Upload │  - Engagement Metrics      │
│  - Module Viewer │  - Cluster Mgmt   │  - Effectiveness Charts    │
│  - Progress      │  - AI Controls    │  - Challenge Analysis      │
└────────────────────────────────────────────────────────────────────┘
                              ↕ HTTPS/REST API
┌────────────────────────────────────────────────────────────────────┐
│                        APPLICATION LAYER                           │
│                   (Node.js Backend Services)                       │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────┐ │
│  │ Authentication   │  │ Training Module  │  │ Feedback       │ │
│  │ Service          │  │ Service          │  │ Processing     │ │
│  │ - JWT Auth       │  │ - CRUD Ops       │  │ Service        │ │
│  │ - RBAC           │  │ - Versioning     │  │ - Collection   │ │
│  └──────────────────┘  └──────────────────┘  └────────────────┘ │
│                                                                    │
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────┐ │
│  │ Cohort Builder   │  │ Recommendation   │  │ Analytics      │ │
│  │ Engine           │  │ Engine           │  │ Service        │ │
│  │ - Clustering     │  │ - Matching       │  │ - Aggregation  │ │
│  │ - Grouping       │  │ - Scoring        │  │ - Reporting    │ │
│  └──────────────────┘  └──────────────────┘  └────────────────┘ │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
                              ↕
┌────────────────────────────────────────────────────────────────────┐
│                        AI & INTELLIGENCE LAYER                     │
│                     (Python AI Services)                           │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌──────────────────┐  ┌──────────────────┐  ┌────────────────┐ │
│  │ Content Chunker  │  │ Adaptation       │  │ Feedback       │ │
│  │ - Text Splitting │  │ Engine           │  │ Analyzer       │ │
│  │ - Topic Modeling │  │ - Translation    │  │ - NLP          │ │
│  │ - Summarization  │  │ - Localization   │  │ - Sentiment    │ │
│  └──────────────────┘  └──────────────────┘  └────────────────┘ │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │              LLM Integration Layer                           │ │
│  │  - OpenAI GPT-4 / Claude API                                 │ │
│  │  - LangChain for Prompt Management                           │ │
│  │  - Embeddings for Semantic Search                            │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
                              ↕
┌────────────────────────────────────────────────────────────────────┐
│                          DATA LAYER                                │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────────┐   │
│  │ MongoDB        │  │ Redis Cache    │  │ Vector Store     │   │
│  │ - Teachers     │  │ - Sessions     │  │ (Pinecone)       │   │
│  │ - Modules      │  │ - Hot Data     │  │ - Embeddings     │   │
│  │ - Feedback     │  │ - Rate Limit   │  │ - Similarity     │   │
│  │ - Clusters     │  └────────────────┘  └──────────────────┘   │
│  │ - Analytics    │                                              │
│  └────────────────┘  ┌────────────────┐                         │
│                      │ File Storage   │                          │
│                      │ (S3/Cloudinary)│                          │
│                      │ - Documents    │                          │
│                      │ - Media        │                          │
│                      └────────────────┘                          │
└────────────────────────────────────────────────────────────────────┘
```

---

## System Components

### 1. Frontend Layer (React Application)

#### Teacher Portal
**Purpose**: Interface for teachers to interact with the platform

**Key Components**:
- **Feedback Form Component**
  - Multi-step form for capturing classroom challenges
  - File upload for supporting materials (optional)
  - Auto-save functionality
  
- **Module Viewer Component**
  - Card-based layout for training modules
  - Video player with progress tracking
  - Interactive quizzes and assessments
  - Download for offline viewing
  
- **Progress Dashboard Component**
  - Visual timeline of completed modules
  - Skill badges and achievements
  - Recommended next steps

**Technologies**:
- React 18+ with functional components and hooks
- React Router for navigation
- React Query for server state management
- Tailwind CSS for styling
- Recharts for progress visualization

#### Admin Dashboard
**Purpose**: Content management and analytics for DIET/SCERT admins

**Key Components**:
- **Content Upload Module**
  - Drag-and-drop file upload
  - Metadata form (subject, grade, tags)
  - Preview before publishing
  
- **Cluster Management**
  - Interactive map view of clusters
  - Teacher list with filters
  - Bulk actions (assign modules, send notifications)
  
- **AI Control Panel**
  - Trigger content chunking
  - Review AI-generated adaptations
  - Approve/edit before publishing
  
- **Analytics Dashboard**
  - Real-time metrics cards
  - Interactive charts (engagement, satisfaction)
  - Export reports (PDF, Excel)

**Technologies**:
- React Admin framework components
- D3.js for advanced visualizations
- React DnD for drag-drop
- Redux Toolkit for complex state

#### Analytics Dashboard
**Purpose**: Data insights for decision-making

**Features**:
- Teacher engagement metrics
- Module effectiveness scores
- Challenge heatmaps by region
- Trend analysis over time
- Comparative cluster analysis

---

### 2. Backend Layer (Go Services)

#### Authentication Service
**Responsibility**: User authentication and authorization

**Features**:
```javascript
// JWT-based authentication
POST /api/auth/register
POST /api/auth/login
POST /api/auth/refresh-token
POST /api/auth/logout

// Role-based access control
- TEACHER: Read modules, submit feedback
- ADMIN: Full access to content management
- COORDINATOR: View analytics, manage clusters
```

**Implementation**:
```go
// middleware/auth.go
package middleware

import (
    "github.com/gin-gonic/gin"
    "github.com/golang-jwt/jwt/v5"
    "net/http"
    "strings"
)

func AuthMiddleware(roles ...string) gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.JSON(http.StatusUnauthorized, gin.H{"error": "Unauthorized"})
            c.Abort()
            return
        }
        
        tokenString := strings.TrimPrefix(authHeader, "Bearer ")
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            return []byte(os.Getenv("JWT_SECRET")), nil
        })
        
        if err != nil || !token.Valid {
            c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid token"})
            c.Abort()
            return
        }
        
        claims := token.Claims.(jwt.MapClaims)
        userRole := claims["role"].(string)
        
        if len(roles) > 0 && !contains(roles, userRole) {
            c.JSON(http.StatusForbidden, gin.H{"error": "Forbidden"})
            c.Abort()
            return
        }
        
        c.Set("user", claims)
        c.Next()
    }
}
```

#### Training Module Service
**Responsibility**: CRUD operations for training content

**API Endpoints**:
```go
POST   /api/training/upload          // Upload base content
GET    /api/training/modules          // List modules with filters
GET    /api/training/modules/:id      // Get specific module
PUT    /api/training/modules/:id      // Update module
DELETE /api/training/modules/:id      // Soft delete
POST   /api/training/modules/:id/version  // Create new version
GET    /api/training/search           // Full-text search
```

**Key Functions**:
- Content versioning with history
- Tag-based organization
- Full-text search with MongoDB Atlas Search
- Module status workflow (draft → review → published)

#### Feedback Processing Service
**Responsibility**: Collect and process teacher feedback

**Workflow**:
```
Teacher submits feedback
    ↓
Validate and sanitize input
    ↓
Store in MongoDB
    ↓
Trigger async AI analysis
    ↓
Update cluster challenge aggregates
    ↓
Notify admins if critical patterns detected
```

**Data Processing**:
```go
// services/feedbackProcessor.go
package services

import (
    "context"
    "gurunirman/internal/models"
    "gurunirman/internal/repository"
)

func ProcessFeedback(ctx context.Context, feedback *models.Feedback) error {
    // 1. Store raw feedback
    if err := repository.CreateFeedback(ctx, feedback); err != nil {
        return err
    }
    
    // 2. Extract keywords using NLP
    keywords, err := extractKeywords(feedback.Comments)
    if err != nil {
        return err
    }
    
    // 3. Update cluster aggregates
    if err := updateClusterStats(ctx, feedback.ClusterID, keywords); err != nil {
        return err
    }
    
    // 4. Queue for AI analysis
    if err := queueAIAnalysis(feedback.ID); err != nil {
        return err
    }
    
    // 5. Check for urgent patterns
    isUrgent, err := detectUrgentPattern(feedback)
    if err != nil {
        return err
    }
    if isUrgent {
        notifyAdmins(feedback.ClusterID)
    }
    
    return nil
}
```

#### Cohort Builder Engine
**Responsibility**: Group teachers into cohorts based on similarities

**Algorithm**:
```go
// services/cohortBuilder.go
package services

import (
    "context"
    "gurunirman/internal/models"
    "gurunirman/internal/repository"
)

func BuildCohorts(ctx context.Context, clusterID string) ([]*models.Cohort, error) {
    teachers, err := repository.FindTeachersByCluster(ctx, clusterID)
    if err != nil {
        return nil, err
    }
    
    // Group by multiple dimensions
    dimensions := []string{
        "subject",        // Primary subject taught
        "grade",          // Grade levels
        "challenges",     // Common challenges
        "resourceLevel",  // Resource availability
    }
    
    // K-means clustering
    cohorts, err := clusterTeachers(teachers, dimensions)
    if err != nil {
        return nil, err
    }
    
    // Save cohorts
    if err := repository.InsertManyCohorts(ctx, cohorts); err != nil {
        return nil, err
    }
    
    return cohorts, nil
}
```

**Cohort Characteristics**:
- Subject-based cohorts (Math, Science, Language)
- Grade-level cohorts (Primary, Middle, Secondary)
- Challenge-based cohorts (Engagement, Assessment, Resources)
- Hybrid cohorts (combination of above)

#### Recommendation Engine
**Responsibility**: Match teachers with relevant training modules

**Matching Algorithm**:
```go
// services/recommendationEngine.go
package services

import (
    "context"
    "gurunirman/internal/models"
    "gurunirman/internal/repository"
)

func GetRecommendations(ctx context.Context, teacherID string, limit int) ([]*models.Module, error) {
    teacher, err := repository.FindTeacherByID(ctx, teacherID)
    if err != nil {
        return nil, err
    }
    
    // Collaborative filtering
    similarTeachers, err := findSimilarTeachers(ctx, teacher)
    if err != nil {
        return nil, err
    }
    theirModules, err := getCompletedModules(ctx, similarTeachers)
    if err != nil {
        return nil, err
    }
    
    // Content-based filtering
    relevantByTag, err := repository.FindModulesByTagsAndSubject(ctx, teacher.Challenges, teacher.Subject)
    if err != nil {
        return nil, err
    }
    
    // Hybrid recommendation score
    scored := combineScores(theirModules, relevantByTag)
    
    // Filter out already completed
    completed, err := repository.FindProgressByTeacher(ctx, teacherID)
    if err != nil {
        return nil, err
    }
    
    filtered := filterCompleted(scored, completed)
    
    if len(filtered) > limit {
        return filtered[:limit], nil
    }
    return filtered, nil
}
```

**Scoring Factors**:
- **Challenge Match** (40%): How well module addresses teacher's challenges
- **Popularity** (20%): Completion rate among similar teachers
- **Recency** (15%): Newer content prioritized
- **Effectiveness** (25%): Avg. usefulness rating from similar cohort

#### Analytics Service
**Responsibility**: Data aggregation and reporting

**Key Metrics Calculated**:
```go
// services/analytics.go
package services

import "math"

type Metrics struct {
    EngagementScore    float64
    Effectiveness      float64
    ResolutionRate     float64
    PersonalizationRate float64
}

func CalculateMetrics(completedModules, assignedModules, helpfulRatings, totalRatings, resolvedChallenges, reportedChallenges, adaptedModules, totalModules int) Metrics {
    return Metrics{
        EngagementScore:    (float64(completedModules) / float64(assignedModules)) * 100,
        Effectiveness:      (float64(helpfulRatings) / float64(totalRatings)) * 100,
        ResolutionRate:     (float64(resolvedChallenges) / float64(reportedChallenges)) * 100,
        PersonalizationRate: (float64(adaptedModules) / float64(totalModules)) * 100,
    }
}
```

**Real-time Aggregations**:
- MongoDB aggregation pipelines for complex queries
- Redis caching for frequently accessed metrics
- Incremental updates for better performance

---

### 3. AI & Intelligence Layer

#### Content Chunker
**Purpose**: Break long training manuals into micro-learning modules

**Process**:
```python
# ai-engine/content_chunker.py

def chunk_content(long_document, target_duration=10):
    """
    Split document into 5-10 minute modules
    
    Args:
        long_document: Full training manual text
        target_duration: Target minutes per chunk
    
    Returns:
        List of micro-modules with metadata
    """
    # 1. Extract document structure
    sections = extract_sections(long_document)
    
    # 2. Topic modeling
    topics = identify_topics_lda(sections)
    
    # 3. Semantic chunking
    chunks = []
    for section in sections:
        # Split by semantic similarity
        sub_chunks = semantic_split(
            section,
            max_tokens=estimate_tokens(target_duration)
        )
        chunks.extend(sub_chunks)
    
    # 4. Generate summaries
    for chunk in chunks:
        chunk['summary'] = llm.summarize(chunk['content'])
        chunk['key_takeaways'] = llm.extract_key_points(chunk['content'])
        chunk['duration_estimate'] = estimate_reading_time(chunk['content'])
    
    return chunks
```

**LLM Prompts Used**:
```python
CHUNKING_PROMPT = """
You are an expert at breaking down educational content into micro-learning modules.

Task: Divide the following training manual into 5-10 minute learning modules.

Requirements:
1. Each module should cover ONE clear concept
2. Maintain logical flow between modules
3. Start with foundational concepts
4. Include practical examples
5. End with actionable takeaways

Manual Content:
{content}

Output format:
[
  {
    "title": "Module title",
    "content": "Module content",
    "duration": "8 minutes",
    "prerequisites": ["Module IDs"],
    "key_takeaways": ["Point 1", "Point 2"]
  }
]
"""
```

#### Adaptation Engine
**Purpose**: Customize content for local context

**Adaptation Parameters**:
```python
# ai-engine/adaptation_service.py

class AdaptationEngine:
    def adapt_content(self, module, cluster_context):
        """
        Adapt module content for specific cluster
        
        Adaptations:
        - Language translation
        - Local examples
        - Resource substitutions
        - Cultural context
        """
        adapted = {}
        
        # 1. Language adaptation
        if cluster_context['language'] != 'english':
            adapted['content'] = self.translate(
                module['content'],
                target_lang=cluster_context['language']
            )
        
        # 2. Example localization
        adapted['examples'] = self.localize_examples(
            module['examples'],
            region=cluster_context['region']
        )
        
        # 3. Resource adjustment
        if cluster_context['resource_level'] == 'low':
            adapted['activities'] = self.low_resource_alternatives(
                module['activities']
            )
        
        # 4. Cultural sensitivity
        adapted['content'] = self.cultural_adaptation(
            adapted['content'],
            culture=cluster_context['culture']
        )
        
        return adapted
```

**Example Adaptation**:
```
Original (Generic):
"Use interactive whiteboards to demonstrate geometric concepts"

Adapted (Low-resource rural cluster):
"Use chalk and floor/ground space to draw large geometric shapes. 
Students can walk the perimeter to understand measurement."
```

#### Feedback Analyzer
**Purpose**: Extract insights from teacher feedback

**NLP Pipeline**:
```python
# ai-engine/feedback_analyzer.py

class FeedbackAnalyzer:
    def analyze_batch(self, feedbacks):
        """
        Analyze batch of feedback for patterns
        """
        results = {
            'common_challenges': [],
            'sentiment': {},
            'urgency_scores': {},
            'suggested_modules': []
        }
        
        # 1. Keyword extraction
        all_text = ' '.join([f['comments'] for f in feedbacks])
        keywords = self.extract_keywords_tfidf(all_text)
        
        # 2. Sentiment analysis
        for feedback in feedbacks:
            sentiment = self.sentiment_model.predict(feedback['comments'])
            results['sentiment'][feedback['id']] = sentiment
        
        # 3. Topic clustering
        topics = self.lda_model.fit_transform(
            [f['comments'] for f in feedbacks]
        )
        results['common_challenges'] = self.interpret_topics(topics)
        
        # 4. Urgency detection
        urgent_keywords = ['urgent', 'critical', 'struggling', 'failing']
        for feedback in feedbacks:
            score = self.calculate_urgency(feedback, urgent_keywords)
            if score > 0.7:
                results['urgency_scores'][feedback['id']] = score
        
        # 5. Module suggestions via embeddings
        feedback_embeddings = self.embed(all_text)
        module_embeddings = self.get_all_module_embeddings()
        similar_modules = self.find_similar(
            feedback_embeddings,
            module_embeddings,
            top_k=5
        )
        results['suggested_modules'] = similar_modules
        
        return results
```

---

## Data Flow

### Primary User Flows

#### Flow 1: Teacher Submits Feedback
```
[Teacher] → Fills feedback form → [Frontend]
    ↓
[Frontend] → POST /api/feedback/submit → [Backend API]
    ↓
[Backend] → Validates & saves to MongoDB → [Database]
    ↓
[Backend] → Queues for AI analysis → [Message Queue]
    ↓
[AI Engine] → Processes feedback → Extracts patterns
    ↓
[AI Engine] → Updates cluster stats → [Database]
    ↓
[Backend] → Sends notification → [Admin Dashboard]
```

#### Flow 2: AI Generates Personalized Module
```
[Admin] → Uploads base content → [Backend API]
    ↓
[Backend] → Stores base module → [MongoDB]
    ↓
[Backend] → Triggers AI chunking → [AI Engine]
    ↓
[AI Engine] → Chunks content into micro-modules
    ↓
[AI Engine] → FOR EACH cluster:
                - Get cluster context
                - Adapt content (language, examples, resources)
                - Generate module version
    ↓
[AI Engine] → Returns adapted modules → [Backend]
    ↓
[Backend] → Saves module versions → [MongoDB]
    ↓
[Recommendation Engine] → Matches modules to teachers
    ↓
[Backend] → Sends notifications → [Teachers]
```

#### Flow 3: Teacher Consumes Module
```
[Teacher] → Logs in → Sees personalized recommendations
    ↓
[Teacher] → Clicks module → [Frontend loads module]
    ↓
[Frontend] → GET /api/training/modules/:id → [Backend]
    ↓
[Backend] → Retrieves adapted version for teacher's cluster
    ↓
[Backend] → Records view event → [Analytics DB]
    ↓
[Frontend] → Teacher completes module → Rates usefulness
    ↓
[Frontend] → POST /api/feedback/module-rating → [Backend]
    ↓
[Backend] → Updates module effectiveness score
    ↓
[Recommendation Engine] → Adjusts future recommendations
```

### Data Synchronization

**Real-time Updates**:
- WebSocket connections for live dashboard updates
- Server-Sent Events (SSE) for notification streams
- Polling fallback for compatibility

**Caching Strategy**:
```
Cache Layer (Redis):
- Session data: TTL 24 hours
- Module metadata: TTL 1 hour
- Analytics snapshots: TTL 15 minutes
- Recommendation results: TTL 30 minutes

Cache invalidation triggers:
- New module published → Invalidate module list cache
- Feedback submitted → Invalidate cluster stats cache
- Module rated → Invalidate effectiveness cache
```

---

## Component Interaction

### Sequence Diagram: End-to-End Training Flow

```
Teacher    Frontend    API Gateway    Backend Services    AI Engine    Database
  │            │            │                │                │           │
  │─Submit────▶│            │                │                │           │
  │  Feedback  │            │                │                │           │
  │            │─POST /api──▶│                │                │           │
  │            │  /feedback │                │                │           │
  │            │            │─Validate──────▶│                │           │
  │            │            │                │─Save──────────▶│           │
  │            │            │                │                │──Write───▶│
  │            │◀──200 OK───│◀───────────────│◀──────────────│◀─────────│
  │◀─Success──│            │                │                │           │
  │  Message   │            │                │                │           │
  │            │            │                │                │           │
  │            │            │                │─Queue AI Job──▶│           │
  │            │            │                │                │           │
  │            │            │                │                │─Analyze──▶│
  │            │            │                │◀───Patterns───│           │
  │            │            │                │                │           │
Admin          │            │                │                │           │
  │            │            │                │                │           │
  │───Login───▶│            │                │                │           │
  │            │─Dashboard─▶│─GET /analytics▶│                │           │
  │            │  Load      │                │─Aggregate─────▶│           │
  │            │            │                │                │──Query───▶│
  │            │◀───Data────│◀───Insights────│◀──────────────│◀─────────│
  │◀─Displays──│            │                │                │           │
  │  Top       │            │                │                │           │
  │ Challenges │            │                │                │           │
  │            │            │                │                │           │
  │─Trigger────▶│            │                │                │           │
  │  AI Module │            │                │                │           │
  │ Generation │─POST /ai/──▶│─Generate──────▶│                │           │
  │            │  generate  │                │─Chunk Content─▶│           │
  │            │            │                │                │           │
  │            │            │                │                │─Adapt for─▶
  │            │            │                │                │  Clusters  │
  │            │            │                │◀──Modules──────│           │
  │            │◀───202─────│◀───Queued──────│                │           │
  │            │  Accepted  │                │                │           │
  │            │            │                │─Save Versions─▶│           │
  │            │            │                │                │──Write───▶│
  │            │            │                │                │           │
Teacher        │            │                │                │           │
  │            │            │                │                │           │
  │─Notification◀───────────│◀───Recommend───│◀───────────────│◀─Match──│
  │  (New      │            │                │                │   Query  │
  │   Module)  │            │                │                │           │
  │            │            │                │                │           │
  │─View Module▶│            │                │                │           │
  │            │─GET /module▶│─Fetch Adapted─▶│                │           │
  │            │            │                │                │──Read────▶│
  │            │◀───Module───│◀───Content─────│◀──────────────│◀─────────│
  │◀─Displays──│            │                │                │           │
  │  Adapted   │            │                │                │           │
  │  Content   │            │                │                │           │
```

---

## Scalability Considerations

### Horizontal Scaling

**Frontend**:
- Static assets served via CDN (Cloudflare, AWS CloudFront)
- Multiple instances behind load balancer
- Container-based deployment (Docker + Kubernetes)

**Backend**:
```yaml
# kubernetes/backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gurunirman-backend
spec:
  replicas: 3  # Start with 3, auto-scale to 10
  selector:
    matchLabels:
      app: gurunirman-backend
  template:
    metadata:
      labels:
        app: gurunirman-backend
    spec:
      containers:
      - name: backend
        image: gurunirman/backend:latest
        resources:
          requests:
            memory: "128Mi"
            cpu: "200m"
          limits:
            memory: "256Mi"
            cpu: "400m"
        env:
        - name: GO_ENV
          value: "production"
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: gurunirman-backend
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**AI Engine**:
- Dedicated GPU instances for LLM inference
- Message queue (RabbitMQ/Bull) for async processing
- Batch processing during off-peak hours
- Caching of common adaptations

### Database Scaling

**MongoDB Sharding**:
```javascript
// Shard key strategy
{
  "teachers": { "clusterId": 1, "teacherId": 1 },    // Shard by cluster
  "modules": { "subject": 1, "createdAt": 1 },       // Shard by subject
  "feedback": { "clusterId": 1, "timestamp": 1 },    // Shard by cluster + time
  "analytics": { "date": 1, "clusterId": 1 }         // Shard by date
}
```

**Read Replicas**:
- Primary for writes
- 2+ secondaries for reads
- Read preference: `secondaryPreferred`
- Analytics queries routed to dedicated replica

### Performance Optimization

**API Response Times**:
| Endpoint | Target | Strategy |
|----------|--------|----------|
| GET /modules | < 200ms | Redis cache + pagination |
| POST /feedback | < 300ms | Async processing |
| GET /analytics | < 500ms | Pre-aggregated data |
| AI generation | < 30s | Queue + progress updates |

**Database Indexing**:
```go
// pkg/database/indexes.go
package database

import (
    "context"
    "go.mongodb.org/mongo-driver/bson"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)

func CreateIndexes(ctx context.Context, db *mongo.Database) error {
    // Critical indexes
    teachersIdx := []mongo.IndexModel{
        {Keys: bson.D{{"clusterId", 1}, {"subject", 1}}},
    }
    _, err := db.Collection("teachers").Indexes().CreateMany(ctx, teachersIdx)
    if err != nil {
        return err
    }
    
    // Text search index
    modulesIdx := []mongo.IndexModel{
        {Keys: bson.D{{"tags", 1}, {"subject", 1}}},
        {Keys: bson.D{{"title", "text"}, {"content", "text"}, {"tags", "text"}}},
    }
    _, err = db.Collection("modules").Indexes().CreateMany(ctx, modulesIdx)
    
    // Geospatial index
    clustersIdx := []mongo.IndexModel{
        {Keys: bson.D{{"location", "2dsphere"}}},
    }
    _, err = db.Collection("clusters").Indexes().CreateMany(ctx, clustersIdx)
    
    return err
}
```

---

## Security Architecture

### Authentication Flow

```
Client                    Backend                   Database
  │                         │                         │
  │──1. Login Request──────▶│                         │
  │   (email, password)     │                         │
  │                         │──2. Query User─────────▶│
  │                         │◀─3. User Data───────────│
  │                         │                         │
  │                         │──4. Verify Password     │
  │                         │    (bcrypt compare)     │
  │                         │                         │
  │                         │──5. Generate JWT        │
  │                         │    (access + refresh)   │
  │◀─6. Tokens + User Data──│                         │
  │                         │                         │
  │──7. API Request─────────▶│                         │
  │   (Authorization: Bearer)                         │
  │                         │──8. Verify JWT          │
  │                         │    (signature + expiry) │
  │                         │                         │
  │                         │──9. Check Role/Permissions
  │                         │                         │
  │◀─10. Protected Data─────│                         │
```

### Security Measures

**1. Input Validation**:
```go
// Using validator/v10
package models

import "github.com/go-playground/validator/v10"

type FeedbackRequest struct {
    TeacherID   string       `json:"teacherId" validate:"required"`
    Comments    string       `json:"comments" validate:"required,max=5000"`
    Challenges  []string     `json:"challenges" validate:"max=10"`
    Attachments []Attachment `json:"attachments"`
}

type Attachment struct {
    Filename string `json:"filename" validate:"required"`
    Size     int64  `json:"size" validate:"max=10485760"` // 10MB limit
}

func ValidateFeedback(feedback *FeedbackRequest) error {
    validate := validator.New()
    return validate.Struct(feedback)
}
```

**2. Rate Limiting**:
```go
// Using golang.org/x/time/rate
package middleware

import (
    "github.com/gin-gonic/gin"
    "golang.org/x/time/rate"
    "net/http"
    "sync"
)

var (
    limiters = make(map[string]*rate.Limiter)
    mu       sync.Mutex
)

func RateLimitMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        ip := c.ClientIP()
        
        mu.Lock()
        limiter, exists := limiters[ip]
        if !exists {
            limiter = rate.NewLimiter(rate.Limit(100.0/900), 100) // 100 requests per 15 min
            limiters[ip] = limiter
        }
        mu.Unlock()
        
        if !limiter.Allow() {
            c.JSON(http.StatusTooManyRequests, gin.H{
                "error": "Too many requests, please try again later",
            })
            c.Abort()
            return
        }
        
        c.Next()
    }
}
```

**3. Data Encryption**:
- **In Transit**: TLS 1.3 for all API communication
- **At Rest**: MongoDB encryption at rest
- **Sensitive Fields**: Field-level encryption for PII

**4. RBAC (Role-Based Access Control)**:
```javascript
const roles = {
  TEACHER: {
    permissions: ['read:own_profile', 'submit:feedback', 'read:modules']
  },
  ADMIN: {
    permissions: ['read:all', 'write:all', 'delete:modules']
  },
  COORDINATOR: {
    permissions: ['read:cluster', 'read:analytics', 'manage:cohorts']
  }
};
```

**5. Security Headers**:
```go
// Using secure middleware
package middleware

import (
    "github.com/gin-gonic/gin"
    "github.com/unrolled/secure"
)

func SecureHeadersMiddleware() gin.HandlerFunc {
    secureMiddleware := secure.New(secure.Options{
        FrameDeny:             true,
        ContentTypeNosniff:    true,
        BrowserXssFilter:      true,
        ContentSecurityPolicy: "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'",
        IsDevelopment:         false,
        SSLRedirect:           true,
        STSSeconds:            31536000,
        STSIncludeSubdomains:  true,
        STSPreload:            true,
    })
    
    return func(c *gin.Context) {
        err := secureMiddleware.Process(c.Writer, c.Request)
        if err != nil {
            c.Abort()
            return
        }
        c.Next()
    }
}
```

---

## Monitoring & Observability

### Logging Strategy
```go
// Using zerolog for structured logging
package logger

import (
    "github.com/rs/zerolog"
    "github.com/rs/zerolog/log"
    "os"
)

func InitLogger() {
    zerolog.TimeFieldFormat = zerolog.TimeFormatUnix
    
    file, err := os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
    if err != nil {
        log.Fatal().Err(err).Msg("Failed to open log file")
    }
    
    multi := zerolog.MultiLevelWriter(os.Stdout, file)
    log.Logger = zerolog.New(multi).With().Timestamp().Str("service", "gurunirman-backend").Logger()
}

// Log key events
log.Info().
    Str("teacherId", teacherID).
    Str("clusterId", clusterID).
    Time("timestamp", time.Now()).
    Msg("Feedback submitted")
```

### Health Checks
```go
// handlers/health.go
package handlers

import (
    "github.com/gin-gonic/gin"
    "net/http"
    "time"
)

var startTime = time.Now()

func HealthCheck(c *gin.Context) {
    health := gin.H{
        "uptime":    time.Since(startTime).Seconds(),
        "timestamp": time.Now().Unix(),
        "status":    "OK",
        "checks": gin.H{
            "database":   checkDatabaseConnection(),
            "redis":      checkRedisConnection(),
            "ai_service": checkAIService(),
        },
    }
    
    checks := health["checks"].(gin.H)
    allHealthy := true
    for _, status := range checks {
        if status != "OK" {
            allHealthy = false
            break
        }
    }
    
    statusCode := http.StatusOK
    if !allHealthy {
        statusCode = http.StatusServiceUnavailable
    }
    
    c.JSON(statusCode, health)
}
```

### Metrics to Track
- API response times (p50, p95, p99)
- Error rates by endpoint
- Database query performance
- AI processing duration
- User engagement metrics
- System resource utilization

---

## Disaster Recovery

**Backup Strategy**:
- **Database**: Daily automated backups, 30-day retention
- **Files**: Replicated across multiple S3 regions
- **Configuration**: Version controlled in Git

**Recovery Time Objective (RTO)**: < 4 hours
**Recovery Point Objective (RPO)**: < 24 hours

---

**Document Version**: 1.0  
**Last Updated**: January 3, 2026  
**Maintained By**: GuruNirman Engineering Team
