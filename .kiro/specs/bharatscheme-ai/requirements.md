# BharatScheme AI - Requirements Document

## 1. Introduction

BharatScheme AI is a multilingual, AI-powered digital assistant designed to democratize access to government welfare schemes across India. The system leverages Natural Language Processing (NLP), semantic similarity modeling, and structured eligibility reasoning to intelligently match citizens with welfare schemes they qualify for.

Unlike traditional government portals that require manual searching and interpretation of complex eligibility criteria, BharatScheme AI enables users to describe their situation in natural language (text or voice) and receive personalized scheme recommendations with clear guidance on application procedures.

## 2. Problem Statement

Citizens across India, particularly in rural and semi-urban areas, face significant barriers in accessing government welfare schemes:

- **Information Asymmetry**: Lack of awareness about available schemes and their eligibility criteria
- **Language Barriers**: Most government portals are primarily in English or Hindi, excluding regional language speakers
- **Digital Literacy Gap**: Complex navigation and form-filling requirements discourage low digital literacy users
- **Intermediary Dependency**: Citizens often rely on middlemen, leading to delays and potential exploitation
- **Eligibility Confusion**: Complex and technical eligibility criteria are difficult to interpret
- **Fragmented Information**: Schemes are scattered across multiple portals and departments

These barriers result in low scheme uptake, exclusion of eligible beneficiaries, and reduced impact of welfare programs.

## 3. Objectives

### Primary Objectives
- Simplify discovery and access to government welfare schemes for all Indian citizens
- Enable natural language interaction in multiple Indian languages
- Provide accurate, AI-powered eligibility matching based on user context
- Reduce dependency on intermediaries for scheme information

### Secondary Objectives
- Increase awareness and uptake of government welfare schemes
- Enhance digital inclusion across rural and semi-urban populations
- Provide transparent and explainable AI-driven recommendations
- Create a scalable platform that can accommodate both central and state schemes

## 4. Scope

### In Scope
- Multilingual natural language query processing (text and voice)
- AI-based extraction of user attributes from unstructured input
- Semantic eligibility matching against structured scheme criteria
- Personalized scheme recommendations with eligibility explanations
- Document checklist and application guidance
- Support for major Indian languages (Hindi, English, Marathi, Tamil, and extensible to others)
- Web and mobile-responsive interface
- Integration with government scheme databases
- Responsible AI practices with data privacy safeguards

### Out of Scope
- Final eligibility determination (system provides guidance only)
- Direct application submission or form filling
- Payment processing or benefit disbursement
- User authentication or profile management (stateless interaction)
- Real-time scheme database updates (periodic updates only)
- Legal advice or grievance redressal


## 5. Functional Requirements

### 5.1 User Interaction Layer

**FR-1.1**: The system shall provide a web-based interface accessible on desktop and mobile devices.

**FR-1.2**: The system shall support text-based natural language input for user queries.

**FR-1.3**: The system shall support voice-based input with speech-to-text conversion.

**FR-1.4**: The system shall allow users to select their preferred language from a list of supported regional languages.

**FR-1.5**: The interface shall be designed for low digital literacy users with large interactive elements, clear labels, and minimal complexity.

### 5.2 Language Processing Module

**FR-2.1**: The system shall automatically detect the language of user input.

**FR-2.2**: The system shall convert voice input to text in the detected language.

**FR-2.3**: The system shall normalize and preprocess text input for consistent processing.

**FR-2.4**: The system shall support at least 5 major Indian languages initially (Hindi, English, Marathi, Tamil, Telugu).

**FR-2.5**: The system shall be extensible to add support for additional regional languages.

### 5.3 NLP-Based Information Extraction

**FR-3.1**: The system shall extract structured attributes from unstructured user input using Named Entity Recognition (NER).

**FR-3.2**: The system shall identify and extract the following user attributes:
- Annual income
- Occupation/employment status
- Location (State, District)
- Social category (SC/ST/OBC/General)
- Landholding size (if applicable)
- Age and gender
- Family composition
- Disability status (if mentioned)

**FR-3.3**: The system shall classify user intent (eligibility check, document information, scheme details, application guidance).

**FR-3.4**: The system shall handle incomplete or ambiguous information by requesting clarification from the user.

### 5.4 AI-Based Semantic Eligibility Matching

**FR-4.1**: The system shall use semantic similarity techniques to compare user attributes with scheme eligibility criteria.

**FR-4.2**: The system shall rank schemes based on relevance scores derived from semantic matching.

**FR-4.3**: The system shall handle complex and multi-condition eligibility criteria.

**FR-4.4**: The system shall provide contextual interpretation of eligibility clauses beyond exact keyword matching.

**FR-4.5**: The system shall categorize eligibility status as:
- **Eligible**: User meets all stated criteria
- **Possibly Eligible**: User meets most criteria but some conditions need verification
- **Not Eligible**: User does not meet key criteria


### 5.5 Scheme Knowledge Base

**FR-5.1**: The system shall maintain a structured database of government welfare schemes containing:
- Scheme name and ID
- Administering department/ministry
- Eligibility criteria (structured format)
- Benefit description
- Required documents
- Application process and links
- Scheme category (agriculture, health, housing, education, etc.)
- Geographic applicability (national/state-specific)

**FR-5.2**: The system shall support periodic updates to the scheme database.

**FR-5.3**: The system shall include both central and state government schemes.

**FR-5.4**: The system shall maintain scheme metadata including last updated date and source.

### 5.6 Response Generation & Simplification

**FR-6.1**: The system shall generate personalized responses containing:
- List of eligible schemes ranked by relevance
- Eligibility status for each scheme
- Simplified benefit summary
- Required documents checklist
- Step-by-step application guidance
- Official application portal links

**FR-6.2**: The system shall provide explanations for why a user qualifies or does not qualify for a scheme.

**FR-6.3**: The system shall translate responses into the user's preferred language.

**FR-6.4**: The system shall use simple, non-technical language appropriate for low literacy users.

**FR-6.5**: The system shall provide visual indicators (icons, colors) to enhance understanding.

### 5.7 Responsible AI & Transparency

**FR-7.1**: The system shall not permanently store sensitive personal information provided by users.

**FR-7.2**: The system shall provide transparent explanations for eligibility determinations.

**FR-7.3**: The system shall display clear disclaimers that:
- AI provides guidance only, not final eligibility determination
- Users must verify eligibility on official portals
- Final decisions rest with government authorities

**FR-7.4**: The system shall log interactions for quality improvement while maintaining user privacy.

**FR-7.5**: The system shall allow users to provide feedback on recommendation accuracy.


## 6. Non-Functional Requirements

### 6.1 Performance

**NFR-1.1**: The system shall respond to user queries within 3 seconds under normal load.

**NFR-1.2**: The system shall support at least 1000 concurrent users.

**NFR-1.3**: Voice-to-text conversion shall complete within 2 seconds for inputs up to 30 seconds.

**NFR-1.4**: The system shall maintain 99% uptime during business hours (9 AM - 6 PM IST).

### 6.2 Scalability

**NFR-2.1**: The system architecture shall support horizontal scaling to accommodate growing user base.

**NFR-2.2**: The scheme database shall be designed to accommodate at least 500 schemes initially, with capacity to scale to 5000+ schemes.

**NFR-2.3**: The system shall support addition of new languages without major architectural changes.

### 6.3 Security & Privacy

**NFR-3.1**: All data transmission shall be encrypted using HTTPS/TLS.

**NFR-3.2**: User queries containing personal information shall not be stored beyond the session duration.

**NFR-3.3**: The system shall comply with Indian data protection regulations and guidelines.

**NFR-3.4**: Access to system logs and analytics shall be restricted and audited.

### 6.4 Usability

**NFR-4.1**: The interface shall be accessible on devices with screen sizes ranging from 320px to 1920px width.

**NFR-4.2**: The system shall follow WCAG 2.1 Level AA accessibility guidelines where feasible.

**NFR-4.3**: The interface shall work on low-bandwidth connections (minimum 2G network).

**NFR-4.4**: Text shall be readable with minimum font size of 14px on mobile devices.

### 6.5 Accuracy

**NFR-5.1**: NLP entity extraction shall achieve at least 85% accuracy on test datasets.

**NFR-5.2**: Semantic eligibility matching shall achieve at least 80% precision in identifying eligible schemes.

**NFR-5.3**: Language detection shall achieve at least 95% accuracy for supported languages.

### 6.6 Maintainability

**NFR-6.1**: The system shall use modular architecture to enable independent updates to components.

**NFR-6.2**: The scheme database shall support bulk updates via structured data imports.

**NFR-6.3**: Code shall follow established coding standards and include comprehensive documentation.


## 7. User Stories

### US-1: Rural Farmer Seeking Agricultural Schemes
**As a** farmer from rural Maharashtra with limited digital literacy  
**I want to** ask in Marathi about schemes for farmers with small landholdings  
**So that** I can discover schemes I'm eligible for without visiting government offices

**Acceptance Criteria:**
- User can input query in Marathi (text or voice)
- System extracts occupation (farmer), location (Maharashtra), and landholding information
- System returns relevant agricultural schemes with eligibility status
- Response is provided in Marathi with simple language

### US-2: Urban Worker Seeking Housing Assistance
**As an** urban worker earning ₹2 lakh annually  
**I want to** find housing schemes I qualify for  
**So that** I can apply for financial assistance to build a home

**Acceptance Criteria:**
- User can describe income and housing needs in natural language
- System identifies income-based housing schemes
- System provides document checklist and application links
- System explains why user qualifies for specific schemes

### US-3: SC/ST Student Seeking Education Schemes
**As a** SC category student from Tamil Nadu  
**I want to** discover scholarship and education schemes  
**So that** I can get financial support for my studies

**Acceptance Criteria:**
- User can specify category and education context
- System filters schemes by social category and education focus
- System provides schemes from both central and state governments
- Response includes benefit amounts and application deadlines

### US-4: Senior Citizen Seeking Pension Schemes
**As a** 65-year-old senior citizen with no regular income  
**I want to** find pension schemes I'm eligible for  
**So that** I can secure financial support in old age

**Acceptance Criteria:**
- System extracts age and income information
- System identifies age-based pension schemes
- System explains eligibility criteria in simple terms
- System provides step-by-step application guidance

### US-5: Woman Entrepreneur Seeking Business Schemes
**As a** woman entrepreneur starting a small business  
**I want to** find schemes that support women-led enterprises  
**So that** I can access loans and training programs

**Acceptance Criteria:**
- System recognizes gender and entrepreneurship context
- System identifies women-specific business schemes
- System provides information on loan amounts and interest rates
- System includes links to application portals


### US-6: User with Incomplete Information
**As a** user who doesn't know all eligibility criteria  
**I want to** get scheme recommendations based on partial information  
**So that** I can still discover relevant schemes

**Acceptance Criteria:**
- System processes queries with incomplete information
- System asks clarifying questions when needed
- System provides "possibly eligible" schemes with conditions
- System explains what additional information is needed

### US-7: Voice Input User
**As a** user with low literacy who prefers speaking  
**I want to** use voice to ask about schemes  
**So that** I don't have to type my query

**Acceptance Criteria:**
- System accepts voice input in regional languages
- Voice-to-text conversion is accurate
- System processes voice input same as text input
- User receives audio or text response based on preference

### US-8: Multi-Scheme Comparison
**As a** user eligible for multiple schemes  
**I want to** see schemes ranked by relevance  
**So that** I can prioritize which schemes to apply for first

**Acceptance Criteria:**
- System ranks schemes by relevance score
- System highlights most beneficial schemes
- System shows comparison of benefits across schemes
- User can view detailed information for each scheme

## 8. Constraints

### Technical Constraints
- System must work on low-end mobile devices with limited processing power
- Must support browsers from the last 3 years (Chrome, Firefox, Safari, Edge)
- Voice input requires microphone access and user permission
- NLP models must be optimized for inference speed vs. accuracy trade-off

### Operational Constraints
- Scheme database updates depend on official government data availability
- System cannot verify user-provided information in real-time
- No integration with government authentication systems (Aadhaar, etc.)
- Limited to schemes with publicly available eligibility criteria

### Regulatory Constraints
- Must comply with Indian IT Act and data protection regulations
- Cannot make legally binding eligibility determinations
- Must include disclaimers about guidance vs. official decisions
- Cannot collect or store sensitive personal data without consent

### Resource Constraints
- Initial deployment limited to 5 major Indian languages
- Scheme database initially covers central schemes and select state schemes
- Support team availability limited to business hours


## 9. Assumptions

- Users have access to internet-enabled devices (smartphone or computer)
- Government scheme data is available in structured or semi-structured format
- Users are willing to provide personal information for eligibility matching
- Official government portals remain accessible for final application submission
- Scheme eligibility criteria remain relatively stable between updates
- Users understand that the system provides guidance, not final decisions
- Regional language support can be achieved through translation APIs or models
- NLP models can be trained or fine-tuned on Indian language datasets
- Users have basic familiarity with digital interfaces (can click buttons, enter text)

## 10. Success Metrics

### User Adoption Metrics
- **Monthly Active Users**: Target 100,000 users within 6 months of launch
- **User Retention**: 40% of users return within 30 days
- **Geographic Reach**: Users from at least 20 states within first year
- **Language Distribution**: At least 30% of queries in regional languages

### System Performance Metrics
- **Query Success Rate**: 90% of queries result in at least one scheme recommendation
- **Response Time**: Average response time under 2 seconds
- **System Uptime**: 99% availability during business hours
- **Voice Input Adoption**: 25% of queries use voice input

### Accuracy Metrics
- **Eligibility Accuracy**: 85% precision in identifying eligible schemes (validated through user feedback)
- **Entity Extraction Accuracy**: 90% accuracy in extracting user attributes
- **User Satisfaction**: Average rating of 4/5 or higher
- **False Positive Rate**: Less than 15% of "eligible" recommendations are incorrect

### Impact Metrics
- **Scheme Applications**: 20% of users proceed to official portals (tracked via referral links)
- **Intermediary Reduction**: 50% of users report not needing intermediary help
- **Awareness Increase**: Users discover average of 3 schemes they weren't aware of
- **Time Savings**: Average 30 minutes saved per user compared to manual portal navigation

### Engagement Metrics
- **Average Session Duration**: 3-5 minutes per session
- **Queries per Session**: Average 2-3 queries per user session
- **Feedback Submission**: 10% of users provide feedback on recommendations
- **Repeat Usage**: 30% of users return for different scheme categories

---

**Document Version**: 1.0  
**Last Updated**: February 19, 2026  
**Status**: Draft
