# Design Document: Arivu Medical Assistant Platform

## Overview

Arivu is a dual-role medical assistant platform that serves both doctors and patients with AI-powered medical assistance. The architecture is built on a modular, role-based design that maintains strict data isolation while sharing core infrastructure components. The system leverages Large Language Models (LLMs) via APIs, Retrieval-Augmented Generation (RAG) for context-aware responses, and voice capabilities for accessibility.

**Key Design Principles:**
- Role-based separation of concerns (Doctor vs. Patient interfaces)
- Data isolation and privacy-first architecture
- Modular component design for maintainability
- Shared infrastructure for common functionality (LLM integration, voice, query screening)
- Scalable RAG architecture with separate knowledge bases per role
- HIPAA-compliant data handling and encryption

## Architecture

### High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Arivu Platform                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────┐          ┌──────────────────────┐     │
│  │   Doctor Interface   │          │   Patient Interface  │     │
│  ├──────────────────────┤          ├──────────────────────┤     │
│  │ • Chat              │          │ • Chat               │     │
│  │ • Content Gen       │          │ • Symptom Checker    │     │
│  │ • Web Search        │          │ • Health Records     │     │
│  │ • PDF Upload        │          │ • Symptom Tracker    │     │
│  │ • Voice             │          │ • Medication Info    │     │
│  └──────────────────────┘          │ • Appointments       │     │
│           │                        │ • Voice              │     │
│           │                        └──────────────────────┘     │
│           │                                 │                    │
│           └─────────────┬───────────────────┘                    │
│                         │                                        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │         Shared Core Services Layer                       │   │
│  ├──────────────────────────────────────────────────────────┤   │
│  │ • Medical Query Screener                                 │   │
│  │ • LLM Integration Service                                │   │
│  │ • Voice Service (Speech-to-Text, Text-to-Speech)        │   │
│  │ • Authentication & Authorization                         │   │
│  │ • Data Encryption & Security                             │   │
│  │ • Audit Logging                                          │   │
│  └──────────────────────────────────────────────────────────┘   │
│           │                                                      │
│  ┌────────┴──────────────────────────────────────────────────┐  │
│  │         Data & Knowledge Layer                            │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │ • Doctor RAG Knowledge Base                              │  │
│  │ • Patient RAG Knowledge Base                             │  │
│  │ • Medical Knowledge Base (General)                       │  │
│  │ • User Data Store (Encrypted)                            │  │
│  │ • Health Records Store                                   │  │
│  │ • Vector Database (Embeddings)                           │  │
│  └──────────────────────────────────────────────────────────┘  │
│           │                                                      │
│  ┌────────┴──────────────────────────────────────────────────┐  │
│  │         External Integrations                             │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │ • LLM APIs (medGemma)                                    │  │
│  │ • Speech-to-Text API                                     │  │
│  │ • Text-to-Speech API (Google Cloud, Azure)              │  │
│  │ • Web Search API                                         │  │
│  │                                                          │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Component Architecture

#### 1. Doctor Interface Components

**Chat Component**
- Accepts medical queries from doctors
- Maintains conversation history
- Integrates with Medical Query Screener
- Retrieves context from Doctor RAG
- Sends augmented queries to LLM Service
- Displays formatted responses with source citations

**Content Generation Component**
- Accepts medical topics and generation parameters
- Generates infographic specifications (JSON format describing visual layout)
- Generates mind map structures (hierarchical JSON)
- Generates slide presentation outlines (structured content)
- Interfaces with rendering services for final output
- Stores generated content for reuse

**Web Search Component**
- Accepts search queries
- Validates queries through Medical Query Screener
- Calls external web search API
- Filters results for medical relevance
- Summarizes results using LLM
- Displays results with source attribution

**PDF Upload Component**
- Accepts PDF file uploads
- Validates file format and size
- Extracts text content from PDFs
- Chunks text into RAG-suitable segments
- Stores chunks in Doctor RAG Knowledge Base
- Maintains document metadata and timestamps

#### 2. Patient Interface Components

**Chat Component**
- Accepts health queries from patients
- Maintains conversation history
- Integrates with Medical Query Screener
- Retrieves context from Patient RAG (personal history + general knowledge)
- Sends augmented queries to LLM Service
- Displays responses in simplified, patient-friendly language

**Symptom Checker Component**
- Accepts symptom descriptions
- Validates symptoms through Medical Query Screener
- Queries Medical Knowledge Base for matching conditions
- Ranks conditions by likelihood
- Displays conditions with severity indicators
- Provides educational information for selected conditions

**Health Records Component**
- Displays patient's historical medical data
- Organizes records by type (consultations, prescriptions, test results)
- Implements filtering by date range and type
- Enforces data isolation (patient only sees own records)
- Displays associated provider information

**Symptom Tracker Component**
- Accepts symptom logging with severity and timestamp
- Stores symptoms in patient's health records
- Displays symptom history with trend analysis
- Generates shareable symptom reports
- Identifies patterns and anomalies

**Medication Reminders Component**
- Accepts medication information (name, dosage, frequency)
- Creates reminder schedule
- Sends notifications at scheduled times
- Records adherence events
- Displays compliance statistics

**Doctor Notes Summary Component**
- Receives doctor's clinical notes
- Generates simplified summaries using LLM
- Highlights key findings and recommendations
- Displays summary alongside original notes
- Updates when notes are modified

**Personalized Health Insights Component**
- Analyzes patient's health data patterns
- Generates insights using LLM
- Provides supporting evidence from records
- Suggests follow-up actions
- Records user preferences for insight types

**Appointment Booking Component**
- Connects to appointment booking system
- Displays available slots
- Confirms bookings
- Stores appointments in health records
- Sends confirmation notifications

#### 3. Shared Core Services

**Medical Query Screener Service**
- Classifies queries as medical or non-medical
- Uses LLM-based classification with medical domain knowledge
- Maintains consistent criteria across doctor and patient roles
- Returns classification confidence score
- Provides rejection feedback for non-medical queries

**LLM Integration Service**
- Manages connections to multiple LLM providers
- Implements rate limiting and quota management
- Handles API failures with retry logic (exponential backoff)
- Validates responses for medical appropriateness
- Tracks token usage and costs
- Supports prompt engineering for role-specific responses

**Voice Service**
- Speech-to-Text: Converts audio to text using medical-specialized models
- Text-to-Speech: Converts responses to speech with appropriate tone
- Handles audio quality issues and background noise
- Supports multiple languages
- Implements audio caching for common responses

**Authentication & Authorization Service**
- Manages user login and session management
- Verifies doctor credentials and medical licenses
- Implements role-based access control (RBAC)
- Enforces data isolation by user ID
- Manages API tokens and refresh tokens

**Data Encryption & Security Service**
- Encrypts sensitive data at rest using AES-256
- Encrypts data in transit using TLS 1.3
- Manages encryption keys securely
- Implements field-level encryption for PHI
- Handles secure key rotation

**Audit Logging Service**
- Records all data access events
- Logs user actions with timestamps
- Tracks data modifications
- Maintains immutable audit trail
- Supports compliance reporting

#### 4. Data & Knowledge Layer

**Doctor RAG Knowledge Base**
- Stores doctor-uploaded PDF content
- Organizes by document and timestamp
- Implements vector embeddings for semantic search
- Supports full-text search
- Maintains document metadata
- Implements TTL for document retention

**Patient RAG Knowledge Base**
- Stores patient's personal medical history
- Includes past consultations, prescriptions, test results
- Stores doctor's notes (with patient access)
- Maintains symptom history
- Implements vector embeddings for semantic search
- Enforces strict data isolation

**Medical Knowledge Base (General)**
- Centralized repository of medical information
- Includes conditions, medications, treatments, procedures
- Sourced from authoritative medical databases
- Regularly updated with latest medical knowledge
- Implements version control for knowledge updates
- Supports multi-language content

**Vector Database**
- Stores embeddings for all RAG content
- Enables semantic similarity search
- Supports efficient retrieval at scale
- Implements caching for frequently accessed embeddings
- Supports batch operations for bulk indexing

**User Data Store**
- Stores encrypted user profiles
- Maintains authentication credentials (hashed)
- Stores user preferences and settings
- Implements data isolation by user ID
- Supports efficient querying by user attributes

**Health Records Store**
- Stores patient health records (encrypted)
- Organizes by record type and date
- Maintains referential integrity
- Supports efficient filtering and search
- Implements backup and recovery mechanisms

### Data Flow Diagrams

#### Doctor Chat Flow

```
Doctor Input
    ↓
Medical Query Screener
    ├─ Non-medical → Reject with feedback
    └─ Medical → Continue
    ↓
Doctor RAG Retrieval
    ├─ Query embedding
    ├─ Vector similarity search
    └─ Retrieve top-k relevant chunks
    ↓
LLM Integration Service
    ├─ Augment query with context
    ├─ Call LLM API
    └─ Validate response
    ↓
Response Formatting & Display
    ├─ Add source citations
    ├─ Format for UI
    └─ Display to doctor
    ↓
Audit Logging
    └─ Log query, context, response
```

#### Patient Chat Flow

```
Patient Input
    ↓
Medical Query Screener
    ├─ Non-medical → Reject with feedback
    └─ Medical → Continue
    ↓
Patient RAG Retrieval
    ├─ Query embedding
    ├─ Vector similarity search
    ├─ Retrieve from personal history
    ├─ Retrieve from general knowledge base
    └─ Rank by relevance
    ↓
LLM Integration Service
    ├─ Augment query with context
    ├─ Call LLM API with patient-friendly prompt
    └─ Validate response
    ↓
Response Simplification
    ├─ Ensure patient-friendly language
    ├─ Add educational context
    └─ Format for UI
    ↓
Display to Patient
    ├─ Add source citations
    ├─ Highlight key information
    └─ Suggest related topics
    ↓
Audit Logging
    └─ Log query, context, response
```

#### PDF Upload & RAG Indexing Flow

```
Doctor Uploads PDF
    ↓
File Validation
    ├─ Check format (PDF)
    ├─ Check file size
    └─ Scan for malware
    ↓
Text Extraction
    ├─ Extract text from PDF
    ├─ Preserve structure
    └─ Handle images/tables
    ↓
Text Chunking
    ├─ Split into semantic chunks
    ├─ Maintain context overlap
    └─ Generate chunk metadata
    ↓
Embedding Generation
    ├─ Generate embeddings for each chunk
    ├─ Use medical-domain model
    └─ Store in vector database
    ↓
RAG Knowledge Base Storage
    ├─ Store chunks with metadata
    ├─ Associate with doctor ID
    ├─ Index for retrieval
    └─ Update search indices
    ↓
Confirmation to Doctor
    └─ Display upload success
```

## Components and Interfaces

### Core Interfaces

#### IQueryScreener
```
interface IQueryScreener {
  classifyQuery(query: string, userRole: 'doctor' | 'patient'): {
    isMedical: boolean
    confidence: number
    reason?: string
  }
}
```

#### IRAGRetriever
```
interface IRAGRetriever {
  retrieveContext(
    query: string,
    userId: string,
    contextSources: string[],
    topK: number
  ): Promise<{
    chunks: Array<{
      content: string
      source: string
      relevanceScore: number
      metadata: Record<string, any>
    }>
    totalRetrieved: number
  }>
}
```

#### ILLMService
```
interface ILLMService {
  generateResponse(
    query: string,
    context: string,
    userRole: 'doctor' | 'patient',
    systemPrompt?: string
  ): Promise<{
    response: string
    tokensUsed: number
    model: string
  }>
  
  generateContent(
    topic: string,
    contentType: 'infographic' | 'mindmap' | 'slides',
    parameters: Record<string, any>
  ): Promise<{
    content: Record<string, any>
    format: string
  }>
}
```

#### IVoiceService
```
interface IVoiceService {
  speechToText(audioBuffer: Buffer): Promise<{
    text: string
    confidence: number
    language: string
  }>
  
  textToSpeech(
    text: string,
    userRole: 'doctor' | 'patient',
    language?: string
  ): Promise<Buffer>
}
```

#### IAuthService
```
interface IAuthService {
  authenticate(
    username: string,
    password: string
  ): Promise<{
    userId: string
    userRole: 'doctor' | 'patient'
    token: string
    expiresIn: number
  }>
  
  authorize(
    userId: string,
    resource: string,
    action: string
  ): Promise<boolean>
}
```

#### IEncryptionService
```
interface IEncryptionService {
  encrypt(data: string, keyId?: string): Promise<string>
  decrypt(encryptedData: string, keyId?: string): Promise<string>
  encryptField(data: any, fieldPath: string): Promise<any>
  decryptField(data: any, fieldPath: string): Promise<any>
}
```

## Data Models

### User Models

```
Doctor {
  id: string (UUID)
  username: string
  email: string (encrypted)
  passwordHash: string
  medicalLicense: string (encrypted)
  licenseVerified: boolean
  specialization: string[]
  createdAt: timestamp
  updatedAt: timestamp
  isActive: boolean
}

Patient {
  id: string (UUID)
  username: string
  email: string (encrypted)
  passwordHash: string
  dateOfBirth: date (encrypted)
  medicalHistory: string[] (references to HealthRecord IDs)
  createdAt: timestamp
  updatedAt: timestamp
  isActive: boolean
  emergencyContact: {
    name: string (encrypted)
    phone: string (encrypted)
    relationship: string
  }
}
```

### Health Records Models

```
HealthRecord {
  id: string (UUID)
  patientId: string
  type: 'consultation' | 'prescription' | 'test_result' | 'note'
  title: string
  content: string (encrypted)
  doctorId: string (if applicable)
  createdAt: timestamp
  updatedAt: timestamp
  metadata: {
    provider: string
    facility: string
    tags: string[]
  }
}

Consultation {
  extends HealthRecord
  type: 'consultation'
  symptoms: string[]
  diagnosis: string
  treatment: string
  followUpDate: date
  notes: string (encrypted)
}

Prescription {
  extends HealthRecord
  type: 'prescription'
  medications: Array<{
    name: string
    dosage: string
    frequency: string
    duration: string
    instructions: string
  }>
  prescribedBy: string
  prescribedDate: date
}

TestResult {
  extends HealthRecord
  type: 'test_result'
  testName: string
  results: Record<string, any> (encrypted)
  normalRange: Record<string, any>
  interpretations: string
  orderedBy: string
}
```

### RAG Models

```
RAGChunk {
  id: string (UUID)
  content: string
  embedding: number[] (vector)
  source: string
  sourceType: 'uploaded_pdf' | 'health_record' | 'general_knowledge'
  ownerId: string (doctor or patient ID)
  metadata: {
    documentId: string
    chunkIndex: number
    createdAt: timestamp
    relevanceScore?: number
  }
}

RAGDocument {
  id: string (UUID)
  ownerId: string
  fileName: string
  fileType: string
  uploadedAt: timestamp
  chunkIds: string[]
  metadata: {
    pageCount?: number
    extractedText: string
    summary?: string
  }
}
```

### Conversation Models

```
Conversation {
  id: string (UUID)
  userId: string
  userRole: 'doctor' | 'patient'
  startedAt: timestamp
  updatedAt: timestamp
  messages: Message[]
  context: {
    retrievedChunks: RAGChunk[]
    sources: string[]
  }
}

Message {
  id: string (UUID)
  conversationId: string
  role: 'user' | 'assistant'
  content: string
  timestamp: timestamp
  metadata: {
    tokensUsed?: number
    model?: string
    voiceInput?: boolean
  }
}
```

### Symptom Tracker Models

```
SymptomLog {
  id: string (UUID)
  patientId: string
  symptom: string
  severity: 1-10
  description: string
  loggedAt: timestamp
  metadata: {
    duration?: string
    triggers?: string[]
    relievedBy?: string[]
  }
}

SymptomReport {
  id: string (UUID)
  patientId: string
  generatedAt: timestamp
  dateRange: {
    startDate: date
    endDate: date
  }
  symptoms: SymptomLog[]
  patterns: {
    frequency: Record<string, number>
    trends: string[]
    anomalies: string[]
  }
  summary: string
}
```

### Medication Models

```
MedicationReminder {
  id: string (UUID)
  patientId: string
  medication: {
    name: string
    dosage: string
    frequency: string
    startDate: date
    endDate?: date
  }
  schedule: {
    times: string[] (HH:MM format)
    daysOfWeek: number[]
  }
  notificationChannel: 'email' | 'sms' | 'push'
  createdAt: timestamp
  isActive: boolean
}

MedicationAdherence {
  id: string (UUID)
  reminderId: string
  patientId: string
  scheduledTime: timestamp
  takenAt?: timestamp
  status: 'pending' | 'taken' | 'missed' | 'skipped'
  notes?: string
}
```

## Correctness Properties

A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.

### Property 1: Medical Query Screening Consistency

*For any* query submitted by either a doctor or patient, the Medical Query Screener SHALL classify it using the same criteria, producing consistent results for identical queries regardless of user role.

**Validates: Requirements 5, 19, 22**

### Property 2: Data Isolation Enforcement

*For any* patient accessing the system, the system SHALL only return health records, RAG context, and personal data belonging to that specific patient, never returning data from other patients.

**Validates: Requirements 13, 24, 27**

### Property 3: Doctor RAG Context Retrieval

*For any* doctor query and uploaded PDF documents, the RAG system SHALL retrieve relevant chunks from the doctor's uploaded documents with relevance scores that correlate with semantic similarity to the query.

**Validates: Requirements 4, 25**

### Property 4: Patient RAG Multi-Source Retrieval

*For any* patient query, the RAG system SHALL retrieve context from all available sources (personal history, doctor notes, general knowledge base) and rank them by relevance to the patient's specific situation.

**Validates: Requirements 7, 26**

### Property 5: LLM Response Validation

*For any* LLM-generated response, the system SHALL validate that it contains appropriate medical disclaimers and does not make definitive diagnoses without proper context.

**Validates: Requirements 1, 7, 21**

### Property 6: Voice Input-Output Round Trip

*For any* user query submitted via voice, the system SHALL transcribe it to text, process it normally, and convert the response back to speech such that the semantic meaning is preserved.

**Validates: Requirements 6, 20, 23**

### Property 7: Encryption at Rest and in Transit

*For any* sensitive data (PHI, passwords, health records), the system SHALL encrypt it when stored and when transmitted, such that unencrypted data is never exposed in logs or memory dumps.

**Validates: Requirements 24**

### Property 8: Audit Trail Completeness

*For any* data access or modification event, the system SHALL create an immutable audit log entry containing the user ID, timestamp, action, and affected data identifiers.

**Validates: Requirements 24, 27**

### Property 9: PDF Upload and RAG Indexing Round Trip

*For any* PDF document uploaded by a doctor, the system SHALL extract text, chunk it, generate embeddings, and store it such that querying with relevant terms retrieves the appropriate chunks.

**Validates: Requirements 4, 25**

### Property 10: Symptom Tracker Data Persistence

*For any* symptom logged by a patient, the system SHALL store it persistently such that querying the symptom history returns all previously logged symptoms in chronological order.

**Validates: Requirements 14**

### Property 11: Medication Reminder Scheduling

*For any* medication reminder created with a specified schedule, the system SHALL trigger notifications at the scheduled times and record adherence events accurately.

**Validates: Requirements 15**

### Property 12: Doctor Notes Summary Accuracy

*For any* doctor's clinical notes, the system SHALL generate a simplified summary that preserves the key medical findings and recommendations while using patient-friendly language.

**Validates: Requirements 16**

### Property 13: Personalized Health Insights Generation

*For any* patient with sufficient health history, the system SHALL generate personalized insights that are grounded in the patient's actual medical data and include supporting evidence.

**Validates: Requirements 17**

### Property 14: Appointment Booking Integration

*For any* appointment booking request, the system SHALL successfully integrate with the appointment system, confirm the booking, and store it in the patient's health records.

**Validates: Requirements 18**

### Property 15: Content Generation Format Consistency

*For any* content generation request (infographic, mind map, slides), the system SHALL generate output in the specified format that can be rendered and downloaded by the doctor.

**Validates: Requirements 2**

### Property 16: Web Search Result Attribution

*For any* web search performed by a doctor, the system SHALL return results with proper source attribution and relevance ranking that correlates with medical relevance.

**Validates: Requirements 3**

### Property 17: Symptom Checker Condition Matching

*For any* set of symptoms entered by a patient, the Symptom Checker SHALL return matching conditions ranked by likelihood based on the medical knowledge base.

**Validates: Requirements 8**

### Property 18: Medication Information Completeness

*For any* medication queried by a patient, the system SHALL return comprehensive information including uses, dosage, side effects, and interactions with the patient's current medications.

**Validates: Requirements 9**

### Property 19: Health Condition Education Accuracy

*For any* health condition queried by a patient, the system SHALL return accurate, simplified educational information sourced from the medical knowledge base.

**Validates: Requirements 10**

### Property 20: LLM API Failure Recovery

*For any* LLM API call that fails, the system SHALL implement retry logic with exponential backoff and eventually either succeed or return a graceful error to the user.

**Validates: Requirements 21**

## Error Handling

### Query Screening Errors

**Non-Medical Query Rejection**
- When a query is classified as non-medical, the system SHALL:
  - Reject the query immediately
  - Display a user-friendly message explaining why
  - Suggest rephrasing the query in medical terms
  - Log the rejection for audit purposes

**Ambiguous Query Handling**
- When a query is ambiguous, the system SHALL:
  - Err on the side of allowing medical queries
  - Request clarification from the user
  - Provide examples of medical queries

### LLM API Errors

**Rate Limiting**
- When rate limits are exceeded, the system SHALL:
  - Queue the request
  - Implement exponential backoff
  - Notify the user of potential delays
  - Prioritize critical queries

**API Failures**
- When an LLM API call fails, the system SHALL:
  - Retry up to 3 times with exponential backoff
  - Return a graceful error message
  - Suggest alternative actions
  - Log the error for debugging

**Invalid Responses**
- When an LLM generates an invalid response, the system SHALL:
  - Validate the response format
  - Check for medical appropriateness
  - Regenerate if necessary
  - Log the issue for analysis

### Data Access Errors

**Unauthorized Access Attempts**
- When a user attempts unauthorized access, the system SHALL:
  - Deny the access immediately
  - Log the attempt with user ID and timestamp
  - Alert security monitoring systems
  - Implement rate limiting on failed attempts

**Data Isolation Violations**
- When a data isolation violation is detected, the system SHALL:
  - Prevent the access
  - Log the violation
  - Alert administrators
  - Trigger security review

### Voice Processing Errors

**Speech-to-Text Failures**
- When speech-to-text fails, the system SHALL:
  - Notify the user
  - Request re-recording
  - Provide alternative input methods
  - Log the failure

**Text-to-Speech Failures**
- When text-to-speech fails, the system SHALL:
  - Display the text response instead
  - Notify the user
  - Provide alternative output methods
  - Log the failure

### Encryption Errors

**Encryption Failures**
- When encryption fails, the system SHALL:
  - Prevent data storage
  - Return an error to the user
  - Log the error
  - Alert administrators

**Decryption Failures**
- When decryption fails, the system SHALL:
  - Prevent data access
  - Log the failure
  - Alert administrators
  - Implement key recovery procedures

## Testing Strategy

### Dual Testing Approach

The testing strategy combines unit tests and property-based tests to ensure comprehensive coverage:

**Unit Tests**: Verify specific examples, edge cases, and error conditions
- Test individual components in isolation
- Verify error handling paths
- Test boundary conditions
- Validate data transformations

**Property-Based Tests**: Verify universal properties across all inputs
- Test that properties hold for randomly generated inputs
- Verify data isolation across multiple users
- Test RAG retrieval consistency
- Validate encryption/decryption round trips
- Verify audit logging completeness

### Property-Based Testing Configuration

Each property-based test SHALL:
- Run minimum 100 iterations with randomized inputs
- Reference the corresponding design property
- Use the tag format: **Feature: arivu-medical-assistant, Property {number}: {property_text}**
- Generate realistic medical data for testing
- Verify properties hold across all generated inputs

### Unit Testing Focus Areas

**Query Screening**
- Test medical query classification accuracy
- Test non-medical query rejection
- Test edge cases (ambiguous queries, domain-specific terms)
- Test consistency across user roles

**Data Isolation**
- Test that patients only see their own records
- Test that doctors only see their own uploaded documents
- Test cross-user data access prevention
- Test filtering by user ID

**RAG Retrieval**
- Test chunk retrieval accuracy
- Test relevance ranking
- Test multi-source retrieval for patients
- Test embedding generation and similarity search

**LLM Integration**
- Test API call formatting
- Test response validation
- Test error handling and retries
- Test rate limiting

**Voice Processing**
- Test speech-to-text accuracy with medical terminology
- Test text-to-speech output quality
- Test audio quality handling
- Test language support

**Encryption**
- Test encryption/decryption round trips
- Test key management
- Test field-level encryption
- Test data at rest and in transit

**Audit Logging**
- Test log entry creation
- Test log completeness
- Test immutability
- Test query performance on large logs

### Test Implementation Strategy

**Property Tests** (using property-based testing framework):
- Generate random users, queries, and medical data
- Verify properties hold across all generated inputs
- Use shrinking to find minimal failing examples
- Run with high iteration counts (100+)

**Unit Tests** (using standard testing framework):
- Test specific examples and edge cases
- Mock external dependencies (LLM APIs, databases)
- Verify error handling paths
- Test integration points between components

### Coverage Goals

- Minimum 80% code coverage for core services
- 100% coverage for security-critical paths (encryption, authentication, data isolation)
- All properties have corresponding property-based tests
- All error handling paths have unit tests
- All data models have validation tests


