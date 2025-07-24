# API Builder - ACE Method Example

## Phase 1: START - Define Your Project

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Project Identity
**What I'm Building:**
[A RESTful API service that provides CRUD operations for a task management system with user authentication and role-based access control]

**The 5-Minute Pitch:**
[TaskFlow API is a modern RESTful service that enables developers to quickly integrate task management capabilities into their applications. It handles user authentication, task CRUD operations, team collaboration features, and provides webhook integrations for real-time updates. Built with Node.js and Express, it offers JWT-based auth, rate limiting, and comprehensive API documentation.]

#### The Reality Check

**Who Will Use This:**
[Frontend developers building task management apps, mobile app developers needing a backend, small teams wanting to integrate task tracking into existing systems]

**The Problem It Solves:**
[Eliminates the need to build task management backend from scratch, provides secure multi-tenant architecture, handles complex permission logic, offers standardized API patterns that developers already know]

**Why They'll Choose This:**
[Production-ready authentication included, well-documented with OpenAPI specs, includes rate limiting and security best practices, offers both REST and webhook integrations, scalable architecture supporting thousands of concurrent users]

#### Constraints & Resources

**Technical Constraints:**
[Must support Node.js 18+, PostgreSQL for data persistence, Redis for caching and sessions, Docker for containerization, 2GB RAM minimum for deployment]

**Time Investment:**
[4 weeks for MVP with core features, 2 additional weeks for advanced features like webhooks and batch operations]

**Financial Reality:**
[Cloud hosting: $50/month for small instances, Domain and SSL: $20/year, Monitoring tools: $30/month, Total monthly: ~$80 for production deployment]

#### Scope Definition

**Core Features (MVP):**
- User registration and JWT authentication
- Task CRUD operations (create, read, update, delete)
- User roles and permissions (admin, user, viewer)
- RESTful API endpoints following best practices
- Basic rate limiting and security headers

**Future Expansions:**
- Webhook notifications for task events
- Batch operations for bulk task management
- Advanced filtering and search capabilities
- API key authentication for service-to-service
- GraphQL endpoint alongside REST

**Out of Scope:**
- Frontend UI (API only)
- Mobile SDKs (REST API is sufficient)
- Real-time websocket connections
- File attachments for tasks
- Third-party integrations (Slack, etc.)

---

## Phase 2: ANALYZE - Deep Requirements Discovery

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Problem Deep Dive

**The Core Challenge:**
[Building a secure, scalable API that handles authentication, authorization, and data operations while maintaining performance under load and preventing common security vulnerabilities]

**Current Pain Points:**
[Developers waste time rebuilding auth systems, inconsistent API patterns across projects, difficulty implementing proper role-based access, complex database relationships for team features, lack of proper rate limiting causing abuse]

**Success Metrics:**
[API response time under 100ms for CRUD operations, Support 10,000 concurrent users, 99.9% uptime, Zero security breaches, Developer onboarding under 30 minutes with documentation]

#### User Journey Mapping

**Primary User Flow:**
[Developer signs up for API access → Reads documentation → Obtains API credentials → Makes first authenticated request → Creates tasks via API → Implements webhooks for updates → Monitors usage via dashboard]

**Critical Touchpoints:**
[API documentation clarity, Authentication flow simplicity, Error message helpfulness, Response time consistency, Rate limit transparency]

**Failure Points:**
[Confusing auth flow loses developers, Poor error messages cause debugging hell, Slow responses impact user experience, Unclear rate limits cause unexpected failures, Missing CORS headers block frontend apps]

#### System Boundaries

**What We'll Build:**
- RESTful API with standard HTTP methods
- JWT-based authentication system
- PostgreSQL schema with migrations
- Redis caching layer
- API documentation with Swagger/OpenAPI
- Rate limiting and security middleware
- Comprehensive error handling
- Health check and monitoring endpoints

**What We'll Integrate:**
- PostgreSQL for primary data storage
- Redis for session and cache management
- SendGrid for email notifications
- Sentry for error tracking
- DataDog for performance monitoring

**What We Won't Touch:**
- Frontend applications
- Mobile app development
- Payment processing
- File storage systems
- Complex workflow automation

#### Success Criteria

**Launch Requirements:**
[100% test coverage for auth endpoints, Load testing passing 1000 concurrent users, Security audit completed with no critical issues, API documentation reviewed by 3 developers, Deployment automation fully configured]

**Quality Benchmarks:**
[Response time p95 < 100ms, Error rate < 0.1%, Test coverage > 90%, Documentation completeness 100%, Security headers score A+]

**Adoption Targets:**
[10 developers using API in first month, 100 active API keys by month 3, 50,000 API calls daily by month 6, 5-star developer experience rating]

---

## Phase 3: CREATE - Implementation Problem Solving

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Current Context

**What I'm Building:**
[Implementing the task creation endpoint with proper validation, authorization checks, and database transactions to ensure data integrity]

**Where I'm Stuck:**
[The authorization middleware is not properly checking team permissions when creating tasks. Users can create tasks in teams they don't belong to. Also having issues with transaction rollback on validation errors.]

**What I've Tried:**
[Added basic role checking in middleware, tried using database triggers for validation, attempted manual transaction management, checked JWT token claims for team IDs]

#### Code State

**Relevant Code:**
```javascript
// Current problematic authorization middleware
const authorizeTeamAccess = async (req, res, next) => {
  const { teamId } = req.body;
  const userId = req.user.id;
  
  // This check is not working correctly
  const hasAccess = await db.query(
    'SELECT 1 FROM team_members WHERE team_id = $1 AND user_id = $2',
    [teamId, userId]
  );
  
  if (!hasAccess.rows.length) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  next();
};

// Task creation endpoint with transaction issues
app.post('/api/tasks', authenticate, authorizeTeamAccess, async (req, res) => {
  const client = await db.getClient();
  
  try {
    await client.query('BEGIN');
    
    const { title, description, teamId, assigneeId } = req.body;
    
    // Validation happens after transaction starts - problematic
    if (!title || title.length < 3) {
      throw new Error('Invalid title');
    }
    
    const result = await client.query(
      'INSERT INTO tasks (title, description, team_id, assignee_id, created_by) VALUES ($1, $2, $3, $4, $5) RETURNING *',
      [title, description, teamId, assigneeId, req.user.id]
    );
    
    await client.query('COMMIT');
    res.json(result.rows[0]);
    
  } catch (error) {
    await client.query('ROLLBACK');
    res.status(400).json({ error: error.message });
  } finally {
    client.release();
  }
});
```

**Error Messages:**
[No specific errors, but users report being able to create tasks in teams they shouldn't access. Also seeing "connection already released" errors occasionally]

**What's Working:**
[Basic authentication is solid, user registration/login flows work perfectly, database schema is well-designed, rate limiting is functioning correctly]

#### Environment Details

**Tech Stack:**
[Node.js 18.17, Express 4.18, PostgreSQL 14, Redis 7.0, node-postgres for DB, jsonwebtoken for JWT, express-rate-limit for rate limiting]

**Constraints:**
[Must maintain backwards compatibility with v1 API, Cannot modify existing database schema, Must support 1000 concurrent connections, Response time must stay under 100ms]

**Dependencies:**
[Database connection pool limited to 20 connections, Redis configured with 2GB memory limit, Running on AWS ECS with 4GB RAM per container]

#### Specific Questions

**Technical Challenges:**
[How to properly structure authorization checks for nested resources? Should team permission checks be in middleware or service layer? Best practice for transaction management with connection pools? How to handle partial failures in bulk operations?]

**Design Decisions:**
[Should we cache team membership in Redis? Is JWT token enrichment with team IDs a security risk? Should we use database views for complex permission queries? Is event sourcing overkill for audit logs?]

**Implementation Approach:**
[Considering moving authorization to service layer for better testability, thinking about using RLS (Row Level Security) in PostgreSQL, evaluating whether to implement CQRS pattern for read/write separation]

---

## Phase 4: EVALUATE - Testing & Quality Assurance

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Implementation Status

**What's Complete:**
- User authentication with JWT tokens
- Basic CRUD endpoints for tasks
- Role-based authorization (admin, user, viewer)
- PostgreSQL schema with migrations
- Redis caching for sessions
- Rate limiting on all endpoints
- OpenAPI documentation
- Docker containerization
- CI/CD pipeline with GitHub Actions

**What's In Progress:**
- Advanced team permission system
- Webhook notification system
- Batch operations for bulk updates
- Performance optimization for large datasets
- Comprehensive integration tests

**What's Pending:**
- API versioning strategy
- Advanced search and filtering
- Audit logging system
- Horizontal scaling setup
- Production monitoring dashboards

#### The Reality Check

**Known Issues:**
[Team authorization has edge cases with nested teams, Bulk operations timeout with >1000 items, Database connection pool exhaustion under heavy load, Webhook delivery lacks retry mechanism, CORS configuration too permissive]

**Performance Bottlenecks:**
[Task listing endpoint slow with >10k tasks (takes 500ms), N+1 queries in team member resolution, Missing database indexes on foreign keys, Redis cache invalidation causing stampedes, JWT validation adding 20ms to every request]

**Security Concerns:**
[Rate limiting can be bypassed with distributed IPs, SQL injection possible in search endpoint, Sensitive data in error messages, Missing security headers in some responses, API keys stored in plain text]

#### Quality Assessment

**Test Coverage:**
[Unit tests: 85% coverage, Integration tests: 60% coverage, E2E tests: 40% coverage, Security tests: Penetration testing scheduled, Load tests: Completed up to 500 concurrent users]

**Code Review Findings:**
[Inconsistent error handling patterns, Magic numbers throughout codebase, Missing input validation in 3 endpoints, Database queries not parameterized in search, Incomplete API documentation for edge cases]

**External Feedback:**
[Beta testers report confusing authentication flow, Documentation lacks real-world examples, Error messages not helpful for debugging, Missing SDK/client libraries, Response format inconsistent between endpoints]

#### The Launch Checklist

**Must-Fix Before Launch:**
- Fix team authorization security holes
- Add missing database indexes
- Implement proper connection pooling
- Complete security audit fixes
- Standardize error responses
- Add request ID tracking
- Implement webhook retries
- Fix N+1 query problems

**Should-Fix If Time:**
- Improve search performance
- Add more detailed logging
- Implement response caching
- Create client SDKs
- Add GraphQL endpoint
- Improve documentation examples

**Can Wait Until Post-Launch:**
- Advanced analytics dashboard
- Multi-region deployment
- Websocket support
- Third-party integrations
- API marketplace listing

---

## Phase 5: COMMIT - Deployment & Operations

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Launch Readiness

**Deployment Plan:**
[Deploy to AWS ECS with blue-green deployment, Use RDS PostgreSQL with multi-AZ setup, ElastiCache Redis cluster for high availability, CloudFront CDN for API responses, Route53 for DNS and health checks]

**Rollback Strategy:**
[Blue-green deployment enables instant rollback, Database migrations include down scripts, Feature flags for gradual rollout, Snapshot RDS before deployment, Keep previous 3 versions ready]

**Success Metrics:**
[API uptime > 99.9%, Response time p95 < 100ms, Zero data loss incidents, Successful authentication rate > 99%, API adoption by 50 developers in month 1]

#### Deployment Details

**Infrastructure Checklist:**
- AWS ECS cluster with auto-scaling
- RDS PostgreSQL with automated backups
- ElastiCache Redis with failover
- Application Load Balancer with health checks
- CloudWatch monitoring and alerts
- S3 bucket for documentation hosting
- Route53 DNS configuration
- SSL certificates configured
- Security groups properly restricted
- IAM roles with least privilege

**Configuration Management:**
[Environment variables in AWS Parameter Store, Secrets in AWS Secrets Manager, Infrastructure as Code with Terraform, Configuration validation on startup, Separate configs for staging/production]

**Monitoring Setup:**
[CloudWatch for infrastructure metrics, Sentry for error tracking, DataDog for APM and custom metrics, PagerDuty for critical alerts, Slack integration for team notifications]

#### Post-Launch Planning

**Day 1 Priorities:**
- Monitor error rates and response times
- Check authentication success rates
- Verify webhook deliveries
- Watch database connection pool
- Monitor rate limit effectiveness
- Check for security anomalies
- Respond to user feedback
- Update status page

**Week 1 Goals:**
- Gather developer feedback
- Fix critical bugs found
- Optimize slow queries identified
- Update documentation based on questions
- Plan first feature update
- Review security logs
- Analyze usage patterns
- Start planning v2 features

**Month 1 Targets:**
- Achieve 99.9% uptime
- Onboard 50 active developers
- Process 1M API requests
- Release 2 patch updates
- Complete SOC2 preparation
- Launch developer forum
- Publish client SDKs
- Plan scaling strategy

#### Retrospective

**What Went Well:**
[JWT authentication implementation was smooth, Database schema proved flexible enough, Docker containerization simplified deployment, CI/CD pipeline caught bugs early, Beta testers provided valuable feedback]

**Lessons Learned:**
[Should have implemented authorization service earlier, Underestimated time for documentation, Load testing should have started sooner, Team permissions more complex than anticipated, Need better local development setup]

**Team Victories:**
[Completed core API 1 week ahead of schedule, Security audit passed on first try, Performance exceeded target metrics, Zero downtime during beta testing, Great developer documentation feedback]