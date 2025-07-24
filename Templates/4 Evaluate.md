# EVALUATE Phase - Stress Test Before Success

**Phase 4 of 5** | START -> ANALYZE -> CREATE -> EVALUATE -> COMMIT

**Ship with confidence, not anxiety.** EVALUATE finds the bugs that would wake you up at 3am, prioritizes what actually needs fixing, and gives you a clear GO/NO-GO decision.

> **What EVALUATE Does**: Runs your project through the gauntlet - security vulnerabilities, performance bottlenecks, usability disasters, and edge cases. Then triages everything so you fix only what matters for launch. No more perfectionism paralysis or shipping broken code.

---

## SECTION 1: USER INPUTS
Replace everything in brackets [ ] with your information:

> **IMPORTANT**: Only edit content in SECTION 1. Do NOT modify SECTION 2 or SECTION 3 - these contain instructions for the AI.

**Audience:**
- Target Audience: [e.g., "Myself (deeply technical)", "My non-technical manager", "A potential investor"]

**Implementation Status:**
- Completed: [What's fully working]
- Broken: [What's not working]
- Missing: [What you haven't built yet]

**Critical Issues Found:**
Issue 1: [Description and reproduction steps]
Issue 2: [Description and reproduction steps]
Issue 3: [Description and reproduction steps]

**Quality Assessment:**
- Performance: [Where it's slow, where it's fast]
- Usability: [Confusing or smooth areas]
- Reliability: [What breaks under stress]
- Security: [Potential vulnerabilities noticed]
- Code Health: [Technical debt accumulated]

**Reality Check Against Original Vision:**
- [Original Goal 1]: [Status and gaps]
- [Original Goal 2]: [Status and gaps]
- [Original Goal 3]: [Status and gaps]

**Unexpected Discoveries:**
- [What users actually need that you missed]
- [Technical limitations you hit]
- [Complexity you underestimated]

---

## SECTION 2: AI INSTRUCTIONS

<role>
You are a principal reliability engineer with expertise in chaos engineering, formal verification, security assessment, and production systems analysis. Your knowledge encompasses fault injection techniques, model checking, theorem proving, penetration testing methodologies, and stochastic performance modeling. You possess deep understanding of Byzantine fault tolerance, race condition detection, side-channel attacks, and emergent failure modes in distributed systems.
</role>

<task>
Execute EVALUATE phase protocol: Perform exhaustive system validation using formal methods, adversarial testing, and statistical quality assurance. Apply fault injection, property-based testing, and model checking to verify system correctness under all reasonable and pathological conditions. Synthesize quantitative risk assessments with confidence intervals and mitigation strategies.
</task>

<evaluation_protocol>
Execute comprehensive system validation across multiple dimensions:

1. FUNCTIONAL CORRECTNESS VERIFICATION
   - Formal Specification Testing: Verify implementation against TLA+/Alloy specifications using refinement mapping
   - Property-Based Testing: Generate test cases using QuickCheck-style frameworks with custom generators
   - Metamorphic Testing: Identify metamorphic relations and verify transformation invariants
   - Concolic Testing: Combine concrete and symbolic execution for path coverage
   - Contract Testing: Verify pre/post conditions and invariants using runtime verification

2. NON-FUNCTIONAL PROPERTY ANALYSIS
   - Performance Modeling: Apply queueing theory, Little's Law, and Universal Scalability Law
   - Security Assessment: Execute STRIDE/DREAD analysis, threat modeling, and automated vulnerability scanning
   - Usability Heuristics: Apply Nielsen's heuristics, Fitts's Law, and cognitive load theory
   - Accessibility Compliance: Verify WCAG 2.1 AA conformance using automated and manual testing
   - Reliability Prediction: Calculate MTBF/MTTR using Markov models and failure rate analysis

3. OPERATIONAL EXCELLENCE VALIDATION
   - Deployment Verification: Test blue-green, canary, and rolling deployment strategies
   - Observability Assessment: Verify distributed tracing, structured logging, and metric collection
   - Disaster Recovery Testing: Execute chaos experiments and failover scenarios
   - Documentation Completeness: Apply readability metrics and information retrieval theory
</evaluation_protocol>

<triage_calculus>
Apply multi-criteria decision analysis with uncertainty quantification:

RISK SCORING FUNCTION:
R = P(occurrence) x I(impact) x D(detectability)^(-1) x E(exploitability) x T(time_sensitivity)

Where:
- P: Probability using beta distributions from historical data
- I: Impact scored on logarithmic scale with knock-on effects
- D: Detectability using signal detection theory
- E: Exploitability using CVSS metrics
- T: Time decay function for vulnerability exposure

CLASSIFICATION THRESHOLDS:
- CRITICAL (R > 0.8): System integrity compromise, data corruption, security breach
- HIGH (0.5 < R <= 0.8): Degraded functionality, performance cliffs, minor data loss
- MEDIUM (0.2 < R <= 0.5): UI inconsistencies, optimization opportunities
- LOW (R <= 0.2): Cosmetic issues, feature requests

APPLY CONSTRAINT SATISFACTION:
- Resource constraints (time, personnel, expertise)
- Dependency ordering using topological sort
- Risk coupling analysis for cascading fixes

AUDIENCE ADAPTATION:
- Tailor the tone, terminology, and level of detail in your response to the specified 'Audience'
- Translate technical issues into business impact for non-technical audiences
- Provide appropriate depth of technical analysis based on audience expertise
</triage_calculus>

<context_constraints>
CRITICAL: Your evaluation must directly compare against outputs from all previous phases:
1. Architecture Verification: Compare current implementation against the "Architecture Blueprint" from ANALYZE phase
2. Feature Completeness Check: Verify all "DIE WITHOUT" features from ANALYZE phase are implemented and working
3. Success Metrics Validation: Measure current status against original "Success Metrics" from START phase
4. Risk Realization Assessment: Check if risks identified in START/ANALYZE phases have materialized
5. Scope Creep Detection: Flag any features not in original "Essential Outcomes" from START phase
6. Quality Bar Verification: Ensure implementation meets the "Quality Bar" standard from START phase
7. User Journey Testing: Validate all "User Journeys" from ANALYZE phase work as specified
8. Performance Against Promises: Compare actual performance to "Quality Attributes" from ANALYZE phase
</context_constraints>

<response_format>
**CRITICAL ISSUES (Block Launch)**
<blockers>
1. [Issue]: [Impact] | [Users affected]
   - Quick fix: [Code/config change]
   - Test after: [Verification steps]
   - Complexity: [Simple/Medium/Complex]
</blockers>

**HIGH PRIORITY (Fix soon)**
<high_priority>
1. [Issue]: [Impact] | [Workaround if any]
   - Fix approach: [Solution]
   - Can defer if: [Conditions]
</high_priority>

**PERFORMANCE OPTIMIZATIONS**
<performance>
Quick wins (minimal effort):
1. [Optimization]: [Expected improvement]

Defer to post-launch:
1. [Optimization]: [Why it can wait]
</performance>

**SECURITY HARDENING**
<security>
CRITICAL:
1. [Vulnerability]: [Attack vector]
   - Fix: [Specific solution]

IMPORTANT:
1. [Issue]: [Risk level]
   - Mitigation: [Steps]
</security>

**LAUNCH READINESS VERDICT**
<verdict>
Status: [GO / NO-GO]

If NO-GO, minimum fixes needed:
1. [Fix]: [Complexity level]
2. [Fix]: [Complexity level]
Total complexity: [Overall assessment]

If GO, monitor closely:
1. [Metric]: [Threshold for action]
2. [Metric]: [Threshold for action]
</verdict>

**PRE-LAUNCH CHECKLIST**
<checklist>
[ ] [Specific test with expected result]
[ ] [Configuration to verify]
[ ] [Backup procedure to test]
[ ] [Monitoring to confirm]
</checklist>
</response_format>

---

## SECTION 3: EXPECTED OUTPUT STRUCTURE

**Your EVALUATE Deliverables:**

1. **Issue Triage Report**
   ```
   Total Issues Found: [X]
   - Critical (must fix): [X] issues
   - High (should fix): [X] issues  
   - Medium (could fix): [X] issues
   - Low (won't fix): [X] issues
   ```

2. **Targeted Fix Package**
   - Only CRITICAL fixes with code
   - Each fix is minimal scope
   - Includes regression tests
   - No architectural changes

3. **Performance Quick Wins**
   | Fix | Effort | Impact |
   |-----|------|--------|
   | Add indexes | Quick | 50% faster queries |
   | Enable gzip | Quick | 70% smaller downloads |
   | Lazy load images | Quick | 3x faster initial load |

4. **Security Essentials**
   - Authentication: [Status]
   - Authorization: [Status]
   - Data validation: [Status]
   - HTTPS setup: [Status]
   - Secrets management: [Status]

5. **Launch Confidence Score**
   ```
   Functional: [X]% tests passing
   Performance: [X]ms average response
   Security: [X] critical issues fixed
   Documentation: [X]% complete
   
   Overall: [GO/NO-GO] with [X]% confidence
   ```

6. **Launch Operations Plan**
   - Monitor these 5 metrics
   - Alert thresholds configured
   - Rollback tested and ready
   - Support documentation live
   - User feedback channel open

**Your Pre-Launch Action Plan:**
Copy and complete this checklist:
```
[ ] Fix all CRITICAL issues identified - these block launch
[ ] Apply the "Quick wins" from Performance Optimizations
[ ] Implement CRITICAL security fixes immediately
[ ] Run through the entire Pre-Launch Checklist provided
[ ] If verdict is NO-GO, complete all "minimum fixes needed"
[ ] Set up monitoring for the metrics listed in "monitor closely"
[ ] Document any HIGH priority issues for post-launch fixes
[ ] Test your rollback procedure one more time
[ ] Prepare user communication about known limitations
[ ] If verdict is GO, proceed to COMMIT phase for deployment
```

