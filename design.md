# Design Document: Arivu Care Platform

## Overview

Arivu Care is a comprehensive medical AI platform built with a modern tech stack leveraging 100% free-tier services. The platform provides AI-powered medical consultations, symptom diagnosis, image analysis, drug discovery, mental health assessments, and educational tools for students. The architecture follows a client-server model with Next.js 15 frontend and FastAPI backend, integrated with multiple AI services (Groq, Hugging Face), vector database (Qdrant), and Supabase for authentication, database, and storage.

### Key Design Principles

1. **Free-Tier First**: All services must operate within free-tier limits
2. **Performance Optimized**: Target < 1.5s initial load, 60fps interactions
3. **Medical Focus**: Content filtering ensures only medical queries are processed
4. **Role-Based Access**: Four user roles with distinct feature access
5. **Cross-Browser Support**: Works on Chrome, Firefox, Safari, and Edge
6. **Graceful Degradation**: Fallback mechanisms for all critical services
7. **HIPAA-Aware**: Security and privacy best practices for medical data

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Frontend (Next.js 15)                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Chat   │  │Diagnosis │  │  Image   │  │ Student  │   │
│  │Interface │  │  Engine  │  │ Analysis │  │  Tools   │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Drug   │  │  Mental  │  │   Data   │  │   Auth   │   │
│  │Discovery │  │  Health  │  │Generator │  │   UI     │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ HTTPS/REST API
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Backend (FastAPI)                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Chat   │  │Diagnosis │  │  Image   │  │ Content  │   │
│  │ Service  │  │ Service  │  │ Service  │  │  Filter  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Drug   │  │  Mental  │  │   Data   │  │   RAG    │   │
│  │ Service  │  │  Health  │  │ Service  │  │  Engine  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Groq API   │    │  Hugging     │    │   Qdrant     │
│ llama-3.3-   │    │   Face       │    │   Cloud      │
│ 70b-versatile│    │  MedGemma    │    │  (Vectors)   │
└──────────────┘    └──────────────┘    └──────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                            ▼
                    ┌──────────────┐
                    │   Supabase   │
                    │ Auth/DB/     │
                    │  Storage     │
                    └──────────────┘
```

### Component Layers

1. **Presentation Layer** (Next.js 15 + React 19)
   - Server-side rendering for initial page load
   - Client-side routing for instant navigation
   - shadcn/ui components with TailwindCSS 4
   - Responsive mobile-first design

2. **API Layer** (FastAPI)
   - RESTful endpoints for all features
   - Request validation with Pydantic
   - Rate limiting and quota management
   - Error handling and logging

3. **Service Layer**
   - Business logic for each feature
   - AI model integration
   - Content filtering
   - Role-based access control

4. **Data Layer**
   - Supabase PostgreSQL for structured data
   - Qdrant Cloud for vector embeddings
   - Supabase Storage for files/images
   - LlamaIndex for RAG operations

5. **External Services**
   - Groq API for LLM and Whisper
   - Hugging Face for MedGemma
   - Pollinations.ai for image generation
   - Web search API for latest data

## Components and Interfaces

### 1. Authentication System

**Purpose**: Manage user authentication, authorization, and session handling

**Components**:
- `AuthProvider`: React context for auth state
- `LoginPage`: Google OAuth login interface
- `RoleSelector`: User role selection component
- `ProtectedRoute`: Route guard for authenticated pages

**Interfaces**:
```typescript
interface User {
  id: string;
  email: string;
  name: string;
  role: 'patient' | 'researcher' | 'student' | 'doctor';
  avatar_url?: string;
  created_at: Date;
  last_login: Date;
}

interface Session {
  user: User;
  access_token: string;
  refresh_token: string;
  expires_at: Date;
}

interface AuthService {
  signInWithGoogle(): Promise<Session>;
  signOut(): Promise<void>;
  getSession(): Promise<Session | null>;
  refreshSession(): Promise<Session>;
}
```

### 2. Medical AI Chatbot

**Purpose**: Provide conversational AI interface with RAG, voice, and web search

**Components**:
- `ChatInterface`: Main chat UI with message list
- `MessageInput`: Text input with voice button
- `VoiceRecorder`: Audio capture and Groq Whisper integration
- `VoiceSynthesizer`: Browser TTS for responses
- `WebSearchIndicator`: Shows when web search is active
- `DisclaimerBanner`: Medical disclaimer display

**Interfaces**:
```typescript
interface Message {
  id: string;
  conversation_id: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  timestamp: Date;
  metadata?: {
    rag_sources?: string[];
    web_search_used?: boolean;
    confidence_score?: number;
  };
}

interface Conversation {
  id: string;
  user_id: string;
  title: string;
  messages: Message[];
  created_at: Date;
  updated_at: Date;
}

interface ChatService {
  sendMessage(conversationId: string, content: string): Promise<Message>;
  getConversations(userId: string): Promise<Conversation[]>;
  createConversation(userId: string): Promise<Conversation>;
  transcribeAudio(audioBlob: Blob): Promise<string>;
  synthesizeSpeech(text: string): Promise<void>;
}
```

### 3. Content Filter

**Purpose**: Ensure queries are medical-related only

**Components**:
- `ContentAnalyzer`: Keyword and AI-based classification
- `FilterWarning`: UI component for non-medical query warnings

**Interfaces**:
```typescript
interface FilterResult {
  is_medical: boolean;
  confidence: number;
  detected_topics: string[];
  suggestion?: string;
}

interface ContentFilterService {
  analyzeQuery(query: string): Promise<FilterResult>;
  getMedicalKeywords(): string[];
  getNonMedicalKeywords(): string[];
}
```

### 4. RAG System

**Purpose**: Retrieve relevant medical context from vector database

**Components**:
- `EmbeddingGenerator`: Generate embeddings using gemini-embedding-001
- `VectorStore`: Qdrant Cloud integration
- `DocumentRetriever`: LlamaIndex query engine
- `ContextBuilder`: Combine retrieved docs with query

**Interfaces**:
```typescript
interface Document {
  id: string;
  content: string;
  metadata: {
    source: string;
    title: string;
    page?: number;
  };
  embedding?: number[];
}

interface RAGService {
  indexDocuments(documents: Document[]): Promise<void>;
  query(query: string, topK: number): Promise<Document[]>;
  generateEmbedding(text: string): Promise<number[]>;
}
```

### 5. Symptom Diagnosis Engine

**Purpose**: Multi-step symptom collection and disease prediction

**Components**:
- `SymptomWizard`: Step-by-step symptom collection UI
- `FollowUpQuestions`: AI-generated clarifying questions
- `DiagnosisReport`: Display predictions with confidence scores
- `TreatmentPlan`: Show recommended treatments

**Interfaces**:
```typescript
interface SymptomData {
  initial_symptoms: string[];
  follow_up_qa: Array<{
    question: string;
    answer: string;
  }>;
  duration: string;
  severity: number; // 1-10
}

interface DiseasePrediction {
  disease: string;
  confidence: number; // 0-1
  symptoms_match: string[];
  details: string;
  next_steps: string[];
}

interface DiagnosisResult {
  predictions: DiseasePrediction[];
  treatment_plan?: TreatmentPlan;
  disclaimer: string;
}

interface DiagnosisService {
  generateFollowUpQuestions(symptoms: string[]): Promise<string[]>;
  predictDisease(symptomData: SymptomData): Promise<DiagnosisResult>;
  generateTreatmentPlan(disease: string, symptomData: SymptomData): Promise<TreatmentPlan>;
}
```

### 6. Medical Image Analyzer

**Purpose**: Analyze medical images using Hugging Face MedGemma

**Components**:
- `ImageUploader`: Drag-and-drop image upload
- `ImagePreview`: Display uploaded image
- `AnalysisReport`: Show findings with annotations
- `OCRExtractor`: Extract text from medical reports

**Interfaces**:
```typescript
interface ImageAnalysisRequest {
  image_url: string;
  image_type: 'xray' | 'skin' | 'report' | 'other';
  user_notes?: string;
}

interface ImageAnalysisResult {
  findings: string[];
  confidence_scores: Record<string, number>;
  annotations?: Array<{
    region: { x: number; y: number; width: number; height: number };
    label: string;
  }>;
  ocr_text?: string;
  disclaimer: string;
}

interface ImageAnalysisService {
  analyzeImage(request: ImageAnalysisRequest): Promise<ImageAnalysisResult>;
  extractTextFromImage(imageUrl: string): Promise<string>;
  uploadImage(file: File): Promise<string>; // Returns Supabase URL
}
```

### 7. Drug Discovery Tool

**Purpose**: Analyze SMILES formulas and recommend drug candidates

**Components**:
- `SMILESInput`: Input field with validation
- `MolecularVisualization`: 2D/3D molecule rendering
- `DrugProperties`: Display predicted properties
- `DrugRecommendations`: Show candidate drugs

**Interfaces**:
```typescript
interface SMILESAnalysis {
  smiles: string;
  molecular_formula: string;
  molecular_weight: number;
  iupac_name: string;
  properties: {
    logP: number;
    h_bond_donors: number;
    h_bond_acceptors: number;
    solubility: string;
  };
  toxicity_prediction: {
    mutagenicity: 'low' | 'moderate' | 'high';
    carcinogenicity: 'low' | 'moderate' | 'high';
  };
  drug_candidates?: DrugCandidate[];
}

interface DrugCandidate {
  smiles: string;
  predicted_activity: string;
  confidence: number;
}

interface DrugDiscoveryService {
  analyzeSMILES(smiles: string): Promise<SMILESAnalysis>;
  recommendDrugsForDisease(disease: string): Promise<DrugCandidate[]>;
}
```

### 8. Mental Health Module

**Purpose**: Mood assessments and wellness recommendations

**Components**:
- `AssessmentSelector`: Choose questionnaire type
- `QuestionnaireForm`: Dynamic form for questions
- `StressScoreDisplay`: Visualize stress levels
- `WellnessRecommendations`: Personalized suggestions
- `ProgressTracker`: Track assessments over time

**Interfaces**:
```typescript
interface Questionnaire {
  id: string;
  type: 'student' | 'parent' | 'corporate' | 'partner';
  questions: Array<{
    id: string;
    text: string;
    type: 'scale' | 'multiple_choice' | 'text';
    options?: string[];
  }>;
}

interface AssessmentResult {
  stress_level: number; // 0-100
  category: 'low' | 'moderate' | 'high' | 'severe';
  recommendations: string[];
  coping_strategies: string[];
  crisis_resources?: string[];
}

interface MentalHealthService {
  getQuestionnaire(type: string): Promise<Questionnaire>;
  submitAssessment(answers: Record<string, any>): Promise<AssessmentResult>;
  getAssessmentHistory(userId: string): Promise<AssessmentResult[]>;
}
```

### 9. Student Learning Tools

**Purpose**: Generate slides, mind maps, and images for education

**Components**:
- `SlideGenerator`: Create educational slides
- `MindMapGenerator`: Visual concept mapping
- `ImageGenerator`: Pollinations.ai integration
- `ExportOptions`: PDF, PNG, JSON export

**Interfaces**:
```typescript
interface SlideGenerationRequest {
  topic: string;
  num_slides: number;
  detail_level: 'basic' | 'intermediate' | 'advanced';
}

interface Slide {
  title: string;
  content: string[];
  images?: string[];
  notes?: string;
}

interface MindMapNode {
  id: string;
  label: string;
  children: MindMapNode[];
  metadata?: Record<string, any>;
}

interface StudentToolsService {
  generateSlides(request: SlideGenerationRequest): Promise<Slide[]>;
  generateMindMap(topic: string): Promise<MindMapNode>;
  generateImage(prompt: string): Promise<string>; // Returns image URL
  exportContent(content: any, format: 'pdf' | 'png' | 'json'): Promise<Blob>;
}
```

### 10. Synthetic Data Generator

**Purpose**: Generate realistic medical datasets for research

**Components**:
- `SchemaBuilder`: Define dataset schema
- `SampleUploader`: Upload sample data
- `DataPreview`: Preview generated data
- `ExportOptions`: CSV/JSON export

**Interfaces**:
```typescript
interface DataSchema {
  fields: Array<{
    name: string;
    type: 'string' | 'number' | 'boolean' | 'date';
    description: string;
    constraints?: {
      min?: number;
      max?: number;
      pattern?: string;
    };
  }>;
}

interface DataGenerationRequest {
  size: number;
  schema?: DataSchema;
  sample_data?: Record<string, any>[];
}

interface DataGenerationResult {
  dataset: Record<string, any>[];
  statistics: {
    row_count: number;
    column_count: number;
    data_quality_score: number;
  };
}

interface DataGeneratorService {
  generateData(request: DataGenerationRequest): Promise<DataGenerationResult>;
  validateSchema(schema: DataSchema): boolean;
  exportData(data: Record<string, any>[], format: 'csv' | 'json'): Promise<Blob>;
}
```

## Data Models

### Database Schema (Supabase PostgreSQL)

```sql
-- Users table (managed by Supabase Auth)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  role TEXT CHECK (role IN ('patient', 'researcher', 'student', 'doctor')),
  avatar_url TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  last_login TIMESTAMP,
  metadata JSONB
);

-- Conversations table
CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  title TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Messages table
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE,
  role TEXT CHECK (role IN ('user', 'assistant', 'system')),
  content TEXT NOT NULL,
  timestamp TIMESTAMP DEFAULT NOW(),
  metadata JSONB
);

-- Diagnosis history table
CREATE TABLE diagnosis_history (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  symptom_data JSONB NOT NULL,
  predictions JSONB NOT NULL,
  treatment_plan JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Image analysis history table
CREATE TABLE image_analysis_history (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  image_url TEXT NOT NULL,
  image_type TEXT,
  analysis_result JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Mental health assessments table
CREATE TABLE mental_health_assessments (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  questionnaire_type TEXT NOT NULL,
  answers JSONB NOT NULL,
  result JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Student content history table
CREATE TABLE student_content_history (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  content_type TEXT CHECK (content_type IN ('slide', 'mindmap', 'image')),
  content_data JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Drug analysis history table
CREATE TABLE drug_analysis_history (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  smiles TEXT,
  disease TEXT,
  analysis_result JSONB NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Usage quotas table (for free-tier management)
CREATE TABLE usage_quotas (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  groq_requests INTEGER DEFAULT 0,
  whisper_requests INTEGER DEFAULT 0,
  image_generations INTEGER DEFAULT 0,
  UNIQUE(user_id, date)
);

-- Indexes for performance
CREATE INDEX idx_conversations_user_id ON conversations(user_id);
CREATE INDEX idx_messages_conversation_id ON messages(conversation_id);
CREATE INDEX idx_diagnosis_user_id ON diagnosis_history(user_id);
CREATE INDEX idx_image_analysis_user_id ON image_analysis_history(user_id);
CREATE INDEX idx_mental_health_user_id ON mental_health_assessments(user_id);
CREATE INDEX idx_student_content_user_id ON student_content_history(user_id);
CREATE INDEX idx_usage_quotas_user_date ON usage_quotas(user_id, date);
```

### Vector Database Schema (Qdrant Cloud)

```python
# Collection configuration for medical documents
collection_config = {
    "name": "medical_documents",
    "vectors": {
        "size": 768,  # gemini-embedding-001 dimension
        "distance": "Cosine"
    },
    "payload_schema": {
        "content": "text",
        "source": "keyword",
        "title": "text",
        "category": "keyword",  # diagnosis, treatment, drug, etc.
        "timestamp": "integer"
    }
}
```

### File Storage Structure (Supabase Storage)

```
buckets/
├── medical-images/
│   ├── {user_id}/
│   │   ├── {image_id}.jpg
│   │   └── {image_id}.png
├── user-avatars/
│   └── {user_id}.jpg
├── generated-content/
│   ├── slides/
│   │   └── {user_id}/{content_id}.pdf
│   ├── mindmaps/
│   │   └── {user_id}/{content_id}.png
│   └── images/
│       └── {user_id}/{content_id}.png
└── medical-documents/
    └── {document_id}.pdf
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

Before defining the correctness properties, let me analyze the acceptance criteria to determine which are testable as properties, examples, or edge cases.


### Core Properties

**Property 1: Authentication Round-Trip**
*For any* successful Google OAuth authentication, creating or retrieving a user profile and then querying for that profile should return the same user data.
**Validates: Requirements 1.2**

**Property 2: Role-Based Access Control**
*For any* user with a specific role (Patient, Researcher, Student, Doctor), accessing features should succeed only for features permitted by that role, and fail for features not permitted by that role.
**Validates: Requirements 1.7, 1.8, 1.9, 1.10, 2.5**

**Property 3: Session Duration Invariant**
*For any* authenticated session, the session should remain valid for any time period less than 24 hours from creation, and should expire after 24 hours.
**Validates: Requirements 1.4**

**Property 4: Logout Cleanup**
*For any* authenticated user, after logging out, attempting to use their session token should fail with an authentication error.
**Validates: Requirements 1.6**

**Property 5: Conversation Persistence Round-Trip**
*For any* conversation with messages, saving the conversation and then loading it should return an equivalent conversation with the same messages in the same order.
**Validates: Requirements 2.6, 2.7**

**Property 6: API Fallback Mechanism**
*For any* chatbot query, if the Groq API fails or is unavailable, the system should successfully complete the request using the Hugging Face API.
**Validates: Requirements 2.2**

**Property 7: RAG Context Retrieval**
*For any* chatbot query, the response generation should invoke the RAG system to retrieve relevant medical context from the vector database.
**Validates: Requirements 2.3**

**Property 8: Web Search Trigger**
*For any* query containing keywords indicating need for current information (e.g., "latest", "recent", "current", "2024"), the chatbot should perform a web search.
**Validates: Requirements 2.4**

**Property 9: Conversation Context Preservation**
*For any* sequence of messages in a conversation, follow-up messages should reference and maintain context from previous messages in the same conversation.
**Validates: Requirements 2.9**

**Property 10: Content Filtering Rejection**
*For any* query that is classified as non-medical (entertainment, politics, general knowledge), the chatbot should politely decline to answer and suggest medical topics instead.
**Validates: Requirements 2.15, 14.1, 14.2, 14.3, 14.4, 14.5, 14.6**

**Property 11: Medical Disclaimer Presence**
*For any* response containing medical information (diagnosis, treatment, analysis), the UI should display a medical disclaimer warning users to consult healthcare professionals.
**Validates: Requirements 2.8, 4.6, 5.7, 7.6, 12.1, 12.2, 12.3, 12.4, 12.5**

**Property 12: Voice Transcription Round-Trip**
*For any* audio input containing clear speech, transcribing via Groq Whisper and then synthesizing via browser TTS should produce semantically equivalent content.
**Validates: Requirements 2.11, 2.12, 2.13**

**Property 13: Student Content Generation**
*For any* student user requesting content generation (slides, mind maps, images), the system should generate valid content and save it to their history.
**Validates: Requirements 3.2, 3.3, 3.4, 3.8**

**Property 14: Student Role Exclusivity**
*For any* user with Doctor role, attempting to access slide generation or mind map features should be denied.
**Validates: Requirements 3.10**

**Property 15: Diagnosis Prediction Ordering**
*For any* completed symptom collection, the disease predictions should be ordered by confidence score in descending order (highest confidence first).
**Validates: Requirements 4.4**

**Property 16: Diagnosis Persistence**
*For any* completed diagnosis, the diagnosis report should be saved to the database and retrievable from the user's history.
**Validates: Requirements 4.7**

**Property 17: Symptom Input Validation**
*For any* symptom input, the system should validate that the input contains medical terminology or symptom descriptions, rejecting non-medical inputs.
**Validates: Requirements 4.8**

**Property 18: Image Size Validation**
*For any* image upload attempt, files larger than 10MB should be rejected with a clear error message.
**Validates: Requirements 5.1**

**Property 19: Image Storage Persistence**
*For any* valid image upload, the image should be stored in Supabase Storage and the returned URL should allow retrieval of the same image.
**Validates: Requirements 5.2**

**Property 20: Image Analysis Confidence Scores**
*For any* image analysis result, the findings should include confidence scores for each detected condition or anomaly.
**Validates: Requirements 5.4**

**Property 21: OCR Text Extraction**
*For any* image containing visible text (medical reports), the OCR extraction should return the text content with reasonable accuracy.
**Validates: Requirements 5.5**

**Property 22: SMILES Validation**
*For any* SMILES formula input, invalid molecular structures should be rejected with a descriptive error message.
**Validates: Requirements 6.1**

**Property 23: Drug Property Prediction**
*For any* valid SMILES formula, the drug discovery tool should predict molecular properties including molecular weight, logP, solubility, and toxicity.
**Validates: Requirements 6.2**

**Property 24: Drug Candidate Confidence**
*For any* drug recommendation, each candidate should include a confidence score between 0 and 1.
**Validates: Requirements 6.5**

**Property 25: Mental Health Questionnaire Selection**
*For any* mental health assessment type (student, parent, corporate, partner), the system should present the questionnaire specific to that type.
**Validates: Requirements 7.1**

**Property 26: Stress Score Calculation**
*For any* completed mental health questionnaire, the system should calculate a stress level score between 0 and 100.
**Validates: Requirements 7.2**

**Property 27: Assessment History Tracking**
*For any* user completing multiple mental health assessments, the system should store all assessments and calculate trends over time.
**Validates: Requirements 7.5**

**Property 28: Data Schema Validation**
*For any* synthetic data generation request with a schema, invalid schemas (missing required fields, invalid types) should be rejected.
**Validates: Requirements 8.1**

**Property 29: Synthetic Data Generation**
*For any* valid data generation request, the system should generate exactly the requested number of records matching the schema.
**Validates: Requirements 8.2**

**Property 30: Statistical Property Preservation**
*For any* sample data provided, the generated synthetic data should maintain similar statistical distributions (mean, standard deviation, correlations).
**Validates: Requirements 8.3**

**Property 31: Data Export Format**
*For any* generated dataset, exporting to CSV or JSON should produce valid files that can be parsed by standard tools.
**Validates: Requirements 8.4**

**Property 32: Response Time Performance**
*For any* chatbot query, the system should return a response within 5 seconds under normal load conditions.
**Validates: Requirements 2.1, 9.2**

**Property 33: Page Load Performance**
*For any* page navigation, the initial content should be visible within 1.5 seconds on standard broadband connections.
**Validates: Requirements 9.1, 15.1**

**Property 34: HTTPS Encryption**
*For any* data transmission between client and server, the connection should use HTTPS with TLS 1.3 or higher.
**Validates: Requirements 10.1**

**Property 35: Data Deletion Compliance**
*For any* user account deletion, all personal and medical data should be removed from the database within 30 days.
**Validates: Requirements 10.3**

**Property 36: Encryption at Rest**
*For any* sensitive medical data stored in the database, the data should be encrypted using AES-256 or equivalent.
**Validates: Requirements 10.4**

**Property 37: Role-Based Data Access**
*For any* attempt to access user data, the request should succeed only if the requesting user has appropriate permissions (owns the data or has admin role).
**Validates: Requirements 10.5**

**Property 38: JWT Token Properties**
*For any* issued authentication token, it should be a valid JWT with expiration time, signature, and user claims.
**Validates: Requirements 10.6**

**Property 39: Cross-Browser Functionality**
*For any* core feature (chat, diagnosis, image analysis), the feature should work correctly in Chrome, Firefox, Safari, and Edge browsers.
**Validates: Requirements 11.1**

**Property 40: Keyboard Accessibility**
*For any* interactive element in the UI, it should be accessible and operable using keyboard navigation alone.
**Validates: Requirements 11.4**

**Property 41: WCAG Color Contrast**
*For any* text element in the UI, the color contrast ratio between text and background should meet WCAG 2.1 AA standards (minimum 4.5:1).
**Validates: Requirements 11.5**

**Property 42: Quota Enforcement**
*For any* user approaching free-tier limits (Groq API, storage, etc.), the system should throttle requests or display warnings before limits are exceeded.
**Validates: Requirements 16.1, 16.2, 16.3, 16.4**

## Error Handling

### Error Categories

1. **Authentication Errors**
   - Invalid OAuth token
   - Expired session
   - Insufficient permissions
   - Response: 401 Unauthorized with clear message

2. **Validation Errors**
   - Invalid input format (SMILES, image size, schema)
   - Missing required fields
   - Out-of-range values
   - Response: 400 Bad Request with field-specific errors

3. **API Errors**
   - Groq API unavailable → Fallback to Hugging Face
   - Hugging Face API unavailable → Return cached response or error
   - Qdrant unavailable → Disable RAG, use LLM only
   - Response: 503 Service Unavailable with retry guidance

4. **Quota Errors**
   - Daily API limit exceeded
   - Storage limit reached
   - Database size limit reached
   - Response: 429 Too Many Requests with quota reset time

5. **Content Filtering Errors**
   - Non-medical query detected
   - Inappropriate content detected
   - Response: 200 OK with polite decline message

6. **Performance Errors**
   - Request timeout (> 30 seconds)
   - Large file upload (> 10MB)
   - Response: 408 Request Timeout or 413 Payload Too Large

### Error Handling Strategy

```typescript
interface ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: Record<string, any>;
    retry_after?: number; // seconds
    fallback_available?: boolean;
  };
}

// Example error handling in service layer
async function handleAPICall<T>(
  primaryCall: () => Promise<T>,
  fallbackCall?: () => Promise<T>
): Promise<T> {
  try {
    return await primaryCall();
  } catch (error) {
    if (fallbackCall && isServiceUnavailable(error)) {
      console.warn('Primary API failed, using fallback');
      return await fallbackCall();
    }
    throw error;
  }
}
```

### Graceful Degradation

1. **Groq API Unavailable**
   - Fallback to Hugging Face API
   - Display warning: "Using backup AI service"

2. **RAG System Unavailable**
   - Use LLM without context retrieval
   - Display warning: "Limited context available"

3. **Voice Features Unavailable**
   - Disable voice buttons
   - Show message: "Voice features require browser support"

4. **Web Search Unavailable**
   - Use cached/RAG data only
   - Display warning: "Latest information may not be available"

5. **Image Generation Unavailable**
   - Show placeholder or error message
   - Suggest trying again later

## Testing Strategy

### Dual Testing Approach

The platform requires both unit tests and property-based tests for comprehensive coverage:

**Unit Tests**: Verify specific examples, edge cases, and error conditions
- Authentication flows (login, logout, session refresh)
- API endpoint responses
- UI component rendering
- Error handling scenarios
- Integration between components

**Property-Based Tests**: Verify universal properties across all inputs
- Generate random user data, messages, images, etc.
- Test properties hold for all generated inputs
- Minimum 100 iterations per property test
- Each test references its design document property

### Property-Based Testing Configuration

**Framework**: Use `fast-check` for TypeScript/JavaScript, `Hypothesis` for Python

**Test Structure**:
```typescript
// Example property test
import fc from 'fast-check';

describe('Property 5: Conversation Persistence Round-Trip', () => {
  it('should preserve conversation data after save and load', async () => {
    await fc.assert(
      fc.asyncProperty(
        fc.array(fc.record({
          role: fc.constantFrom('user', 'assistant'),
          content: fc.string({ minLength: 1, maxLength: 1000 })
        })),
        async (messages) => {
          // Feature: arivu-care-platform, Property 5: Conversation Persistence Round-Trip
          const conversation = await saveConversation(messages);
          const loaded = await loadConversation(conversation.id);
          
          expect(loaded.messages).toHaveLength(messages.length);
          expect(loaded.messages).toEqual(messages);
        }
      ),
      { numRuns: 100 }
    );
  });
});
```

**Test Tags**: Each property test must include a comment:
```
// Feature: arivu-care-platform, Property {number}: {property_text}
```

### Unit Test Focus Areas

1. **Authentication**
   - Google OAuth callback handling
   - Session token validation
   - Role assignment logic
   - Logout cleanup

2. **Content Filtering**
   - Medical keyword detection
   - Non-medical query rejection
   - Edge cases (ambiguous queries)

3. **API Integration**
   - Groq API request/response
   - Hugging Face fallback
   - Qdrant vector search
   - Error handling

4. **Data Validation**
   - SMILES formula validation
   - Image size/format validation
   - Schema validation
   - Input sanitization

5. **UI Components**
   - Chat message rendering
   - Voice button states
   - Loading indicators
   - Error messages

### Integration Tests

1. **End-to-End Flows**
   - Complete diagnosis flow (symptoms → questions → prediction → treatment)
   - Image upload and analysis flow
   - Student content generation flow
   - Mental health assessment flow

2. **Cross-Component Integration**
   - Chat + RAG + Web Search
   - Voice input + Transcription + Chat
   - Authentication + Role-based access
   - Data persistence + Retrieval

### Performance Tests

1. **Load Testing**
   - 100 concurrent users
   - Measure response times
   - Monitor resource usage

2. **Stress Testing**
   - Approach free-tier limits
   - Test quota enforcement
   - Verify graceful degradation

3. **Browser Testing**
   - Test on Chrome, Firefox, Safari, Edge
   - Verify responsive design
   - Check accessibility features

### Test Coverage Goals

- Unit test coverage: > 60%
- Property test coverage: All 42 properties
- Integration test coverage: All major user flows
- Browser compatibility: 100% on supported browsers
- Accessibility compliance: 100% WCAG 2.1 AA

## Implementation Notes

### Technology-Specific Considerations

**Next.js 15**:
- Use App Router for server-side rendering
- Implement React Server Components for initial page load
- Use client components for interactive features
- Leverage Next.js Image component for optimization

**FastAPI**:
- Use Pydantic models for request/response validation
- Implement dependency injection for services
- Use async/await for all I/O operations
- Add middleware for CORS, rate limiting, logging

**Supabase**:
- Use Row Level Security (RLS) for data access control
- Implement database triggers for audit logging
- Use Supabase Realtime for live updates (optional)
- Configure storage buckets with appropriate policies

**Groq API**:
- Implement request queuing to manage rate limits
- Cache responses where appropriate
- Monitor daily quota usage
- Implement exponential backoff for retries

**Qdrant Cloud**:
- Use collections with appropriate vector dimensions
- Implement batch indexing for efficiency
- Use filters for metadata-based search
- Monitor storage usage

**LlamaIndex**:
- Configure chunk size and overlap for documents
- Use appropriate embedding model (gemini-embedding-001)
- Implement query transformations for better retrieval
- Cache embeddings to reduce API calls

### Security Considerations

1. **Input Sanitization**
   - Sanitize all user inputs before processing
   - Validate file uploads (type, size, content)
   - Prevent SQL injection with parameterized queries
   - Prevent XSS with proper escaping

2. **Authentication Security**
   - Use secure, httpOnly cookies for tokens
   - Implement CSRF protection
   - Rotate refresh tokens regularly
   - Log authentication events

3. **Data Privacy**
   - Encrypt sensitive data at rest
   - Use HTTPS for all communications
   - Implement data retention policies
   - Provide data export/deletion options

4. **API Security**
   - Store API keys in environment variables
   - Never expose keys in client-side code
   - Implement rate limiting per user
   - Monitor for suspicious activity

### Performance Optimization

1. **Frontend Optimization**
   - Code splitting by route
   - Lazy load components
   - Optimize images (WebP, responsive sizes)
   - Use CDN for static assets
   - Implement service worker for offline support

2. **Backend Optimization**
   - Cache frequently accessed data (Redis optional)
   - Use database indexes for common queries
   - Implement connection pooling
   - Batch API requests where possible

3. **Database Optimization**
   - Use appropriate indexes
   - Implement query optimization
   - Archive old data
   - Monitor query performance

4. **API Optimization**
   - Batch embeddings generation
   - Cache RAG results
   - Implement request deduplication
   - Use streaming for long responses

### Monitoring and Logging

1. **Application Metrics**
   - Response times (p50, p95, p99)
   - Error rates by endpoint
   - API quota usage
   - Database size and query performance

2. **User Metrics**
   - Daily/monthly active users
   - Feature usage statistics
   - Session duration
   - Conversion rates

3. **System Health**
   - API availability
   - Database connection pool
   - Storage usage
   - Memory and CPU usage

4. **Logging Strategy**
   - Structured logging (JSON format)
   - Log levels: DEBUG, INFO, WARN, ERROR
   - Include request IDs for tracing
   - Sanitize sensitive data from logs

### Deployment Strategy

1. **Development Environment**
   - Local Next.js dev server
   - Local FastAPI with hot reload
   - Supabase local development
   - Environment variables in .env.local

2. **Staging Environment** (Optional)
   - Deploy to Vercel (free tier)
   - Deploy FastAPI to Railway (free tier)
   - Use Supabase staging project
   - Test with production-like data

3. **Production Environment**
   - Deploy frontend to Vercel
   - Deploy backend to Railway or Render
   - Use Supabase production project
   - Configure custom domain
   - Set up monitoring and alerts

### Future Enhancements

1. **Phase 2 Features**
   - Real-time collaboration for researchers
   - Advanced analytics dashboard
   - Mobile app (React Native)
   - Offline mode support

2. **Scalability Improvements**
   - Migrate to paid tiers as user base grows
   - Implement caching layer (Redis)
   - Add load balancing
   - Optimize database queries

3. **Feature Enhancements**
   - Multi-language support
   - Advanced visualization tools
   - Integration with wearable devices
   - Telemedicine features (if needed)

4. **AI Improvements**
   - Fine-tune models on medical data
   - Implement feedback loop for model improvement
   - Add more specialized AI models
   - Improve RAG retrieval accuracy
