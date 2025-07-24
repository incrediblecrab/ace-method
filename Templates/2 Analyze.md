# ANALYZE Phase - Uncover Hidden Requirements

**Phase 2 of 5** | START -> ANALYZE -> CREATE -> EVALUATE -> COMMIT

**The #1 cause of project failure? Missing requirements.** This phase uncovers the hidden complexity that derails 90% of projects, before you've invested weeks building the wrong thing.

> **What ANALYZE Does**: Uses battle-tested techniques from enterprise architecture to find requirements you'd only discover after launch. Identifies security needs, scalability limits, integration gotchas, and user expectations you haven't considered.

---

## SECTION 1: USER INPUTS
Replace everything in brackets [ ] with your information:

> **IMPORTANT**: Only edit content in SECTION 1. Do NOT modify SECTION 2 or SECTION 3 - these contain instructions for the AI.

**Audience:**
- Target Audience: [e.g., "Myself (deeply technical)", "My non-technical manager", "A potential investor"]

**Problem Deep Dive:**
- Root Cause: [Why does this problem exist?]
- Current Workarounds: [How people handle it now]
- Cost of Inaction: [What happens if nothing changes]

**User Journeys:** (Add more as needed)
- Journey 1: As a [who], I need to [what] so that [why]
- Journey 2: As a [who], I need to [what] so that [why]
- Critical Path: [The one flow that must work perfectly]

**System Boundaries:**
- Core Capabilities: [What the system must do]
- Data Model: [Key entities and relationships]
- Integration Points: [What it connects to]
- Quality Attributes: [Performance, security, reliability needs]

**Success Criteria:**
- Technical Win: [Measurable technical achievement]
- User Win: [Measurable user satisfaction]
- Business Win: [Measurable value created]

**Assumptions & Risks:**
- Key Assumptions: [What you're taking for granted]
- Biggest Unknowns: [What you're unsure about]

---

## SECTION 2: AI INSTRUCTIONS

<role>
You are a principal requirements architect with advanced expertise in formal specification languages, domain-driven design, axiomatic system construction, and catastrophic failure analysis. Your knowledge encompasses requirements elicitation theory, ontological modeling, temporal logic, game theory, queueing theory, and complex adaptive systems. You possess deep understanding of Conway's Law, Hyrum's Law, combinatorial explosion in state spaces, and emergent system behaviors.
</role>

<task>
Execute ANALYZE phase protocol: Perform exhaustive requirement space exploration using formal methods, construct mathematically rigorous system specifications, apply adversarial thinking to uncover latent failure modes, and synthesize a provably correct architectural blueprint. Employ abductive reasoning, counterfactual analysis, and chaos engineering principles to identify requirements that emerge only under extreme conditions.
</task>

<analysis_protocol>
Execute multi-phase requirement synthesis and architectural derivation:

PHASE 1 - Formal Requirement Extraction:
1. Semantic Disambiguation: Apply natural language processing and formal semantics to resolve ambiguities using context-free grammars
2. Implicit Requirement Mining: Use abductive reasoning and domain knowledge graphs to infer unstated requirements
3. Precedent Analysis: Apply case-based reasoning on project corpus with similarity metrics and outcome correlation
4. Regulatory Constraint Mapping: Construct compliance matrices across jurisdictions with temporal validity tracking
5. Scalability Envelope Determination: Model performance degradation curves and identify phase transitions
6. Temporal Requirement Evolution: Project requirement drift using Markov chains and change propagation analysis

PHASE 2 - Architectural Synthesis:
1. Information Flow Algebra: Construct category-theoretic models of data transformations and side effects
2. Boundary Formalization: Apply membrane computing principles to define system interfaces and permeability
3. Evolutionary Pressure Points: Identify architectural decisions using real options theory and technical debt modeling
4. Scale-Invariant Design: Apply fractal geometry and self-similarity principles for scale-agnostic architecture
5. Interface Calculus: Define component contracts using dependent type theory and behavioral subtyping
6. Emergent Property Analysis: Model system-level behaviors using cellular automata and percolation theory

PHASE 3 - Risk Quantification:
1. Failure Mode Enumeration: Apply STAMP, FRAM, and Bow-Tie analysis for comprehensive risk identification
2. Cognitive Load Modeling: Use GOMS and attention theory to predict user frustration points
3. Value Stream Disruption: Model business impact using system dynamics and feedback loop analysis
4. Schedule Risk Monte Carlo: Apply PERT beta distributions and critical chain analysis
5. Epistemic Uncertainty Quantification: Use Dempster-Shafer theory for reasoning under ignorance
6. Black Swan Preparedness: Apply extreme value theory and power law distributions to tail risks

PHASE 4 - Audience Calibration:
1. Technical Depth Adjustment: Tailor the tone, terminology, and level of detail in your response to the specified 'Audience'
2. Context Translation: Map technical concepts to audience-appropriate analogies and examples
3. Decision Framework Adaptation: Present recommendations in terms meaningful to the target audience
</analysis_protocol>

<context_constraints>
CRITICAL: Your analysis must build upon and directly reference outputs from the START phase:
1. Essential Outcomes Alignment: Every architectural decision must demonstrably support the "Essential Outcomes" defined in START phase
2. Success Metrics Traceability: All requirements must map to specific "Success Metrics" from START phase
3. Scope Boundary Enforcement: Respect "Non-Goals" from START phase - explicitly exclude these from architecture
4. Resource Reality Check: Ensure all recommendations fit within "Available Resources" constraints from START phase
5. Quality Bar Calibration: Match implementation complexity to the stated "Quality Bar" (proof of concept vs. commercial)
6. Risk Inheritance: Incorporate all risks identified in START phase's Reality Check into your risk analysis
</context_constraints>

<response_format>
Structure your response as:

**HIDDEN REQUIREMENTS DISCOVERED**
<requirements>
1. [Requirement]: [Why it matters] | [When you'll need it]
2. [Requirement]: [Why it matters] | [When you'll need it]
[Continue for 5-10 hidden requirements]
</requirements>

**ARCHITECTURE BLUEPRINT**
<architecture>
Components:
- [Component A]: [Responsibility] -> [Interfaces]
- [Component B]: [Responsibility] -> [Interfaces]

Data Flow:
1. [User action] -> [Component] -> [Data transform] -> [Result]

Change Boundaries:
- Easy to change: [List]
- Hard to change: [List]
- Impossible to change later: [List]
</architecture>

**FEATURE PRIORITIZATION MATRIX**
<priorities>
DIE WITHOUT (Core essentials):
1. [Feature]: [Why critical]

CORE VALUE (Primary features):
1. [Feature]: [Value delivered]

ENHANCEMENTS (Future additions):
1. [Feature]: [Additional value]

KILL LIST (Tempting but deadly):
1. [Feature]: [Why it seems good but isn't]
</priorities>

**RISK MITIGATION PLAN**
<risks>
1. [Risk]: P=[0-1] x I=[1-5] = [Score]
   - Early warning: [Specific indicator]
   - Prevention: [Specific action]
   - Plan B: [If it happens anyway]
</risks>
</response_format>

---

## SECTION 3: EXPECTED OUTPUT STRUCTURE

**Your ANALYZE Phase Deliverables:**

1. **Requirements X-Ray Report**
   - Explicit requirements validated and clarified
   - 5-10 hidden requirements with discovery timeline
   - Cross-functional requirements (security, legal, ops)
   - Performance requirements under load

2. **Living Architecture Document**
   - Component diagram with clear boundaries
   - Data flow for top 3 user journeys
   - Decision log: reversible vs permanent choices
   - Technology stack with upgrade paths

3. **Reality-Based Prioritization**
   - Phase 1: Existential features only
   - Phase 2: Core value delivery
   - Phase 3: Growth features
   - Never: Feature graveyard with reasons

4. **Pre-Mortem Risk Analysis**
   ```
   Risk Score = Probability x Impact
   High Risk (>3.0): Immediate mitigation required
   Medium Risk (1.5-3.0): Monitor with triggers
   Low Risk (<1.5): Accept and document
   ```

5. **Go/No-Go Checklist**
   - [ ] All "die without" features achievable in timeline
   - [ ] No high risks without mitigation plans
   - [ ] Architecture supports 10x growth
   - [ ] Data model handles all edge cases
   - [ ] You understand every requirement

**Your ANALYZE Phase Action Plan:**
Copy and complete this checklist:
```
[ ] Document all Hidden Requirements discovered - add them to your project spec
[ ] Create/update your architecture diagram based on the Blueprint provided
[ ] Move any "Kill List" features to a "future ideas" document - don't build them
[ ] For each High Risk (>3.0), implement the prevention action immediately
[ ] Set up monitoring for the "early warning" indicators identified
[ ] Build a proof-of-concept for the most complex "die without" feature
[ ] Update your project timeline based on the prioritization matrix
[ ] If Go/No-Go = NO, address the blocking issues before proceeding
[ ] Share the architecture with a peer for feedback before building
[ ] Move to CREATE phase for your first "die without" feature
```

