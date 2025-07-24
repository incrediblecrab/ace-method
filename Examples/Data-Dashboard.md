# Data Dashboard - ACE Method Example

## Phase 1: START - Define Your Project

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Project Identity
**What I'm Building:**
[A real-time data visualization dashboard that displays database metrics, KPIs, and business intelligence insights with interactive charts and automated reporting capabilities]

**The 5-Minute Pitch:**
[MetricFlow is a modern analytics dashboard that transforms raw database data into actionable insights. It connects to multiple data sources (PostgreSQL, MySQL, MongoDB), provides real-time updates via WebSocket, offers 15+ visualization types, and includes automated daily/weekly reports. Built with React and D3.js, it features drill-down capabilities, custom dashboards, and role-based access control for different stakeholders.]

#### The Reality Check

**Who Will Use This:**
[Business analysts monitoring KPIs, C-suite executives tracking company metrics, Product managers analyzing user behavior, Operations teams monitoring system health, Sales teams tracking performance]

**The Problem It Solves:**
[Eliminates manual Excel report generation, provides real-time visibility into business metrics, democratizes data access across organization, reduces time from question to insight from days to minutes]

**Why They'll Choose This:**
[No SQL knowledge required for end users, real-time updates without page refresh, mobile-responsive design for on-the-go access, white-label customization options, exports to multiple formats (PDF, Excel, PNG)]

#### Constraints & Resources

**Technical Constraints:**
[Must support real-time data with <5 second latency, Handle datasets up to 10M rows, Support 100 concurrent users, Compatible with Chrome/Firefox/Safari, 4GB RAM minimum for server]

**Time Investment:**
[6 weeks for MVP with core visualizations, 3 weeks for real-time features, 2 weeks for automated reporting, 1 week for deployment and testing]

**Financial Reality:**
[Frontend hosting: $20/month on Vercel, Backend server: $100/month for adequate specs, Database read replicas: $150/month, WebSocket server: $50/month, Total: ~$320/month for production]

#### Scope Definition

**Core Features (MVP):**
- Multi-database connection support
- 5 essential chart types (line, bar, pie, table, metric cards)
- User authentication and role management
- Dashboard builder with drag-and-drop
- Basic filtering and date range selection

**Future Expansions:**
- Advanced visualizations (heatmaps, sankey, treemaps)
- Automated anomaly detection
- Natural language queries
- Embedded dashboard sharing
- API for programmatic access

**Out of Scope:**
- ETL pipeline management
- Data warehouse creation
- Machine learning predictions
- Native mobile apps
- Data input/editing capabilities

---

## Phase 2: ANALYZE - Deep Requirements Discovery

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Problem Deep Dive

**The Core Challenge:**
[Creating a system that can query multiple databases efficiently, transform diverse data structures into visualizations, handle real-time updates without overwhelming the browser, and maintain performance with large datasets while keeping the interface intuitive for non-technical users]

**Current Pain Points:**
[Business users wait days for custom reports, Excel crashes with large datasets, No single source of truth across departments, Manual report generation prone to errors, Lack of real-time visibility into critical metrics]

**Success Metrics:**
[Dashboard load time under 3 seconds, Query response time under 2 seconds for 1M rows, 90% of users can create basic dashboard without training, 50% reduction in time spent on reporting, Zero data discrepancies vs source systems]

#### User Journey Mapping

**Primary User Flow:**
[User logs in → Selects data source → Chooses metrics to visualize → Drags charts onto canvas → Configures filters and time ranges → Saves dashboard → Sets up automated reports → Shares with team]

**Critical Touchpoints:**
[Database connection setup simplicity, Metric selection intuitiveness, Chart customization options, Performance with large datasets, Sharing and permissions management]

**Failure Points:**
[Complex database authentication fails, Slow queries timeout and frustrate users, Confusing chart configuration loses users, Poor mobile experience for executives, Incorrect data aggregations erode trust]

#### System Boundaries

**What We'll Build:**
- React-based frontend with D3.js visualizations
- Node.js backend with GraphQL API
- Database connection pooling system
- Redis-based caching layer
- WebSocket server for real-time updates
- PDF generation service
- User management system
- Query optimization engine

**What We'll Integrate:**
- PostgreSQL, MySQL, MongoDB drivers
- Auth0 for authentication
- SendGrid for email reports
- AWS S3 for report storage
- Stripe for billing (future)

**What We Won't Touch:**
- Database administration
- Data cleaning/preparation
- ETL processes
- Predictive analytics
- Data governance tools

#### Success Criteria

**Launch Requirements:**
[Support for 3 major databases tested, 5 chart types fully functional, Load testing with 1M row datasets, Security audit completed, User documentation complete]

**Quality Benchmarks:**
[Page load time < 3 seconds, Chart render time < 1 second, 99.9% uptime, Query accuracy 100%, Mobile responsive score > 95]

**Adoption Targets:**
[20 active users in week 1, 100 dashboards created by month 1, 80% daily active usage, 4.5/5 user satisfaction score, 10 automated reports configured]

---

## Phase 3: CREATE - Implementation Problem Solving

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Current Context

**What I'm Building:**
[Implementing the real-time data synchronization system that updates dashboard charts when underlying database data changes without requiring page refresh]

**Where I'm Stuck:**
[WebSocket connections are dropping under load, causing missed updates. Also struggling with efficient change detection in source databases without impacting their performance. The current polling approach creates too much database load.]

**What I've Tried:**
[Implemented database triggers for change detection, tried polling with incremental timestamps, attempted using database replication logs, explored using change data capture (CDC) tools]

#### Code State

**Relevant Code:**
```javascript
// Current problematic WebSocket implementation
class RealtimeDataSync {
  constructor() {
    this.connections = new Map();
    this.pollingIntervals = new Map();
  }

  // This is causing too much database load
  startDataSync(dashboardId, queries) {
    const interval = setInterval(async () => {
      for (const query of queries) {
        try {
          const newData = await this.executeQuery(query);
          const oldData = this.cache.get(query.id);
          
          // Inefficient comparison
          if (JSON.stringify(newData) !== JSON.stringify(oldData)) {
            this.broadcastUpdate(dashboardId, query.id, newData);
            this.cache.set(query.id, newData);
          }
        } catch (error) {
          console.error('Query failed:', error);
        }
      }
    }, 5000); // 5 second polling
    
    this.pollingIntervals.set(dashboardId, interval);
  }

  // WebSocket connections dropping
  broadcastUpdate(dashboardId, queryId, data) {
    const connections = this.connections.get(dashboardId) || [];
    
    connections.forEach(ws => {
      // No error handling for failed sends
      ws.send(JSON.stringify({
        type: 'data-update',
        queryId,
        data
      }));
    });
  }

  async executeQuery(query) {
    // Direct database query without connection pooling
    const connection = await this.getDbConnection(query.datasource);
    const result = await connection.query(query.sql);
    connection.close(); // Connection churn
    return result;
  }
}

// Frontend WebSocket handling with issues
const useRealtimeData = (dashboardId) => {
  const [data, setData] = useState({});
  const wsRef = useRef(null);

  useEffect(() => {
    // Reconnection logic missing
    wsRef.current = new WebSocket(`ws://localhost:3001/dashboard/${dashboardId}`);
    
    wsRef.current.onmessage = (event) => {
      const update = JSON.parse(event.data);
      // Race condition with React state updates
      setData(prev => ({
        ...prev,
        [update.queryId]: update.data
      }));
    };

    return () => wsRef.current?.close();
  }, [dashboardId]);

  return data;
};
```

**Error Messages:**
[WebSocket connection timeout after 50 concurrent users, "Too many connections" from PostgreSQL, Browser console shows "WebSocket is already in CLOSING state", Memory usage growing unbounded on server]

**What's Working:**
[Initial dashboard load is fast, Static charts render perfectly, Database connection setup is solid, User authentication working well, Basic filtering performs adequately]

#### Environment Details

**Tech Stack:**
[React 18 with TypeScript, D3.js for visualizations, Node.js 18 with Express, PostgreSQL 14, Redis 7, Socket.io for WebSocket, Bull for job queues]

**Constraints:**
[Cannot modify source databases, Must support 100 concurrent dashboards, Query results can be up to 100MB, 5-second maximum latency for updates, Server limited to 8GB RAM]

**Dependencies:**
[Database read replicas available, Redis cluster with 16GB RAM, CDN for static assets, Load balancer supports WebSocket, Kubernetes for container orchestration]

#### Specific Questions

**Technical Challenges:**
[How to implement efficient change detection without database triggers? Best approach for WebSocket connection pooling? How to handle partial updates for large datasets? Should we use Server-Sent Events instead of WebSocket?]

**Design Decisions:**
[Should updates be pushed or pulled? Is GraphQL subscriptions better than WebSocket? How to batch updates efficiently? Where to implement data diffing - server or client? How to handle offline clients?]

**Implementation Approach:**
[Considering CDC tools like Debezium, Evaluating Redis Streams for event queue, Thinking about differential sync algorithm, Exploring cursor-based pagination for large results]

---

## Phase 4: EVALUATE - Testing & Quality Assurance

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Implementation Status

**What's Complete:**
- Core dashboard builder with drag-and-drop
- 5 chart types (line, bar, pie, table, metric)
- Database connection management
- User authentication and roles
- Basic real-time updates
- PDF export functionality
- Responsive design
- Query builder interface
- Caching layer implementation

**What's In Progress:**
- WebSocket optimization for scale
- Advanced chart types
- Automated report scheduling
- Performance optimization for large datasets
- Mobile app development

**What's Pending:**
- Natural language query interface
- Anomaly detection features
- Embedded dashboard API
- White-label customization
- Advanced user permissions

#### The Reality Check

**Known Issues:**
[WebSocket connections drop after 50 users, Large datasets (>100k rows) cause browser freezing, PDF exports timeout for complex dashboards, Date timezone handling inconsistent, Memory leaks in real-time sync service]

**Performance Bottlenecks:**
[Initial dashboard load takes 5+ seconds with 10+ charts, Database queries not optimized (missing indexes), Chart re-renders on every update causing flicker, No query result pagination, Cache invalidation too aggressive]

**Security Concerns:**
[SQL injection possible in custom query builder, Cross-tenant data leakage in shared queries, API keys exposed in frontend code, Missing rate limiting on API endpoints, Insufficient audit logging]

#### Quality Assessment

**Test Coverage:**
[Unit tests: 75% coverage, Integration tests: 45% coverage, E2E tests: 30% coverage, Performance tests: Completed for 10k rows, Security scan: 3 high-severity issues found]

**Code Review Findings:**
[Inconsistent TypeScript usage, Missing error boundaries in React, Database connections not properly pooled, No request cancellation on unmount, Hard-coded configuration values]

**External Feedback:**
[Beta users report confusion with chart configuration, Mobile experience needs work for tablets, Export functionality doesn't preserve formatting, Real-time updates feel jarring, Need better loading states]

#### The Launch Checklist

**Must-Fix Before Launch:**
- Fix WebSocket scaling issues
- Resolve SQL injection vulnerabilities
- Optimize queries for large datasets
- Fix memory leaks in sync service
- Implement proper error handling
- Add request rate limiting
- Fix timezone inconsistencies
- Complete security audit remediations

**Should-Fix If Time:**
- Improve chart configuration UX
- Add more loading states
- Implement query caching
- Optimize bundle size
- Add keyboard shortcuts
- Improve mobile responsiveness
- Create video tutorials
- Set up status page

**Can Wait Until Post-Launch:**
- Advanced chart types
- Natural language queries
- Native mobile apps
- Advanced permissions
- Custom themes
- API documentation
- Integration marketplace
- Multi-language support

---

## Phase 5: COMMIT - Deployment & Operations

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Launch Readiness

**Deployment Plan:**
[Deploy frontend to Vercel with preview deployments, Backend on AWS ECS with auto-scaling, RDS PostgreSQL for application data, ElastiCache Redis for caching, CloudFront CDN for static assets, Route53 for DNS management]

**Rollback Strategy:**
[Vercel instant rollback for frontend, Blue-green deployment for backend, Database migrations with rollback scripts, Feature flags for gradual feature release, Backup snapshots before each deployment]

**Success Metrics:**
[Dashboard load time < 3 seconds, 99.9% uptime SLA, 100 daily active users by week 2, Zero data accuracy issues, 90% user satisfaction score]

#### Deployment Details

**Infrastructure Checklist:**
- Vercel project configured with env vars
- AWS ECS cluster with task definitions
- RDS Multi-AZ deployment enabled
- Redis cluster with automatic failover
- S3 buckets for report storage
- CloudWatch alarms configured
- Auto-scaling policies set
- SSL certificates installed
- WAF rules configured
- Backup retention policies set

**Configuration Management:**
[Environment variables in Vercel and AWS Systems Manager, Database credentials in AWS Secrets Manager, Feature flags in LaunchDarkly, Infrastructure as Code with Terraform, Configuration validation on deploy]

**Monitoring Setup:**
[Vercel Analytics for frontend performance, CloudWatch for infrastructure metrics, Sentry for error tracking, Mixpanel for user analytics, StatusPage for public status, PagerDuty for incident response]

#### Post-Launch Planning

**Day 1 Priorities:**
- Monitor error rates across all services
- Check dashboard load times
- Verify real-time updates working
- Watch database connection pools
- Monitor WebSocket stability
- Review user signup flow
- Check export functionality
- Respond to immediate feedback

**Week 1 Goals:**
- Analyze usage patterns
- Optimize slow queries identified
- Fix any critical bugs
- Gather user feedback sessions
- Create knowledge base articles
- Monitor infrastructure costs
- Plan first feature update
- Set up user onboarding flow

**Month 1 Targets:**
- Reach 500 active dashboards
- Achieve < 3s load time consistently
- Launch 3 new chart types
- Complete mobile optimizations
- Establish power user community
- Create partnership program
- Launch affiliate program
- Plan enterprise features

#### Retrospective

**What Went Well:**
[Chart library selection (D3.js) proved flexible, Drag-and-drop interface intuitive for users, Caching strategy dramatically improved performance, Beta user feedback invaluable for UX improvements, TypeScript caught many bugs early]

**Lessons Learned:**
[WebSocket scaling more complex than anticipated, Should have started with read replicas, Real-time sync needed more planning, Mobile experience should've been priority, Query optimization critical for large datasets]

**Team Victories:**
[Delivered beautiful visualizations users love, PDF exports exceeded expectations, Performance goals met after optimization, Security audit passed after fixes, Created extensible architecture for future growth]