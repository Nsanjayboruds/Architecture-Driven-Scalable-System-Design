# AI System Design - GuruNirman Platform

## Overview

The AI layer is the core intelligence of GuruNirman, responsible for:
1. **Content Chunking**: Breaking long training manuals into micro-learning modules
2. **Content Adaptation**: Localizing content for specific clusters (language, resources, culture)
3. **Feedback Analysis**: Extracting insights from teacher feedback
4. **Recommendation Engine**: Matching teachers with relevant training modules
5. **Pattern Recognition**: Identifying emerging challenges across clusters

---

## Architecture

### High-Level AI Pipeline

```
┌──────────────────────────────────────────────────────────────────┐
│                        INPUT SOURCES                             │
├──────────────────────────────────────────────────────────────────┤
│  Base Training Content  │  Teacher Feedback  │  Teacher Profiles │
│  (PDFs, Documents)      │  (Text, Voice)     │  (Metadata)       │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│                    PREPROCESSING LAYER                           │
├──────────────────────────────────────────────────────────────────┤
│  • Text Extraction (OCR, PDF parsing)                            │
│  • Data Cleaning & Normalization                                 │
│  • Language Detection                                            │
│  • Tokenization                                                  │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│                       AI CORE SERVICES                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────┐  ┌────────────────────┐                │
│  │ Content Chunker    │  │ Adaptation Engine  │                │
│  │ - LLM-based        │  │ - Translation      │                │
│  │ - Semantic split   │  │ - Localization     │                │
│  └────────────────────┘  └────────────────────┘                │
│                                                                  │
│  ┌────────────────────┐  ┌────────────────────┐                │
│  │ Feedback Analyzer  │  │ Recommendation     │                │
│  │ - NLP Pipeline     │  │ - Collaborative    │                │
│  │ - Sentiment        │  │ - Content-based    │                │
│  └────────────────────┘  └────────────────────┘                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│                      EMBEDDING & VECTOR STORE                    │
├──────────────────────────────────────────────────────────────────┤
│  • Module Embeddings (1536-dim vectors)                          │
│  • Teacher Profile Embeddings                                    │
│  • Feedback Embeddings                                           │
│  • Semantic Search via Vector Similarity                         │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│                         OUTPUTS                                  │
├──────────────────────────────────────────────────────────────────┤
│  Micro-modules  │  Adapted Content  │  Insights  │  Recommendations│
└──────────────────────────────────────────────────────────────────┘
```

---

## 1. Content Chunking Engine

### Objective
Transform lengthy training manuals (50-100 pages) into bite-sized, focused micro-learning modules (5-10 minutes each).

### Approach

#### Stage 1: Document Analysis
```python
def analyze_document(document_text):
    """
    Analyze document structure and extract metadata
    """
    analysis = {
        'total_words': count_words(document_text),
        'estimated_reading_time': calculate_reading_time(document_text),
        'structure': {
            'chapters': extract_chapters(document_text),
            'sections': extract_sections(document_text),
            'subsections': extract_subsections(document_text)
        },
        'topics': identify_topics_lda(document_text, n_topics=10),
        'key_concepts': extract_key_concepts(document_text)
    }
    return analysis
```

**Techniques Used**:
- **Header Detection**: Identify document hierarchy (H1, H2, H3)
- **Topic Modeling**: Latent Dirichlet Allocation (LDA) to discover themes
- **Named Entity Recognition**: Extract key concepts, methodologies

#### Stage 2: Semantic Chunking
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from sentence_transformers import SentenceTransformer

def semantic_chunk(document, target_duration=10):
    """
    Split document into semantically coherent chunks
    
    Args:
        document: Full document text
        target_duration: Target minutes per chunk
    
    Returns:
        List of chunks with metadata
    """
    # Calculate target tokens (avg reading: 200 words/min)
    target_words = target_duration * 200
    target_tokens = int(target_words * 1.3)  # ~1.3 tokens per word
    
    # Initialize semantic splitter
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=target_tokens,
        chunk_overlap=200,  # Maintain context
        separators=["\n\n", "\n", ". ", " "],
        length_function=count_tokens
    )
    
    # Split document
    initial_chunks = splitter.split_text(document)
    
    # Refine chunks for semantic coherence
    model = SentenceTransformer('all-MiniLM-L6-v2')
    refined_chunks = []
    
    for chunk in initial_chunks:
        # Check if chunk is semantically complete
        if is_semantically_complete(chunk, model):
            refined_chunks.append(chunk)
        else:
            # Merge with next chunk if incomplete
            refined_chunks[-1] += " " + chunk
    
    return refined_chunks

def is_semantically_complete(chunk, model):
    """
    Check if chunk represents a complete thought
    """
    sentences = chunk.split('. ')
    if len(sentences) < 3:
        return False
    
    # Check embedding similarity of first and last sentence
    embeddings = model.encode([sentences[0], sentences[-1]])
    similarity = cosine_similarity([embeddings[0]], [embeddings[1]])[0][0]
    
    # If similarity is high, chunk is likely self-contained
    return similarity > 0.5
```

#### Stage 3: LLM-Enhanced Structuring
```python
from openai import OpenAI

client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

def structure_chunk_with_llm(chunk_text):
    """
    Use LLM to create structured micro-learning module
    """
    prompt = f"""
You are an expert instructional designer. Transform the following educational content 
into a structured micro-learning module suitable for teachers.

Content:
{chunk_text}

Create a JSON structure with:
1. title: Clear, engaging title (5-8 words)
2. learningObjectives: 2-3 specific, measurable objectives
3. introduction: Hook to engage learners (2-3 sentences)
4. mainContent: Core teaching content
   - Organized into 2-4 subsections
   - Each subsection has: heading, explanation, examples
5. practicalActivity: One concrete classroom activity
6. keyTakeaways: 3-5 bullet points
7. reflectionQuestion: One thought-provoking question
8. estimatedDuration: Realistic time in minutes

Ensure content is:
- Practical and actionable
- Directly applicable to Indian classrooms
- Written in clear, simple language
- Includes specific examples

Output only valid JSON.
"""
    
    response = client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=[
            {"role": "system", "content": "You are an expert instructional designer specializing in teacher training."},
            {"role": "user", "content": prompt}
        ],
        response_format={"type": "json_object"},
        temperature=0.7
    )
    
    module_structure = json.loads(response.choices[0].message.content)
    return module_structure
```

### Example Transformation

**Input** (Fragment of 50-page manual):
```
Chapter 3: Assessment Strategies

Assessment is a crucial component of effective teaching. Teachers must use 
various assessment methods to gauge student understanding. Formative assessments 
help track progress, while summative assessments evaluate overall learning...
[continues for 10 pages]
```

**Output** (Micro-module):
```json
{
  "moduleId": "M001-3",
  "title": "Quick Formative Assessment Techniques",
  "duration": 8,
  "learningObjectives": [
    "Implement 3 quick formative assessment techniques in daily lessons",
    "Use assessment data to adjust teaching strategies",
    "Provide effective feedback to students based on assessments"
  ],
  "introduction": "How do you know if your students are really learning during the lesson? Formative assessments give you real-time insights into student understanding, allowing you to adjust your teaching on the spot.",
  "mainContent": {
    "sections": [
      {
        "heading": "What is Formative Assessment?",
        "content": "...",
        "examples": ["Exit tickets", "Think-Pair-Share"]
      },
      {
        "heading": "3 Quick Techniques You Can Use Today",
        "content": "...",
        "examples": [
          {
            "technique": "Thumbs Up/Down",
            "description": "Students show understanding with hand signals",
            "whenToUse": "After explaining a new concept"
          }
        ]
      }
    ]
  },
  "practicalActivity": {
    "title": "Try It Tomorrow",
    "description": "Choose one technique and use it in your next math lesson...",
    "materials": ["None required"],
    "duration": 5
  },
  "keyTakeaways": [
    "Formative assessment happens during learning, not after",
    "Quick techniques take less than 5 minutes",
    "Use results to adjust your teaching immediately"
  ],
  "reflectionQuestion": "Which formative assessment technique would work best with your current topic?"
}
```

---

## 2. Content Adaptation Engine

### Objective
Customize micro-modules for specific clusters based on:
- **Language**: Translate to local language
- **Resources**: Adjust activities for low/medium/high resource settings
- **Culture**: Use locally relevant examples
- **Challenges**: Emphasize aspects relevant to cluster's common challenges

### Multi-Dimensional Adaptation

```python
class AdaptationEngine:
    def __init__(self):
        self.translator = TranslationService()
        self.examples_db = LocalExamplesDatabase()
        self.resource_adjuster = ResourceLevelAdjuster()
    
    def adapt_module(self, base_module, cluster_context):
        """
        Adapt base module for specific cluster
        
        Args:
            base_module: Structured module from chunking engine
            cluster_context: {
                'clusterId': 'C001',
                'language': 'hindi',
                'resourceLevel': 'low',
                'region': 'rural_north',
                'commonChallenges': ['student_engagement', 'resource_shortage']
            }
        
        Returns:
            Adapted module
        """
        adapted = copy.deepcopy(base_module)
        
        # 1. Language Translation
        if cluster_context['language'] != 'english':
            adapted = self.translate_content(adapted, cluster_context['language'])
        
        # 2. Localize Examples
        adapted = self.localize_examples(adapted, cluster_context)
        
        # 3. Adjust for Resource Level
        adapted = self.adjust_resources(adapted, cluster_context['resourceLevel'])
        
        # 4. Cultural Adaptation
        adapted = self.cultural_adaptation(adapted, cluster_context['region'])
        
        # 5. Emphasize Relevant Challenges
        adapted = self.emphasize_challenges(adapted, cluster_context['commonChallenges'])
        
        # Add adaptation metadata
        adapted['adaptationMetadata'] = {
            'baseModuleId': base_module['moduleId'],
            'clusterId': cluster_context['clusterId'],
            'adaptations': ['language', 'examples', 'resources', 'culture'],
            'adaptedAt': datetime.now().isoformat()
        }
        
        return adapted
```

#### 2.1 Language Translation

```python
def translate_content(self, module, target_language):
    """
    Translate module content while preserving structure
    """
    # Use GPT-4 for context-aware translation
    prompt = f"""
Translate the following educational module from English to {target_language}.

Requirements:
1. Maintain educational terminology accuracy
2. Use simple, clear language appropriate for teachers
3. Keep formatting and structure intact
4. Preserve JSON structure
5. Translate all text fields but keep field names in English

Module:
{json.dumps(module, indent=2)}

Return the translated module as valid JSON.
"""
    
    response = client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=[
            {"role": "system", "content": f"You are an expert translator specializing in educational content. Translate to {target_language} while maintaining accuracy and simplicity."},
            {"role": "user", "content": prompt}
        ],
        response_format={"type": "json_object"},
        temperature=0.3  # Low temperature for consistent translation
    )
    
    translated_module = json.loads(response.choices[0].message.content)
    return translated_module
```

#### 2.2 Example Localization

```python
def localize_examples(self, module, context):
    """
    Replace generic examples with locally relevant ones
    """
    region = context['region']
    subject = module.get('subject', 'general')
    
    # Query local examples database
    local_examples = self.examples_db.get_examples(
        region=region,
        subject=subject,
        count=5
    )
    
    # Use LLM to intelligently substitute examples
    prompt = f"""
Given this teaching module and a set of locally relevant examples, 
replace generic examples with local ones where appropriate.

Module:
{json.dumps(module['mainContent'], indent=2)}

Local Examples for {region}:
{json.dumps(local_examples, indent=2)}

Guidelines:
1. Maintain the learning objective
2. Use local examples that teachers will recognize
3. Ensure examples are culturally appropriate
4. Keep the same structure

Return the updated mainContent section as JSON.
"""
    
    # LLM call to substitute examples...
    # [Implementation similar to previous LLM calls]
    
    return module
```

#### 2.3 Resource Level Adjustment

```python
def adjust_resources(self, module, resource_level):
    """
    Modify activities based on available resources
    
    resource_level: 'low' | 'medium' | 'high'
    """
    if 'practicalActivity' not in module:
        return module
    
    activity = module['practicalActivity']
    
    # Resource substitution rules
    substitutions = {
        'low': {
            'smartboard': 'chalkboard and chart paper',
            'tablets': 'printed worksheets',
            'projector': 'blackboard drawings',
            'internet': 'textbook examples',
            'manipulatives': 'locally available materials (stones, sticks)'
        },
        'medium': {
            'smartboard': 'whiteboard with markers',
            'tablets': 'shared tablets (1 per group)',
            'projector': 'printed large charts'
        }
    }
    
    if resource_level in ['low', 'medium']:
        # Use LLM to adapt activity
        prompt = f"""
Adapt this classroom activity for a {resource_level}-resource setting.

Original Activity:
{json.dumps(activity, indent=2)}

Resource Substitutions:
{json.dumps(substitutions[resource_level], indent=2)}

Guidelines:
1. Maintain the learning objective
2. Make activity feasible with available resources
3. Keep it practical and simple
4. Ensure pedagogical effectiveness is not compromised

Return adapted activity as JSON.
"""
        
        # LLM adaptation...
        adapted_activity = self.llm_adapt(prompt)
        module['practicalActivity'] = adapted_activity
    
    return module
```

**Example**:

Original (High-resource):
```json
{
  "practicalActivity": {
    "title": "Interactive Geometry Exploration",
    "description": "Students use tablets with GeoGebra app to explore geometric transformations",
    "materials": ["Tablets (1 per student)", "GeoGebra app", "WiFi connection"],
    "steps": [...]
  }
}
```

Adapted (Low-resource):
```json
{
  "practicalActivity": {
    "title": "Hands-On Geometry Exploration",
    "description": "छात्र रस्सी और लकड़ी के डंडों का उपयोग करके ज्यामितीय रूपांतरणों का पता लगाते हैं",
    "materials": ["रस्सी", "लकड़ी के डंडे या छड़ियाँ", "चाक", "फर्श की जगह"],
    "steps": [
      "फर्श पर एक बड़ा वर्ग बनाने के लिए रस्सी और डंडों का उपयोग करें",
      "छात्रों को वर्ग को घुमाने और प्रतिबिंबित करने के लिए कहें",
      "परिवर्तनों पर चर्चा करें और चाक से चिह्नित करें"
    ]
  }
}
```

---

## 3. Feedback Analysis Engine

### Objective
Extract actionable insights from teacher feedback to:
- Identify common challenges
- Detect urgent issues
- Recommend relevant training modules
- Inform content creation priorities

### NLP Pipeline

```python
class FeedbackAnalyzer:
    def __init__(self):
        self.sentiment_model = pipeline("sentiment-analysis", model="nlptown/bert-base-multilingual-uncased-sentiment")
        self.ner_model = pipeline("ner", model="xlm-roberta-large-finetuned-conll03-english")
        self.embedder = SentenceTransformer('paraphrase-multilingual-mpnet-base-v2')
        
    def analyze_feedback(self, feedback_text, metadata):
        """
        Comprehensive feedback analysis
        
        Args:
            feedback_text: Teacher's feedback text
            metadata: {
                'teacherId', 'subject', 'grade', 'clusterId'
            }
        
        Returns:
            Analysis results
        """
        analysis = {
            'feedbackId': generate_id(),
            'processedAt': datetime.now().isoformat(),
            'sentiment': self.analyze_sentiment(feedback_text),
            'keywords': self.extract_keywords(feedback_text),
            'topics': self.identify_topics(feedback_text),
            'urgency': self.calculate_urgency(feedback_text),
            'entities': self.extract_entities(feedback_text),
            'suggestedModules': self.recommend_modules(feedback_text, metadata),
            'similarFeedback': self.find_similar_feedback(feedback_text),
            'actionableInsights': self.extract_insights(feedback_text)
        }
        
        return analysis
```

#### 3.1 Sentiment Analysis

```python
def analyze_sentiment(self, text):
    """
    Analyze emotional tone of feedback
    """
    # Multi-lingual BERT for sentiment
    result = self.sentiment_model(text[:512])[0]  # Truncate for model limits
    
    # Map 5-star rating to sentiment
    score = int(result['label'].split()[0])
    
    sentiment_map = {
        1: {'label': 'VERY_NEGATIVE', 'score': -1.0},
        2: {'label': 'NEGATIVE', 'score': -0.5},
        3: {'label': 'NEUTRAL', 'score': 0.0},
        4: {'label': 'POSITIVE', 'score': 0.5},
        5: {'label': 'VERY_POSITIVE', 'score': 1.0}
    }
    
    return sentiment_map.get(score, {'label': 'NEUTRAL', 'score': 0.0})
```

#### 3.2 Keyword & Topic Extraction

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.decomposition import LatentDirichletAllocation

def extract_keywords(self, text, top_n=10):
    """
    Extract important keywords using TF-IDF
    """
    # Simple TF-IDF for keyword extraction
    vectorizer = TfidfVectorizer(
        max_features=100,
        stop_words='english',  # Multi-language stop words needed
        ngram_range=(1, 2)
    )
    
    tfidf_matrix = vectorizer.fit_transform([text])
    feature_names = vectorizer.get_feature_names_out()
    
    # Get top keywords
    scores = tfidf_matrix.toarray()[0]
    top_indices = scores.argsort()[-top_n:][::-1]
    keywords = [feature_names[i] for i in top_indices if scores[i] > 0]
    
    return keywords

def identify_topics(self, text):
    """
    Identify latent topics in feedback
    """
    # For single document, use LLM for topic identification
    prompt = f"""
Analyze this teacher feedback and identify the main educational topics/challenges mentioned.

Feedback:
{text}

List 2-4 specific topics from this list:
- student_engagement
- concept_clarity
- classroom_management
- assessment_methods
- differentiation
- resource_shortage
- parent_communication
- technology_integration
- time_management
- curriculum_pacing
- special_needs
- language_barriers
- peer_learning
- motivation
- homework_completion

Return only topic names as a JSON array, in order of prominence.
"""
    
    response = client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=[{"role": "user", "content": prompt}],
        response_format={"type": "json_object"},
        temperature=0.3
    )
    
    topics = json.loads(response.choices[0].message.content)
    return topics['topics']
```

#### 3.3 Urgency Detection

```python
def calculate_urgency(self, text):
    """
    Calculate urgency score (0-1) based on linguistic markers
    """
    # Urgency indicators
    urgent_keywords = [
        'urgent', 'critical', 'emergency', 'immediately', 'crisis',
        'failing', 'severe', 'desperate', 'cannot', 'unable',
        'dropping out', 'giving up', 'completely lost', 'no progress'
    ]
    
    severity_keywords = [
        'all students', 'entire class', 'every student', 'nobody',
        'never', 'always', 'constantly', 'daily', 'repeatedly'
    ]
    
    # Count matches
    text_lower = text.lower()
    urgent_count = sum(1 for keyword in urgent_keywords if keyword in text_lower)
    severity_count = sum(1 for keyword in severity_keywords if keyword in text_lower)
    
    # Sentiment factor (negative sentiment increases urgency)
    sentiment = self.analyze_sentiment(text)
    sentiment_factor = abs(min(sentiment['score'], 0))  # 0 to 1
    
    # Length factor (longer feedback may indicate deeper issues)
    length_factor = min(len(text.split()) / 200, 1.0)  # Normalized
    
    # Calculate weighted urgency score
    urgency_score = (
        0.4 * min(urgent_count / 3, 1.0) +
        0.3 * min(severity_count / 2, 1.0) +
        0.2 * sentiment_factor +
        0.1 * length_factor
    )
    
    return {
        'score': round(urgency_score, 2),
        'level': 'CRITICAL' if urgency_score > 0.7 else 'HIGH' if urgency_score > 0.5 else 'MEDIUM' if urgency_score > 0.3 else 'LOW',
        'factors': {
            'urgentKeywords': urgent_count,
            'severityKeywords': severity_count,
            'sentiment': sentiment['label']
        }
    }
```

#### 3.4 Module Recommendation from Feedback

```python
def recommend_modules(self, feedback_text, metadata):
    """
    Recommend training modules based on feedback content
    """
    # 1. Get feedback embedding
    feedback_embedding = self.embedder.encode(feedback_text)
    
    # 2. Query vector database for similar modules
    similar_modules = vector_db.query(
        vector=feedback_embedding,
        filter={
            'subject': metadata['subject'],
            'grade': {'$in': metadata['grade']}
        },
        top_k=10
    )
    
    # 3. Re-rank using LLM for relevance
    prompt = f"""
Given this teacher feedback and a list of training modules, rank the modules 
by relevance to addressing the teacher's challenges.

Feedback:
{feedback_text}

Available Modules:
{json.dumps([m['metadata'] for m in similar_modules], indent=2)}

Return top 5 module IDs in order of relevance, with a brief reason for each.
Format: JSON array of objects with 'moduleId' and 'reason' fields.
"""
    
    response = client.chat.completions.create(
        model="gpt-4-turbo-preview",
        messages=[{"role": "user", "content": prompt}],
        response_format={"type": "json_object"},
        temperature=0.3
    )
    
    recommendations = json.loads(response.choices[0].message.content)
    return recommendations['modules']
```

---

## 4. Recommendation Engine

### Hybrid Approach
Combines multiple recommendation strategies:

1. **Collaborative Filtering**: "Teachers like you also completed..."
2. **Content-Based**: Match module tags to teacher challenges/interests
3. **Knowledge-Based**: Rules based on pedagogical progression
4. **Contextual**: Consider cluster, recent feedback, time of year

```python
class RecommendationEngine:
    def __init__(self):
        self.embedder = SentenceTransformer('all-MiniLM-L6-v2')
        
    def generate_recommendations(self, teacher_id, limit=5):
        """
        Generate personalized module recommendations
        """
        # Get teacher profile
        teacher = db.users.find_one({'userId': teacher_id})
        
        # Get teacher's history
        completed_modules = self.get_completed_modules(teacher_id)
        recent_feedback = self.get_recent_feedback(teacher_id, days=30)
        
        # Calculate scores from multiple sources
        collab_scores = self.collaborative_filtering(teacher_id, completed_modules)
        content_scores = self.content_based_filtering(teacher, recent_feedback)
        knowledge_scores = self.knowledge_based_rules(teacher, completed_modules)
        context_scores = self.contextual_scoring(teacher)
        
        # Hybrid combination (weighted average)
        combined_scores = self.combine_scores({
            'collaborative': (collab_scores, 0.25),
            'content': (content_scores, 0.35),
            'knowledge': (knowledge_scores, 0.25),
            'contextual': (context_scores, 0.15)
        })
        
        # Filter out already completed
        filtered_scores = {
            mid: score for mid, score in combined_scores.items()
            if mid not in completed_modules
        }
        
        # Get top recommendations
        top_module_ids = sorted(filtered_scores.keys(), 
                                key=lambda x: filtered_scores[x], 
                                reverse=True)[:limit]
        
        # Fetch module details and add explanations
        recommendations = []
        for module_id in top_module_ids:
            module = db.trainingModules.find_one({'moduleId': module_id})
            
            recommendations.append({
                'moduleId': module_id,
                'title': module['title'],
                'matchScore': filtered_scores[module_id],
                'reasons': self.explain_recommendation(
                    module_id, teacher, recent_feedback, completed_modules
                ),
                'estimatedImpact': self.estimate_impact(module_id, teacher),
                'duration': module['estimatedDuration']
            })
        
        return recommendations
```

#### 4.1 Collaborative Filtering

```python
def collaborative_filtering(self, teacher_id, completed_modules):
    """
    Find similar teachers and recommend what they completed
    """
    # Find similar teachers using cosine similarity on embeddings
    teacher_profile = db.users.find_one({'userId': teacher_id})
    teacher_embedding = self.create_teacher_embedding(teacher_profile)
    
    # Query for similar teachers in same cluster
    similar_teachers = db.users.find({
        'clusterId': teacher_profile['clusterId'],
        'userId': {'$ne': teacher_id},
        'role': 'TEACHER'
    })
    
    # Calculate similarity scores
    similarities = []
    for other_teacher in similar_teachers:
        other_embedding = self.create_teacher_embedding(other_teacher)
        similarity = cosine_similarity([teacher_embedding], [other_embedding])[0][0]
        similarities.append((other_teacher['userId'], similarity))
    
    # Get top 20 similar teachers
    top_similar = sorted(similarities, key=lambda x: x[1], reverse=True)[:20]
    
    # Find modules they completed and liked
    module_scores = defaultdict(float)
    
    for similar_teacher_id, similarity in top_similar:
        their_progress = db.moduleProgress.find({
            'teacherId': similar_teacher_id,
            'status': 'COMPLETED',
            'rating': {'$gte': 4}  # Only highly rated modules
        })
        
        for progress in their_progress:
            if progress['moduleId'] not in completed_modules:
                # Weight by similarity and their rating
                module_scores[progress['moduleId']] += similarity * (progress['rating'] / 5.0)
    
    # Normalize scores
    max_score = max(module_scores.values()) if module_scores else 1
    normalized_scores = {mid: score/max_score for mid, score in module_scores.items()}
    
    return normalized_scores

def create_teacher_embedding(self, teacher_profile):
    """
    Create embedding from teacher characteristics
    """
    # Concatenate relevant features
    text = f"{teacher_profile['subject']} {' '.join(teacher_profile['grade'])} {' '.join(teacher_profile['challenges'])}"
    embedding = self.embedder.encode(text)
    return embedding
```

#### 4.2 Content-Based Filtering

```python
def content_based_filtering(self, teacher, recent_feedback):
    """
    Match module content to teacher's challenges and interests
    """
    # Extract challenges from teacher profile and recent feedback
    challenges = set(teacher['challenges'])
    
    for feedback in recent_feedback:
        if 'aiAnalysis' in feedback and 'topics' in feedback['aiAnalysis']:
            challenges.update(feedback['aiAnalysis']['topics'])
    
    # Create query embedding
    query_text = f"{teacher['subject']} {' '.join(challenges)}"
    query_embedding = self.embedder.encode(query_text)
    
    # Query vector database
    similar_modules = vector_db.query(
        vector=query_embedding,
        filter={
            'subject': teacher['subject'],
            'status': 'PUBLISHED'
        },
        top_k=50
    )
    
    # Convert to scores
    scores = {m['metadata']['moduleId']: m['score'] for m in similar_modules}
    
    # Boost score if module explicitly tagged with challenge
    for module_id, score in scores.items():
        module = db.trainingModules.find_one({'moduleId': module_id})
        tag_overlap = len(set(module['tags']) & challenges)
        scores[module_id] = score * (1 + 0.1 * tag_overlap)
    
    return scores
```

---

## 5. Vector Store & Embeddings

### Setup (Pinecone)

```python
import pinecone
from sentence_transformers import SentenceTransformer

# Initialize Pinecone
pinecone.init(api_key=os.getenv("PINECONE_API_KEY"), environment="us-west1-gcp")

# Create index
index_name = "gurunirman-modules"
if index_name not in pinecone.list_indexes():
    pinecone.create_index(
        name=index_name,
        dimension=384,  # all-MiniLM-L6-v2 dimension
        metric="cosine"
    )

index = pinecone.Index(index_name)

# Initialize embedder
embedder = SentenceTransformer('all-MiniLM-L6-v2')
```

### Indexing Modules

```python
def index_module(module):
    """
    Create and store module embedding
    """
    # Create text representation of module
    text = f"{module['title']} {module['description']} {' '.join(module['tags'])}"
    
    # Generate embedding
    embedding = embedder.encode(text).tolist()
    
    # Store in Pinecone with metadata
    index.upsert(vectors=[{
        'id': module['moduleId'],
        'values': embedding,
        'metadata': {
            'title': module['title'],
            'subject': module['subject'],
            'grade': module['grade'],
            'tags': module['tags'],
            'difficulty': module['difficulty'],
            'duration': module['estimatedDuration']
        }
    }])
```

### Semantic Search

```python
def semantic_search(query_text, filters=None, top_k=10):
    """
    Search modules using semantic similarity
    """
    # Generate query embedding
    query_embedding = embedder.encode(query_text).tolist()
    
    # Query Pinecone
    results = index.query(
        vector=query_embedding,
        filter=filters,
        top_k=top_k,
        include_metadata=True
    )
    
    return results['matches']

# Example usage
results = semantic_search(
    query_text="engaging students in mathematics using games",
    filters={'subject': {'$eq': 'Mathematics'}},
    top_k=5
)
```

---

## 6. AI Infrastructure & Deployment

### Model Selection

| Component | Model | Reason |
|-----------|-------|--------|
| Content Chunking | GPT-4 Turbo | Best understanding of educational content |
| Translation | GPT-4 / mBART | Multi-lingual quality |
| Sentiment Analysis | multilingual-BERT | Good for Indian languages |
| Embeddings | all-MiniLM-L6-v2 | Fast, good quality, small size |
| NER | XLM-RoBERTa | Multi-lingual support |

### Cost Optimization

```python
# Caching strategy
import hashlib
from functools import lru_cache

def cache_key(text):
    """Generate cache key for text"""
    return hashlib.md5(text.encode()).hexdigest()

@lru_cache(maxsize=1000)
def cached_llm_call(prompt_hash, model="gpt-4-turbo-preview"):
    """
    Cache LLM responses to avoid duplicate API calls
    """
    # Check Redis cache first
    cached_response = redis_client.get(f"llm:{prompt_hash}")
    if cached_response:
        return json.loads(cached_response)
    
    # Make API call
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        temperature=0.3
    )
    
    result = response.choices[0].message.content
    
    # Cache for 30 days
    redis_client.setex(f"llm:{prompt_hash}", 2592000, json.dumps(result))
    
    return result
```

### Batch Processing

```python
from celery import Celery

app = Celery('ai_tasks', broker='redis://localhost:6379/0')

@app.task
def batch_chunk_content(upload_id):
    """
    Background task for content chunking
    """
    upload = db.uploads.find_one({'uploadId': upload_id})
    
    # Update status
    db.uploads.update_one(
        {'uploadId': upload_id},
        {'$set': {'processingStatus': 'PROCESSING', 'processingStage': 'CHUNKING'}}
    )
    
    try:
        # Extract text from file
        text = extract_text_from_file(upload['url'])
        
        # Chunk content
        chunks = semantic_chunk(text, target_duration=10)
        
        # Structure each chunk
        modules = []
        for i, chunk in enumerate(chunks):
            structured_module = structure_chunk_with_llm(chunk)
            structured_module['moduleId'] = f"{upload['metadata']['subject'][:3].upper()}{i+1:03d}"
            modules.append(structured_module)
        
        # Save modules
        for module in modules:
            db.trainingModules.insert_one(module)
        
        # Update upload status
        db.uploads.update_one(
            {'uploadId': upload_id},
            {'$set': {
                'processingStatus': 'COMPLETED',
                'generatedModules': [m['moduleId'] for m in modules]
            }}
        )
        
    except Exception as e:
        db.uploads.update_one(
            {'uploadId': upload_id},
            {'$set': {
                'processingStatus': 'FAILED',
                'error': {'message': str(e)}
            }}
        )

# Trigger batch job
batch_chunk_content.delay(upload_id='UP12345')
```

---

## 7. Performance Metrics

### AI System KPIs

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Chunking Accuracy | > 85% | Human eval: coherence score |
| Translation Quality | > 90% BLEU | Back-translation + human review |
| Recommendation Precision | > 70% | Click-through rate + completion rate |
| Feedback Analysis Accuracy | > 80% | Compare AI topics to manual labels |
| Processing Time | < 5 min per doc | Automated logging |
| Cost per Module | < $0.50 | API cost tracking |

### Monitoring

```python
import logging
from prometheus_client import Counter, Histogram

# Metrics
llm_calls = Counter('llm_api_calls_total', 'Total LLM API calls', ['model', 'component'])
llm_latency = Histogram('llm_api_latency_seconds', 'LLM API latency', ['model'])
llm_cost = Counter('llm_api_cost_usd', 'LLM API cost in USD', ['model'])

def monitored_llm_call(prompt, model="gpt-4-turbo-preview", component="unknown"):
    """
    LLM call with monitoring
    """
    start_time = time.time()
    
    try:
        response = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}]
        )
        
        # Record metrics
        latency = time.time() - start_time
        llm_calls.labels(model=model, component=component).inc()
        llm_latency.labels(model=model).observe(latency)
        
        # Calculate cost (simplified)
        tokens = response.usage.total_tokens
        cost = calculate_cost(model, tokens)
        llm_cost.labels(model=model).inc(cost)
        
        logging.info(f"LLM call successful: {model}, {latency:.2f}s, ${cost:.4f}")
        
        return response.choices[0].message.content
        
    except Exception as e:
        logging.error(f"LLM call failed: {model}, {str(e)}")
        raise
```

---

**AI System Version**: 1.0  
**Last Updated**: January 3, 2026  
**Maintained By**: GuruNirman AI Team
