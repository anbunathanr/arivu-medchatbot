# Requirements Document: Arivu Care Platform

## Introduction

Arivu Care is a production-grade medical AI platform designed to provide accessible healthcare assistance through AI-powered diagnostics, consultations, and health management tools. The platform leverages 100% free-tier and open-source technologies to deliver medical insights, symptom analysis, mental health assessments, and drug discovery tools while maintaining HIPAA-aware data handling practices.

## Glossary

- **Platform**: The Arivu Care medical AI system
- **User**: Any authenticated person using the platform (Patient, Researcher, Student, or Doctor)
- **Patient**: A user role focused on personal health management and consultations
- **Researcher**: A user role with access to drug discovery and synthetic data generation tools
- **Student**: A user role with educational access including slide generation, mind maps, and image generation
- **Doctor**: A user role with comprehensive access to diagnostic and research tools (excluding student-specific features)
- **Chatbot**: The AI-powered conversational interface using Groq API (llama-3.3-70b-versatile model)
- **RAG_System**: Retrieval-Augmented Generation system using Qdrant Cloud and LlamaIndex with gemini-embedding-001
- **Voice_Interface**: Cross-browser voice input/output system using Groq Whisper and browser TTS
- **Diagnosis_Engine**: AI-powered symptom analysis and disease prediction system with follow-up questions
- **Image_Analyzer**: Medical image analysis system using Hugging Face MedGemma multimodal model
- **Drug_Discovery_Tool**: SMILES-based molecular analysis and drug recommendation system
- **Mental_Health_Module**: Mood assessment and wellness recommendation system with multiple questionnaire types
- **Data_Generator**: Synthetic medical data generation tool for research purposes
- **Auth_System**: Supabase-based authentication and authorization system with Google OAuth
- **Session**: An authenticated user's active connection to the platform
- **Conversation_History**: Persistent record of user-chatbot interactions stored in Supabase
- **Medical_Disclaimer**: Legal notice about AI limitations and professional medical advice requirements
- **Web_Search_Engine**: Internet search capability for retrieving latest medical information
- **Content_Filter**: System to ensure queries are medical-related only
- **Slide_Generator**: AI-powered educational slide creation tool for students
- **Mind_Map_Generator**: Visual concept mapping tool for medical education
- **Image_Generator**: Pollinations.ai integration for generating medical illustrations

## Requirements

### Requirement 1: User Authentication and Authorization

**User Story:** As a user, I want to securely authenticate and access role-appropriate features, so that my data is protected and I can use relevant platform capabilities.

#### Acceptance Criteria

1. WHEN a user visits the platform, THE Auth_System SHALL display Google OAuth login options
2. WHEN a user successfully authenticates via Google OAuth, THE Auth_System SHALL create or retrieve their user profile
3. WHEN a user selects their role (Patient, Researcher, Student, Doctor), THE Auth_System SHALL assign appropriate permissions
4. WHILE a user is authenticated, THE Platform SHALL maintain their session for up to 24 hours
5. WHEN a user's session expires, THE Auth_System SHALL redirect them to the login page
6. WHEN a user logs out, THE Auth_System SHALL terminate their session and clear authentication tokens
7. WHERE a user has Patient role, THE Platform SHALL grant access to chatbot with web search, diagnosis, image analysis, and mental health features
8. WHERE a user has Researcher role, THE Platform SHALL grant access to chatbot with web search, drug discovery, and synthetic data generation features
9. WHERE a user has Student role, THE Platform SHALL grant access to chatbot with web search, slide generation, mind maps, image generation, and all educational features
10. WHERE a user has Doctor role, THE Platform SHALL grant access to chatbot with web search, diagnosis, image analysis, drug discovery, mental health features, and synthetic data generation (excluding slide generation and mind maps)

### Requirement 2: Medical AI Chatbot with Voice and Web Search

**User Story:** As a user, I want to have real-time conversations with a medical AI using text or voice across all browsers with web search capability, so that I can get immediate health-related information including latest medical data.

#### Acceptance Criteria

1. WHEN a user sends a message to the chatbot, THE Chatbot SHALL respond within 5 seconds using Groq API (llama-3.3-70b-versatile model)
2. IF Groq API is unavailable, THEN THE Chatbot SHALL fallback to Hugging Face API
3. WHEN the chatbot generates a response, THE RAG_System SHALL retrieve relevant medical context from Qdrant vector database using LlamaIndex with gemini-embedding-001
4. WHEN a user query requires current medical information, THE Chatbot SHALL perform web search to retrieve latest data
5. WHERE a user has Patient, Researcher, Student, or Doctor role, THE Chatbot SHALL provide web search functionality
6. WHEN a conversation occurs, THE Platform SHALL persist the conversation history to Supabase database
7. WHEN a user returns to the chatbot, THE Platform SHALL load their previous conversation history
8. WHEN the chatbot provides medical information, THE Platform SHALL display a medical disclaimer
9. WHILE processing a query, THE Chatbot SHALL maintain conversation context from previous messages
10. WHEN a user's query contains ambiguous symptoms, THE Chatbot SHALL ask clarifying follow-up questions
10. WHEN a user activates voice input within the chat interface, THE Voice_Interface SHALL capture audio and send it to Groq Whisper API for transcription
11. WHEN voice transcription is complete, THE Platform SHALL process the text as a chatbot message
12. WHEN the chatbot generates a response, THE Voice_Interface SHALL provide option to synthesize speech output using browser-native TTS (Web Speech API)
13. WHEN voice features are used, THE Platform SHALL work across Chrome, Firefox, Safari, and Edge browsers
14. IF Groq Whisper is unavailable, THEN THE Platform SHALL fallback to browser-native Speech Recognition where available
15. WHEN a user sends a non-medical query, THE Chatbot SHALL politely decline and redirect to medical topics only

### Requirement 3: Student Learning Features

**User Story:** As a student, I want access to educational tools including slide generation, mind maps, and visual learning aids, so that I can enhance my medical education.

#### Acceptance Criteria

1. WHERE a user has Student role, THE Platform SHALL enable slide generation, mind map creation, and image generation features
2. WHEN a student user requests slide generation, THE Platform SHALL generate educational slides based on medical topics using AI
3. WHEN a student requests a mind map, THE Platform SHALL create visual mind maps for medical concepts and relationships
4. WHEN a student requests image generation, THE Platform SHALL use Pollinations.ai (Flux model) to generate medical illustrations
5. WHEN a student accesses the chatbot, THE Platform SHALL provide web search functionality for latest medical information
6. WHEN educational content is generated, THE Platform SHALL provide export options (PDF, PNG, JSON)
7. WHEN a student accesses the chat interface, THE Platform SHALL provide voice support for hands-free learning
8. WHEN a student generates content, THE Platform SHALL save it to their learning history
9. WHERE a student has usage limits, THE Platform SHALL track and enforce daily generation quotas
10. WHERE a user has Doctor role, THE Platform SHALL NOT provide access to slide generation and mind map features


### Requirement 4: Symptom Diagnosis Flow

**User Story:** As a patient, I want to describe my symptoms and receive AI-powered disease predictions, so that I can understand potential health conditions and seek appropriate care.

#### Acceptance Criteria

1. WHEN a user initiates symptom diagnosis, THE Diagnosis_Engine SHALL present a multi-step symptom collection wizard
2. WHEN a user enters initial symptoms, THE Diagnosis_Engine SHALL generate relevant follow-up questions using AI
3. WHEN sufficient symptom data is collected, THE Diagnosis_Engine SHALL predict potential diseases with confidence scores
4. WHEN disease predictions are generated, THE Platform SHALL display them ranked by confidence score (0-100%)
5. WHEN a disease prediction is displayed, THE Diagnosis_Engine SHALL generate a recommended treatment plan
6. WHEN diagnosis results are shown, THE Platform SHALL display a prominent medical disclaimer advising professional consultation
7. WHEN a diagnosis is completed, THE Platform SHALL save the diagnosis report to the user's history
8. WHILE collecting symptoms, THE Diagnosis_Engine SHALL validate input for medical relevance

### Requirement 5: Medical Image Analysis

**User Story:** As a patient, I want to upload medical images for AI analysis, so that I can get preliminary insights about potential conditions.

#### Acceptance Criteria

1. WHEN a user uploads an image file (JPEG, PNG), THE Platform SHALL validate the file size is under 10MB
2. WHEN a valid image is uploaded, THE Image_Analyzer SHALL store it in Supabase Storage
3. WHEN an image is stored, THE Image_Analyzer SHALL send it to Hugging Face MedGemma API for analysis
4. WHEN MedGemma analyzes an image, THE Image_Analyzer SHALL return findings with confidence scores
5. WHERE the image contains text (medical reports), THE Image_Analyzer SHALL perform OCR extraction
6. WHEN analysis is complete, THE Platform SHALL display findings with visual annotations
7. WHEN image analysis results are shown, THE Platform SHALL display a medical disclaimer
8. WHEN analysis fails or confidence is low, THE Platform SHALL recommend professional medical review

### Requirement 6: Drug Discovery Tools

**User Story:** As a researcher, I want to analyze molecular structures and discover drug candidates, so that I can accelerate pharmaceutical research.

#### Acceptance Criteria

1. WHEN a researcher enters a SMILES formula, THE Drug_Discovery_Tool SHALL validate the molecular structure syntax
2. WHEN a valid SMILES formula is provided, THE Drug_Discovery_Tool SHALL predict drug properties (solubility, toxicity, bioavailability)
3. WHEN a disease name is entered, THE Drug_Discovery_Tool SHALL recommend potential drug candidates from a curated database
4. WHEN drug properties are predicted, THE Platform SHALL display molecular visualization using 2D/3D rendering
5. WHEN drug recommendations are generated, THE Drug_Discovery_Tool SHALL provide confidence scores and research references
6. WHEN a researcher saves a drug analysis, THE Platform SHALL store it in their research history
7. WHILE analyzing molecules, THE Drug_Discovery_Tool SHALL use AI models to predict binding affinity

### Requirement 7: Mental Health Assessment

**User Story:** As a user, I want to complete mental health assessments and receive wellness recommendations, so that I can monitor and improve my mental wellbeing.

#### Acceptance Criteria

1. WHEN a user selects a mental health assessment, THE Mental_Health_Module SHALL present role-appropriate questionnaires (Student, Parent, Corporate, Partner)
2. WHEN a user completes a questionnaire, THE Mental_Health_Module SHALL calculate stress level scores (0-100)
3. WHEN stress levels are calculated, THE Mental_Health_Module SHALL generate personalized wellness recommendations
4. WHEN recommendations are provided, THE Platform SHALL suggest coping strategies and resources
5. WHEN a user completes multiple assessments over time, THE Mental_Health_Module SHALL track progress and display trends
6. WHEN mental health results are shown, THE Platform SHALL display crisis resources and professional help information
7. WHILE completing assessments, THE Platform SHALL ensure user privacy and data confidentiality

### Requirement 8: Synthetic Medical Data Generation

**User Story:** As a researcher, I want to generate synthetic medical datasets, so that I can conduct research without compromising patient privacy.

#### Acceptance Criteria

1. WHEN a researcher defines a data schema (fields, types, constraints), THE Data_Generator SHALL validate the schema structure
2. WHEN a valid schema is provided, THE Data_Generator SHALL generate synthetic medical records matching the schema
3. WHERE a researcher provides sample data, THE Data_Generator SHALL expand it while maintaining statistical properties
4. WHEN synthetic data is generated, THE Platform SHALL allow export to CSV or JSON formats
5. WHEN generating data, THE Data_Generator SHALL ensure no real patient information is included
6. WHEN data generation is complete, THE Platform SHALL display summary statistics and data quality metrics
7. WHILE generating large datasets, THE Platform SHALL show progress indicators

### Requirement 9: Performance and Responsiveness

**User Story:** As a user, I want the platform to load quickly and respond promptly, so that I can efficiently access healthcare services.

#### Acceptance Criteria

1. WHEN a user navigates to any page, THE Platform SHALL load the page within 3 seconds on standard broadband connections
2. WHEN the chatbot processes a query, THE Platform SHALL display a response within 5 seconds
3. WHEN images are uploaded, THE Platform SHALL provide upload progress feedback
4. WHILE AI processing occurs, THE Platform SHALL display loading indicators
5. WHEN the platform is accessed on mobile devices, THE Platform SHALL render responsive layouts optimized for screen size
6. WHEN multiple users access the platform simultaneously, THE Platform SHALL maintain performance within acceptable limits

### Requirement 10: Security and Data Privacy

**User Story:** As a user, I want my medical data to be secure and private, so that my sensitive health information is protected.

#### Acceptance Criteria

1. WHEN user data is transmitted, THE Platform SHALL use HTTPS encryption for all communications
2. WHEN medical data is stored, THE Platform SHALL implement HIPAA-aware data handling practices
3. WHEN a user deletes their account, THE Platform SHALL remove all personal and medical data within 30 days
4. WHILE storing conversation history, THE Platform SHALL encrypt sensitive medical information at rest
5. WHEN accessing user data, THE Platform SHALL enforce role-based access controls
6. WHEN authentication tokens are issued, THE Auth_System SHALL use secure, time-limited JWT tokens
7. IF a security breach is detected, THEN THE Platform SHALL log the incident and notify administrators

### Requirement 11: Browser Compatibility and Accessibility

**User Story:** As a user, I want to access the platform on all modern browsers with accessibility support, so that I can use the system regardless of my browser choice or abilities.

#### Acceptance Criteria

1. WHEN the platform is accessed via Chrome, Firefox, Safari, or Edge browsers, THE Platform SHALL provide full functionality
2. WHEN voice features are used, THE Platform SHALL use Groq Whisper for input and browser-native TTS for output
3. IF Groq Whisper is unavailable, THEN THE Platform SHALL fallback to browser Speech Recognition where supported
4. WHEN a user navigates using keyboard only, THE Platform SHALL provide full keyboard accessibility
5. WHEN screen readers are used, THE Platform SHALL provide appropriate ARIA labels and semantic HTML
6. WHEN color contrast is insufficient, THE Platform SHALL meet WCAG 2.1 AA standards
7. WHEN forms are submitted with errors, THE Platform SHALL provide clear, accessible error messages
8. WHERE browser-specific features are unavailable, THE Platform SHALL provide graceful fallbacks

### Requirement 12: Medical Disclaimers and Legal Compliance

**User Story:** As a platform operator, I want to display appropriate medical disclaimers, so that users understand the AI's limitations and seek professional care when needed.

#### Acceptance Criteria

1. WHEN a user first accesses diagnostic features, THE Platform SHALL display a comprehensive medical disclaimer
2. WHEN AI-generated medical information is shown, THE Platform SHALL include inline disclaimers about AI limitations
3. WHEN diagnosis or analysis results are displayed, THE Platform SHALL recommend consulting licensed healthcare professionals
4. WHEN the platform provides health information, THE Platform SHALL clarify it is for informational purposes only
5. WHEN users interact with mental health features, THE Platform SHALL display crisis hotline information
6. WHEN the platform is used, THE Platform SHALL comply with applicable medical information regulations

### Requirement 13: Testing and Quality Assurance

**User Story:** As a developer, I want comprehensive test coverage, so that the platform is reliable and bugs are caught early.

#### Acceptance Criteria

1. WHEN code is written, THE Platform SHALL maintain unit test coverage above 60%
2. WHEN critical features are implemented, THE Platform SHALL include integration tests for end-to-end flows
3. WHEN AI APIs are integrated, THE Platform SHALL include fallback mechanisms tested through automated tests
4. WHEN UI components are created, THE Platform SHALL include component tests for user interactions
5. WHEN the platform is deployed, THE Platform SHALL pass all automated test suites

### Requirement 14: Content Filtering and Medical Focus

**User Story:** As a platform operator, I want to ensure AI responses are limited to medical topics, so that the platform maintains its healthcare focus and prevents misuse.

#### Acceptance Criteria

1. WHEN a user submits a query, THE Platform SHALL analyze the query for medical relevance
2. WHEN a query is non-medical (entertainment, politics, general knowledge), THE Platform SHALL politely decline and suggest medical topics
3. WHEN a query is medical-related (symptoms, treatment, diagnosis, healthcare), THE Platform SHALL process it normally
4. WHEN filtering content, THE Platform SHALL use keyword detection and AI classification
5. WHEN a user repeatedly submits non-medical queries, THE Platform SHALL display educational message about platform purpose
6. WHERE a query is ambiguous, THE Platform SHALL ask clarifying questions to determine medical relevance

### Requirement 15: Modern UI and Performance Optimization

**User Story:** As a user, I want a modern, fast-loading interface, so that I can efficiently access healthcare services without delays.

#### Acceptance Criteria

1. WHEN the platform loads, THE Platform SHALL display initial content within 1.5 seconds
2. WHEN navigating between pages, THE Platform SHALL use client-side routing for instant transitions
3. WHEN images or media load, THE Platform SHALL use lazy loading and progressive enhancement
4. WHEN the platform renders UI, THE Platform SHALL use modern design patterns with shadcn/ui and TailwindCSS 4
5. WHEN animations occur, THE Platform SHALL use GPU-accelerated CSS transforms
6. WHEN data is fetched, THE Platform SHALL implement optimistic UI updates and skeleton loaders
7. WHILE the platform operates, THE Platform SHALL maintain 60fps scrolling and interactions
8. WHEN the platform is accessed on mobile, THE Platform SHALL provide touch-optimized controls and gestures

### Requirement 16: Free-Tier Resource Management

**User Story:** As a platform operator, I want to ensure AI responses are limited to medical topics, so that the platform maintains its healthcare focus and prevents misuse.

#### Acceptance Criteria

1. WHEN a user submits a query, THE Platform SHALL analyze the query for medical relevance
2. WHEN a query is non-medical (entertainment, politics, general knowledge), THE Platform SHALL politely decline and suggest medical topics
3. WHEN a query is medical-related (symptoms, treatment, diagnosis, healthcare), THE Platform SHALL process it normally
4. WHEN filtering content, THE Platform SHALL use keyword detection and AI classification
5. WHEN a user repeatedly submits non-medical queries, THE Platform SHALL display educational message about platform purpose
6. WHERE a query is ambiguous, THE Platform SHALL ask clarifying questions to determine medical relevance

### Requirement 17: Modern UI and Performance Optimization

**User Story:** As a user, I want a modern, fast-loading interface, so that I can efficiently access healthcare services without delays.

#### Acceptance Criteria

1. WHEN the platform loads, THE Platform SHALL display initial content within 1.5 seconds
2. WHEN navigating between pages, THE Platform SHALL use client-side routing for instant transitions
3. WHEN images or media load, THE Platform SHALL use lazy loading and progressive enhancement
4. WHEN the platform renders UI, THE Platform SHALL use modern design patterns with shadcn/ui and TailwindCSS 4
5. WHEN animations occur, THE Platform SHALL use GPU-accelerated CSS transforms
6. WHEN data is fetched, THE Platform SHALL implement optimistic UI updates and skeleton loaders
7. WHILE the platform operates, THE Platform SHALL maintain 60fps scrolling and interactions
8. WHEN the platform is accessed on mobile, THE Platform SHALL provide touch-optimized controls and gestures

**User Story:** As a platform operator, I want to efficiently manage free-tier service limits, so that the platform remains operational without costs.

#### Acceptance Criteria

1. WHEN Groq API approaches daily limits (14,400 requests), THE Platform SHALL throttle requests or switch to Hugging Face API
2. WHEN Supabase storage approaches 1GB limit, THE Platform SHALL implement data retention policies
3. WHEN Qdrant vector database approaches 1GB limit, THE Platform SHALL optimize embeddings or archive old data
4. WHEN Supabase database approaches 500MB limit, THE Platform SHALL archive or compress historical data
5. WHILE monitoring resource usage, THE Platform SHALL log metrics for capacity planning
6. IF any service quota is exceeded, THEN THE Platform SHALL display user-friendly error messages

## Non-Functional Requirements

### Performance
- Initial page load: < 1.5 seconds on standard broadband
- Subsequent page transitions: < 300ms (client-side routing)
- API response times: < 5 seconds for chatbot queries
- Image upload processing: < 10 seconds for images under 10MB
- Concurrent users: Support at least 100-500 simultaneous users
- Frame rate: Maintain 60fps for animations and scrolling
- Web search integration: < 3 seconds for search results

### Security
- All data transmission via HTTPS/TLS 1.3
- HIPAA-aware data handling practices
- Secure authentication with OAuth 2.0
- Encrypted storage for sensitive medical data
- Role-based access control (RBAC)

### Accessibility
- WCAG 2.1 AA compliance
- Full keyboard navigation support
- Screen reader compatibility
- Minimum color contrast ratio 4.5:1
- Responsive design for mobile devices (320px - 1920px)

### Reliability
- 99% uptime during business hours
- Graceful degradation when services are unavailable
- Automatic fallback from Groq to Hugging Face API
- Error logging and monitoring

### Usability
- Intuitive navigation with clear information architecture
- Consistent UI patterns using shadcn/ui components and TailwindCSS 4
- Loading indicators for all async operations with skeleton loaders
- Clear error messages with actionable guidance
- Mobile-first responsive design with touch-optimized controls
- Modern, clean aesthetic with smooth animations
- Voice integration within chat interface (not separate feature)
- Content filtering with helpful redirection messages

### Maintainability
- TypeScript for type safety
- Component-based architecture
- Comprehensive documentation
- Unit test coverage > 60%
- Code linting and formatting standards

## Constraints and Assumptions

### Technology Stack
- Frontend: Next.js 15, React 19, TypeScript, TailwindCSS 4, shadcn/ui
- Backend: FastAPI, Python 3.12
- Database: Supabase PostgreSQL (500MB free tier)
- Auth: Supabase Auth with Google OAuth (50,000 MAU free)
- Storage: Supabase Storage (1GB free tier)
- Vector DB: Qdrant Cloud (1GB free tier)
- Embeddings: Google Gemini gemini-embedding-001
- Primary AI: Groq API with llama-3.3-70b-versatile model (14,400 req/day free)
- Backup AI: Hugging Face API with MedGemma (free tier)
- RAG Framework: LlamaIndex for document retrieval
- Voice Input: Groq Whisper API (free tier) with fallback to browser Speech Recognition
- Voice Output: Browser-native Web Speech API (TTS)
- Image Generation: Pollinations.ai with Flux model (free tier)
- Web Search: Integration for latest medical data retrieval

### Constraints
1. Must use only free-tier services (no paid subscriptions)
2. Groq API limited to 14,400 requests per day (shared between LLM and Whisper)
3. Supabase storage limited to 1GB
4. Supabase database limited to 500MB
5. Qdrant Cloud limited to 1GB vector storage
6. Voice features must work across Chrome, Firefox, Safari, and Edge browsers
7. Voice input uses Groq Whisper with fallback to browser Speech Recognition
8. Voice output uses browser-native TTS only (no third-party services)
9. Local development environment only (no cloud hosting initially)
10. Must comply with medical information regulations
11. Content must be filtered to medical topics only
12. No OpenAI API usage (use Groq and Hugging Face only)

### Assumptions
1. Users have modern browsers (Chrome, Firefox, Safari, or Edge) with JavaScript enabled
2. Users have stable internet connections (minimum 1 Mbps)
3. Medical AI models (Groq llama-3.3-70b-versatile, Hugging Face MedGemma) maintain acceptable accuracy
4. Free-tier service limits are sufficient for MVP user base (target 100-500 users)
5. Users understand AI limitations and will seek professional medical care when needed
6. Synthetic data generation does not require real patient data access
7. Groq Whisper provides accurate transcription for medical terminology
8. Browser-native TTS provides acceptable voice quality across all browsers
9. Pollinations.ai maintains free tier access for image generation
10. Content filtering effectively prevents non-medical query abuse
11. Cross-browser voice support is achievable with Groq Whisper and browser TTS

## Success Metrics

### User Engagement
- Daily active users (DAU): Target 100+ within 3 months, 500+ within 6 months
- Average session duration: > 5 minutes
- Chatbot conversations per user: > 3 per week
- Feature adoption rate: > 40% of users try diagnosis or image analysis
- Student feature usage: > 60% of students use slide/mind map generation
- Voice feature usage: > 30% of users try voice input/output

### Technical Performance
- Initial page load: < 1.5 seconds (95th percentile)
- Page transitions: < 300ms (95th percentile)
- API response time: < 5 seconds (95th percentile)
- Error rate: < 2% of all requests
- Test coverage: > 60%
- Content filtering accuracy: > 95%

### Quality Metrics
- User-reported bugs: < 5 critical bugs per month
- Accessibility compliance: 100% WCAG 2.1 AA
- Security incidents: 0 data breaches
- API fallback success rate: > 95%

### Resource Efficiency
- Groq API usage: < 80% of daily limit
- Supabase storage usage: < 80% of 1GB limit
- Database size: < 80% of 500MB limit
- Vector database usage: < 80% of 1GB limit

### User Satisfaction
- User feedback rating: > 4.0/5.0
- Feature usefulness rating: > 4.0/5.0
- Would recommend to others: > 70%
- Return user rate: > 50% within 7 days
