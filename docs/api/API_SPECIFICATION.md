# API Specification - GuruNirman Platform

## Base URL
```
Development: http://localhost:5000/api
Production: https://api.gurunirman.edu.in/api
```

## Authentication
All authenticated endpoints require a JWT token in the Authorization header:
```
Authorization: Bearer <access_token>
```

## Response Format

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "message": "Operation successful"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": { ... }
  }
}
```

## HTTP Status Codes
- `200 OK`: Request successful
- `201 Created`: Resource created successfully
- `400 Bad Request`: Invalid request data
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server error

---

## API Endpoints

## 1. Authentication & User Management

### 1.1 Register User
**Endpoint**: `POST /auth/register`  
**Access**: Public  
**Description**: Register a new user (teacher/admin)

**Request Body**:
```json
{
  "email": "teacher@example.com",
  "password": "SecurePass123!",
  "name": "Priya Sharma",
  "role": "TEACHER",
  "metadata": {
    "clusterId": "C001",
    "subject": ["Mathematics", "Science"],
    "grade": ["6", "7", "8"],
    "language": "hindi",
    "phone": "+91-9876543210"
  }
}
```

**Response**: `201 Created`
```json
{
  "success": true,
  "data": {
    "userId": "U12345",
    "email": "teacher@example.com",
    "name": "Priya Sharma",
    "role": "TEACHER",
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
  },
  "message": "User registered successfully"
}
```

**Validation Rules**:
- Email: Valid email format, unique
- Password: Min 8 chars, 1 uppercase, 1 number, 1 special char
- Name: 2-100 characters
- Role: TEACHER | ADMIN | COORDINATOR

---

### 1.2 Login
**Endpoint**: `POST /auth/login`  
**Access**: Public  
**Description**: Authenticate user and receive tokens

**Request Body**:
```json
{
  "email": "teacher@example.com",
  "password": "SecurePass123!"
}
```

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "user": {
      "userId": "U12345",
      "email": "teacher@example.com",
      "name": "Priya Sharma",
      "role": "TEACHER",
      "clusterId": "C001"
    },
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 3600
  }
}
```

---

### 1.3 Refresh Token
**Endpoint**: `POST /auth/refresh-token`  
**Access**: Public (requires refresh token)  
**Description**: Get new access token using refresh token

**Request Body**:
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 3600
  }
}
```

---

### 1.4 Get User Profile
**Endpoint**: `GET /auth/profile`  
**Access**: Authenticated  
**Description**: Get current user's profile

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "userId": "U12345",
    "email": "teacher@example.com",
    "name": "Priya Sharma",
    "role": "TEACHER",
    "clusterId": "C001",
    "subject": ["Mathematics", "Science"],
    "grade": ["6", "7", "8"],
    "challenges": ["student_engagement", "assessment"],
    "language": "hindi",
    "stats": {
      "modulesCompleted": 15,
      "feedbackSubmitted": 8,
      "lastActive": "2026-01-03T10:30:00Z"
    }
  }
}
```

---

### 1.5 Update Profile
**Endpoint**: `PUT /auth/profile`  
**Access**: Authenticated  
**Description**: Update user profile

**Request Body**:
```json
{
  "name": "Priya Sharma",
  "challenges": ["student_engagement", "classroom_management"],
  "language": "hindi",
  "phone": "+91-9876543210"
}
```

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "userId": "U12345",
    "name": "Priya Sharma",
    "challenges": ["student_engagement", "classroom_management"],
    "updatedAt": "2026-01-03T11:00:00Z"
  },
  "message": "Profile updated successfully"
}
```

---

## 2. Feedback Management

### 2.1 Submit Feedback
**Endpoint**: `POST /feedback/submit`  
**Access**: TEACHER  
**Description**: Submit classroom feedback

**Request Body**:
```json
{
  "subject": "Mathematics",
  "grade": "8",
  "topic": "Linear Equations",
  "challenges": ["student_engagement", "concept_clarity"],
  "description": "Students are struggling to understand the concept of variables in equations. Need more practical examples.",
  "severity": "MEDIUM",
  "attachments": [
    {
      "filename": "student_work.pdf",
      "url": "https://s3.amazonaws.com/..."
    }
  ]
}
```

**Response**: `201 Created`
```json
{
  "success": true,
  "data": {
    "feedbackId": "F12345",
    "teacherId": "U12345",
    "clusterId": "C001",
    "subject": "Mathematics",
    "challenges": ["student_engagement", "concept_clarity"],
    "status": "SUBMITTED",
    "createdAt": "2026-01-03T12:00:00Z",
    "estimatedProcessingTime": "2-5 minutes"
  },
  "message": "Feedback submitted successfully. AI analysis in progress."
}
```

**Challenge Options**:
- `student_engagement`: Student participation issues
- `concept_clarity`: Students don't understand concepts
- `resource_shortage`: Lack of teaching materials
- `classroom_management`: Discipline/behavior issues
- `assessment`: Difficulties in testing/evaluation
- `differentiation`: Mixed ability students
- `technology`: Issues with tech integration
- `other`: Custom challenge

---

### 2.2 Get My Feedback History
**Endpoint**: `GET /feedback/teacher/me`  
**Access**: TEACHER  
**Description**: Get current teacher's feedback history

**Query Parameters**:
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10)
- `status` (optional): SUBMITTED | PROCESSED | RESOLVED
- `subject` (optional): Filter by subject

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "feedbacks": [
      {
        "feedbackId": "F12345",
        "subject": "Mathematics",
        "challenges": ["student_engagement"],
        "description": "Students struggling with...",
        "status": "PROCESSED",
        "aiInsights": {
          "recommendedModules": ["M001", "M045"],
          "similarChallenges": 15,
          "suggestedActions": ["Use manipulatives", "Group activities"]
        },
        "createdAt": "2026-01-03T12:00:00Z"
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 3,
      "totalItems": 28,
      "itemsPerPage": 10
    }
  }
}
```

---

### 2.3 Get Cluster Feedback Summary
**Endpoint**: `GET /feedback/cluster/:clusterId`  
**Access**: ADMIN, COORDINATOR  
**Description**: Get aggregated feedback for a cluster

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "clusterId": "C001",
    "summary": {
      "totalFeedbacks": 145,
      "lastWeek": 23,
      "avgResponseTime": "3.2 minutes"
    },
    "topChallenges": [
      {
        "challenge": "student_engagement",
        "count": 45,
        "percentage": 31,
        "trend": "increasing"
      },
      {
        "challenge": "resource_shortage",
        "count": 38,
        "percentage": 26,
        "trend": "stable"
      }
    ],
    "challengesBySubject": {
      "Mathematics": ["concept_clarity", "student_engagement"],
      "Science": ["resource_shortage", "practical_activities"]
    },
    "sentimentAnalysis": {
      "positive": 15,
      "neutral": 60,
      "negative": 25
    },
    "urgentIssues": [
      {
        "feedbackId": "F12340",
        "teacherId": "T001",
        "urgencyScore": 0.85,
        "summary": "Multiple students failing..."
      }
    ]
  }
}
```

---

### 2.4 Get Feedback Analytics
**Endpoint**: `GET /feedback/analytics`  
**Access**: ADMIN  
**Description**: System-wide feedback analytics

**Query Parameters**:
- `startDate`: ISO date string
- `endDate`: ISO date string
- `groupBy`: cluster | subject | grade

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "overview": {
      "totalFeedbacks": 1245,
      "avgResponseTime": "4.1 minutes",
      "resolutionRate": 78
    },
    "timeSeries": [
      {
        "date": "2026-01-01",
        "count": 45,
        "processed": 43,
        "resolved": 35
      }
    ],
    "byCluster": [
      {
        "clusterId": "C001",
        "name": "Delhi North",
        "feedbackCount": 145,
        "topChallenge": "student_engagement"
      }
    ]
  }
}
```

---

## 3. Training Modules

### 3.1 Upload Base Content
**Endpoint**: `POST /training/upload`  
**Access**: ADMIN  
**Description**: Upload new training content for AI processing

**Request Body** (multipart/form-data):
```
file: [PDF/DOCX file]
metadata: {
  "title": "Effective Mathematics Teaching",
  "subject": "Mathematics",
  "grade": ["6", "7", "8"],
  "tags": ["pedagogy", "engagement", "differentiation"],
  "language": "english",
  "estimatedDuration": 45
}
```

**Response**: `202 Accepted`
```json
{
  "success": true,
  "data": {
    "uploadId": "UP12345",
    "filename": "math_teaching_guide.pdf",
    "status": "PROCESSING",
    "estimatedTime": "5-10 minutes",
    "nextSteps": [
      "Content extraction",
      "AI chunking",
      "Cluster adaptation"
    ]
  },
  "message": "Content uploaded. AI processing initiated."
}
```

---

### 3.2 Get Module List
**Endpoint**: `GET /training/modules`  
**Access**: Authenticated  
**Description**: Get list of training modules

**Query Parameters**:
- `page`: Page number
- `limit`: Items per page
- `subject`: Filter by subject
- `grade`: Filter by grade
- `tags`: Comma-separated tags
- `search`: Full-text search
- `forMe`: true (returns personalized for current teacher)

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "modules": [
      {
        "moduleId": "M001",
        "title": "Engaging Students in Math",
        "description": "Strategies to increase participation...",
        "duration": 8,
        "difficulty": "BEGINNER",
        "subject": "Mathematics",
        "grade": ["6", "7"],
        "tags": ["engagement", "activities"],
        "language": "hindi",
        "rating": 4.5,
        "completionCount": 234,
        "isCompleted": false,
        "thumbnail": "https://cdn.example.com/thumb.jpg",
        "createdAt": "2026-01-01T00:00:00Z"
      }
    ],
    "pagination": {
      "currentPage": 1,
      "totalPages": 15,
      "totalItems": 147
    }
  }
}
```

---

### 3.3 Get Module Details
**Endpoint**: `GET /training/modules/:moduleId`  
**Access**: Authenticated  
**Description**: Get detailed module content (adapted for user's cluster)

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "moduleId": "M001",
    "version": "v1.2-C001",
    "title": "गणित में छात्रों को जोड़ना",
    "description": "भागीदारी बढ़ाने के लिए रणनीतियाँ...",
    "duration": 8,
    "content": {
      "sections": [
        {
          "title": "परिचय",
          "content": "...",
          "duration": 2
        },
        {
          "title": "गतिविधियाँ",
          "content": "...",
          "examples": [
            {
              "title": "संख्या खेल",
              "description": "कम संसाधनों के साथ...",
              "materials": ["चाक", "फर्श की जगह"]
            }
          ],
          "duration": 5
        }
      ],
      "keyTakeaways": [
        "Point 1",
        "Point 2"
      ],
      "assessmentQuestions": [
        {
          "question": "Which strategy is most effective for...",
          "options": ["A", "B", "C", "D"],
          "correctAnswer": "B"
        }
      ]
    },
    "relatedModules": ["M002", "M015"],
    "prerequisites": [],
    "adaptations": {
      "language": "hindi",
      "resourceLevel": "low",
      "culturalContext": "rural_north_india"
    }
  }
}
```

---

### 3.4 Get Personalized Recommendations
**Endpoint**: `GET /training/recommendations`  
**Access**: TEACHER  
**Description**: Get AI-recommended modules for current teacher

**Query Parameters**:
- `limit`: Number of recommendations (default: 5)

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "recommendations": [
      {
        "moduleId": "M001",
        "title": "Engaging Students in Math",
        "matchScore": 0.92,
        "reasons": [
          "Addresses your challenge: student_engagement",
          "Popular among Math teachers in your cluster",
          "Rated 4.7/5 by similar teachers"
        ],
        "estimatedImpact": "HIGH",
        "duration": 8
      },
      {
        "moduleId": "M045",
        "title": "Formative Assessment Techniques",
        "matchScore": 0.87,
        "reasons": [
          "Complements your recent completed modules",
          "Addresses assessment challenges in your feedback"
        ],
        "estimatedImpact": "MEDIUM",
        "duration": 10
      }
    ],
    "basedOn": {
      "recentFeedback": ["F12345", "F12340"],
      "challenges": ["student_engagement", "assessment"],
      "completedModules": ["M010", "M020"]
    }
  }
}
```

---

### 3.5 Rate Module
**Endpoint**: `POST /training/modules/:moduleId/rate`  
**Access**: TEACHER  
**Description**: Rate module usefulness

**Request Body**:
```json
{
  "rating": 5,
  "isHelpful": true,
  "comment": "Very practical strategies. Implemented in my class successfully.",
  "implementationStatus": "IMPLEMENTED",
  "impact": "HIGH"
}
```

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "ratingId": "R12345",
    "moduleId": "M001",
    "rating": 5,
    "isHelpful": true,
    "timestamp": "2026-01-03T14:00:00Z"
  },
  "message": "Thank you for your feedback!"
}
```

---

### 3.6 Track Module Progress
**Endpoint**: `POST /training/modules/:moduleId/progress`  
**Access**: TEACHER  
**Description**: Update module completion progress

**Request Body**:
```json
{
  "sectionIndex": 2,
  "completed": true,
  "timeSpent": 5.5,
  "quizScore": 8
}
```

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "moduleId": "M001",
    "progress": 66,
    "sectionsCompleted": 2,
    "totalSections": 3,
    "isCompleted": false
  }
}
```

---

## 4. Cohort & Cluster Management

### 4.1 Get Cluster Details
**Endpoint**: `GET /clusters/:clusterId`  
**Access**: ADMIN, COORDINATOR  
**Description**: Get cluster information

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "clusterId": "C001",
    "name": "Delhi North Cluster",
    "region": "Delhi",
    "language": "hindi",
    "resourceLevel": "medium",
    "teacherCount": 45,
    "schools": 12,
    "stats": {
      "avgModuleCompletionRate": 72,
      "avgSatisfactionScore": 4.2,
      "totalFeedbackSubmitted": 145
    },
    "topChallenges": [
      "student_engagement",
      "resource_shortage"
    ],
    "coordinator": {
      "name": "Rajesh Kumar",
      "email": "rajesh@diet.gov.in"
    }
  }
}
```

---

### 4.2 Build Cohorts
**Endpoint**: `POST /cohorts/build`  
**Access**: ADMIN  
**Description**: Trigger AI-based cohort building

**Request Body**:
```json
{
  "clusterId": "C001",
  "dimensions": ["subject", "challenges", "grade"],
  "minCohortSize": 5
}
```

**Response**: `202 Accepted`
```json
{
  "success": true,
  "data": {
    "jobId": "JOB12345",
    "status": "PROCESSING",
    "estimatedTime": "2-3 minutes"
  },
  "message": "Cohort building initiated"
}
```

---

### 4.3 Get Cohorts
**Endpoint**: `GET /cohorts`  
**Access**: ADMIN, COORDINATOR  
**Description**: List cohorts in a cluster

**Query Parameters**:
- `clusterId`: Cluster ID (required)

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "cohorts": [
      {
        "cohortId": "COH001",
        "name": "Math Teachers - Engagement Focus",
        "subject": "Mathematics",
        "primaryChallenge": "student_engagement",
        "teacherCount": 12,
        "avgGrade": "6-8",
        "recommendedModules": ["M001", "M002", "M015"],
        "createdAt": "2026-01-02T00:00:00Z"
      }
    ]
  }
}
```

---

## 5. Analytics & Reporting

### 5.1 Teacher Engagement Metrics
**Endpoint**: `GET /analytics/engagement`  
**Access**: ADMIN, COORDINATOR  
**Description**: Get teacher engagement analytics

**Query Parameters**:
- `clusterId`: Filter by cluster
- `startDate`: Start date (ISO format)
- `endDate`: End date (ISO format)

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "overview": {
      "totalTeachers": 450,
      "activeTeachers": 389,
      "engagementRate": 86.4
    },
    "moduleCompletion": {
      "avgCompletionRate": 72,
      "totalModulesCompleted": 3245,
      "avgTimePerModule": 8.5
    },
    "feedbackActivity": {
      "totalFeedbackSubmitted": 1245,
      "avgFeedbackPerTeacher": 3.2,
      "responseRate": 94
    },
    "trends": {
      "weekOverWeek": {
        "engagement": 5.2,
        "completion": 3.1
      }
    },
    "byCluster": [
      {
        "clusterId": "C001",
        "name": "Delhi North",
        "engagementRate": 89,
        "completionRate": 75
      }
    ]
  }
}
```

---

### 5.2 Module Effectiveness
**Endpoint**: `GET /analytics/effectiveness`  
**Access**: ADMIN  
**Description**: Analyze training module effectiveness

**Query Parameters**:
- `moduleId`: Specific module (optional)
- `subject`: Filter by subject
- `clusterId`: Filter by cluster

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "topPerformingModules": [
      {
        "moduleId": "M001",
        "title": "Engaging Students in Math",
        "metrics": {
          "completionRate": 94,
          "avgRating": 4.7,
          "helpfulPercentage": 92,
          "implementationRate": 78
        },
        "impact": {
          "challengeResolutionRate": 85,
          "followupFeedback": "positive"
        }
      }
    ],
    "underperformingModules": [
      {
        "moduleId": "M050",
        "title": "Advanced Assessment",
        "metrics": {
          "completionRate": 45,
          "avgRating": 3.2,
          "helpfulPercentage": 58
        },
        "issues": [
          "Too advanced for target audience",
          "Lacks practical examples"
        ],
        "recommendations": [
          "Add beginner prerequisites",
          "Include video demonstrations"
        ]
      }
    ]
  }
}
```

---

### 5.3 Challenge Analysis
**Endpoint**: `GET /analytics/challenges`  
**Access**: ADMIN, COORDINATOR  
**Description**: Analyze classroom challenges across system

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "topChallenges": [
      {
        "challenge": "student_engagement",
        "totalReports": 345,
        "trendDirection": "increasing",
        "percentageChange": 12,
        "affectedClusters": 15,
        "availableModules": 8,
        "avgResolutionTime": "14 days"
      }
    ],
    "challengesByRegion": {
      "North": ["student_engagement", "resource_shortage"],
      "South": ["technology", "assessment"]
    },
    "emergingChallenges": [
      {
        "challenge": "hybrid_teaching",
        "firstReported": "2025-12-15",
        "growthRate": 45,
        "clustersAffected": 5
      }
    ],
    "heatmap": [
      {
        "clusterId": "C001",
        "lat": 28.7041,
        "lng": 77.1025,
        "challengeIntensity": 0.85,
        "primaryChallenge": "student_engagement"
      }
    ]
  }
}
```

---

### 5.4 Satisfaction Scores
**Endpoint**: `GET /analytics/satisfaction`  
**Access**: ADMIN  
**Description**: Teacher satisfaction metrics

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "overall": {
      "avgRating": 4.3,
      "nps": 72,
      "satisfactionRate": 86
    },
    "byDimension": {
      "contentQuality": 4.5,
      "relevance": 4.6,
      "easeOfUse": 4.2,
      "support": 4.0
    },
    "trends": {
      "monthly": [
        { "month": "2025-12", "score": 4.1 },
        { "month": "2026-01", "score": 4.3 }
      ]
    },
    "testimonials": [
      {
        "teacherId": "T001",
        "comment": "Very helpful and practical",
        "rating": 5,
        "date": "2026-01-02"
      }
    ]
  }
}
```

---

### 5.5 Export Report
**Endpoint**: `GET /analytics/export`  
**Access**: ADMIN  
**Description**: Export analytics report

**Query Parameters**:
- `format`: pdf | excel | csv
- `reportType`: engagement | effectiveness | challenges | satisfaction
- `startDate`: ISO date
- `endDate`: ISO date

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "downloadUrl": "https://s3.amazonaws.com/reports/report_12345.pdf",
    "expiresIn": 3600,
    "fileSize": 2.5
  }
}
```

---

## 6. AI Services

### 6.1 Trigger Content Chunking
**Endpoint**: `POST /ai/chunk-content`  
**Access**: ADMIN  
**Description**: Manually trigger AI chunking for uploaded content

**Request Body**:
```json
{
  "uploadId": "UP12345",
  "targetDuration": 10,
  "chunkingStrategy": "semantic"
}
```

**Response**: `202 Accepted`
```json
{
  "success": true,
  "data": {
    "jobId": "AI_JOB_12345",
    "status": "QUEUED",
    "estimatedTime": "3-5 minutes"
  }
}
```

---

### 6.2 Generate Cluster Adaptations
**Endpoint**: `POST /ai/adapt-module`  
**Access**: ADMIN  
**Description**: Generate cluster-specific adaptations

**Request Body**:
```json
{
  "moduleId": "M001",
  "clusterIds": ["C001", "C002", "C003"]
}
```

**Response**: `202 Accepted`
```json
{
  "success": true,
  "data": {
    "jobId": "AI_JOB_12346",
    "adaptations": [
      {
        "clusterId": "C001",
        "language": "hindi",
        "status": "PROCESSING"
      }
    ]
  }
}
```

---

### 6.3 Get AI Job Status
**Endpoint**: `GET /ai/jobs/:jobId`  
**Access**: ADMIN  
**Description**: Check AI processing job status

**Response**: `200 OK`
```json
{
  "success": true,
  "data": {
    "jobId": "AI_JOB_12345",
    "type": "CONTENT_CHUNKING",
    "status": "COMPLETED",
    "progress": 100,
    "startedAt": "2026-01-03T10:00:00Z",
    "completedAt": "2026-01-03T10:04:32Z",
    "result": {
      "chunksCreated": 5,
      "moduleIds": ["M001-1", "M001-2", "M001-3", "M001-4", "M001-5"]
    }
  }
}
```

---

## Webhooks

### Webhook Events
Admin can configure webhooks to receive notifications for key events:

**Event Types**:
- `feedback.submitted`: New feedback received
- `module.completed`: Teacher completed module
- `cohort.created`: New cohort formed
- `challenge.urgent`: Urgent challenge detected
- `module.published`: New module published

**Webhook Payload**:
```json
{
  "event": "feedback.submitted",
  "timestamp": "2026-01-03T12:00:00Z",
  "data": {
    "feedbackId": "F12345",
    "teacherId": "T001",
    "clusterId": "C001",
    "challenges": ["student_engagement"]
  }
}
```

---

## Rate Limiting

**Limits by Role**:
- TEACHER: 100 requests / 15 minutes
- COORDINATOR: 500 requests / 15 minutes
- ADMIN: 1000 requests / 15 minutes

**Response when rate limited** (429):
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retryAfter": 300
  }
}
```

---

## Pagination

All list endpoints support pagination with consistent parameters:

**Query Parameters**:
- `page`: Page number (1-indexed)
- `limit`: Items per page (max 100)
- `sort`: Sort field (e.g., `createdAt`)
- `order`: `asc` or `desc`

**Response includes**:
```json
{
  "data": [...],
  "pagination": {
    "currentPage": 1,
    "totalPages": 10,
    "totalItems": 95,
    "itemsPerPage": 10,
    "hasNext": true,
    "hasPrev": false
  }
}
```

---

## Error Codes

| Code | Description |
|------|-------------|
| `INVALID_REQUEST` | Request validation failed |
| `UNAUTHORIZED` | Missing or invalid authentication |
| `FORBIDDEN` | Insufficient permissions |
| `NOT_FOUND` | Resource not found |
| `CONFLICT` | Resource already exists |
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `SERVER_ERROR` | Internal server error |
| `AI_PROCESSING_ERROR` | AI service error |
| `DATABASE_ERROR` | Database operation failed |

---

**API Version**: v1  
**Last Updated**: January 3, 2026  
**Support**: api-support@gurunirman.edu.in
