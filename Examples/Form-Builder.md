# Form Builder - ACE Method Example

## Phase 1: START - Define Your Project

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Project Identity
**What I'm Building:**
[A dynamic form generator that allows users to create complex forms with drag-and-drop interface, advanced validation rules, conditional logic, and seamless integration with various backends and databases]

**The 5-Minute Pitch:**
[FormCraft Pro is a powerful no-code form builder that enables anyone to create sophisticated forms in minutes. It features a drag-and-drop interface, 20+ field types, complex conditional logic, multi-step forms, real-time validation, and integrations with popular services. Built with React and Node.js, it generates accessible, mobile-responsive forms that can be embedded anywhere while providing detailed submission analytics.]

#### The Reality Check

**Who Will Use This:**
[Marketing teams creating lead capture forms, HR departments building application forms, Developers needing quick form prototypes, Agencies creating client forms, Educational institutions for surveys]

**The Problem It Solves:**
[Eliminates weeks of form development time, No coding required for complex forms, Provides consistent validation and UX, Handles data storage and integration, Ensures accessibility compliance]

**Why They'll Choose This:**
[Intuitive drag-and-drop builder, Complex logic without code, 50+ pre-built templates, WCAG 2.1 compliant forms, Real-time collaboration, White-label options]

#### Constraints & Resources

**Technical Constraints:**
[Must support forms with 100+ fields, Handle 10k submissions/hour, Work in IE11 and modern browsers, Load forms under 2 seconds, Support offline submissions]

**Time Investment:**
[4 weeks for core builder interface, 2 weeks for validation engine, 2 weeks for conditional logic, 1 week for integrations, 1 week for analytics dashboard]

**Financial Reality:**
[Hosting infrastructure: $150/month, Database storage: $100/month, CDN for assets: $50/month, Email service: $30/month, SSL and domains: $20/month, Total: ~$350/month]

#### Scope Definition

**Core Features (MVP):**
- Drag-and-drop form builder
- 10 essential field types
- Basic validation rules
- Form preview and testing
- Simple submission storage
- Embed code generation

**Future Expansions:**
- Advanced conditional logic
- Multi-step form wizard
- Payment integration
- File upload handling
- Workflow automation
- A/B testing capabilities

**Out of Scope:**
- PDF generation
- E-signature functionality
- Advanced reporting
- Native mobile apps
- Offline-first architecture

---

## Phase 2: ANALYZE - Deep Requirements Discovery

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Problem Deep Dive

**The Core Challenge:**
[Creating a form builder that balances ease of use for non-technical users with the flexibility developers need, while generating forms that perform well, are accessible, handle complex validation and conditional logic, and integrate seamlessly with existing systems]

**Current Pain Points:**
[Form builders too limited for complex needs, Custom forms take weeks to develop, Validation logic becomes unmaintainable, Poor mobile experience on forms, Data stuck in form builder silos]

**Success Metrics:**
[Form creation time under 10 minutes, Support 50+ field types, Handle 1M submissions/month, 99.9% uptime for embedded forms, Zero accessibility violations]

#### User Journey Mapping

**Primary User Flow:**
[User enters builder → Drags fields onto canvas → Configures field properties → Sets up validation rules → Adds conditional logic → Previews form → Publishes and gets embed code → Monitors submissions]

**Critical Touchpoints:**
[Builder interface intuitiveness, Field configuration clarity, Validation rule setup, Preview accuracy, Publishing process, Embed implementation, Submission handling]

**Failure Points:**
[Complex UI overwhelms new users, Conditional logic too hard to configure, Preview doesn't match live form, Embed code breaks on some sites, Submissions lost during high traffic]

#### System Boundaries

**What We'll Build:**
- React-based form builder UI
- Form rendering engine
- Validation rule system
- Conditional logic processor
- Submission handling API
- Data storage layer
- Analytics dashboard
- Integration framework

**What We'll Integrate:**
- PostgreSQL for form data
- Redis for caching
- S3 for file uploads
- SendGrid for notifications
- Stripe for payments
- Zapier for workflows

**What We Won't Touch:**
- CRM functionality
- Email marketing
- Document management
- Project management
- Customer support tools

#### Success Criteria

**Launch Requirements:**
[10 field types fully functional, Validation engine tested thoroughly, Mobile responsive forms, Accessibility audit passed, Load tested to 1k concurrent forms]

**Quality Benchmarks:**
[Form load time < 1 second, Builder performs smoothly with 100 fields, Zero data loss on submissions, WCAG 2.1 AA compliance, 99.9% API uptime]

**Adoption Targets:**
[1000 forms created month 1, 100k submissions month 1, 500 active users month 3, 50 paid customers month 6, 4.5/5 user satisfaction]

---

## Phase 3: CREATE - Implementation Problem Solving

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Current Context

**What I'm Building:**
[Implementing the conditional logic engine that allows form fields to show/hide, become required/optional, or change validation rules based on other field values, while maintaining performance with complex rule sets]

**Where I'm Stuck:**
[The conditional logic evaluation is causing performance issues with forms having multiple interdependent rules. Also struggling with circular dependencies where field A depends on B, and B depends on A. The UI for building these rules is confusing users.]

**What I've Tried:**
[Implemented rule evaluation on every change, tried caching computed states, used directed graph for dependencies, attempted to detect circular dependencies, built visual rule builder]

#### Code State

**Relevant Code:**
```javascript
// Current problematic conditional logic implementation
class ConditionalLogicEngine {
  constructor(form) {
    this.form = form;
    this.rules = new Map();
    this.fieldStates = new Map();
  }

  // This recalculates everything on every change - slow
  evaluateRules(changedFieldId, newValue) {
    const startTime = performance.now();
    
    // Brute force approach
    for (const [fieldId, rules] of this.rules) {
      for (const rule of rules) {
        const result = this.evaluateRule(rule);
        this.applyRuleResult(fieldId, rule.action, result);
      }
    }
    
    console.log(`Rule evaluation took ${performance.now() - startTime}ms`);
  }

  // No circular dependency detection
  evaluateRule(rule) {
    const { conditions, operator = 'AND' } = rule;
    
    if (operator === 'AND') {
      return conditions.every(cond => this.evaluateCondition(cond));
    } else {
      return conditions.some(cond => this.evaluateCondition(cond));
    }
  }

  evaluateCondition(condition) {
    const { fieldId, operator, value } = condition;
    const fieldValue = this.getFieldValue(fieldId);
    
    // This can cause infinite loops
    switch (operator) {
      case 'equals':
        return fieldValue === value;
      case 'not_equals':
        return fieldValue !== value;
      case 'contains':
        return fieldValue?.includes(value);
      case 'greater_than':
        return Number(fieldValue) > Number(value);
      // ... more operators
    }
  }
}

// Form builder UI with complex rule builder
const RuleBuilder = ({ field, onRuleChange }) => {
  const [rules, setRules] = useState([]);
  
  // This UI is too complex for users
  const addRule = () => {
    setRules([...rules, {
      id: generateId(),
      conditions: [{
        fieldId: '',
        operator: 'equals',
        value: ''
      }],
      operator: 'AND',
      action: {
        type: 'show',
        target: field.id
      }
    }]);
  };
  
  // Nested conditions make this unmanageable
  return (
    <div className="rule-builder">
      {rules.map(rule => (
        <div key={rule.id} className="rule">
          <select value={rule.operator}>
            <option value="AND">All conditions must be true</option>
            <option value="OR">Any condition must be true</option>
          </select>
          
          {rule.conditions.map((condition, index) => (
            <ConditionBuilder
              key={index}
              condition={condition}
              onChange={(newCondition) => {
                // Complex state update logic
              }}
            />
          ))}
          
          <ActionSelector
            action={rule.action}
            onChange={(newAction) => {
              // More complex updates
            }}
          />
        </div>
      ))}
    </div>
  );
};

// Form renderer with performance issues
class FormRenderer {
  constructor(formConfig) {
    this.config = formConfig;
    this.logicEngine = new ConditionalLogicEngine(formConfig);
    this.fieldComponents = new Map();
  }

  // Re-renders entire form on any change
  handleFieldChange = (fieldId, value) => {
    // This triggers full re-evaluation
    this.logicEngine.evaluateRules(fieldId, value);
    
    // Forces re-render of all fields
    this.forceUpdate();
  };

  renderField(field) {
    const state = this.logicEngine.getFieldState(field.id);
    
    if (state.hidden) {
      return null;
    }
    
    // No memoization - recreates component every render
    return (
      <FieldComponent
        key={field.id}
        field={field}
        state={state}
        onChange={(value) => this.handleFieldChange(field.id, value)}
      />
    );
  }
}
```

**Error Messages:**
["Maximum update depth exceeded" in React, "RangeError: Maximum call stack size exceeded", "Performance warning: Long script execution", Users report "Form freezing when typing"]

**What's Working:**
[Basic drag-and-drop working well, Simple forms without logic perform great, Field validation is solid, Form preview accurate, Submission handling reliable]

#### Environment Details

**Tech Stack:**
[React 18 with TypeScript, Redux for state management, Node.js 18 backend, PostgreSQL 14, Redis for caching, Formik for form handling, Yup for validation]

**Constraints:**
[Forms must work in IE11, Support 100+ fields per form, Complex logic with 50+ rules, Form load time under 2 seconds, Builder must be responsive]

**Dependencies:**
[React DnD for drag-drop, Monaco Editor for code, Ant Design components, Express.js backend, Bull for job queues, Sharp for image processing]

#### Specific Questions

**Technical Challenges:**
[How to efficiently evaluate conditional rules? Best way to detect circular dependencies? Should logic run client or server-side? How to optimize React re-renders?]

**Design Decisions:**
[Graph-based vs imperative rule engine? How to simplify rule builder UI? Where to compute field visibility? Should we use Web Workers for logic?]

**Implementation Approach:**
[Considering dependency graph with topological sort, Evaluating React Hook Form for performance, Thinking about rule templates for common patterns, Exploring visual programming for rules]

---

## Phase 4: EVALUATE - Testing & Quality Assurance

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Implementation Status

**What's Complete:**
- Drag-and-drop form builder
- 15 field types implemented
- Basic validation rules
- Form preview functionality
- Submission handling
- Simple conditional logic
- Embed code generation
- Basic analytics dashboard
- PostgreSQL integration

**What's In Progress:**
- Complex conditional logic optimization
- Performance improvements
- Advanced validation rules
- Multi-step forms
- Integration marketplace

**What's Pending:**
- Payment processing
- File upload optimization
- A/B testing features
- Advanced analytics
- White-label options

#### The Reality Check

**Known Issues:**
[Conditional logic causing performance degradation, Circular dependencies crash the browser, Form builder slow with 50+ fields, Preview not accurate for complex logic, Memory leaks in long editing sessions]

**Performance Bottlenecks:**
[Rule evaluation taking 200ms+ on complex forms, React re-rendering entire form on changes, Database queries for form loading slow, Large forms exceed 2MB payload, Analytics queries timing out]

**Security Concerns:**
[XSS possible in custom HTML fields, No rate limiting on submissions, Form data not encrypted at rest, Missing CSRF protection, Embed code exposes API keys]

#### Quality Assessment

**Test Coverage:**
[Unit tests: 70% coverage, Integration tests: 45% coverage, E2E tests: 30% coverage, Accessibility tests: WCAG 2.1 AA passing, Performance tests: 50 fields maximum]

**Code Review Findings:**
[State management overly complex, No code splitting implemented, Missing error boundaries, Inconsistent naming conventions, Database queries need optimization]

**External Feedback:**
[Users find conditional logic UI confusing, Form load time too slow on mobile, Need more field types, Want custom CSS options, Preview should be more accurate]

#### The Launch Checklist

**Must-Fix Before Launch:**
- Fix conditional logic performance
- Resolve circular dependencies
- Implement security headers
- Add rate limiting
- Optimize form loading
- Fix memory leaks
- Improve rule builder UI
- Add error boundaries

**Should-Fix If Time:**
- Add more field types
- Implement code splitting
- Create onboarding tour
- Add form templates
- Improve mobile builder
- Add bulk actions
- Create API documentation
- Set up monitoring

**Can Wait Until Post-Launch:**
- Payment integration
- Advanced analytics
- A/B testing
- White labeling
- Workflow automation
- Native apps
- Offline support
- Advanced themes

---

## Phase 5: COMMIT - Deployment & Operations

### SECTION 1: USER INPUTS - REPLACE CONTENT WITHIN [...] WITH YOUR INFORMATION

#### Launch Readiness

**Deployment Plan:**
[Deploy frontend to Vercel with CDN, Backend on AWS ECS Fargate, RDS PostgreSQL for data, ElastiCache Redis, S3 for form assets, CloudFront for embedded forms]

**Rollback Strategy:**
[Vercel instant rollback capability, ECS blue-green deployments, Database migrations versioned, Redis cache can be cleared, Feature flags for risky features]

**Success Metrics:**
[Form load time < 1.5 seconds globally, 99.9% uptime for form rendering, Support 10k concurrent forms, Zero data loss incidents, 95% user satisfaction score]

#### Deployment Details

**Infrastructure Checklist:**
- Vercel project configured
- ECS services defined
- RDS Multi-AZ enabled
- Redis cluster ready
- S3 buckets created
- CloudFront configured
- Route53 DNS setup
- SSL certificates valid
- WAF rules active
- Monitoring enabled

**Configuration Management:**
[Environment variables in Vercel/ECS, Database credentials in Secrets Manager, Feature flags in LaunchDarkly, Infrastructure as Code with Terraform, Automated deployments with GitHub Actions]

**Monitoring Setup:**
[Vercel Analytics for frontend, CloudWatch for infrastructure, Sentry for error tracking, Mixpanel for user analytics, Custom form submission metrics, Uptime monitoring with Pingdom]

#### Post-Launch Planning

**Day 1 Priorities:**
- Monitor form load times
- Check submission success rates
- Verify conditional logic performance
- Watch for JavaScript errors
- Monitor server resources
- Review security alerts
- Track user signups
- Respond to feedback

**Week 1 Goals:**
- Analyze usage patterns
- Optimize slow queries
- Fix reported bugs
- Create video tutorials
- Launch customer support
- Gather testimonials
- Plan first update
- Monitor costs

**Month 1 Targets:**
- 5000 forms created
- 500k submissions processed
- 1000 active users
- 20 paying customers
- 4.5/5 satisfaction rating
- 10 integration partners
- Form template library
- Community forum launch

#### Retrospective

**What Went Well:**
[Drag-and-drop interface intuitive for users, Form rendering performance exceeded goals, Accessibility compliance achieved first try, Embed functionality works seamlessly, Analytics dashboard provides valuable insights]

**Lessons Learned:**
[Conditional logic complexity underestimated, Should have used virtualization earlier, Rule builder needs user research, Performance testing should include complex forms, Mobile builder needs different approach]

**Team Victories:**
[Created best-in-class form builder UX, Solved complex conditional logic challenges, Built scalable architecture, Achieved accessibility goals, Received amazing user feedback]