# Email Service - ACE Method Example

## Phase 1: START - Define Your Project

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Project Identity
**What I'm Building:**
[A scalable email service platform that handles template-based transactional emails, marketing campaigns, with advanced queuing, delivery tracking, bounce handling, and analytics]

**The 5-Minute Pitch:**
[MailFlow Engine is an enterprise-grade email delivery platform that ensures your emails reach the inbox. It features a powerful template engine with personalization, intelligent send-time optimization, automatic bounce and complaint handling, real-time analytics, and multi-provider failover. Built with Node.js and Redis, it processes millions of emails daily while maintaining high deliverability rates and providing detailed insights into email performance.]

#### The Reality Check

**Who Will Use This:**
[SaaS platforms sending transactional emails, E-commerce sites with order notifications, Marketing teams running campaigns, Developers needing reliable email API, Enterprises requiring email compliance]

**The Problem It Solves:**
[Eliminates email delivery complexity, Ensures high inbox placement rates, Provides real-time delivery insights, Handles provider outages gracefully, Maintains sender reputation automatically]

**Why They'll Choose This:**
[99.9% delivery success rate, Multi-provider redundancy, Real-time delivery tracking, Automatic list hygiene, GDPR/CAN-SPAM compliant, Beautiful email templates]

#### Constraints & Resources

**Technical Constraints:**
[Must send 1M emails/hour peak, Support templates up to 2MB, Handle 10k concurrent SMTP connections, Process webhooks in real-time, Store 90 days of email history]

**Time Investment:**
[3 weeks for core sending engine, 2 weeks for template system, 2 weeks for analytics, 1 week for bounce handling, 1 week for provider integrations]

**Financial Reality:**
[SendGrid costs: $500/month for 1M emails, AWS SES: $100/month backup, Server infrastructure: $300/month, Redis cluster: $150/month, Analytics storage: $100/month, Total: ~$1150/month]

#### Scope Definition

**Core Features (MVP):**
- Template-based email sending
- Basic personalization
- Send queue with retry
- Delivery tracking
- Bounce handling
- Simple analytics API

**Future Expansions:**
- Visual template editor
- A/B testing capabilities
- Send-time optimization
- Advanced segmentation
- Email validation service
- Dedicated IP management

**Out of Scope:**
- Email client/inbox
- Calendar integration
- SMS messaging
- Push notifications
- Contact management CRM

---

## Phase 2: ANALYZE - Deep Requirements Discovery

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Problem Deep Dive

**The Core Challenge:**
[Building an email service that maintains high deliverability across providers, handles template rendering at scale, processes delivery events in real-time, manages sender reputation, and provides actionable analytics while complying with anti-spam regulations]

**Current Pain Points:**
[Emails landing in spam folders, No visibility into delivery failures, Template changes require developer time, Provider outages cause email loss, Bounce handling is manual and error-prone]

**Success Metrics:**
[Inbox placement rate > 95%, Email send latency < 2 seconds, Process 1M emails/hour, Real-time event processing < 1 second, 99.9% platform uptime]

#### User Journey Mapping

**Primary User Flow:**
[Developer integrates API → Creates email template → Triggers email send → System queues and validates → Renders template with data → Sends via best provider → Tracks delivery events → Updates analytics dashboard]

**Critical Touchpoints:**
[API integration simplicity, Template creation ease, Send request latency, Delivery success rate, Event webhook reliability, Analytics dashboard clarity]

**Failure Points:**
[Complex API authentication, Template syntax errors, Queue backup during surges, Provider rate limits hit, Webhook processing delays, Analytics queries timeout]

#### System Boundaries

**What We'll Build:**
- High-performance sending engine
- Template management system
- Multi-provider abstraction
- Queue and retry mechanism
- Event processing pipeline
- Analytics and reporting
- Bounce/complaint handling
- REST API and SDKs

**What We'll Integrate:**
- SendGrid as primary provider
- AWS SES as backup
- Mailgun for EU sending
- PostgreSQL for data
- Redis for queuing
- Elasticsearch for analytics

**What We Won't Touch:**
- Email composition UI
- Contact list management
- Landing page builders
- Survey functionality
- Social media integration

#### Success Criteria

**Launch Requirements:**
[Send 100k emails/hour tested, 95% inbox placement verified, Failover tested and working, Analytics processing real-time, API documentation complete]

**Quality Benchmarks:**
[Send latency p95 < 1 second, Template render < 50ms, Webhook processing < 500ms, Dashboard load < 2 seconds, API uptime > 99.9%]

**Adoption Targets:**
[10M emails sent month 1, 100 active API users, 50k templates created, 95% delivery success rate, 5 enterprise customers]

---

## Phase 3: CREATE - Implementation Problem Solving

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Current Context

**What I'm Building:**
[Implementing the intelligent queue system that handles email prioritization, provider selection based on performance, automatic failover when providers fail, and rate limit management across multiple providers]

**Where I'm Stuck:**
[The queue is getting backed up during high volume periods. Provider failover isn't working smoothly - emails get lost during provider switches. Also struggling with rate limit coordination across multiple providers with different limits.]

**What I've Tried:**
[Implemented priority queues in Redis, added circuit breakers for providers, tried round-robin provider selection, used provider webhooks for rate limits, attempted predictive rate limiting]

#### Code State

**Relevant Code:**
```javascript
// Current problematic queue implementation
class EmailQueue {
  constructor() {
    this.redis = new Redis.Cluster(redisNodes);
    this.providers = new Map();
    this.providerHealth = new Map();
  }

  async queueEmail(email, priority = 'normal') {
    const queueName = `email:queue:${priority}`;
    
    // This doesn't handle queue overflow
    await this.redis.lpush(queueName, JSON.stringify(email));
  }

  // Provider selection not intelligent
  async selectProvider(email) {
    const providers = Array.from(this.providers.keys());
    
    // Simple round-robin, ignores performance
    const index = await this.redis.incr('provider:index') % providers.length;
    return providers[index];
  }

  // Rate limiting per provider is broken
  async sendEmail(email) {
    const provider = await this.selectProvider(email);
    const rateLimiter = this.getRateLimiter(provider);
    
    // This doesn't account for provider-specific limits
    const canSend = await rateLimiter.checkLimit();
    if (!canSend) {
      // Re-queues at end, losing priority
      await this.queueEmail(email);
      return;
    }

    try {
      const result = await this.providers.get(provider).send(email);
      await this.recordSuccess(provider);
      return result;
    } catch (error) {
      await this.recordFailure(provider);
      
      // No intelligent retry with different provider
      throw error;
    }
  }
}

// Template rendering with performance issues
class TemplateEngine {
  constructor() {
    this.templates = new Map();
    this.compiled = new Map();
  }

  // No caching of compiled templates
  async renderTemplate(templateId, data) {
    const template = await this.loadTemplate(templateId);
    
    // Handlebars compilation is slow
    const compiled = Handlebars.compile(template.content);
    
    // No protection against template injection
    return compiled(data);
  }

  // Personalization slowing down rendering
  async personalizeContent(content, recipient) {
    // Too many database queries
    const preferences = await db.query(
      'SELECT * FROM user_preferences WHERE email = $1',
      [recipient.email]
    );
    
    const history = await db.query(
      'SELECT * FROM email_history WHERE recipient = $1 ORDER BY sent_at DESC LIMIT 10',
      [recipient.email]
    );
    
    // Complex personalization logic
    return this.applyPersonalization(content, {
      ...recipient,
      preferences: preferences.rows[0],
      history: history.rows
    });
  }
}

// Event processing with bottlenecks
class EventProcessor {
  async processWebhook(provider, events) {
    // No batching - processes one by one
    for (const event of events) {
      try {
        await this.processEvent(provider, event);
      } catch (error) {
        console.error('Event processing failed:', error);
        // Events are lost on error
      }
    }
  }

  async processEvent(provider, event) {
    const { messageId, eventType, timestamp } = event;
    
    // No deduplication
    await db.query(
      'INSERT INTO email_events (message_id, event_type, timestamp, raw_data) VALUES ($1, $2, $3, $4)',
      [messageId, eventType, timestamp, JSON.stringify(event)]
    );
    
    // Inefficient analytics update
    await this.updateAnalytics(messageId, eventType);
    
    // No real-time notifications
    if (eventType === 'bounce' || eventType === 'complaint') {
      await this.handleSuppression(event);
    }
  }
}
```

**Error Messages:**
["Queue overflow - Redis memory limit reached", "Provider rate limit exceeded", "Template compilation timeout", "Duplicate event processing detected", "Analytics query timeout after 30s"]

**What's Working:**
[Basic email sending functional, Template storage working well, Provider authentication solid, Simple analytics queries fast, Bounce detection accurate]

#### Environment Details

**Tech Stack:**
[Node.js 18, Bull queue for jobs, Redis Cluster 7, PostgreSQL 14, Handlebars templates, SendGrid/SES/Mailgun APIs, Express.js, TypeScript]

**Constraints:**
[Must handle 1M emails/hour, Template render under 100ms, Support 1MB email size, 90-day retention required, Real-time analytics needed]

**Dependencies:**
[Redis cluster 32GB RAM, PostgreSQL 1TB storage, 3 email providers configured, Elasticsearch cluster, Kubernetes for scaling, Prometheus monitoring]

#### Specific Questions

**Technical Challenges:**
[How to implement intelligent provider selection? Best approach for distributed rate limiting? Should we pre-render templates? How to handle provider webhook delays?]

**Design Decisions:**
[Queue per provider vs global queue? Where to store email content - Redis or S3? How to deduplicate webhook events? Should analytics be real-time or batch?]

**Implementation Approach:**
[Considering weighted provider selection, Evaluating token bucket for rate limiting, Thinking about template pre-compilation, Exploring event sourcing for analytics]

---

## Phase 4: EVALUATE - Testing & Quality Assurance

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Implementation Status

**What's Complete:**
- Core email sending API
- Template management system
- Basic queue implementation
- Provider integrations (3)
- Bounce handling
- Simple analytics
- REST API endpoints
- Basic SDKs (Node, Python)
- Webhook processing

**What's In Progress:**
- Queue optimization
- Intelligent routing
- Template caching
- Real-time analytics
- Advanced personalization

**What's Pending:**
- Visual template editor
- A/B testing
- Send-time optimization
- IP warming tools
- Advanced segmentation

#### The Reality Check

**Known Issues:**
[Queue backs up at 500k emails/hour, Provider failover loses 1-2% of emails, Template rendering slow with personalization, Analytics queries timeout on large datasets, Webhook events sometimes processed twice]

**Performance Bottlenecks:**
[Template compilation taking 200ms+, Database queries for personalization slow, Redis memory usage growing unbounded, Event processing not keeping up, Analytics aggregation queries too slow]

**Security Concerns:**
[API keys stored in plain text, No rate limiting on API endpoints, Template injection possible, Webhook endpoints not authenticated, PII not encrypted at rest]

#### Quality Assessment

**Test Coverage:**
[Unit tests: 80% coverage, Integration tests: 60% coverage, Load tests: 500k emails/hour max, Provider failover tests: Manual only, E2E tests: 40% coverage]

**Code Review Findings:**
[Queue logic overly complex, No circuit breaker pattern, Missing database indexes, Inconsistent error handling, Memory leaks in template engine]

**External Feedback:**
[Users want better template editor, Analytics dashboard too slow, Need more detailed bounce reasons, Want real-time delivery notifications, Missing email validation API]

#### The Launch Checklist

**Must-Fix Before Launch:**
- Fix queue scalability issues
- Implement proper failover
- Add API rate limiting
- Encrypt sensitive data
- Fix webhook authentication
- Optimize template rendering
- Add event deduplication
- Create operational dashboards

**Should-Fix If Time:**
- Add template caching
- Implement batch processing
- Create status page
- Add more providers
- Improve error messages
- Build admin UI
- Add health checks
- Create migrations

**Can Wait Until Post-Launch:**
- Visual editor
- A/B testing
- Advanced analytics
- IP management
- Email validation
- Mobile SDKs
- GraphQL API
- Marketplace

---

## Phase 5: COMMIT - Deployment & Operations

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Launch Readiness

**Deployment Plan:**
[Deploy to AWS EKS across 3 regions, Redis cluster with automatic failover, RDS PostgreSQL with read replicas, Elasticsearch for analytics, Multi-provider setup with automatic switching, CloudFront for webhook endpoints]

**Rollback Strategy:**
[Kubernetes rolling updates with history, Database migrations with down scripts, Redis snapshots every hour, Feature flags for risky changes, Provider configuration versioned]

**Success Metrics:**
[Send 1M emails/hour sustained, 99% delivery success rate, <2 second send latency, 99.9% API uptime, Zero email loss during deploys]

#### Deployment Details

**Infrastructure Checklist:**
- EKS clusters in 3 regions
- Redis cluster configured
- RDS with Multi-AZ
- Elasticsearch cluster
- Load balancers ready
- DNS with failover
- SSL certificates
- Monitoring setup
- Secrets management
- Backup strategies

**Configuration Management:**
[Kubernetes secrets for API keys, ConfigMaps for settings, Helm charts for deployment, ArgoCD for GitOps, Vault for provider credentials]

**Monitoring Setup:**
[Prometheus for metrics, Grafana dashboards, CloudWatch alarms, Email delivery dashboard, Provider health monitors, PagerDuty escalation]

#### Post-Launch Planning

**Day 1 Priorities:**
- Monitor send rates
- Check delivery rates
- Verify failover working
- Watch queue depths
- Monitor provider health
- Review bounce rates
- Check API latency
- Respond to issues

**Week 1 Goals:**
- Analyze sending patterns
- Optimize slow templates
- Fix any bugs found
- Create user guides
- Set up support
- Review provider costs
- Plan improvements
- Gather feedback

**Month 1 Targets:**
- 50M emails delivered
- 99% success rate
- 100 API integrations
- 10k templates created
- 5 enterprise clients
- 3 new providers added
- Mobile SDKs launched
- $10k MRR achieved

#### Retrospective

**What Went Well:**
[Multi-provider architecture worked perfectly, Template system flexible and powerful, Analytics provided valuable insights, API design praised by developers, Bounce handling prevented major issues]

**Lessons Learned:**
[Queue design crucial for scale, Provider rate limits more complex than expected, Template caching essential for performance, Real-time analytics harder than batch, Documentation saves support time]

**Team Victories:**
[Achieved 1.2M emails/hour capacity, Built resilient failover system, Created intuitive template language, Delivered real-time analytics dashboard, Maintained 99.95% uptime in testing]