# ANALYZE Phase: Strategic Problem Definition

Transform ambiguity into actionable clarity through systematic analysis and strategic thinking.

> Part of the [ACE Method](./ACE-Method.md) framework. For CLI-specific analysis, see [CLI-ACE-Method.md](./CLI-ACE-Method.md).

## Phase Overview

The ANALYZE phase is the foundation of successful software development. This phase transforms vague requirements into precise technical specifications, identifies hidden complexities, and establishes clear success criteria.

### Key Deliverables
1. Problem Definition Document
2. Technical Requirements Specification
3. Risk Assessment Matrix
4. Solution Architecture Design
5. Success Metrics Framework

## 1. Problem Definition Framework

### 1.1 Problem Statement Template

```markdown
## Problem Statement

### Executive Summary
**Problem**: [One sentence that a non-technical stakeholder would understand]
**Impact**: [Quantified business/user impact]
**Urgency**: [Critical | High | Medium | Low]

### Detailed Analysis
**Current State**: 
- What exists today
- Pain points and inefficiencies
- Technical debt considerations

**Desired State**:
- Target outcomes
- Success indicators
- Non-functional requirements

**Gap Analysis**:
- Technical gaps
- Process gaps
- Resource gaps
```

### 1.2 Root Cause Analysis

Apply the Five Whys technique systematically:

```markdown
## Five Whys Analysis

**Observable Problem**: [Initial symptom]

1. **Why?** [First level cause]
   - Evidence: [Data/logs/metrics]
   - Impact: [Scope and severity]

2. **Why?** [Second level cause]
   - Evidence: [Deeper investigation]
   - Impact: [Cascading effects]

3. **Why?** [Third level cause]
   - Evidence: [System analysis]
   - Impact: [Architectural implications]

4. **Why?** [Fourth level cause]
   - Evidence: [Process examination]
   - Impact: [Organizational effects]

5. **Why?** [Root cause]
   - Evidence: [Fundamental issue]
   - Solution direction: [High-level approach]
```

### 1.3 Stakeholder Analysis

```markdown
## Stakeholder Map

### Primary Stakeholders

**End Users**
- Interest: High
- Influence: Medium
- Requirements: Usability, Performance
- Success Criteria: Task completion in minimal steps

**Product Owner**
- Interest: High
- Influence: High
- Requirements: Features, Timeline
- Success Criteria: Delivery on schedule

**Engineering**
- Interest: High
- Influence: High
- Requirements: Maintainability
- Success Criteria: Clean architecture

**Operations**
- Interest: Medium
- Influence: High
- Requirements: Observability
- Success Criteria: Low error rate

### Communication Plan
- Engineering team regular sync meetings
- Stakeholder progress updates as needed
- Executive briefings at key milestones
```

## 2. Technical Requirements Analysis

### 2.1 Functional Requirements

```markdown
## Functional Requirements Specification

### Core Features (MUST HAVE)
1. **Feature Name**: [Descriptive name]
   - **Description**: [What it does]
   - **Acceptance Criteria**: 
     - [ ] Criterion 1
     - [ ] Criterion 2
   - **Technical Notes**: [Implementation considerations]
   - **Dependencies**: [External systems/APIs]

### Enhanced Features (SHOULD HAVE)
[Similar structure]

### Future Features (NICE TO HAVE)
[Similar structure]
```

### 2.2 Non-Functional Requirements

```markdown
## Non-Functional Requirements

### Performance
- **Response Time**: Define percentile targets based on user expectations
- **Throughput**: Set targets based on expected load
- **Concurrency**: Plan for peak user capacity

### Scalability
- **Horizontal**: Design for elastic scaling based on demand
- **Vertical**: Plan resource limits for individual instances
- **Data Volume**: Plan for significant growth from initial deployment

### Reliability
- **Availability**: Define uptime targets based on business requirements
- **Recovery Time Objective (RTO)**: Define based on business criticality
- **Recovery Point Objective (RPO)**: Set according to data loss tolerance

### Security
- **Authentication**: OAuth 2.0 / SAML 2.0
- **Authorization**: Role-based (RBAC)
- **Encryption**: TLS 1.3 in transit, AES-256 at rest
- **Compliance**: SOC2, GDPR, HIPAA

### Usability
- **Accessibility**: WCAG 2.1 AA compliance
- **Browser Support**: Define supported browsers and versions
- **Mobile**: Responsive design, native app considerations
- **Localization**: Multi-language support (i18n)
```

## 3. Risk Assessment and Mitigation

### 3.1 Risk Matrix Template

```markdown
## Risk Assessment Matrix

### Technical Risks

**API Rate Limits**
- Probability: High
- Impact: High
- Score: 9
- Mitigation Strategy: Implement caching layer
- Owner: Backend Team

**Database Scaling**
- Probability: Medium
- Impact: High
- Score: 6
- Mitigation Strategy: Design for sharding
- Owner: DBA Team

**Third-party Service Outage**
- Probability: Low
- Impact: High
- Score: 3
- Mitigation Strategy: Circuit breaker pattern
- Owner: DevOps

### Business Risks
[Similar structure]

### Security Risks
[Similar structure]

### Risk Score Calculation
- Score = Probability (1-3) x Impact (1-3)
- Priority: 7-9 (Critical), 4-6 (High), 1-3 (Medium)
```

### 3.2 Contingency Planning

```markdown
## Contingency Plans

### Scenario 1: [Risk Description]
**Trigger**: [Conditions that activate this plan]
**Response**:
1. Immediate actions
2. Communication protocol
3. Rollback procedures
4. Recovery steps

**Success Metrics**: [How we know the issue is resolved]
```

## 4. Solution Architecture Design

### 4.1 High-Level Architecture

```markdown
## Architecture Overview

### System Context

The system architecture consists of three main components:
- **Users**: The primary interface layer where users interact with the system
- **System**: The core application logic and processing layer
- **External Services**: Third-party integrations and external dependencies

### Component Architecture
- **Presentation Layer**: React/Vue/Angular considerations
- **API Layer**: REST/GraphQL/gRPC decision
- **Business Logic**: Microservices/Monolith trade-offs
- **Data Layer**: SQL/NoSQL selection criteria
- **Infrastructure**: Cloud/On-premise/Hybrid approach
```

### 4.2 Technology Stack Analysis

```markdown
## Technology Selection

### Programming Languages

**Python**
- Use Case: Backend API
- Pros: Fast development, ML libraries
- Cons: Performance
- Decision: Selected

**Go**
- Use Case: Microservices
- Pros: Performance, concurrency
- Cons: Learning curve
- Decision: Consider

**JavaScript**
- Use Case: Frontend
- Pros: Universal, large ecosystem
- Cons: Type safety
- Decision: Selected

### Frameworks and Libraries
[Similar analysis structure]

### Data Storage
[Similar analysis structure]

### Infrastructure
[Similar analysis structure]
```

### 4.3 Data Flow Design

```markdown
## Data Flow Specification

### Input Sources
1. **User Interface**
   - Format: JSON
   - Validation: Frontend + Backend
   - Rate limiting: 100 req/min per user

2. **API Integration**
   - Format: REST/JSON
   - Authentication: API keys
   - Retry strategy: Exponential backoff

### Processing Pipeline
1. **Ingestion**: Message queue (Kafka/RabbitMQ)
2. **Validation**: Schema validation service
3. **Transformation**: Data processing workers
4. **Storage**: Primary database + cache
5. **Analytics**: Real-time streaming analytics

### Output Channels
[Detailed specifications]
```

## 5. Constraint Analysis

### 5.1 Technical Constraints

```markdown
## Technical Constraints

### Legacy System Integration
- **System**: [Name and version]
- **Constraints**: [API limitations, data formats]
- **Workarounds**: [Proposed solutions]

### Performance Constraints
- **CPU**: Document available processing resources
- **Memory**: Identify memory limitations
- **Network**: Note bandwidth constraints
- **Storage**: Define storage capacity limits

### Platform Constraints
- **Cloud Provider**: AWS/GCP/Azure specific
- **Deployment**: Kubernetes 1.20+
- **Database**: PostgreSQL 13+
```

### 5.2 Business Constraints

```markdown
## Business Constraints

### Project Milestones
- Define critical deadlines based on business requirements
- Establish key milestones for Alpha, Beta, and Production releases
- Align timelines with stakeholder expectations

### Budget
- **Development**: $[Amount]
- **Infrastructure**: $[Amount]/month
- **Maintenance**: $[Amount]/year

### Resources
- **Team Size**: [Number] engineers
- **Skill Requirements**: [Specific expertise needed]
- **External Dependencies**: [Vendor timelines]
```

## 6. Success Metrics Framework

### 6.1 Technical Metrics

```markdown
## Technical Success Metrics

### Performance KPIs
- **Response Time**: Set appropriate percentile targets
- **Error Rate**: Define acceptable error thresholds
- **Throughput**: Establish transaction per second goals

### Quality Metrics
- **Code Coverage**: Define appropriate coverage targets
- **Technical Debt**: Monitor and manage technical debt ratio
- **Security Vulnerabilities**: Establish vulnerability thresholds by severity

### Operational Metrics
- **Deployment Frequency**: Daily
- **Lead Time**: Optimize for rapid delivery
- **MTTR**: Minimize recovery time
```

### 6.2 Business Metrics

```markdown
## Business Success Metrics

### User Engagement
- **Daily Active Users**: [Target]
- **Session Duration**: [Target]
- **Feature Adoption**: [Percentage]

### Business Impact
- **Revenue Impact**: [Measurement]
- **Cost Reduction**: [Measurement]
- **Customer Satisfaction**: [NPS Score]

### Efficiency Gains
- **Process Time Reduction**: [Percentage]
- **Automation Rate**: [Percentage]
- **Error Reduction**: [Percentage]
```

## 7. Analysis Tools and Techniques

### 7.1 SWOT Analysis

```markdown
## SWOT Analysis

### Strengths
- Existing infrastructure
- Team expertise
- Market position

### Weaknesses
- Technical debt
- Resource constraints
- Knowledge gaps

### Opportunities
- Market trends
- Technology advances
- Partnership potential

### Threats
- Competition
- Technology obsolescence
- Regulatory changes
```

### 7.2 Impact/Effort Matrix

```markdown
## Feature Prioritization Matrix

### High Impact, Low Effort (Quick Wins)
1. Feature A: [Description]
2. Feature B: [Description]

### High Impact, High Effort (Major Projects)
1. Feature C: [Description]
2. Feature D: [Description]

### Low Impact, Low Effort (Fill-ins)
1. Feature E: [Description]

### Low Impact, High Effort (Avoid)
1. Feature F: [Description]
```

## 8. Documentation Standards

### 8.1 Architecture Decision Records (ADR)

```markdown
# ADR-001: [Decision Title]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Context
[What is the issue that we're seeing that is motivating this decision?]

## Decision
[What is the change that we're proposing and/or doing?]

## Consequences
[What becomes easier or more difficult to do because of this change?]

## Alternatives Considered
1. **Option A**: [Description]
   - Pros: [List]
   - Cons: [List]
2. **Option B**: [Description]
   - Pros: [List]
   - Cons: [List]
```

### 8.2 Technical Specification Template

```markdown
# Technical Specification: [Feature Name]

## Overview
[High-level description]

## Goals
1. [Specific goal 1]
2. [Specific goal 2]

## Non-Goals
1. [What this does NOT address]

## Design
[Detailed technical design]

## Implementation Plan
[Step-by-step approach]

## Testing Strategy
[How we validate success]

## Rollout Plan
[Deployment strategy]
```

## 9. Analysis Checklist

Before proceeding to the CREATE phase, ensure:

### Problem Understanding
- [ ] Problem clearly defined and quantified
- [ ] Root cause analysis completed
- [ ] All stakeholders identified and consulted
- [ ] Success criteria established

### Technical Analysis
- [ ] All requirements documented
- [ ] Constraints identified and addressed
- [ ] Risks assessed with mitigation plans
- [ ] Architecture design reviewed

### Solution Validation
- [ ] Technical feasibility confirmed
- [ ] Resource availability verified
- [ ] Timeline realistic and agreed
- [ ] Budget approved

### Documentation
- [ ] All templates completed
- [ ] Decisions recorded in ADRs
- [ ] Specifications peer-reviewed
- [ ] Sign-off obtained

## 10. Transition to CREATE Phase

### Handoff Checklist
- [ ] All analysis documentation complete
- [ ] Technical specifications approved
- [ ] Development environment prepared
- [ ] Team assignments made
- [ ] Sprint planning completed

### Key Artifacts for CREATE Phase
1. Technical Requirements Document
2. Solution Architecture Diagram
3. Risk Mitigation Plan
4. Success Metrics Dashboard
5. Implementation Timeline

### Next Steps
Proceed to [02 Create.md](./02 Create.md) for implementation guidance.

## Quick Reference Card - ANALYZE Phase

### Key Activities Checklist
- [ ] **Requirement Gathering**
  - [ ] Business objectives defined
  - [ ] Technical constraints identified
  - [ ] Success metrics established
- [ ] **Stakeholder Analysis**
  - [ ] Primary stakeholders mapped
  - [ ] User personas created
  - [ ] Communication plan set
- [ ] **Technical Assessment**
  - [ ] Architecture patterns selected
  - [ ] Technology stack chosen
  - [ ] Integration points mapped
- [ ] **Risk Analysis**
  - [ ] Technical risks identified
  - [ ] Business risks assessed
  - [ ] Contingency plans created
- [ ] **Project Planning**
  - [ ] Timeline established
  - [ ] Milestones defined
  - [ ] Resources allocated

### Key Deliverables
1. **Requirements Document** - Complete specification of needs
2. **Architecture Decision Records** - Key technical choices
3. **Risk Register** - Identified risks with mitigation plans
4. **Implementation Roadmap** - Phased delivery plan
5. **Success Metrics Dashboard** - How we measure success

### Critical Questions
- What problem are we solving?
- Who are our users?
- What are the constraints?
- What could go wrong?
- How do we measure success?

### Common Pitfalls to Avoid
- ❌ Jumping to implementation without clear requirements
- ❌ Ignoring non-functional requirements
- ❌ Underestimating complexity
- ❌ Not involving stakeholders early
- ❌ Skipping risk assessment

## Additional Resources
- [Resources & References](./ACE-Method.md#resources--references)
- [Implementation Guide for LLMs](./ACE-Method.md#implementation-guide-for-llms)

---

**Navigation**:
- Previous: [ACE-Method.md](./ACE-Method.md) or [CLI-ACE-Method.md](./CLI-ACE-Method.md)
- Current: **01 Analyze.md**
- Next: [02 Create.md](./02 Create.md)

**Remember**: The quality of analysis directly impacts the success of implementation. Time invested here saves multiples in the CREATE and EVALUATE phases.