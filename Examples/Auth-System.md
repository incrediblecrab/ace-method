# Auth System - ACE Method Example

## Phase 1: START - Define Your Project

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Project Identity
**What I'm Building:**
[A comprehensive authentication and authorization system with user registration, secure login, JWT token management, OAuth integration, and multi-factor authentication support]

**The 5-Minute Pitch:**
[SecureAuth is a production-ready authentication system that handles the complete user identity lifecycle. It provides secure registration with email verification, password-less login options, JWT and refresh token management, OAuth2 integration with major providers, and optional MFA. Built with security-first principles, it includes rate limiting, account lockout protection, and comprehensive audit logging while maintaining GDPR compliance.]

#### The Reality Check

**Who Will Use This:**
[SaaS applications needing enterprise-grade auth, Startups wanting to avoid building auth from scratch, Mobile apps requiring secure token management, Enterprise applications needing SSO integration]

**The Problem It Solves:**
[Eliminates months of authentication development, Provides battle-tested security implementations, Handles complex token lifecycle management, Ensures compliance with security standards, Reduces risk of auth vulnerabilities]

**Why They'll Choose This:**
[OWASP-compliant security practices, Support for modern auth flows (PKCE, WebAuthn), Easy integration with 5-line setup, Extensive documentation and SDKs, Regular security updates and patches]

#### Constraints & Resources

**Technical Constraints:**
[Must support 10k+ concurrent users, JWT tokens with 15-minute expiry, Refresh tokens with secure rotation, PostgreSQL for user data, Redis for session management, Node.js 18+ runtime]

**Time Investment:**
[4 weeks for core auth system, 2 weeks for OAuth integrations, 2 weeks for MFA implementation, 1 week for SDK development, 1 week for security testing]

**Financial Reality:**
[Database hosting: $100/month, Redis cluster: $50/month, Email service: $30/month, SMS for MFA: $50/month, SSL certificates: $20/year, Total: ~$230/month]

#### Scope Definition

**Core Features (MVP):**
- User registration with email verification
- Secure password storage (Argon2)
- JWT-based authentication
- Refresh token rotation
- Password reset flow
- Basic rate limiting
- Session management

**Future Expansions:**
- OAuth2 providers (Google, GitHub, Microsoft)
- Multi-factor authentication (TOTP, SMS)
- WebAuthn/Passkeys support
- Enterprise SSO (SAML, LDAP)
- Advanced threat detection

**Out of Scope:**
- User profile management
- Payment integration
- CMS functionality
- Analytics dashboard
- Custom OAuth provider hosting

---

## Phase 2: ANALYZE - Deep Requirements Discovery

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Problem Deep Dive

**The Core Challenge:**
[Building a secure authentication system that prevents common attacks (credential stuffing, session hijacking, token theft) while maintaining excellent user experience and supporting modern authentication methods without sacrificing performance or scalability]

**Current Pain Points:**
[Developers spend weeks implementing basic auth, Security vulnerabilities from DIY implementations, Complex token management causes bugs, Poor user experience with traditional passwords, Difficulty adding enterprise features later]

**Success Metrics:**
[Authentication time under 200ms, Zero security breaches, 99.99% uptime for auth services, Support 1M monthly active users, Less than 0.1% failed login rate for legitimate users]

#### User Journey Mapping

**Primary User Flow:**
[User visits app → Clicks sign up → Enters email/password → Receives verification email → Clicks verify link → Logs in → Receives JWT + refresh token → Makes authenticated requests → Token auto-refreshes → Logs out securely]

**Critical Touchpoints:**
[Registration form simplicity, Email delivery speed, Password requirements clarity, Login error messaging, Token refresh transparency, Session timeout handling]

**Failure Points:**
[Email verification not received, Password requirements too complex, Login attempts blocked incorrectly, Token refresh fails silently, Session expires during critical action, MFA setup too complicated]

#### System Boundaries

**What We'll Build:**
- REST API for authentication endpoints
- JWT token generation and validation
- Refresh token rotation system
- Email verification service
- Password reset functionality
- Rate limiting and brute force protection
- Audit logging system
- Admin dashboard for user management

**What We'll Integrate:**
- PostgreSQL for user storage
- Redis for session/token blacklist
- SendGrid for email delivery
- Twilio for SMS MFA
- Google/GitHub OAuth providers
- Argon2 for password hashing

**What We Won't Touch:**
- User profile features
- Authorization rules engine
- Custom email templates
- Billing/subscription management
- Analytics tracking

#### Success Criteria

**Launch Requirements:**
[Security audit passed with no critical issues, Load tested to 10k concurrent users, 100% test coverage on auth flows, OWASP Top 10 compliance verified, Documentation complete with examples]

**Quality Benchmarks:**
[Login response time < 200ms, Token generation < 50ms, 99.99% availability SLA, Zero security incidents, Password hash time 100-200ms]

**Adoption Targets:**
[10 applications integrated in month 1, 100k user registrations by month 3, 1M authentications daily by month 6, 5 enterprise customers by end of year]

---

## Phase 3: CREATE - Implementation Problem Solving

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Current Context

**What I'm Building:**
[Implementing secure refresh token rotation to prevent token theft while maintaining seamless user experience across multiple devices and handling edge cases like network failures]

**Where I'm Stuck:**
[The refresh token rotation is causing users to get logged out unexpectedly when they have multiple tabs open. Also struggling with race conditions when multiple requests try to refresh simultaneously. The token family tracking is getting complex.]

**What I've Tried:**
[Implemented mutex locks for token refresh, tried token versioning system, attempted to detect token reuse, added grace period for old tokens, used Redis transactions for atomicity]

#### Code State

**Relevant Code:**
```javascript
// Current problematic refresh token implementation
class TokenService {
  async refreshTokens(refreshToken) {
    // Race condition: Multiple requests can execute simultaneously
    const tokenData = await this.validateRefreshToken(refreshToken);
    if (!tokenData) {
      throw new AuthError('Invalid refresh token');
    }

    // Check if token was already used (replay attack)
    const isUsed = await redis.get(`used_token:${refreshToken}`);
    if (isUsed) {
      // This is too aggressive - logs out all devices
      await this.revokeAllUserTokens(tokenData.userId);
      throw new AuthError('Token reuse detected');
    }

    // Mark token as used
    await redis.setex(`used_token:${refreshToken}`, 86400, '1');

    // Generate new tokens
    const newAccessToken = this.generateAccessToken(tokenData.userId);
    const newRefreshToken = this.generateRefreshToken(tokenData.userId);

    // Store new refresh token
    await redis.setex(
      `refresh_token:${newRefreshToken}`,
      604800, // 7 days
      JSON.stringify({ userId: tokenData.userId, family: tokenData.family })
    );

    // Problem: Old refresh token still valid for race conditions
    await redis.del(`refresh_token:${refreshToken}`);

    return { accessToken: newAccessToken, refreshToken: newRefreshToken };
  }

  // Multiple tabs issue
  async validateAccessToken(token) {
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      
      // This check is too strict for multiple tabs
      const isBlacklisted = await redis.get(`blacklist:${token}`);
      if (isBlacklisted) {
        throw new AuthError('Token blacklisted');
      }

      return decoded;
    } catch (error) {
      if (error.name === 'TokenExpiredError') {
        // Auto-refresh attempt causes issues
        throw new AuthError('Token expired', 'TOKEN_EXPIRED');
      }
      throw error;
    }
  }
}

// Frontend token refresh with race conditions
class AuthClient {
  async makeAuthenticatedRequest(url, options) {
    let response = await fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${this.accessToken}`
      }
    });

    if (response.status === 401) {
      // Multiple simultaneous 401s cause multiple refresh attempts
      await this.refreshTokens();
      
      // Retry with new token
      response = await fetch(url, {
        ...options,
        headers: {
          ...options.headers,
          'Authorization': `Bearer ${this.accessToken}`
        }
      });
    }

    return response;
  }

  async refreshTokens() {
    // No protection against concurrent refresh attempts
    const response = await fetch('/auth/refresh', {
      method: 'POST',
      body: JSON.stringify({ refreshToken: this.refreshToken })
    });

    if (!response.ok) {
      // This logs out user even for network errors
      this.logout();
      throw new Error('Token refresh failed');
    }

    const { accessToken, refreshToken } = await response.json();
    this.accessToken = accessToken;
    this.refreshToken = refreshToken;
    this.saveTokens();
  }
}
```

**Error Messages:**
["Token reuse detected" when user has multiple tabs, "Invalid refresh token" during normal usage, Race condition causes "Token not found" errors, Users report random logouts]

**What's Working:**
[Basic login/logout flow is solid, Password hashing with Argon2 working well, Email verification system reliable, Rate limiting preventing brute force, OAuth integration functioning correctly]

#### Environment Details

**Tech Stack:**
[Node.js 18, Express.js, PostgreSQL 14, Redis 7, JWT with RS256, Argon2id for passwords, Node-jsonwebtoken library, Express-rate-limit]

**Constraints:**
[Tokens must work across multiple devices, 15-minute access token lifetime required, Cannot use sessions for stateless architecture, Must support 10k concurrent users, Need to maintain GDPR compliance]

**Dependencies:**
[Redis cluster with 4GB RAM, PostgreSQL with connection pooling, SendGrid for transactional emails, Load balancer with sticky sessions disabled, Docker containers with horizontal scaling]

#### Specific Questions

**Technical Challenges:**
[How to handle refresh token rotation with multiple tabs? Best practice for grace period on old tokens? How to detect actual token theft vs legitimate reuse? Should we implement sliding sessions?]

**Design Decisions:**
[One refresh token per device or per session? How long should token families be tracked? Is Redis the right choice for token storage? Should we use JWT for refresh tokens too?]

**Implementation Approach:**
[Considering device fingerprinting for tokens, Evaluating token binding to prevent theft, Thinking about WebSocket for token updates, Exploring shared worker for tab coordination]

---

## Phase 4: EVALUATE - Testing & Quality Assurance

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Implementation Status

**What's Complete:**
- User registration with email verification
- Password hashing with Argon2id
- JWT access token generation
- Basic refresh token system
- Password reset flow
- Rate limiting on all endpoints
- OAuth integration (Google, GitHub)
- Audit logging framework
- Admin user management UI

**What's In Progress:**
- Refresh token rotation fixes
- Multi-factor authentication
- WebAuthn support
- Enterprise SSO integration
- Advanced threat detection

**What's Pending:**
- Account recovery options
- Passwordless login
- Device management UI
- Security alert system
- Compliance reporting

#### The Reality Check

**Known Issues:**
[Refresh token rotation causing logouts with multiple tabs, Race conditions in concurrent token refresh, Token blacklist growing unbounded in Redis, Email delivery delays during high load, OAuth state parameter not properly validated]

**Performance Bottlenecks:**
[Argon2 hashing taking 300ms on high load, Database connection pool exhaustion at 5k users, Redis memory usage growing rapidly, JWT validation adding 50ms latency, Audit log queries slow without indexes]

**Security Concerns:**
[Missing CSRF protection on some endpoints, Session fixation possible in OAuth flow, Timing attacks on user enumeration, Weak password policy enforcement, Insufficient rate limiting on password reset]

#### Quality Assessment

**Test Coverage:**
[Unit tests: 92% coverage, Integration tests: 78% coverage, Security tests: Penetration test scheduled, Load tests: Completed up to 5k users, E2E tests: 65% coverage]

**Code Review Findings:**
[Inconsistent error handling across modules, Sensitive data in log files, Missing input validation on 2 endpoints, Hard-coded configuration in OAuth module, Incomplete transaction handling in database]

**External Feedback:**
[Beta testers report confusion with MFA setup, Password requirements too restrictive, Email verification link expires too quickly, Error messages reveal too much information, Missing password strength indicator]

#### The Launch Checklist

**Must-Fix Before Launch:**
- Fix refresh token rotation race conditions
- Implement CSRF protection everywhere
- Add missing security headers
- Fix OAuth state validation
- Resolve session fixation vulnerability
- Index audit log tables
- Implement token blacklist cleanup
- Fix concurrent refresh bug

**Should-Fix If Time:**
- Optimize Argon2 parameters
- Implement request signing
- Add anomaly detection
- Improve error messages
- Create SDK libraries
- Add password strength meter
- Implement account lockout
- Set up monitoring dashboard

**Can Wait Until Post-Launch:**
- WebAuthn support
- Passwordless login
- Advanced MFA options
- Enterprise SSO
- Compliance reporting
- Device management
- Security alerts
- Custom token claims

---

## Phase 5: COMMIT - Deployment & Operations

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Launch Readiness

**Deployment Plan:**
[Deploy to AWS with multi-region setup, Primary in us-east-1, replica in eu-west-1, RDS PostgreSQL with encryption at rest, ElastiCache Redis with TLS, Application Load Balancer with WAF, ECS Fargate for auto-scaling]

**Rollback Strategy:**
[Blue-green deployment with instant switch, Database migrations with tested rollbacks, Feature flags for gradual rollout, Redis snapshot before deployment, Previous 5 versions kept ready]

**Success Metrics:**
[99.99% uptime for auth services, Authentication latency < 200ms globally, Zero security incidents, 95% successful login rate, Support 1M monthly active users]

#### Deployment Details

**Infrastructure Checklist:**
- AWS ECS clusters in 2 regions
- RDS Multi-AZ with encryption
- ElastiCache Redis with auth enabled
- ALB with health checks configured
- WAF rules for common attacks
- CloudFront for static assets
- S3 buckets for backups
- KMS keys for encryption
- VPC with proper security groups
- CloudWatch alarms set

**Configuration Management:**
[Secrets in AWS Secrets Manager, Environment configs in Parameter Store, Infrastructure as Code with Terraform, Automated secret rotation enabled, Separate configs per environment]

**Monitoring Setup:**
[CloudWatch for infrastructure, Sentry for application errors, DataDog for APM and custom metrics, AWS GuardDuty for threat detection, PagerDuty for critical alerts, Slack webhooks for notifications]

#### Post-Launch Planning

**Day 1 Priorities:**
- Monitor authentication success rates
- Check token refresh performance
- Verify email delivery rates
- Watch for security anomalies
- Monitor database connections
- Check Redis memory usage
- Review error logs
- Respond to user issues

**Week 1 Goals:**
- Analyze authentication patterns
- Optimize slow endpoints
- Fix any critical bugs
- Gather integration feedback
- Update SDK documentation
- Review security logs
- Plan MFA rollout
- Create status page

**Month 1 Targets:**
- 99.99% uptime achieved
- 100k registered users
- 10 production integrations
- MFA adoption at 30%
- Zero security incidents
- 5 SDKs published
- Enterprise features ready
- SOC2 audit started

#### Retrospective

**What Went Well:**
[JWT implementation rock solid, Argon2 configuration optimal for security/performance, OAuth integration smoother than expected, Rate limiting prevented all brute force attempts, Audit logging proved invaluable for debugging]

**Lessons Learned:**
[Refresh token complexity underestimated, Should have used message queue for emails, Multiple tab support needed earlier planning, Redis memory planning crucial, Security testing should start earlier]

**Team Victories:**
[Zero security vulnerabilities in final audit, Performance exceeded all benchmarks, Clean architecture enabled easy testing, Documentation praised by early adopters, Smooth deployment with zero downtime]