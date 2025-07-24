# ACE Method Quick Template

**All 5 Phases in One Document** | START -> ANALYZE -> CREATE -> EVALUATE -> COMMIT

> **Power User Mode**: Copy this entire template into your AI assistant to run the complete ACE Method workflow in a single conversation. Perfect for experienced users or smaller projects.

> **Note on Audience Field**: The "Target Audience" field appears in each phase. You can either:
> - Fill it once at the top and copy to all phases for consistency
> - Adjust it per phase if your audience changes (e.g., technical details for CREATE, business focus for COMMIT)
> - The AI will tailor its response complexity and terminology based on this field

> **Need Examples?** Check the `/Examples` directory for 10 real projects showing how to fill out these templates (API Builder, Auth System, Cache Layer, Data Dashboard, and more)

---

## PHASE 1: START - Define Your Vision

**Project Identity:**
- Name: [What you want to call this]
- Type: [Software, hardware, research, creative work, etc.]
- Core Purpose: [One sentence describing the end result]

**Audience:**
- Target Audience: [e.g., "Myself (deeply technical)", "My non-technical manager", "A potential investor"]

**Impact & Users:**
- End Users: [Who will use this]
- Problem Being Solved: [The pain point or opportunity]
- Success Metrics: [How you'll measure if this worked]

**Your Context:**
- Your Background: [Relevant skills and experience]
- Available Resources: [Time, money, tools, people]
- Quality Bar: [Proof of concept, professional, commercial, etc.]

**Scope Definition:**
- Essential Outcomes: [What absolutely must work]
- Bonus Outcomes: [What would be nice to have]
- Non-Goals: [What you're explicitly NOT doing]

---

## PHASE 2: ANALYZE - Uncover Hidden Requirements

**Audience:**
- Target Audience: [e.g., "Myself (deeply technical)", "My non-technical manager", "A potential investor"]

**Problem Deep Dive:**
- Root Cause: [Why does this problem exist?]
- Current Workarounds: [How people handle it now]
- Cost of Inaction: [What happens if nothing changes]

**User Journeys:**
- Journey 1: As a [who], I need to [what] so that [why]
- Journey 2: As a [who], I need to [what] so that [why]
- Critical Path: [The one flow that must work perfectly]

**System Boundaries:**
- Core Capabilities: [What the system must do]
- Data Model: [Key entities and relationships]
- Integration Points: [What it connects to]
- Quality Attributes: [Performance, security, reliability needs]

---

## PHASE 3: CREATE - Implementation Issues

**Audience:**
- Target Audience: [e.g., "Myself (deeply technical)", "My non-technical manager", "A potential investor"]

**Current Context:**
- Working On: [Feature/component/problem]
- Project State: [What works so far, what's next]

**The Situation:**
- Expected: [What should happen]
- Reality: [What actually happens]
- Error/Issue: [Specific error or unexpected behavior]

**Code Context:**
```[language]
[Your current implementation if applicable]
```

**My Specific Questions:**
1. [Primary blocker/question]
2. [Secondary concern if any]

---

## PHASE 4: EVALUATE - Quality Check

**Audience:**
- Target Audience: [e.g., "Myself (deeply technical)", "My non-technical manager", "A potential investor"]

**Implementation Status:**
- Completed: [What's fully working]
- Broken: [What's not working]
- Missing: [What you haven't built yet]

**Critical Issues Found:**
- Issue 1: [Description and reproduction steps]
- Issue 2: [Description and reproduction steps]

**Quality Assessment:**
- Performance: [Where it's slow, where it's fast]
- Usability: [Confusing or smooth areas]
- Reliability: [What breaks under stress]

---

## PHASE 5: COMMIT - Ship It

**Audience:**
- Target Audience: [e.g., "Myself (deeply technical)", "My non-technical manager", "A potential investor"]

**Launch Readiness:**
- Bugs Fixed: [Critical issues resolved]
- Documentation: [What you've written for users/devs]
- Final Testing: [What's been validated]

**Deployment Details:**
- Platform: [Where it's going live]
- URL/Access: [How users will find it]
- Configuration: [Environment vars, secrets, settings]

**Project Retrospective:**
- Hardest Challenge: [What nearly broke you]
- Proudest Achievement: [What exceeded expectations]
- Key Lesson: [What you'd do differently]

---

## AI INSTRUCTIONS

You are an expert systems architect and implementation specialist. Process this ACE Method template in sequence:

1. **START**: Perform feasibility analysis, identify risks, recommend architecture, and create learning path
2. **ANALYZE**: Discover hidden requirements, design system architecture, prioritize features, and identify risks
3. **CREATE**: Diagnose issues, provide working solutions, explain fixes, and suggest next steps
4. **EVALUATE**: Triage issues by severity, identify critical fixes, assess launch readiness
5. **COMMIT**: Create deployment guide, write documentation, setup monitoring, and plan next phases

For each phase, provide:
- Specific, actionable recommendations
- Code/configuration examples where applicable
- Risk assessments with mitigation strategies
- Clear next steps and success criteria

**IMPORTANT**: Tailor the tone, terminology, and level of detail in your response to the specified 'Audience' field in each phase. Adjust technical depth, use appropriate analogies, and present information in terms meaningful to the target audience.

**CRITICAL - Context Continuity**: Each phase must build upon and reference outputs from previous phases:
- ANALYZE must address Essential Outcomes and Success Metrics from START
- CREATE must follow the Architecture Blueprint from ANALYZE
- EVALUATE must compare implementation against all prior phase commitments
- COMMIT must verify resolution of issues from EVALUATE and achievement of original vision

Focus on practical implementation over theory. Be concise but thorough. Adapt complexity to user's technical level.

---

## HOW TO USE THIS TEMPLATE

### Step-by-Step Instructions

1. **Replace all [square bracket] placeholders** with your specific information
2. **Keep all 5 phases together** - the AI needs full context to deliver best results
3. **Copy the ENTIRE template** - from top to bottom, including AI instructions
4. **Paste into your AI assistant** - Works with ChatGPT, Claude, Gemini, etc.
5. **Save the response** - It becomes your project documentation

### Pro Tips

- **Be specific**: "e-commerce site" -> "Shopify store selling handmade jewelry"
- **Include constraints**: Mention deadlines, budget, tech stack preferences
- **Add context**: Previous attempts, existing code, team situation
- **Iterate freely**: Run CREATE phase multiple times for different features

### Tips for Success:
- Be specific in your inputs - vague inputs yield vague outputs
- Include actual error messages and code snippets
- Update earlier phases as you learn more
- Use the AI's feedback to refine your approach

### Common Patterns:
- **Software Project**: Focus on architecture in ANALYZE, code issues in CREATE
- **Research Project**: Emphasize methodology in ANALYZE, validation in EVALUATE
- **Business Process**: Detail workflows in ANALYZE, metrics in COMMIT
- **Creative Work**: Define aesthetic goals in START, audience testing in EVALUATE

---

## ACTION CHECKLISTS

After running the ACE Method, use these phase-specific checklists:

**After START Phase:**
```
[ ] Adjust project scope based on Reality Check
[ ] Choose technology stack from recommendations
[ ] Begin learning first "Core essential"
[ ] Implement critical risk mitigations
```

**After ANALYZE Phase:**
```
[ ] Document all Hidden Requirements
[ ] Create architecture diagram
[ ] Build proof-of-concept for complex features
[ ] Set up risk monitoring
```

**After CREATE Phase:**
```
[ ] Apply and test the fix provided
[ ] Run all verification tests
[ ] Document the solution
[ ] Implement "next steps" suggestions
```

**After EVALUATE Phase:**
```
[ ] Fix all CRITICAL issues
[ ] Apply performance quick wins
[ ] Run pre-launch checklist
[ ] Set up monitoring
```

**After COMMIT Phase:**
```
[ ] Execute deployment steps
[ ] Set up monitoring tools
[ ] Publish documentation
[ ] Monitor metrics for 24 hours
[ ] Celebrate your launch!
```