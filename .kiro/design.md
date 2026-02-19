# BharatScheme AI - Design Document

## 1. System Overview

BharatScheme AI is a cloud-native, microservices-based application that leverages AI/ML technologies to provide intelligent government scheme recommendations. The system follows a layered architecture pattern with clear separation of concerns:

- **Presentation Layer**: Web/mobile interface for user interaction
- **API Gateway Layer**: Request routing, rate limiting, and authentication
- **NLP Processing Layer**: Language detection, entity extraction, and intent classification
- **Matching Engine Layer**: Semantic similarity and eligibility reasoning
- **Data Layer**: Scheme database and caching infrastructure
- **Response Generation Layer**: Multilingual response formatting and translation

The system is designed for horizontal scalability, fault tolerance, and low-latency responses. It uses stateless request processing to ensure privacy and scalability.

### Key Design Principles

1. **Privacy by Design**: No persistent storage of user personal data
2. **Multilingual First**: All components support multiple Indian languages
3. **Explainable AI**: Transparent reasoning for all recommendations
4. **Graceful Degradation**: System remains functional even if AI components fail
5. **Mobile-First**: Optimized for low-bandwidth and low-end devices
6. **Modular Architecture**: Independent deployment and scaling of components

## 2. Architecture Diagram Explanation

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER DEVICES                              │
│              (Web Browser / Mobile App / Voice)                  │
└────────────────────────┬────────────────────────────────────────┘
                         │ HTTPS
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CDN / LOAD BALANCER                          │
│                    (CloudFlare / AWS ALB)                        │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API GATEWAY                                 │
│         (Rate Limiting, Routing, Request Validation)             │
└────────┬────────────────────────────────────────────────────────┘
         │
         ├──────────────────┬──────────────────┬──────────────────┐
         ▼                  ▼                  ▼                  ▼
┌────────────────┐  ┌────────────────┐  ┌────────────────┐  ┌──────────┐
│  LANGUAGE      │  │  NLP SERVICE   │  │  MATCHING      │  │ RESPONSE │
│  PROCESSOR     │  │  (Entity       │  │  ENGINE        │  │ GENERATOR│
│  (Detection,   │  │  Extraction,   │  │  (Semantic     │  │ (Format, │
│  Translation,  │  │  Intent        │  │  Similarity,   │  │ Translate│
│  STT)          │  │  Classification)│  │  Ranking)      │  │ Simplify)│
└────────┬───────┘  └────────┬───────┘  └────────┬───────┘  └────┬─────┘
         │                   │                   │                │
         └───────────────────┴───────────────────┴────────────────┘
                                      │
                                      ▼
                         ┌────────────────────────┐
                         │   CACHING LAYER        │
                         │   (Redis)              │
                         └────────────┬───────────┘
                                      │
                                      ▼
                         ┌────────────────────────┐
                         │   SCHEME DATABASE      │
                         │   (PostgreSQL)         │
                         └────────────────────────┘
```

### Architecture Flow

1. **User Request**: User submits query via web/mobile interface (text or voice)
2. **API Gateway**: Validates request, applies rate limiting, routes to appropriate service
3. **Language Processing**: Detects language, converts speech to text if needed
4. **NLP Service**: Extracts entities (income, location, category) and classifies intent
5. **Matching Engine**: Performs semantic similarity matching against scheme database
6. **Response Generator**: Formats results, translates to user language, simplifies content
7. **Cache Layer**: Stores frequently accessed schemes and embeddings for performance
8. **Database**: Persistent storage for scheme data and metadata


## 3. Component Description

### 3.1 Presentation Layer

**Technology Stack**: React.js (Web), React Native (Mobile)

**Responsibilities**:
- Render user interface with multilingual support
- Capture text and voice input
- Display scheme recommendations with visual indicators
- Handle user language preference selection
- Provide accessible UI for low digital literacy users

**Key Features**:
- Progressive Web App (PWA) for offline capability
- Responsive design (320px to 1920px)
- Large touch targets (minimum 44x44px)
- Voice input with visual feedback
- Simple navigation with minimal steps
- Icon-based communication for low literacy users

**Components**:
- `QueryInput`: Text/voice input component with language selector
- `SchemeCard`: Displays individual scheme with eligibility badge
- `DocumentChecklist`: Shows required documents with checkboxes
- `ApplicationGuide`: Step-by-step application instructions
- `FeedbackForm`: Collects user feedback on recommendations

### 3.2 API Gateway Layer

**Technology**: Kong / AWS API Gateway / Express.js

**Responsibilities**:
- Request routing to microservices
- Rate limiting (100 requests/minute per IP)
- Request/response validation
- CORS handling
- API versioning
- Logging and monitoring

**Endpoints**:
```
POST /api/v1/query          - Submit user query
POST /api/v1/voice          - Submit voice input
GET  /api/v1/schemes/:id    - Get scheme details
POST /api/v1/feedback       - Submit feedback
GET  /api/v1/languages      - Get supported languages
```


### 3.3 Language Processing Service

**Technology**: Python (FastAPI), Google Speech-to-Text API / Whisper, IndicNLP

**Responsibilities**:
- Detect input language (Hindi, English, Marathi, Tamil, Telugu)
- Convert voice to text with language-specific models
- Normalize text (remove special characters, standardize numbers)
- Romanization handling (e.g., "mera income 2 lakh hai")

**Components**:
- **Language Detector**: Uses `langdetect` or `fasttext` for language identification
- **Speech-to-Text Engine**: Integrates with Google Cloud Speech-to-Text or OpenAI Whisper
- **Text Normalizer**: Standardizes currency formats, numbers, and common abbreviations
- **Transliteration Handler**: Converts romanized Indian language text to native script

**Input/Output**:
```python
# Input
{
  "text": "मेरी आय 2 लाख है और मैं किसान हूं",
  "mode": "text",  # or "voice"
  "audio_data": null  # base64 encoded audio if mode=voice
}

# Output
{
  "detected_language": "hi",
  "normalized_text": "मेरी आय 200000 है और मैं किसान हूं",
  "confidence": 0.95
}
```

### 3.4 NLP Service (Entity Extraction & Intent Classification)

**Technology**: Python (FastAPI), spaCy, Hugging Face Transformers, Custom NER models

**Responsibilities**:
- Extract structured entities from unstructured text
- Classify user intent (eligibility_check, document_info, scheme_details)
- Handle multi-turn conversations for clarification
- Support code-mixed and transliterated text

**Entity Types**:
- `INCOME`: Annual income amount
- `OCCUPATION`: Job/profession (farmer, student, worker, etc.)
- `LOCATION`: State, district, or city
- `CATEGORY`: Social category (SC, ST, OBC, General)
- `AGE`: User age or age group
- `GENDER`: Male, female, other
- `LANDHOLDING`: Land size in acres/hectares
- `DISABILITY`: Disability status
- `FAMILY_SIZE`: Number of family members


**NER Model Architecture**:
- Base Model: `ai4bharat/indic-bert` or `xlm-roberta-base`
- Fine-tuned on custom dataset of Indian government scheme queries
- Multi-label classification for intent detection
- Confidence thresholding for entity validation

**Input/Output**:
```python
# Input
{
  "text": "मेरी आय 200000 है और मैं किसान हूं",
  "language": "hi"
}

# Output
{
  "entities": {
    "income": 200000,
    "occupation": "farmer",
    "location": null,
    "category": null
  },
  "intent": "eligibility_check",
  "confidence": 0.88,
  "missing_fields": ["location", "category"]
}
```

### 3.5 Matching Engine (Semantic Eligibility Matching)

**Technology**: Python (FastAPI), Sentence Transformers, FAISS, Custom Ranking Algorithm

**Responsibilities**:
- Encode user profile and scheme criteria as embeddings
- Compute semantic similarity scores
- Apply rule-based filters for hard constraints
- Rank schemes by relevance
- Determine eligibility status (Eligible, Possibly Eligible, Not Eligible)

**Matching Algorithm**:

1. **Hard Filter Stage**: Apply mandatory criteria (location, category, age limits)
2. **Semantic Matching Stage**: 
   - Encode user profile: `user_embedding = encode(user_attributes)`
   - Encode scheme criteria: `scheme_embedding = encode(scheme_eligibility)`
   - Compute cosine similarity: `score = cosine_similarity(user_embedding, scheme_embedding)`
3. **Ranking Stage**: Combine semantic score with rule-based score
4. **Threshold Application**: 
   - Score > 0.8: Eligible
   - Score 0.6-0.8: Possibly Eligible
   - Score < 0.6: Not Eligible


**Embedding Model**: 
- `sentence-transformers/paraphrase-multilingual-mpnet-base-v2`
- Fine-tuned on scheme-user profile pairs
- 768-dimensional embeddings

**Input/Output**:
```python
# Input
{
  "user_profile": {
    "income": 200000,
    "occupation": "farmer",
    "location": "Maharashtra",
    "category": "General",
    "landholding": 2.5
  },
  "top_k": 10
}

# Output
{
  "schemes": [
    {
      "scheme_id": "PM-KISAN-001",
      "scheme_name": "PM-KISAN",
      "eligibility_status": "Eligible",
      "relevance_score": 0.92,
      "match_explanation": "You meet all criteria: farmer with landholding < 5 acres"
    },
    {
      "scheme_id": "MAHA-AGRI-002",
      "scheme_name": "Maharashtra Farmer Support",
      "eligibility_status": "Possibly Eligible",
      "relevance_score": 0.75,
      "match_explanation": "You meet most criteria. Verify district eligibility."
    }
  ]
}
```

### 3.6 Scheme Database

**Technology**: PostgreSQL with pgvector extension for vector similarity search

**Responsibilities**:
- Store structured scheme information
- Store pre-computed scheme embeddings
- Support efficient similarity search
- Maintain scheme metadata and versioning

**Schema Design**: See Section 7 for detailed schema


### 3.7 Response Generation Layer

**Technology**: Python (FastAPI), Google Translate API / IndicTrans, Template Engine

**Responsibilities**:
- Format scheme recommendations into user-friendly responses
- Translate responses to user's preferred language
- Simplify technical language for low literacy users
- Generate explanations for eligibility decisions
- Create document checklists and application guides

**Components**:
- **Response Formatter**: Structures data into readable format
- **Translation Service**: Translates content to regional languages
- **Simplification Engine**: Converts technical terms to simple language
- **Explanation Generator**: Creates human-readable eligibility reasoning

**Input/Output**:
```python
# Input
{
  "schemes": [...],  # from matching engine
  "user_language": "hi",
  "user_profile": {...}
}

# Output
{
  "response_text": "आपके लिए 3 योजनाएं मिलीं:\n\n1. PM-KISAN...",
  "schemes": [
    {
      "scheme_name": "पीएम-किसान",
      "eligibility_status": "पात्र",
      "benefits": "₹6000 प्रति वर्ष",
      "documents": ["आधार कार्ड", "बैंक पासबुक", "भूमि दस्तावेज"],
      "application_link": "https://pmkisan.gov.in",
      "explanation": "आप इस योजना के लिए पात्र हैं क्योंकि आप 5 एकड़ से कम भूमि वाले किसान हैं।"
    }
  ]
}
```

### 3.8 Caching Layer

**Technology**: Redis

**Responsibilities**:
- Cache frequently accessed schemes
- Cache pre-computed embeddings
- Store session data temporarily
- Rate limiting counters

**Cache Strategy**:
- Scheme data: TTL 24 hours
- Embeddings: TTL 7 days
- Session data: TTL 30 minutes
- LRU eviction policy


## 4. Data Flow

### 4.1 Text Query Flow

```
1. User enters text query in Hindi: "मेरी आय 2 लाख है और मैं महाराष्ट्र का किसान हूं"

2. API Gateway receives request, validates, and routes to Language Processor

3. Language Processor:
   - Detects language: Hindi (hi)
   - Normalizes text: "मेरी आय 200000 है और मैं महाराष्ट्र का किसान हूं"
   - Returns to API Gateway

4. NLP Service receives normalized text:
   - Extracts entities: {income: 200000, occupation: "farmer", location: "Maharashtra"}
   - Classifies intent: "eligibility_check"
   - Returns structured data

5. Matching Engine receives user profile:
   - Queries database for schemes matching location
   - Computes semantic similarity for each scheme
   - Ranks schemes by relevance score
   - Returns top 10 schemes with eligibility status

6. Response Generator receives matched schemes:
   - Formats response in structured template
   - Translates to Hindi
   - Simplifies technical terms
   - Generates explanations
   - Returns formatted response

7. API Gateway returns response to frontend

8. Frontend displays schemes with visual indicators
```

### 4.2 Voice Query Flow

```
1. User speaks query in Marathi

2. Frontend captures audio, encodes as base64, sends to API Gateway

3. API Gateway routes to Language Processor

4. Language Processor:
   - Calls Speech-to-Text API with Marathi language hint
   - Receives transcribed text
   - Normalizes text
   - Returns to API Gateway

5. Flow continues same as text query from step 4 onwards
```


### 4.3 Scheme Update Flow

```
1. Admin uploads scheme data CSV/JSON via admin portal

2. Data Validation Service:
   - Validates schema compliance
   - Checks for duplicate schemes
   - Validates eligibility criteria format

3. Embedding Generation Service:
   - Generates embeddings for new/updated schemes
   - Stores embeddings in database

4. Database Update:
   - Inserts/updates scheme records
   - Updates metadata (last_updated timestamp)

5. Cache Invalidation:
   - Clears relevant cache entries
   - Pre-warms cache with popular schemes

6. Admin receives confirmation
```

## 5. API Design

### 5.1 Query Submission API

**Endpoint**: `POST /api/v1/query`

**Request**:
```json
{
  "query": "मेरी आय 2 लाख है और मैं किसान हूं",
  "language": "hi",
  "mode": "text",
  "user_preferences": {
    "max_results": 10,
    "include_state_schemes": true
  }
}
```

**Response**:
```json
{
  "status": "success",
  "request_id": "req_123456",
  "extracted_profile": {
    "income": 200000,
    "occupation": "farmer",
    "location": null,
    "category": null
  },
  "missing_fields": ["location", "category"],
  "clarification_needed": true,
  "clarification_question": "कृपया अपना राज्य बताएं",
  "schemes": [],
  "processing_time_ms": 450
}
```


**Response with Schemes**:
```json
{
  "status": "success",
  "request_id": "req_123457",
  "extracted_profile": {
    "income": 200000,
    "occupation": "farmer",
    "location": "Maharashtra",
    "category": "General"
  },
  "schemes": [
    {
      "scheme_id": "PM-KISAN-001",
      "scheme_name": "प्रधानमंत्री किसान सम्मान निधि",
      "scheme_name_en": "PM-KISAN",
      "eligibility_status": "Eligible",
      "relevance_score": 0.92,
      "benefits": "₹6,000 प्रति वर्ष तीन किस्तों में",
      "documents_required": [
        "आधार कार्ड",
        "बैंक खाता पासबुक",
        "भूमि स्वामित्व दस्तावेज"
      ],
      "application_link": "https://pmkisan.gov.in",
      "application_steps": [
        "PM-KISAN वेबसाइट पर जाएं",
        "'नया किसान पंजीकरण' पर क्लिक करें",
        "आधार नंबर दर्ज करें",
        "फॉर्म भरें और दस्तावेज अपलोड करें"
      ],
      "explanation": "आप इस योजना के लिए पात्र हैं क्योंकि आप 5 एकड़ से कम भूमि वाले किसान हैं।",
      "category": "agriculture",
      "administering_body": "कृषि और किसान कल्याण मंत्रालय"
    }
  ],
  "total_schemes_found": 3,
  "processing_time_ms": 680
}
```

### 5.2 Voice Input API

**Endpoint**: `POST /api/v1/voice`

**Request**:
```json
{
  "audio_data": "base64_encoded_audio_data",
  "audio_format": "webm",
  "language_hint": "hi",
  "user_preferences": {
    "max_results": 10
  }
}
```

**Response**: Same as Query Submission API


### 5.3 Scheme Details API

**Endpoint**: `GET /api/v1/schemes/{scheme_id}`

**Query Parameters**:
- `language`: Preferred language code (default: en)

**Response**:
```json
{
  "scheme_id": "PM-KISAN-001",
  "scheme_name": "प्रधानमंत्री किसान सम्मान निधि",
  "description": "यह योजना छोटे और सीमांत किसानों को वित्तीय सहायता प्रदान करती है।",
  "eligibility_criteria": {
    "occupation": ["farmer"],
    "landholding_max": 5.0,
    "income_max": null,
    "location": ["All India"],
    "category": ["All"]
  },
  "benefits": "₹6,000 प्रति वर्ष",
  "documents_required": ["आधार कार्ड", "बैंक पासबुक", "भूमि दस्तावेज"],
  "application_process": "...",
  "official_website": "https://pmkisan.gov.in",
  "last_updated": "2026-01-15T10:30:00Z"
}
```

### 5.4 Feedback API

**Endpoint**: `POST /api/v1/feedback`

**Request**:
```json
{
  "request_id": "req_123457",
  "scheme_id": "PM-KISAN-001",
  "feedback_type": "accuracy",
  "rating": 5,
  "comment": "Very helpful, found the right scheme",
  "was_eligible": true
}
```

**Response**:
```json
{
  "status": "success",
  "message": "Thank you for your feedback"
}
```

### 5.5 Supported Languages API

**Endpoint**: `GET /api/v1/languages`

**Response**:
```json
{
  "languages": [
    {"code": "en", "name": "English", "native_name": "English"},
    {"code": "hi", "name": "Hindi", "native_name": "हिन्दी"},
    {"code": "mr", "name": "Marathi", "native_name": "मराठी"},
    {"code": "ta", "name": "Tamil", "native_name": "தமிழ்"},
    {"code": "te", "name": "Telugu", "native_name": "తెలుగు"}
  ]
}
```


## 6. Database Schema

### 6.1 Schemes Table

```sql
CREATE TABLE schemes (
    scheme_id VARCHAR(50) PRIMARY KEY,
    scheme_name_en VARCHAR(255) NOT NULL,
    scheme_name_hi TEXT,
    scheme_name_mr TEXT,
    scheme_name_ta TEXT,
    scheme_name_te TEXT,
    description_en TEXT NOT NULL,
    description_hi TEXT,
    description_mr TEXT,
    description_ta TEXT,
    description_te TEXT,
    category VARCHAR(50) NOT NULL,  -- agriculture, health, education, housing, etc.
    scheme_type VARCHAR(20) NOT NULL,  -- central, state
    state VARCHAR(50),  -- NULL for central schemes
    administering_body VARCHAR(255),
    benefits_en TEXT NOT NULL,
    benefits_hi TEXT,
    official_website VARCHAR(500),
    application_link VARCHAR(500),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    embedding vector(768)  -- pgvector extension for semantic search
);

CREATE INDEX idx_schemes_category ON schemes(category);
CREATE INDEX idx_schemes_state ON schemes(state);
CREATE INDEX idx_schemes_active ON schemes(is_active);
CREATE INDEX idx_schemes_embedding ON schemes USING ivfflat (embedding vector_cosine_ops);
```

### 6.2 Eligibility Criteria Table

```sql
CREATE TABLE eligibility_criteria (
    criteria_id SERIAL PRIMARY KEY,
    scheme_id VARCHAR(50) REFERENCES schemes(scheme_id) ON DELETE CASCADE,
    criteria_type VARCHAR(50) NOT NULL,  -- income, age, occupation, location, category, etc.
    operator VARCHAR(20),  -- <=, >=, =, IN, NOT IN
    value_numeric DECIMAL(15,2),
    value_text TEXT,
    value_array TEXT[],
    is_mandatory BOOLEAN DEFAULT TRUE,
    criteria_text_en TEXT,
    criteria_text_hi TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_eligibility_scheme ON eligibility_criteria(scheme_id);
CREATE INDEX idx_eligibility_type ON eligibility_criteria(criteria_type);
```


### 6.3 Documents Table

```sql
CREATE TABLE required_documents (
    document_id SERIAL PRIMARY KEY,
    scheme_id VARCHAR(50) REFERENCES schemes(scheme_id) ON DELETE CASCADE,
    document_name_en VARCHAR(255) NOT NULL,
    document_name_hi TEXT,
    document_name_mr TEXT,
    document_name_ta TEXT,
    document_name_te TEXT,
    is_mandatory BOOLEAN DEFAULT TRUE,
    document_type VARCHAR(50),  -- identity, income, residence, etc.
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_documents_scheme ON required_documents(scheme_id);
```

### 6.4 Application Steps Table

```sql
CREATE TABLE application_steps (
    step_id SERIAL PRIMARY KEY,
    scheme_id VARCHAR(50) REFERENCES schemes(scheme_id) ON DELETE CASCADE,
    step_number INT NOT NULL,
    step_description_en TEXT NOT NULL,
    step_description_hi TEXT,
    step_description_mr TEXT,
    step_description_ta TEXT,
    step_description_te TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_steps_scheme ON application_steps(scheme_id);
```

### 6.5 Feedback Table

```sql
CREATE TABLE feedback (
    feedback_id SERIAL PRIMARY KEY,
    request_id VARCHAR(100),
    scheme_id VARCHAR(50) REFERENCES schemes(scheme_id),
    feedback_type VARCHAR(50),  -- accuracy, usability, helpfulness
    rating INT CHECK (rating >= 1 AND rating <= 5),
    comment TEXT,
    was_eligible BOOLEAN,
    user_language VARCHAR(10),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_feedback_scheme ON feedback(scheme_id);
CREATE INDEX idx_feedback_created ON feedback(created_at);
```


### 6.6 Analytics Table (Optional)

```sql
CREATE TABLE query_analytics (
    analytics_id SERIAL PRIMARY KEY,
    request_id VARCHAR(100) UNIQUE,
    user_language VARCHAR(10),
    input_mode VARCHAR(20),  -- text, voice
    extracted_entities JSONB,
    schemes_returned INT,
    processing_time_ms INT,
    user_state VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_analytics_created ON query_analytics(created_at);
CREATE INDEX idx_analytics_language ON query_analytics(user_language);
```

### 6.7 Sample Data

```sql
-- Sample Scheme
INSERT INTO schemes (
    scheme_id, scheme_name_en, scheme_name_hi, description_en, 
    category, scheme_type, administering_body, benefits_en, 
    official_website, application_link
) VALUES (
    'PM-KISAN-001',
    'Pradhan Mantri Kisan Samman Nidhi',
    'प्रधानमंत्री किसान सम्मान निधि',
    'Income support to all landholding farmers families',
    'agriculture',
    'central',
    'Ministry of Agriculture and Farmers Welfare',
    '₹6,000 per year in three equal installments',
    'https://pmkisan.gov.in',
    'https://pmkisan.gov.in/RegistrationForm.aspx'
);

-- Sample Eligibility Criteria
INSERT INTO eligibility_criteria (
    scheme_id, criteria_type, operator, value_numeric, is_mandatory
) VALUES 
    ('PM-KISAN-001', 'landholding', '<=', 5.0, TRUE),
    ('PM-KISAN-001', 'occupation', '=', NULL, TRUE);

UPDATE eligibility_criteria 
SET value_array = ARRAY['farmer', 'agricultural_worker']
WHERE scheme_id = 'PM-KISAN-001' AND criteria_type = 'occupation';
```


## 7. AI Model Description

### 7.1 Language Detection Model

**Model**: `fasttext` or `langdetect`

**Purpose**: Identify input language from text

**Training Data**: Not required (pre-trained)

**Performance**: 
- Accuracy: >95% for supported languages
- Inference time: <10ms

### 7.2 Speech-to-Text Model

**Model**: Google Cloud Speech-to-Text API or OpenAI Whisper

**Purpose**: Convert voice input to text

**Supported Languages**: Hindi, English, Marathi, Tamil, Telugu

**Configuration**:
- Sample rate: 16kHz
- Encoding: LINEAR16 or WEBM_OPUS
- Language hints enabled for better accuracy

**Performance**:
- Word Error Rate (WER): <15% for clear audio
- Inference time: <2 seconds for 30-second audio

### 7.3 Named Entity Recognition (NER) Model

**Base Model**: `ai4bharat/indic-bert` or `xlm-roberta-base`

**Fine-tuning Dataset**: 
- 10,000+ annotated queries in Hindi, English, Marathi
- Entity types: INCOME, OCCUPATION, LOCATION, CATEGORY, AGE, GENDER, LANDHOLDING

**Architecture**:
- Token classification with CRF layer
- Multi-task learning for entity extraction and intent classification

**Training Configuration**:
```python
{
  "model": "ai4bharat/indic-bert",
  "max_length": 128,
  "batch_size": 16,
  "learning_rate": 2e-5,
  "epochs": 10,
  "optimizer": "AdamW",
  "loss": "CrossEntropyLoss"
}
```

**Performance Targets**:
- Entity extraction F1 score: >85%
- Intent classification accuracy: >90%
- Inference time: <200ms per query


### 7.4 Semantic Similarity Model

**Base Model**: `sentence-transformers/paraphrase-multilingual-mpnet-base-v2`

**Purpose**: Generate embeddings for user profiles and scheme criteria

**Fine-tuning Strategy**:
- Contrastive learning on (user_profile, eligible_scheme) pairs
- Triplet loss: anchor (user), positive (eligible scheme), negative (ineligible scheme)

**Training Data**:
- 5,000+ synthetic user profiles
- 500+ real schemes with eligibility criteria
- Positive and negative pairs generated programmatically

**Embedding Dimension**: 768

**Similarity Metric**: Cosine similarity

**Performance Targets**:
- Precision@5: >80% (top 5 schemes include eligible ones)
- Recall@10: >90% (top 10 schemes include most eligible ones)
- Inference time: <100ms for embedding generation + similarity computation

### 7.5 Translation Model

**Model**: Google Translate API or IndicTrans2

**Purpose**: Translate responses to user's preferred language

**Supported Language Pairs**:
- English ↔ Hindi
- English ↔ Marathi
- English ↔ Tamil
- English ↔ Telugu

**Performance**:
- BLEU score: >40 for technical content
- Inference time: <500ms per response

### 7.6 Model Deployment

**Serving Infrastructure**: 
- TorchServe or TensorFlow Serving for custom models
- API calls for third-party models (Google, OpenAI)

**Model Versioning**: 
- MLflow for experiment tracking and model registry
- A/B testing for model updates

**Monitoring**:
- Latency tracking per model
- Accuracy metrics from user feedback
- Model drift detection


## 8. Deployment Architecture

### 8.1 Infrastructure Overview

**Cloud Provider**: AWS / Google Cloud Platform / Azure

**Deployment Strategy**: Containerized microservices with Kubernetes orchestration

**Architecture Components**:

```
┌─────────────────────────────────────────────────────────────┐
│                    CLOUDFLARE CDN                            │
│              (DDoS Protection, SSL, Caching)                 │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  AWS/GCP LOAD BALANCER                       │
│            (Application Load Balancer / ALB)                 │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              KUBERNETES CLUSTER (EKS/GKE)                    │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ API Gateway  │  │ Language     │  │ NLP Service  │     │
│  │ (3 replicas) │  │ Processor    │  │ (5 replicas) │     │
│  │              │  │ (2 replicas) │  │              │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ Matching     │  │ Response     │  │ Frontend     │     │
│  │ Engine       │  │ Generator    │  │ (Static)     │     │
│  │ (3 replicas) │  │ (2 replicas) │  │              │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
└────────────────────────┬────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ PostgreSQL   │  │ Redis Cache  │  │ S3 / Cloud   │
│ (RDS/Cloud   │  │ (ElastiCache)│  │ Storage      │
│ SQL)         │  │              │  │ (Models,     │
│              │  │              │  │ Logs)        │
└──────────────┘  └──────────────┘  └──────────────┘
```


### 8.2 Container Configuration

**Base Images**:
- Frontend: `node:18-alpine`
- Backend Services: `python:3.11-slim`
- API Gateway: `kong:3.4` or `node:18-alpine`

**Resource Allocation**:

| Service | CPU | Memory | Replicas | Auto-scale |
|---------|-----|--------|----------|------------|
| API Gateway | 0.5 | 512MB | 3 | 3-10 |
| Language Processor | 1.0 | 1GB | 2 | 2-5 |
| NLP Service | 2.0 | 4GB | 5 | 5-15 |
| Matching Engine | 2.0 | 4GB | 3 | 3-10 |
| Response Generator | 1.0 | 2GB | 2 | 2-5 |

**Auto-scaling Triggers**:
- CPU utilization > 70%
- Memory utilization > 80%
- Request queue depth > 100

### 8.3 Database Configuration

**PostgreSQL**:
- Instance type: db.t3.large (2 vCPU, 8GB RAM) for development
- Instance type: db.r6g.xlarge (4 vCPU, 32GB RAM) for production
- Storage: 100GB SSD with auto-scaling
- Backup: Daily automated backups with 7-day retention
- Read replicas: 2 for load distribution

**Redis**:
- Instance type: cache.t3.medium (2 vCPU, 3.09GB RAM)
- Cluster mode enabled for high availability
- Eviction policy: allkeys-lru

### 8.4 Monitoring & Logging

**Monitoring Stack**:
- Prometheus for metrics collection
- Grafana for visualization
- AlertManager for alerting

**Key Metrics**:
- Request rate (requests/second)
- Response time (p50, p95, p99)
- Error rate (4xx, 5xx)
- Model inference latency
- Database query performance
- Cache hit rate

**Logging**:
- Centralized logging with ELK stack (Elasticsearch, Logstash, Kibana)
- Structured JSON logs
- Log retention: 30 days

**Alerts**:
- Response time > 5 seconds
- Error rate > 5%
- Service downtime
- Database connection pool exhaustion


### 8.5 CI/CD Pipeline

**Version Control**: GitHub / GitLab

**Pipeline Stages**:

1. **Code Commit** → Trigger pipeline
2. **Lint & Format** → ESLint, Black, Prettier
3. **Unit Tests** → Jest, Pytest (coverage > 80%)
4. **Build Docker Images** → Multi-stage builds
5. **Security Scan** → Trivy, Snyk
6. **Push to Registry** → Docker Hub / ECR
7. **Deploy to Staging** → Kubernetes staging namespace
8. **Integration Tests** → API tests, E2E tests
9. **Deploy to Production** → Blue-green deployment
10. **Smoke Tests** → Health checks, basic functionality

**Deployment Strategy**: 
- Blue-green deployment for zero downtime
- Canary releases for gradual rollout (10% → 50% → 100%)
- Automatic rollback on error rate spike

### 8.6 Disaster Recovery

**Backup Strategy**:
- Database: Daily full backup + continuous WAL archiving
- Configuration: Version controlled in Git
- Models: Stored in S3 with versioning enabled

**Recovery Time Objective (RTO)**: 1 hour

**Recovery Point Objective (RPO)**: 15 minutes

**DR Plan**:
1. Maintain hot standby in different availability zone
2. Automated failover for database
3. Multi-region deployment for critical services (future)


## 9. Security & Responsible AI Design

### 9.1 Security Measures

**Network Security**:
- All traffic encrypted with TLS 1.3
- API Gateway with rate limiting (100 requests/minute per IP)
- DDoS protection via CloudFlare
- Web Application Firewall (WAF) rules
- Private subnets for backend services
- Security groups restricting inter-service communication

**Data Security**:
- No persistent storage of user personal data
- Session data encrypted in Redis with TTL
- Database encryption at rest (AES-256)
- Encrypted backups
- Secrets management via AWS Secrets Manager / HashiCorp Vault

**Application Security**:
- Input validation and sanitization
- SQL injection prevention (parameterized queries)
- XSS protection (Content Security Policy headers)
- CORS configuration for allowed origins
- API authentication for admin endpoints (JWT)
- Regular security audits and penetration testing

**Compliance**:
- GDPR-inspired data handling practices
- Indian IT Act compliance
- No collection of sensitive personal data without explicit need
- Data retention policy: Session data only (30 minutes)

### 9.2 Responsible AI Principles

**Transparency**:
- Clear explanations for eligibility decisions
- Confidence scores displayed to users
- Disclaimer that AI provides guidance, not final decisions
- Open about system limitations

**Fairness**:
- No discrimination based on protected attributes
- Regular bias audits on model predictions
- Diverse training data across demographics
- Equal access across languages and regions


**Privacy**:
- Stateless architecture (no user profiles)
- Minimal data collection (only what's needed for matching)
- No tracking or profiling of users
- Anonymous analytics (aggregated only)
- User control over data sharing

**Accountability**:
- Logging of system decisions for audit
- Feedback mechanism for incorrect recommendations
- Human oversight for model updates
- Clear ownership and contact information

**Safety**:
- Fallback to rule-based matching if AI fails
- Confidence thresholds to prevent false positives
- Regular model validation against ground truth
- Monitoring for model drift and degradation

### 9.3 Bias Mitigation

**Training Data Diversity**:
- Balanced representation across states, languages, categories
- Synthetic data generation for underrepresented groups
- Regular audits of training data distribution

**Model Evaluation**:
- Disaggregated evaluation by demographic groups
- Fairness metrics (demographic parity, equal opportunity)
- Regular testing with diverse user personas

**Mitigation Strategies**:
- Debiasing techniques during training
- Post-processing adjustments for fairness
- Human review of edge cases
- Continuous monitoring of prediction patterns

### 9.4 Explainability

**Explanation Types**:
- **Feature-based**: "You qualify because income < ₹2 lakh and occupation = farmer"
- **Similarity-based**: "This scheme matches your profile with 92% confidence"
- **Contrastive**: "You don't qualify because landholding > 5 acres"

**Implementation**:
- Rule-based explanations for hard filters
- Attention weights for semantic matching
- Natural language generation for user-friendly explanations


## 10. Scalability Considerations

### 10.1 Horizontal Scaling

**Stateless Services**:
- All backend services designed to be stateless
- No session affinity required
- Easy to add/remove replicas based on load

**Load Balancing**:
- Round-robin distribution across service replicas
- Health checks to remove unhealthy instances
- Connection draining during scale-down

**Auto-scaling Configuration**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nlp-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nlp-service
  minReplicas: 5
  maxReplicas: 15
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 10.2 Database Scaling

**Read Scaling**:
- PostgreSQL read replicas for query distribution
- Read-heavy queries routed to replicas
- Write queries to primary instance

**Caching Strategy**:
- Redis for frequently accessed schemes
- Embedding cache to avoid recomputation
- Cache warming for popular schemes
- TTL-based invalidation

**Query Optimization**:
- Indexed columns for fast lookups
- Vector index (IVFFlat) for similarity search
- Query result pagination
- Connection pooling (PgBouncer)


### 10.3 Model Serving Optimization

**Batch Inference**:
- Batch multiple requests for efficient GPU utilization
- Dynamic batching with max wait time of 50ms

**Model Optimization**:
- ONNX conversion for faster inference
- Quantization (INT8) for reduced memory footprint
- Model distillation for smaller, faster models

**Caching**:
- Cache embeddings for schemes (rarely change)
- Cache language detection results for common phrases

### 10.4 Content Delivery

**Static Assets**:
- Frontend served via CDN (CloudFlare)
- Aggressive caching for JS/CSS bundles
- Image optimization and lazy loading

**API Response Caching**:
- Cache scheme details by ID (24-hour TTL)
- Cache language list (7-day TTL)
- No caching for user queries (privacy)

### 10.5 Geographic Distribution

**Current**: Single region deployment

**Future** (Phase 2):
- Multi-region deployment for lower latency
- Regional databases with scheme data replication
- GeoDNS routing to nearest region
- Edge computing for language detection and STT

### 10.6 Performance Targets

| Metric | Target | Current Capacity |
|--------|--------|------------------|
| Concurrent Users | 1,000 | 5,000 (with auto-scaling) |
| Requests/Second | 100 | 500 (with auto-scaling) |
| Response Time (p95) | <3s | <2s |
| Database Queries/Second | 500 | 2,000 (with replicas) |
| Cache Hit Rate | >80% | 85% |
| Uptime | 99% | 99.5% |


### 10.7 Cost Optimization

**Compute**:
- Spot instances for non-critical workloads
- Auto-scaling to match demand
- Right-sizing based on actual usage

**Storage**:
- S3 lifecycle policies for old logs
- Compression for database backups
- Tiered storage (hot/cold)

**Network**:
- CDN to reduce origin traffic
- Data transfer optimization
- Regional deployment to minimize cross-region costs

**AI/ML**:
- Self-hosted models vs. API costs analysis
- Batch processing for non-real-time tasks
- Model caching to reduce API calls

## 11. Technology Stack Summary

### Frontend
- **Framework**: React.js 18
- **Mobile**: React Native or PWA
- **State Management**: Redux Toolkit
- **UI Library**: Material-UI / Chakra UI
- **Build Tool**: Vite
- **Testing**: Jest, React Testing Library

### Backend
- **Language**: Python 3.11
- **Framework**: FastAPI
- **API Gateway**: Kong / Express.js
- **Task Queue**: Celery (for async tasks)
- **Message Broker**: RabbitMQ / Redis

### AI/ML
- **NLP**: spaCy, Hugging Face Transformers
- **Embeddings**: Sentence Transformers
- **Model Serving**: TorchServe / TensorFlow Serving
- **ML Ops**: MLflow, Weights & Biases

### Data
- **Database**: PostgreSQL 15 with pgvector
- **Cache**: Redis 7
- **Search**: PostgreSQL full-text search
- **Storage**: AWS S3 / Google Cloud Storage

### Infrastructure
- **Container**: Docker
- **Orchestration**: Kubernetes (EKS/GKE)
- **CI/CD**: GitHub Actions / GitLab CI
- **Monitoring**: Prometheus, Grafana
- **Logging**: ELK Stack
- **CDN**: CloudFlare


## 12. Implementation Phases

### Phase 1: MVP (Months 1-3)
- Basic web interface with text input
- Support for English and Hindi
- NER model for entity extraction
- Rule-based eligibility matching
- 50 central government schemes
- PostgreSQL database
- Single-region deployment

### Phase 2: AI Enhancement (Months 4-6)
- Voice input support
- Semantic similarity matching
- Add Marathi, Tamil, Telugu support
- 200+ schemes (central + select states)
- Redis caching
- Improved UI/UX for low literacy users

### Phase 3: Scale & Optimize (Months 7-9)
- Mobile app (React Native)
- Auto-scaling infrastructure
- Model optimization (ONNX, quantization)
- 500+ schemes (all states)
- Advanced analytics and monitoring
- A/B testing framework

### Phase 4: Advanced Features (Months 10-12)
- Multi-turn conversations
- Personalized recommendations
- Document verification guidance
- Integration with government portals
- Multi-region deployment
- Advanced explainability features

## 13. Risks & Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Inaccurate NER extraction | High | Medium | Extensive testing, user feedback loop, confidence thresholds |
| Scheme data outdated | High | High | Automated data refresh pipeline, partnerships with govt |
| Low adoption in rural areas | High | Medium | Offline mode, SMS integration, partnerships with NGOs |
| Model bias | Medium | Medium | Diverse training data, regular bias audits, human oversight |
| High infrastructure costs | Medium | Low | Cost optimization, efficient caching, right-sizing |
| Privacy concerns | High | Low | Stateless architecture, clear disclaimers, compliance |
| API rate limits (3rd party) | Medium | Medium | Self-hosted alternatives, request batching, caching |


## 14. Testing Strategy

### 14.1 Unit Testing
- Test coverage target: >80%
- Framework: Pytest (backend), Jest (frontend)
- Mock external dependencies (APIs, database)
- Test entity extraction accuracy
- Test eligibility matching logic

### 14.2 Integration Testing
- API endpoint testing
- Database integration tests
- Service-to-service communication tests
- End-to-end query flow tests

### 14.3 Model Testing
- Accuracy metrics on held-out test set
- Bias evaluation across demographics
- Adversarial testing with edge cases
- Performance benchmarking (latency, throughput)

### 14.4 User Acceptance Testing
- Beta testing with 100+ real users
- Usability testing with low literacy users
- Multilingual testing by native speakers
- Feedback collection and iteration

### 14.5 Load Testing
- Simulate 1,000 concurrent users
- Stress testing to find breaking points
- Sustained load testing (24 hours)
- Spike testing for sudden traffic increases

### 14.6 Security Testing
- Penetration testing
- Vulnerability scanning
- SQL injection testing
- XSS and CSRF testing
- Rate limiting validation

## 15. Success Criteria

### Technical Success
- ✅ Response time <3 seconds (p95)
- ✅ NER accuracy >85%
- ✅ Matching precision >80%
- ✅ System uptime >99%
- ✅ Support 5+ languages
- ✅ Handle 1,000 concurrent users

### User Success
- ✅ 100,000 monthly active users within 6 months
- ✅ User satisfaction rating >4/5
- ✅ 30% of users return within 30 days
- ✅ 20% proceed to official application portals

### Business Success
- ✅ Increase scheme awareness by 50%
- ✅ Reduce intermediary dependency by 30%
- ✅ Coverage of 500+ schemes
- ✅ Reach users in 20+ states

---

**Document Version**: 1.0  
**Last Updated**: February 15, 2026  
**Status**: Draft  
**Authors**: BharatScheme AI Team
