# DecisionLens AI Powered by NexVoid - System Design Document

## 1. System Architecture Overview

The DecisionLens AI follows a microservices architecture designed for trustworthy AI, explainability, and India-scale deployment. The system prioritizes human-in-the-loop decision support while maintaining high availability and security.

### Core Design Principles
- **Trustworthy AI**: Every AI decision includes confidence scores and explanations
- **Human-Centric**: AI augments human understanding rather than replacing human judgment
- **Fail-Safe**: System defaults to conservative interpretations when uncertain
- **Transparent**: All processing steps are auditable and explainable
- **Inclusive**: Designed for diverse linguistic and educational backgrounds

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    User Interface Layer                         │
├─────────────────────────────────────────────────────────────────┤
│  Web App  │  Mobile App  │  Voice Interface  │  API Gateway     │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                   Application Services Layer                    │
├─────────────────────────────────────────────────────────────────┤
│ Document    │ User Mgmt  │ Audit &    │ Analytics │             |
│ Processing  │ Service    │ Service    │ Complianc │ Service     │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                     AI/NLP Pipeline Layer                       │
├─────────────────────────────────────────────────────────────────┤
│ OCR Engine │ Language   │ Text         │ Intent     │ Risk      │
│            │ Detection  │ Simplifier   │ Extractor  │ Assessor  │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                      Data & Storage Layer                       │
├─────────────────────────────────────────────────────────────────┤
│ Document   │ User Data  │ Knowledge    │ Audit      │ Cache     │
│ Store      │ Store      │ Base         │ Logs       │ Layer     │
└─────────────────────────────────────────────────────────────────┘
```

## 2. Component Diagram Description

### Frontend Components
- **Web Application**: React-based responsive interface with accessibility features
- **Mobile Application**: Progressive Web App (PWA) with offline capabilities
- **Voice Interface**: Speech-to-text integration for document dictation
- **Admin Dashboard**: Human reviewer interface for quality assurance

### Backend Services
- **API Gateway**: Request routing, authentication, rate limiting, and API versioning
- **Document Processing Service**: Orchestrates the entire document analysis pipeline
- **User Management Service**: Authentication, authorization, and user preferences
- **Notification Service**: Multi-channel alerts (SMS, email, push notifications)
- **Audit Service**: Compliance logging and decision traceability
- **Analytics Service**: Usage patterns, accuracy metrics, and performance monitoring

### AI/NLP Components
- **OCR Engine**: Multi-language optical character recognition with confidence scoring
- **Language Detection**: Automatic identification of document language and script
- **Text Simplification Engine**: Legal-to-plain language transformation
- **Intent Extraction**: Identification of required actions and obligations
- **Risk Assessment**: Consequence analysis and urgency classification
- **Explanation Generator**: Human-readable justifications for AI decisions

### Data Layer
- **Document Store**: Encrypted storage for uploaded documents with auto-deletion
- **Knowledge Base**: Curated database of government procedures and legal frameworks
- **User Data Store**: Preferences, history, and personalization data
- **Audit Database**: Immutable logs for compliance and debugging
- **Cache Layer**: Redis-based caching for frequently accessed data

## 3. Data Flow

### End-to-End Processing Flow

```
User Upload → Document Validation → OCR Processing → Language Detection
     ↓
Content Extraction → Intent Analysis → Risk Assessment → Action Generation
     ↓
Human Review (if needed) → Explanation Generation → User Presentation
     ↓
User Feedback → Model Improvement → Audit Logging
```

### Detailed Data Flow Steps

1. **Document Ingestion**
   - User uploads document (PDF/image) or provides URL
   - System validates file format, size, and basic security checks
   - Document stored temporarily with encryption

2. **Content Extraction**
   - OCR engine processes images with confidence scoring
   - Text extraction with layout preservation
   - Quality assessment and error flagging

3. **Language & Context Detection**
   - Automatic language identification
   - Document type classification (tax notice, legal summons, etc.)
   - Sender authority verification

4. **AI Analysis Pipeline**
   - Legal text simplification with multiple complexity levels
   - Key information extraction (dates, amounts, obligations)
   - Risk and urgency assessment
   - Action recommendation generation

5. **Quality Assurance**
   - Confidence scoring for all AI outputs
   - Human review trigger for low-confidence results
   - Cross-validation with knowledge base

6. **User Presentation**
   - Multi-level explanations generation
   - Action plan creation with timelines
   - Risk communication with clear warnings

7. **Feedback Loop**
   - User satisfaction collection
   - Accuracy validation
   - Model retraining data preparation

## 4. AI/NLP Pipeline Design

### OCR & Document Ingestion

**Technology Stack**: Tesseract OCR + Custom Indian Language Models
- **Multi-script Support**: Devanagari, Tamil, Telugu, Bengali, and Latin scripts
- **Confidence Scoring**: Character-level and word-level confidence metrics
- **Layout Preservation**: Maintains document structure for context understanding
- **Quality Enhancement**: Image preprocessing for better OCR accuracy

**Implementation**:
```python
class OCRProcessor:
    def process_document(self, document):
        # Preprocess image for better OCR
        enhanced_image = self.enhance_image(document)
        
        # Multi-language OCR with confidence
        ocr_result = self.tesseract_ocr(enhanced_image, languages=['hin', 'eng', 'tam'])
        
        # Confidence-based quality assessment
        if ocr_result.confidence < 0.8:
            return self.flag_for_human_review(ocr_result)
        
        return ocr_result
```

### Language Detection

**Approach**: Hybrid statistical and neural language detection
- **Script Detection**: Unicode-based script identification
- **Language Classification**: FastText-based language detection
- **Code-Mixed Handling**: Detection of English-Hindi mixed content
- **Confidence Thresholds**: Minimum 90% confidence for automatic processing

### Legal/Official Text Simplification

**Strategy**: Multi-stage simplification with legal safety
- **Stage 1**: Complex sentence breaking and restructuring
- **Stage 2**: Legal terminology replacement with plain language
- **Stage 3**: Context-aware explanation generation
- **Safety Check**: Legal accuracy validation before simplification

**Prompt Engineering Example**:
```
System: You are a legal communication expert specializing in Indian administrative language. 
Your task is to simplify official notices while maintaining legal accuracy.

Rules:
1. Preserve all dates, amounts, and legal obligations exactly
2. Replace complex terms with simple explanations in parentheses
3. Break long sentences into shorter, clearer statements
4. Highlight action items clearly
5. If uncertain about legal implications, mark for human review

Input: [Original legal text]
Output: [Simplified version with confidence score]
```

### Intent and Obligation Extraction

**Named Entity Recognition (NER)**:
- **Custom Entities**: Deadlines, penalties, required actions, contact information
- **Legal Entities**: Court names, case numbers, legal sections
- **Financial Entities**: Amounts, tax codes, payment methods
- **Temporal Entities**: Dates, time periods, deadlines

**Intent Classification**:
- **Primary Intents**: Payment required, appearance needed, information update, compliance check
- **Urgency Levels**: Immediate (24h), urgent (7 days), normal (30 days), informational
- **Consequence Severity**: Critical, high, medium, low impact

### Risk & Consequence Identification

**Risk Assessment Framework**:
```python
class RiskAssessor:
    def assess_risk(self, document_analysis):
        risk_factors = {
            'deadline_proximity': self.calculate_time_urgency(document_analysis.deadlines),
            'penalty_severity': self.assess_penalties(document_analysis.consequences),
            'legal_implications': self.evaluate_legal_impact(document_analysis.legal_refs),
            'complexity_level': self.measure_complexity(document_analysis.requirements)
        }
        
        overall_risk = self.weighted_risk_calculation(risk_factors)
        explanation = self.generate_risk_explanation(risk_factors)
        
        return RiskAssessment(level=overall_risk, explanation=explanation)
```

**Risk Categories**:
- **Critical**: Legal action, asset seizure, license cancellation
- **High**: Significant penalties, compliance violations
- **Medium**: Minor penalties, administrative consequences
- **Low**: Informational, no immediate action required

### Action Recommendation Logic

**Decision Tree Approach**:
1. **Urgency Assessment**: Time-sensitive actions prioritized
2. **Complexity Evaluation**: Simple actions recommended first
3. **Resource Requirements**: Consider user capabilities and resources
4. **Alternative Paths**: Provide multiple approaches when possible

**Action Template Generation**:
```
Action: [Clear action description]
Deadline: [Specific date and time remaining]
Required Documents: [List of needed materials]
Where to Go: [Physical location or online portal]
Estimated Time: [Time required to complete]
Difficulty Level: [Easy/Medium/Hard]
Consequences of Inaction: [Clear warning]
```

## 5. Prompt Engineering Strategy

### Safety-First Prompt Design

**Core Principles**:
- **Conservative Interpretation**: When in doubt, recommend seeking professional help
- **Explicit Limitations**: Clear disclaimers about AI limitations
- **Verification Prompts**: Encourage users to verify critical information
- **Escalation Triggers**: Automatic referral to human experts for complex cases

**Prompt Templates**:

**Document Analysis Prompt**:
```
You are an AI assistant helping Indian citizens understand official documents. 
Your role is to explain, not to provide legal advice.

CRITICAL RULES:
1. Never guarantee legal outcomes or provide legal advice
2. Always recommend consulting professionals for complex matters
3. Preserve exact dates, amounts, and legal references
4. If unsure about any interpretation, state uncertainty clearly
5. Provide confidence scores for all major interpretations

Document Type: {document_type}
Content: {document_content}

Provide:
1. Simple explanation (6th grade level)
2. Key actions required with deadlines
3. Risk assessment with confidence score
4. Recommendation for professional consultation if needed
```

**Risk Communication Prompt**:
```
Explain the consequences of not taking action on this official notice.
Use clear, non-alarming language while being truthful about risks.

Format:
- What happens if I do nothing: [Clear explanation]
- How serious is this: [Risk level with reasoning]
- Time I have to act: [Specific timeline]
- My confidence in this assessment: [Percentage with explanation]
```

### Multi-Language Prompt Adaptation

**Language-Specific Considerations**:
- **Hindi**: Formal vs. informal register appropriate for government communications
- **Regional Languages**: Cultural context and local administrative practices
- **English**: Indian English conventions and legal terminology
- **Code-Mixed**: Handling of English-Hindi mixed official documents

## 6. Human-in-the-Loop & Confidence Scoring

### Confidence Scoring Framework

**Multi-Level Confidence Assessment**:
```python
class ConfidenceScorer:
    def calculate_confidence(self, analysis_result):
        scores = {
            'ocr_confidence': analysis_result.ocr_quality,
            'language_confidence': analysis_result.language_detection_score,
            'intent_confidence': analysis_result.intent_classification_score,
            'risk_confidence': analysis_result.risk_assessment_confidence,
            'action_confidence': analysis_result.action_recommendation_score
        }
        
        # Weighted average with minimum threshold
        overall_confidence = self.weighted_average(scores)
        
        # Flag for human review if below threshold
        if overall_confidence < 0.75:
            return self.flag_for_review(analysis_result, scores)
        
        return ConfidenceResult(score=overall_confidence, breakdown=scores)
```

### Human Review Integration

**Review Triggers**:
- Overall confidence score below 75%
- High-risk consequences identified
- Complex legal language detected
- User explicitly requests human verification
- New document types not seen before

**Review Workflow**:
1. **Automatic Flagging**: System identifies cases needing human review
2. **Expert Assignment**: Route to appropriate domain expert (legal, tax, education)
3. **Review Interface**: Streamlined dashboard for expert validation
4. **Feedback Integration**: Expert corrections fed back to improve AI models
5. **User Notification**: Clear communication about review status and timeline

**Expert Dashboard Features**:
- Side-by-side view of original document and AI interpretation
- Confidence score breakdown for each analysis component
- Quick correction tools for common errors
- Escalation options for complex cases requiring specialist knowledge

### Quality Assurance Loop

**Continuous Improvement Process**:
- **User Feedback Collection**: Simple thumbs up/down with optional comments
- **Outcome Tracking**: Follow-up on whether users successfully completed actions
- **Expert Validation**: Regular sampling of AI decisions for accuracy assessment
- **Model Retraining**: Monthly model updates based on collected feedback

## 7. Multilingual & Accessibility Design

### Language Support Architecture

**Hierarchical Language Processing**:
```
Primary Languages (Full Support):
├── Hindi (Devanagari script)
├── English (Indian context)
└── Regional Languages:
    ├── Tamil (Tamil script)
    ├── Telugu (Telugu script)
    ├── Bengali (Bengali script)
    ├── Marathi (Devanagari script)
    ├── Gujarati (Gujarati script)
    ├── Kannada (Kannada script)
    ├── Malayalam (Malayalam script)
    ├── Punjabi (Gurmukhi script)
    ├── Odia (Odia script)
    └── Assamese (Assamese script)
```

**Translation Strategy**:
- **Document-to-Document**: Maintain legal accuracy in translations
- **Context-Aware**: Preserve administrative and legal context
- **Cultural Adaptation**: Adjust explanations for regional administrative practices
- **Quality Assurance**: Native speaker validation for critical translations

### Accessibility Features

**Visual Accessibility**:
- **High Contrast Mode**: 4.5:1 minimum contrast ratio
- **Font Scaling**: 100% to 200% text size adjustment
- **Screen Reader Support**: ARIA labels and semantic HTML
- **Color Independence**: Information not conveyed through color alone

**Motor Accessibility**:
- **Keyboard Navigation**: Full functionality without mouse
- **Voice Input**: Speech-to-text for document content input
- **Touch Targets**: Minimum 44px touch targets for mobile
- **Gesture Alternatives**: Alternative input methods for complex gestures

**Cognitive Accessibility**:
- **Simple Language Options**: Multiple complexity levels
- **Clear Navigation**: Consistent layout and navigation patterns
- **Progress Indicators**: Clear steps and completion status
- **Error Prevention**: Confirmation dialogs for critical actions

**Audio Accessibility**:
- **Text-to-Speech**: Audio playback of explanations and instructions
- **Multiple Voices**: Gender and accent options for user preference
- **Speed Control**: Adjustable playback speed
- **Audio Descriptions**: Descriptions of visual elements

## 8. Security, Privacy & Compliance Considerations

### Data Protection Framework

**Encryption Standards**:
- **In Transit**: TLS 1.3 for all communications
- **At Rest**: AES-256 encryption for stored documents
- **Key Management**: Hardware Security Modules (HSM) for key storage
- **Database Encryption**: Transparent Data Encryption (TDE) for databases

**Privacy by Design**:
```python
class PrivacyController:
    def process_document(self, document, user_consent):
        # Minimal data extraction
        extracted_data = self.extract_minimal_required_data(document)
        
        # Automatic PII detection and masking
        sanitized_data = self.mask_personal_information(extracted_data)
        
        # Time-bound storage
        self.schedule_deletion(document.id, days=90)
        
        # Audit trail
        self.log_processing_activity(user_consent, extracted_data.metadata)
        
        return sanitized_data
```

**Compliance Requirements**:
- **Digital Personal Data Protection Act 2023**: Full compliance with Indian data protection laws
- **Right to Deletion**: User-initiated data deletion within 24 hours
- **Data Minimization**: Process only data necessary for service delivery
- **Consent Management**: Granular consent for different data processing activities

### Security Architecture

**Authentication & Authorization**:
- **Multi-Factor Authentication**: SMS OTP + biometric authentication
- **Role-Based Access Control**: User, reviewer, admin, and auditor roles
- **Session Management**: Secure session handling with automatic timeout
- **API Security**: OAuth 2.0 + JWT tokens for API access

**Threat Protection**:
- **Input Validation**: Comprehensive validation of all user inputs
- **Rate Limiting**: API rate limiting to prevent abuse
- **DDoS Protection**: CloudFlare-based DDoS mitigation
- **Malware Scanning**: All uploaded documents scanned for malware

**Audit & Monitoring**:
- **Comprehensive Logging**: All user actions and system decisions logged
- **Real-time Monitoring**: Security incident detection and alerting
- **Compliance Reporting**: Automated compliance reports for auditors
- **Incident Response**: Defined procedures for security incidents

## 9. Error Handling & Misinterpretation Safeguards

### Error Classification System

**Error Categories**:
1. **Technical Errors**: OCR failures, system downtime, processing errors
2. **Interpretation Errors**: Incorrect understanding of document content
3. **Translation Errors**: Inaccurate language translations
4. **Action Errors**: Incorrect or incomplete action recommendations

### Safeguard Mechanisms

**Multi-Layer Validation**:
```python
class SafeguardSystem:
    def validate_interpretation(self, document, interpretation):
        validations = [
            self.cross_reference_knowledge_base(interpretation),
            self.check_logical_consistency(interpretation),
            self.validate_temporal_constraints(interpretation.deadlines),
            self.verify_legal_accuracy(interpretation.legal_refs)
        ]
        
        confidence_penalty = sum(v.penalty for v in validations if not v.passed)
        
        if confidence_penalty > 0.3:
            return self.flag_for_human_review(interpretation, validations)
        
        return ValidationResult(passed=True, confidence_adjustment=confidence_penalty)
```

**Conservative Decision Making**:
- **Default to Safety**: When uncertain, recommend seeking professional help
- **Multiple Interpretations**: Present alternative interpretations when confidence is low
- **Escalation Triggers**: Automatic escalation for high-risk or complex cases
- **User Warnings**: Clear disclaimers about AI limitations and recommendation to verify

**Feedback-Driven Improvement**:
- **Error Reporting**: Easy mechanism for users to report incorrect interpretations
- **Expert Validation**: Regular review of flagged cases by domain experts
- **Model Updates**: Continuous improvement based on identified errors
- **Knowledge Base Updates**: Regular updates to reference knowledge base

### Graceful Degradation

**Service Continuity**:
- **Partial Processing**: Provide available information even if some components fail
- **Offline Capabilities**: Basic document viewing and note-taking when offline
- **Alternative Channels**: Phone and email support when system is unavailable
- **Status Communication**: Clear communication about system limitations and expected resolution times

## 10. Scalability & Deployment Strategy

### Cloud-Native Architecture

**Microservices Deployment**:
```yaml
# Kubernetes deployment example
apiVersion: apps/v1
kind: Deployment
metadata:
  name: document-processor
spec:
  replicas: 10
  selector:
    matchLabels:
      app: document-processor
  template:
    spec:
      containers:
      - name: processor
        image: citizen-platform/document-processor:v1.0
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        env:
        - name: AI_MODEL_ENDPOINT
          value: "https://ai-models.internal/v1"
```

**Auto-Scaling Strategy**:
- **Horizontal Pod Autoscaling**: Scale based on CPU and memory usage
- **Vertical Pod Autoscaling**: Adjust resource allocation based on demand
- **Predictive Scaling**: Scale proactively based on historical patterns
- **Regional Distribution**: Multi-region deployment for reduced latency

### Performance Optimization

**Caching Strategy**:
- **Redis Cluster**: Distributed caching for frequently accessed data
- **CDN Integration**: Static asset delivery through CloudFlare
- **Database Optimization**: Read replicas and query optimization
- **AI Model Caching**: Cache model predictions for similar documents

**Load Balancing**:
- **Application Load Balancer**: Intelligent request routing
- **Database Load Balancing**: Read/write splitting for database operations
- **AI Model Load Balancing**: Distribute AI processing across multiple instances
- **Geographic Load Balancing**: Route users to nearest data center

### Monitoring & Observability

**Comprehensive Monitoring Stack**:
```python
# Monitoring configuration
monitoring_config = {
    'metrics': {
        'application': ['response_time', 'error_rate', 'throughput'],
        'ai_models': ['accuracy', 'confidence_scores', 'processing_time'],
        'infrastructure': ['cpu_usage', 'memory_usage', 'disk_io']
    },
    'alerts': {
        'critical': ['system_down', 'data_breach', 'ai_accuracy_drop'],
        'warning': ['high_latency', 'resource_usage_spike', 'error_rate_increase']
    },
    'dashboards': ['user_experience', 'system_health', 'business_metrics']
}
```

**Key Performance Indicators**:
- **System Performance**: 99.5% uptime, <3s response time, <2% error rate
- **AI Performance**: >95% accuracy, >80% confidence scores, <30s processing time
- **User Experience**: >4.2/5 satisfaction, <10% abandonment rate, >60% task completion

## 11. Tech Stack

### Backend Infrastructure
- **Runtime**: Node.js 18+ with TypeScript for type safety
- **Framework**: Express.js with Helmet for security
- **Database**: PostgreSQL 14+ for structured data, MongoDB for document storage
- **Cache**: Redis Cluster for distributed caching
- **Message Queue**: Apache Kafka for event streaming
- **Search**: Elasticsearch for document search and analytics

### AI/ML Stack
- **ML Framework**: PyTorch for model development and training
- **Model Serving**: TorchServe for production model deployment
- **OCR Engine**: Tesseract 5.0+ with custom Indian language models
- **NLP Library**: spaCy with custom Indian language pipelines
- **Translation**: Custom transformer models fine-tuned for Indian languages
- **Vector Database**: Pinecone for semantic search and similarity matching

### Frontend Technologies
- **Web Framework**: React 18 with TypeScript
- **State Management**: Redux Toolkit for predictable state management
- **UI Components**: Material-UI with custom accessibility enhancements
- **Mobile**: Progressive Web App (PWA) with offline capabilities
- **Voice Interface**: Web Speech API with fallback to cloud services
- **Accessibility**: React-aria for comprehensive accessibility support

### DevOps & Infrastructure
- **Container Platform**: Docker with multi-stage builds
- **Orchestration**: Kubernetes with Helm charts
- **Cloud Provider**: AWS with multi-region deployment
- **CI/CD**: GitHub Actions with automated testing and deployment
- **Monitoring**: Prometheus + Grafana for metrics, ELK stack for logging
- **Security**: Vault for secrets management, Falco for runtime security

### Development Tools
- **API Documentation**: OpenAPI 3.0 with Swagger UI
- **Testing**: Jest for unit tests, Cypress for E2E testing
- **Code Quality**: ESLint, Prettier, SonarQube for code analysis
- **Documentation**: GitBook for user documentation, JSDoc for code documentation

## 12. Limitations & Future Enhancements

### Current Limitations

**Technical Constraints**:
- **Language Support**: Initial release supports 11 Indian languages
- **Complex Legal Cases**: Cannot handle highly specialized legal domains

**AI Model Limitations**:
- **Context Understanding**: May miss nuanced legal implications in complex cases
- **Regional Variations**: Administrative procedures may vary by state/district
- **Handwritten Text**: Limited accuracy for handwritten documents
- **Document Quality**: Performance degrades with poor quality scans

**Regulatory Constraints**:
- **Legal Advice Prohibition**: Cannot provide legal advice or guarantee outcomes
- **Jurisdiction Scope**: Initially focused on Indian legal and administrative framework
- **Professional Services**: Cannot replace qualified legal or professional consultation

### Future Enhancement Roadmap

**Phase 2 Enhancements (6-12 months)**:
- **Advanced OCR**: Handwritten text recognition with 85% accuracy
- **Voice Interface**: Complete voice-driven interaction in 5 major Indian languages
- **Mobile App**: Native iOS and Android applications with enhanced offline capabilities
- **Integration APIs**: Direct integration with government portals for status updates

**Phase 3 Enhancements (12-18 months)**:
- **Predictive Analytics**: Forecast potential compliance issues based on user patterns
- **Blockchain Verification**: Document authenticity verification using blockchain
- **Advanced AI**: GPT-4 level language understanding for complex legal reasoning
- **Professional Network**: Integration with verified lawyers and consultants

**Long-term Vision (18+ months)**:
- **International Expansion**: Support for documents from other countries
- **Legal Research**: Advanced legal research capabilities with case law analysis
- **Collaborative Platform**: Multi-user family/business account management
- **Government Integration**: Direct submission capabilities to government systems

### Research & Development Areas

**AI Research Priorities**:
- **Explainable AI**: Enhanced explanation generation for complex legal decisions
- **Few-shot Learning**: Rapid adaptation to new document types with minimal training data
- **Multimodal AI**: Integration of text, image, and audio processing for comprehensive understanding
- **Federated Learning**: Privacy-preserving model training across distributed user data

**User Experience Research**:
- **Accessibility Studies**: Comprehensive usability testing with diverse user groups
- **Cognitive Load Analysis**: Optimization of information presentation for better comprehension
- **Cultural Adaptation**: Region-specific customization of explanations and guidance
- **Digital Literacy Support**: Enhanced onboarding and education for less tech-savvy users

---