# SwasthiQ - System Design Document

## 1. Executive Summary

SwasthiQ is a voice-first, multilingual AI assistant designed for healthcare education in India. The system provides accessible explanations of medical concepts and documents through natural language processing, speech recognition, and retrieval-augmented generation (RAG) capabilities.

**Key Design Principles:**
- Educational focus with clear safety boundaries
- Voice-first interaction in Indian languages
- Serverless, scalable architecture
- Privacy-preserving design
- Real-time performance optimization

## 2. High-Level Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Frontend      │    │   API Gateway    │    │   Backend       │
│   (React PWA)   │◄──►│   (AWS)          │◄──►│   (Lambda)      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   CDN           │    │   AI Services    │    │   Data Layer    │
│   (CloudFront)  │    │   (Transcribe,   │    │   (S3, Vector   │
│                 │    │   Polly, LLM)    │    │   DB, DynamoDB) │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

### 2.1 Architecture Layers

**Presentation Layer:**
- React.js Progressive Web App
- WebRTC for voice recording
- Responsive UI with accessibility features

**API Layer:**
- AWS API Gateway with REST endpoints
- Rate limiting and authentication
- Request/response transformation

**Business Logic Layer:**
- AWS Lambda functions (Node.js)
- Microservices architecture
- Event-driven processing

**Data Layer:**
- S3 for document and audio storage
- Vector database for RAG embeddings
- DynamoDB for session management

**AI/ML Layer:**
- AWS Transcribe for speech-to-text
- AWS Polly for text-to-speech
- Large Language Model integration
- Vector similarity search

## 3. Major System Components

### 3.1 Frontend Application (React PWA)

**Core Modules:**
```
src/
├── components/
│   ├── VoiceRecorder/
│   ├── DocumentUploader/
│   ├── ChatInterface/
│   └── LanguageSelector/
├── services/
│   ├── apiClient.js
│   ├── audioService.js
│   └── storageService.js
└── utils/
    ├── audioUtils.js
    └── validationUtils.js
```

**Key Features:**
- Real-time voice recording with visual feedback
- Drag-and-drop document upload
- Chat-style conversation interface
- Multi-language support
- Offline capability for basic features

### 3.2 API Gateway & Lambda Functions

**Core Lambda Functions:**

```
functions/
├── voiceProcessor/
│   ├── transcribeAudio.js
│   └── synthesizeSpeech.js
├── documentProcessor/
│   ├── uploadDocument.js
│   ├── extractText.js
│   └── generateSummary.js
├── questionAnswering/
│   ├── processQuery.js
│   ├── retrieveContext.js
│   └── generateResponse.js
└── utilities/
    ├── contentModeration.js
    └── sessionManager.js
```

**API Endpoints:**
- `POST /voice/transcribe` - Convert speech to text
- `POST /voice/synthesize` - Convert text to speech
- `POST /documents/upload` - Upload medical documents
- `GET /documents/{id}/summary` - Get document summary
- `POST /chat/query` - Process user questions
- `GET /chat/history` - Retrieve conversation history

### 3.3 Document Processing Pipeline

**Processing Flow:**
1. **Upload Handler** - Validates file type and size
2. **Text Extractor** - Extracts text from PDF/DOC files
3. **Content Parser** - Identifies medical terminology
4. **Summarizer** - Generates concise summaries
5. **Embedder** - Creates vector embeddings for RAG

**Implementation:**
```javascript
// Document processing pipeline
const processDocument = async (documentId) => {
  const text = await extractText(documentId);
  const summary = await generateSummary(text);
  const embeddings = await createEmbeddings(text);
  await storeInVectorDB(documentId, embeddings);
  return { summary, status: 'processed' };
};
```

### 3.4 RAG (Retrieval-Augmented Generation) System

**Components:**
- **Vector Database** - Stores medical knowledge embeddings
- **Retriever** - Finds relevant context for queries
- **Generator** - Produces grounded responses
- **Safety Filter** - Ensures educational boundaries

**RAG Pipeline:**
```javascript
const generateResponse = async (query, language) => {
  // 1. Convert query to embeddings
  const queryEmbedding = await embedQuery(query);
  
  // 2. Retrieve relevant context
  const context = await vectorDB.search(queryEmbedding, limit: 5);
  
  // 3. Apply safety filters
  const safeContext = await moderateContent(context);
  
  // 4. Generate response with LLM
  const response = await llm.generate({
    query,
    context: safeContext,
    language,
    template: 'educational_healthcare'
  });
  
  // 5. Add disclaimers
  return addHealthcareDisclaimers(response);
};
```

## 4. Data Flow Between Components

### 4.1 Voice Interaction Flow

```
User Voice Input
      ↓
WebRTC Recording
      ↓
Frontend Audio Processing
      ↓
API Gateway (/voice/transcribe)
      ↓
AWS Transcribe Service
      ↓
Lambda Function (processQuery)
      ↓
RAG Pipeline
      ↓
LLM Response Generation
      ↓
AWS Polly (TTS)
      ↓
Audio Response to Frontend
      ↓
User Hears Response
```

### 4.2 Document Processing Flow

```
Document Upload
      ↓
S3 Storage
      ↓
Lambda Trigger (processDocument)
      ↓
Text Extraction
      ↓
Medical Content Analysis
      ↓
Summary Generation
      ↓
Vector Embedding Creation
      ↓
Vector DB Storage
      ↓
Notification to Frontend
```

### 4.3 Question Answering Flow

```
User Query (Text/Voice)
      ↓
Query Processing & Validation
      ↓
Vector Similarity Search
      ↓
Context Retrieval
      ↓
Safety & Content Moderation
      ↓
LLM Response Generation
      ↓
Response Post-processing
      ↓
Disclaimer Addition
      ↓
Response Delivery (Text/Voice)
```

## 5. AI/RAG Pipeline Design

### 5.1 Knowledge Base Architecture

**Medical Knowledge Sources:**
- Curated medical textbooks and journals
- WHO health guidelines
- Government health resources
- Peer-reviewed medical literature

**Vector Database Schema:**
```javascript
{
  id: "doc_chunk_001",
  content: "Medical text chunk",
  embeddings: [0.1, 0.2, ...], // 1536-dimensional vector
  metadata: {
    source: "WHO Guidelines",
    topic: "diabetes",
    language: "en",
    confidence: 0.95,
    last_updated: "2024-01-15"
  }
}
```

### 5.2 Embedding Strategy

**Text Chunking:**
- 512-token chunks with 50-token overlap
- Semantic boundary preservation
- Medical terminology awareness

**Embedding Model:**
- OpenAI text-embedding-ada-002 or equivalent
- Fine-tuned on medical terminology
- Multilingual support for Indian languages

### 5.3 Retrieval Strategy

**Hybrid Search:**
```javascript
const retrieveContext = async (query, language) => {
  // Semantic search
  const semanticResults = await vectorDB.search(
    await embedQuery(query), 
    { limit: 10, threshold: 0.7 }
  );
  
  // Keyword search for medical terms
  const keywordResults = await elasticSearch.search(query, {
    fields: ['medical_terms', 'content'],
    language: language
  });
  
  // Combine and rank results
  return rankAndMerge(semanticResults, keywordResults);
};
```

### 5.4 Response Generation

**LLM Configuration:**
```javascript
const generateEducationalResponse = async (query, context, language) => {
  const prompt = `
    You are SwasthiQ, an educational healthcare assistant.
    
    STRICT GUIDELINES:
    - Provide educational information only
    - Never diagnose or recommend treatments
    - Always suggest consulting healthcare professionals
    - Use simple, understandable language
    - Include relevant disclaimers
    
    Context: ${context}
    User Question: ${query}
    Language: ${language}
    
    Response:
  `;
  
  return await llm.complete(prompt, {
    temperature: 0.3,
    max_tokens: 500,
    language: language
  });
};
```

## 6. Speech-to-Text and Text-to-Speech Integration

### 6.1 Speech-to-Text (AWS Transcribe)

**Configuration:**
```javascript
const transcribeConfig = {
  LanguageCode: 'hi-IN', // Hindi, English, regional languages
  MediaFormat: 'webm',
  MediaSampleRateHertz: 16000,
  Settings: {
    VocabularyName: 'medical-terms-vocabulary',
    ShowSpeakerLabels: false,
    MaxSpeakerLabels: 1
  }
};
```

**Real-time Processing:**
- WebSocket streaming for live transcription
- Custom vocabulary for medical terms
- Noise reduction and audio enhancement
- Confidence scoring and error handling

### 6.2 Text-to-Speech (AWS Polly)

**Voice Selection:**
```javascript
const pollyConfig = {
  VoiceId: getVoiceForLanguage(language), // Aditi (Hindi), Joanna (English)
  OutputFormat: 'mp3',
  SampleRate: '22050',
  TextType: 'text',
  Engine: 'neural' // Higher quality voices
};
```

**Language Support:**
- Hindi: Aditi (female), Raveena (female)
- English (Indian): Aditi, Raveena
- Regional languages as available
- SSML for pronunciation control

### 6.3 Audio Processing Pipeline

```javascript
const processAudioInteraction = async (audioBlob, language) => {
  // 1. Upload audio to S3
  const audioKey = await uploadAudio(audioBlob);
  
  // 2. Transcribe speech
  const transcription = await transcribeAudio(audioKey, language);
  
  // 3. Process query
  const response = await processQuery(transcription.text, language);
  
  // 4. Generate speech
  const audioResponse = await synthesizeSpeech(response.text, language);
  
  // 5. Return both text and audio
  return {
    transcription: transcription.text,
    response: response.text,
    audio: audioResponse.audioUrl
  };
};
```

## 7. Technology Choices and Justification

### 7.1 Frontend Technology Stack

**React.js + PWA:**
- **Justification:** Component-based architecture, large ecosystem, PWA capabilities for offline use
- **Alternatives Considered:** Vue.js, Angular
- **Decision Factors:** Team expertise, community support, mobile-first design

**WebRTC for Audio:**
- **Justification:** Native browser support, real-time capabilities, no additional plugins
- **Alternatives Considered:** MediaRecorder API, third-party libraries
- **Decision Factors:** Cross-browser compatibility, performance

### 7.2 Backend Technology Stack

**AWS Lambda + Node.js:**
- **Justification:** Serverless scaling, cost-effective, JavaScript ecosystem
- **Alternatives Considered:** Express.js on EC2, Python Flask
- **Decision Factors:** Auto-scaling, pay-per-use, rapid development

**AWS API Gateway:**
- **Justification:** Integrated with Lambda, built-in rate limiting, monitoring
- **Alternatives Considered:** Application Load Balancer, Kong
- **Decision Factors:** Serverless integration, AWS ecosystem

### 7.3 AI/ML Services

**AWS Transcribe/Polly:**
- **Justification:** Native AWS integration, Indian language support, medical vocabulary
- **Alternatives Considered:** Google Speech API, Azure Cognitive Services
- **Decision Factors:** Regional language support, custom vocabulary, cost

**Vector Database (Pinecone/Weaviate):**
- **Justification:** Optimized for similarity search, managed service, scalability
- **Alternatives Considered:** Elasticsearch, FAISS, Chroma
- **Decision Factors:** Performance, managed infrastructure, API simplicity

### 7.4 Data Storage

**Amazon S3:**
- **Justification:** Scalable object storage, integrated with other AWS services
- **Use Cases:** Document storage, audio files, static assets

**DynamoDB:**
- **Justification:** Serverless NoSQL, fast performance, auto-scaling
- **Use Cases:** Session management, user preferences, conversation history

**Vector Database:**
- **Justification:** Specialized for embedding storage and similarity search
- **Use Cases:** Medical knowledge embeddings, RAG context retrieval

## 8. Security and Privacy Considerations

### 8.1 Data Protection

**Encryption:**
- TLS 1.3 for data in transit
- AES-256 encryption for data at rest
- Client-side encryption for sensitive uploads

**Data Minimization:**
```javascript
const sanitizeUserData = (data) => {
  // Remove PII from uploaded documents
  const sanitized = removePII(data);
  
  // Hash user identifiers
  const hashedId = hashUserId(data.userId);
  
  // Limit data retention
  scheduleDataDeletion(hashedId, '30days');
  
  return sanitized;
};
```

### 8.2 Access Control

**API Security:**
- JWT tokens for session management
- Rate limiting per user/IP
- Input validation and sanitization
- CORS policy enforcement

**AWS IAM Policies:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "transcribe:StartTranscriptionJob",
        "polly:SynthesizeSpeech",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::swasthiq-*/*"
    }
  ]
}
```

### 8.3 Content Safety

**Medical Content Moderation:**
```javascript
const moderateHealthQuery = async (query) => {
  // Check for diagnostic requests
  if (containsDiagnosticIntent(query)) {
    return {
      allowed: false,
      reason: 'diagnostic_request',
      response: 'I cannot provide medical diagnosis. Please consult a healthcare professional.'
    };
  }
  
  // Check for treatment advice requests
  if (containsTreatmentIntent(query)) {
    return {
      allowed: false,
      reason: 'treatment_request',
      response: 'I cannot recommend treatments. Please consult your doctor.'
    };
  }
  
  return { allowed: true };
};
```

### 8.4 Privacy Compliance

**Data Handling:**
- No storage of personally identifiable health information
- Session-based data with automatic expiration
- User consent management
- Right to data deletion

**Audit Logging:**
```javascript
const auditLog = {
  timestamp: new Date().toISOString(),
  userId: hashUserId(userId),
  action: 'query_processed',
  queryType: 'educational',
  language: 'hi-IN',
  responseGenerated: true,
  disclaimerShown: true
};
```

## 9. Performance Optimization

### 9.1 Latency Optimization

**Voice Processing:**
- Streaming transcription for real-time feedback
- Pre-warmed Lambda functions
- CDN for audio file delivery
- Optimized audio compression

**Response Generation:**
- Cached embeddings for common queries
- Parallel processing of retrieval and generation
- Response streaming for long answers

### 9.2 Scalability Design

**Auto-scaling Configuration:**
```javascript
const lambdaConfig = {
  reservedConcurrency: 100,
  provisionedConcurrency: 10,
  timeout: 30,
  memorySize: 1024,
  environment: {
    VECTOR_DB_URL: process.env.VECTOR_DB_URL,
    LLM_API_KEY: process.env.LLM_API_KEY
  }
};
```

**Database Optimization:**
- Read replicas for vector database
- Connection pooling for database access
- Caching layer with Redis/ElastiCache

### 9.3 Cost Optimization

**Resource Management:**
- Lambda function right-sizing
- S3 lifecycle policies for document cleanup
- Spot instances for batch processing
- Usage-based pricing monitoring

## 10. Monitoring and Observability

### 10.1 Application Monitoring

**Key Metrics:**
- Voice processing latency
- RAG retrieval accuracy
- User session duration
- Error rates by component

**Monitoring Stack:**
```javascript
const metrics = {
  transcriptionLatency: cloudWatch.metric('TranscriptionLatency'),
  ragAccuracy: cloudWatch.metric('RAGAccuracy'),
  userSatisfaction: cloudWatch.metric('UserSatisfaction'),
  systemErrors: cloudWatch.metric('SystemErrors')
};
```

### 10.2 Health Checks

**Service Health:**
- Lambda function health checks
- Vector database connectivity
- External API availability
- Audio processing pipeline status

### 10.3 Alerting

**Critical Alerts:**
- High error rates (>5%)
- Latency spikes (>5 seconds)
- Service unavailability
- Security incidents

## 11. Deployment Architecture

### 11.1 Infrastructure as Code

**AWS CDK Stack:**
```javascript
class SwasthiQStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);
    
    // S3 Buckets
    const documentBucket = new s3.Bucket(this, 'DocumentBucket');
    const audioBucket = new s3.Bucket(this, 'AudioBucket');
    
    // Lambda Functions
    const voiceProcessor = new lambda.Function(this, 'VoiceProcessor');
    const queryProcessor = new lambda.Function(this, 'QueryProcessor');
    
    // API Gateway
    const api = new apigateway.RestApi(this, 'SwasthiQAPI');
    
    // CloudFront Distribution
    const distribution = new cloudfront.Distribution(this, 'CDN');
  }
}
```

### 11.2 CI/CD Pipeline

**Deployment Stages:**
1. **Development** - Feature branches, unit tests
2. **Staging** - Integration tests, performance tests
3. **Production** - Blue-green deployment, rollback capability

**Pipeline Configuration:**
```yaml
stages:
  - test
  - build
  - deploy-staging
  - integration-tests
  - deploy-production

deploy-production:
  script:
    - cdk deploy --require-approval never
    - run-smoke-tests
  only:
    - main
```

## 12. Future Enhancements

### 12.1 Phase 2 Features

**Mobile Applications:**
- Native iOS/Android apps
- Offline voice processing
- Push notifications for health tips

**Advanced AI Capabilities:**
- Multi-modal input (image + voice + text)
- Personalized health education paths
- Integration with wearable devices

### 12.2 Technical Improvements

**Performance Enhancements:**
- Edge computing for voice processing
- Custom LLM fine-tuning for medical domain
- Advanced caching strategies

**Feature Additions:**
- Real-time language translation
- Health literacy assessment
- Integration with telemedicine platforms

## 13. Risk Mitigation

### 13.1 Technical Risks

**AI Model Accuracy:**
- Continuous model evaluation and retraining
- Human-in-the-loop validation for medical content
- A/B testing for response quality

**Service Dependencies:**
- Multi-cloud strategy for critical services
- Fallback mechanisms for AI services
- Circuit breakers for external APIs

### 13.2 Compliance Risks

**Healthcare Regulations:**
- Regular legal compliance reviews
- Medical professional content validation
- Clear user education on system limitations

**Data Privacy:**
- Privacy impact assessments
- Regular security audits
- Compliance monitoring automation

This system design provides a comprehensive foundation for building SwasthiQ as a scalable, secure, and user-friendly healthcare education platform while maintaining strict boundaries around medical advice and diagnosis.
