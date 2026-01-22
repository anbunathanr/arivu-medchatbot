# Implementation Plan: Arivu Medical Assistant Platform

## Overview

This implementation plan breaks down the Arivu platform into discrete TypeScript development tasks. The architecture is built incrementally, starting with core infrastructure and shared services, then implementing role-specific features (doctor and patient interfaces), and finally integrating all components. Each task builds on previous work with no orphaned code.

## Tasks

- [ ] 1. Set up project structure and core infrastructure
  - Create TypeScript project with Node.js backend (Express.js) and frontend (React/TypeScript)
  - Set up database schema for users, health records, RAG chunks, conversations, and audit logs
  - Configure environment variables for LLM APIs, voice services, and database connections
  - Implement base error handling and logging infrastructure
  - Set up testing framework (Jest for unit tests, fast-check for property-based tests)
  - _Requirements: 21, 24, 27, 28_

- [ ] 2. Implement authentication and authorization service
  - Create user authentication with JWT tokens and refresh token rotation
  - Implement role-based access control (RBAC) for doctor and patient roles
  - Create credential verification for doctors (medical license validation)
  - Implement session management and logout with session data clearing
  - _Requirements: 24_

- [ ] 2.1 Write property tests for authentication
  - **Property 1: Medical Query Screening Consistency**
  - **Validates: Requirements 24**

- [ ] 3. Implement encryption and security service
  - Create AES-256 encryption for data at rest
  - Implement TLS 1.3 for data in transit
  - Create field-level encryption for PHI (Protected Health Information)
  - Implement secure key management and rotation
  - _Requirements: 24_

- [ ] 3.1 Write property tests for encryption
  - **Property 7: Encryption at Rest and in Transit**
  - **Validates: Requirements 24**

- [ ] 4. Implement audit logging service
  - Create immutable audit log storage
  - Log all data access events with user ID, timestamp, and action
  - Implement audit log querying and compliance reporting
  - _Requirements: 24, 27_

- [ ] 4.1 Write property tests for audit logging
  - **Property 8: Audit Trail Completeness**
  - **Validates: Requirements 24, 27**

- [ ] 5. Implement medical query screener service
  - Create LLM-based query classification (medical vs. non-medical)
  - Implement consistent classification logic for both doctor and patient roles
  - Add ambiguous query handling (err on side of allowing medical queries)
  - Implement rejection feedback messages
  - _Requirements: 5, 19, 22_

- [ ] 5.1 Write property tests for query screener
  - **Property 1: Medical Query Screening Consistency**
  - **Validates: Requirements 5, 19, 22**

- [ ] 6. Implement LLM integration service
  - Create abstraction layer for multiple LLM providers (OpenAI, Anthropic, etc.)
  - Implement rate limiting and quota management
  - Add retry logic with exponential backoff for API failures
  - Implement response validation for medical appropriateness
  - Create role-specific prompt engineering (doctor vs. patient language)
  - _Requirements: 21_

- [ ] 6.1 Write property tests for LLM integration
  - **Property 5: LLM Response Validation**
  - **Validates: Requirements 21**

- [ ] 7. Implement voice service (speech-to-text and text-to-speech)
  - Integrate medical-specialized speech-to-text API (Deepgram or AssemblyAI)
  - Integrate text-to-speech API (Google Cloud or Azure)
  - Implement audio quality handling and background noise filtering
  - Add role-appropriate voice and tone selection
  - _Requirements: 6, 20, 23_

- [ ] 7.1 Write property tests for voice service
  - **Property 6: Voice Input-Output Round Trip**
  - **Validates: Requirements 6, 20, 23**

- [ ] 8. Implement vector database and RAG infrastructure
  - Set up vector database (Pinecone, Weaviate, or Milvus) for embeddings
  - Create embedding generation using medical-domain models
  - Implement semantic similarity search
  - Create caching layer for frequently accessed embeddings
  - _Requirements: 25, 26, 28_

- [ ] 8.1 Write property tests for RAG infrastructure
  - **Property 3: Doctor RAG Context Retrieval**
  - **Property 4: Patient RAG Multi-Source Retrieval**
  - **Validates: Requirements 25, 26, 28**

- [ ] 9. Implement doctor RAG knowledge base
  - Create storage for doctor-uploaded PDF content
  - Implement PDF upload validation and text extraction
  - Create text chunking for RAG-suitable segments
  - Implement document metadata tracking and TTL management
  - _Requirements: 4, 25_

- [ ] 9.1 Write property tests for doctor RAG
  - **Property 9: PDF Upload and RAG Indexing Round Trip**
  - **Validates: Requirements 4, 25**

- [ ] 10. Implement patient RAG knowledge base
  - Create storage for patient's personal medical history
  - Implement integration with health records store
  - Create storage for doctor's notes with patient access
  - Implement symptom history storage and retrieval
  - _Requirements: 7, 26_

- [ ] 10.1 Write property tests for patient RAG
  - **Property 4: Patient RAG Multi-Source Retrieval**
  - **Validates: Requirements 7, 26**

- [ ] 11. Implement general medical knowledge base
  - Create centralized repository of medical information
  - Implement data sourcing from authoritative medical databases
  - Create version control for knowledge updates
  - Implement multi-language support
  - _Requirements: 8, 9, 10_

- [ ] 12. Implement data isolation and access control
  - Create user ID-based data filtering for all queries
  - Implement cross-user data leakage prevention
  - Create doctor-specific data access (only their uploaded documents)
  - Create patient-specific data access (only their health records)
  - _Requirements: 24, 27_

- [ ] 12.1 Write property tests for data isolation
  - **Property 2: Data Isolation Enforcement**
  - **Validates: Requirements 24, 27**

- [ ] 13. Implement doctor chat interface backend
  - Create chat endpoint accepting doctor queries
  - Integrate medical query screener
  - Implement doctor RAG context retrieval
  - Integrate LLM service with doctor-specific prompts
  - Implement response formatting with source citations
  - Create conversation history storage
  - _Requirements: 1, 25_

- [ ] 13.1 Write property tests for doctor chat
  - **Property 1: Medical Query Screening Consistency**
  - **Property 3: Doctor RAG Context Retrieval**
  - **Validates: Requirements 1, 25**

- [ ] 14. Implement doctor content generation service
  - Create infographic generation endpoint (JSON specification format)
  - Create mind map generation endpoint (hierarchical JSON)
  - Create slide presentation generation endpoint (structured content)
  - Implement LLM-based content specification generation
  - Create rendering service integration for final output
  - _Requirements: 2_

- [ ] 14.1 Write property tests for content generation
  - **Property 15: Content Generation Format Consistency**
  - **Validates: Requirements 2**

- [ ] 15. Implement doctor web search feature
  - Create web search endpoint accepting search queries
  - Integrate medical web search API
  - Implement result filtering for medical relevance
  - Create LLM-based result summarization
  - Implement source attribution and relevance ranking
  - _Requirements: 3_

- [ ] 15.1 Write property tests for web search
  - **Property 16: Web Search Result Attribution**
  - **Validates: Requirements 3**

- [ ] 16. Implement doctor PDF upload and processing
  - Create PDF upload endpoint with file validation
  - Implement text extraction from PDFs
  - Create text chunking for RAG
  - Implement embedding generation and storage
  - Create document metadata tracking
  - _Requirements: 4, 25_

- [ ] 16.1 Write property tests for PDF upload
  - **Property 9: PDF Upload and RAG Indexing Round Trip**
  - **Validates: Requirements 4, 25**

- [ ] 17. Implement doctor voice support
  - Integrate speech-to-text for doctor voice input
  - Create voice query processing through normal chat pipeline
  - Integrate text-to-speech for doctor voice output
  - Implement doctor-appropriate voice tone
  - _Requirements: 6, 23_

- [ ] 17.1 Write property tests for doctor voice
  - **Property 6: Voice Input-Output Round Trip**
  - **Validates: Requirements 6, 23**

- [ ] 18. Checkpoint - Ensure all doctor features tests pass
  - Ensure all doctor feature tests pass
  - Verify no data isolation issues
  - Ask the user if questions arise

- [ ] 19. Implement patient chat interface backend
  - Create chat endpoint accepting patient queries
  - Integrate medical query screener
  - Implement patient RAG context retrieval (multi-source)
  - Integrate LLM service with patient-friendly prompts
  - Implement response simplification for patient language
  - Create conversation history storage
  - _Requirements: 7, 26_

- [ ] 19.1 Write property tests for patient chat
  - **Property 1: Medical Query Screening Consistency**
  - **Property 4: Patient RAG Multi-Source Retrieval**
  - **Validates: Requirements 7, 26**

- [ ] 20. Implement patient symptom checker
  - Create symptom input endpoint with validation
  - Implement medical knowledge base querying for matching conditions
  - Create condition ranking by likelihood
  - Implement educational information retrieval
  - _Requirements: 8_

- [ ] 20.1 Write property tests for symptom checker
  - **Property 17: Symptom Checker Condition Matching**
  - **Validates: Requirements 8**

- [ ] 21. Implement patient medication information service
  - Create medication search endpoint
  - Implement medical knowledge base querying
  - Create interaction detection with patient's current medications
  - Implement disclaimer inclusion
  - _Requirements: 9_

- [ ] 21.1 Write property tests for medication information
  - **Property 18: Medication Information Completeness**
  - **Validates: Requirements 9**

- [ ] 22. Implement patient health condition education
  - Create condition search endpoint
  - Implement medical knowledge base querying
  - Create patient-friendly language conversion
  - Implement related condition linking
  - _Requirements: 10_

- [ ] 22.1 Write property tests for condition education
  - **Property 19: Health Condition Education Accuracy**
  - **Validates: Requirements 10**

- [ ] 23. Implement patient health records access
  - Create health records retrieval endpoint with data isolation
  - Implement record organization by type
  - Create filtering by date range, type, and keyword
  - Implement consultation display with notes and documents
  - _Requirements: 13, 27_

- [ ] 23.1 Write property tests for health records
  - **Property 2: Data Isolation Enforcement**
  - **Validates: Requirements 13, 27**

- [ ] 24. Implement patient symptom tracker
  - Create symptom logging endpoint
  - Implement symptom storage with severity and timestamp
  - Create symptom history retrieval with chronological ordering
  - Implement trend analysis
  - Create shareable symptom report generation
  - _Requirements: 14_

- [ ] 24.1 Write property tests for symptom tracker
  - **Property 10: Symptom Tracker Data Persistence**
  - **Validates: Requirements 14**

- [ ] 25. Implement patient medication reminders
  - Create medication profile management endpoint
  - Implement reminder schedule creation
  - Create notification triggering at scheduled times
  - Implement adherence event recording
  - Create compliance statistics calculation
  - _Requirements: 15_

- [ ] 25.1 Write property tests for medication reminders
  - **Property 11: Medication Reminder Scheduling**
  - **Validates: Requirements 15**

- [ ] 26. Implement patient doctor notes summary
  - Create clinical notes availability endpoint
  - Implement LLM-based summary generation
  - Create patient-friendly language conversion
  - Implement key findings highlighting
  - Create original notes display alongside summary
  - _Requirements: 16_

- [ ] 26.1 Write property tests for notes summary
  - **Property 12: Doctor Notes Summary Accuracy**
  - **Validates: Requirements 16**

- [ ] 27. Implement patient personalized health insights
  - Create health data pattern analysis
  - Implement LLM-based insight generation
  - Create supporting evidence retrieval from records
  - Implement follow-up action recommendations
  - Create preference recording for insight dismissal
  - _Requirements: 17_

- [ ] 27.1 Write property tests for health insights
  - **Property 13: Personalized Health Insights Generation**
  - **Validates: Requirements 17**

- [ ] 28. Implement patient appointment booking integration
  - Create appointment booking system integration
  - Implement available slot display
  - Create booking confirmation and storage
  - Implement confirmation notification
  - Create upcoming appointments display
  - _Requirements: 18_

- [ ] 28.1 Write property tests for appointment booking
  - **Property 14: Appointment Booking Integration**
  - **Validates: Requirements 18**

- [ ] 29. Implement patient voice support
  - Integrate speech-to-text for patient voice input
  - Create voice query processing through normal chat pipeline
  - Integrate text-to-speech for patient voice output
  - Implement patient-appropriate voice tone
  - _Requirements: 20, 23_

- [ ] 29.1 Write property tests for patient voice
  - **Property 6: Voice Input-Output Round Trip**
  - **Validates: Requirements 20, 23**

- [ ] 30. Checkpoint - Ensure all patient features tests pass
  - Ensure all patient feature tests pass
  - Verify no data isolation issues
  - Ask the user if questions arise

- [ ] 31. Implement appointment and follow-up questions handler
  - Create endpoint for appointment-related queries
  - Implement health records retrieval for follow-up information
  - Create appointment booking integration
  - _Requirements: 11_

- [ ] 32. Implement wellness advice service
  - Create wellness advice endpoint
  - Implement health profile and history retrieval
  - Create LLM-based personalized recommendations
  - Implement adherence tracking
  - _Requirements: 12_

- [ ] 33. Implement queue management and rate limiting
  - Create request queuing for concurrent queries
  - Implement LLM API rate limiting
  - Create queue management with priority handling
  - _Requirements: 28_

- [ ] 33.1 Write property tests for queue management
  - **Validates: Requirements 28**

- [ ] 34. Implement caching and performance optimization
  - Create embedding caching layer
  - Implement response caching for common queries
  - Create progressive loading for large responses
  - _Requirements: 28_

- [ ] 35. Implement data archival strategy
  - Create archival policy for old health records
  - Implement data retention management
  - Create archival retrieval for historical data
  - _Requirements: 28_

- [ ] 36. Implement frontend for doctor interface
  - Create React components for doctor chat
  - Implement content generation UI (infographic, mind map, slides)
  - Create web search interface
  - Implement PDF upload interface
  - Create voice mode toggle and audio controls
  - _Requirements: 1, 2, 3, 4, 6_

- [ ] 36.1 Write integration tests for doctor frontend
  - Test end-to-end doctor workflows
  - _Requirements: 1, 2, 3, 4, 6_

- [ ] 37. Implement frontend for patient interface
  - Create React components for patient chat
  - Implement symptom checker UI
  - Create health records display
  - Implement symptom tracker UI
  - Create medication reminders UI
  - Implement appointment booking UI
  - Create voice mode toggle and audio controls
  - _Requirements: 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 20_

- [ ] 37.1 Write integration tests for patient frontend
  - Test end-to-end patient workflows
  - _Requirements: 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 20_

- [ ] 38. Implement API documentation and OpenAPI spec
  - Create OpenAPI/Swagger documentation for all endpoints
  - Document authentication and authorization
  - Document error responses and status codes
  - _Requirements: 21_

- [ ] 39. Implement comprehensive integration tests
  - Test doctor-patient interaction flows
  - Test data isolation across multiple users
  - Test RAG retrieval with real medical data
  - Test voice input/output round trips
  - Test encryption/decryption workflows
  - _Requirements: 1, 7, 24, 25, 26, 27_

- [ ] 40. Final checkpoint - Ensure all tests pass
  - Ensure all unit tests pass
  - Ensure all property-based tests pass
  - Ensure all integration tests pass
  - Verify no data isolation issues
  - Ask the user if questions arise

## Notes

- All tasks are required for comprehensive implementation
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation
- Property tests validate universal correctness properties
- Unit tests validate specific examples and edge cases
- All code is written in TypeScript with Node.js backend and React frontend
- Data isolation is enforced at every layer (database queries, API endpoints, RAG retrieval)
- All sensitive data is encrypted at rest and in transit
- All data access is logged for audit purposes
