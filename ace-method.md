# ACE Method

Ship code that works. Fast.

## Overview

ACE is a lightweight framework for rapid development. Three phases, minimal overhead, maximum velocity.

- **Analyze** → Find the real problem
- **Create** → Build the simplest solution
- **Evaluate** → Ensure it won't break

## Quick Start for LLMs

When given a development task, generate this checklist:

```markdown
## ACE Implementation Plan

**ANALYZE**
- [ ] Problem: [one sentence description]
- [ ] Constraints: [technical limitations]
- [ ] Solution: [simplest approach]

**CREATE**
- [ ] Build core feature
- [ ] Add error handling
- [ ] Deploy to staging

**EVALUATE** 
- [ ] Test main flows
- [ ] Deploy to production
- [ ] Add monitoring
```

## The Framework

### Analyze
*Time: Minutes, not hours*

- [ ] Problem statement (one sentence)
- [ ] Technical constraints
- [ ] Simplest viable approach

### Create  
*Time: Hours, not days*

- [ ] Core functionality only
- [ ] Basic error handling
- [ ] Deploy to staging

### Evaluate
*Time: Ship when ready*

- [ ] Tests for critical paths
- [ ] Production deployment
- [ ] Basic monitoring

## Core Principles

**Speed over perfection**  
Working code in production beats perfect code in development.

**Iterate in production**  
Real users provide better feedback than any planning session.

**10x headroom**  
Build for current load, architect for 10x growth.

## Anti-patterns

- Analysis paralysis
- Feature creep  
- Premature optimization
- Over-engineering

## Example

```
ANALYZE: Users can't reset passwords
→ Email service exists, just needs reset flow

CREATE: Password reset endpoint + email template
→ 2 hours, deployed to staging  

EVALUATE: Tested happy path + rate limiting
→ Shipped to production, monitoring 200s
```

## FAQ

**Q: What about edge cases?**  
A: Ship first, iterate based on real usage.

**Q: When to use ACE?**  
A: New features, prototypes, any "just ship it" scenario.

**Q: What about technical debt?**  
A: Refactor when you hit scale, not before.

## References

Inspired by:
- [OpenAI Cookbook](https://cookbook.openai.com/) - Rapid prototyping patterns
- [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook) - Production-ready examples
- [Together AI Cookbooks](https://www.together.ai/cookbooks) - Scalable deployment strategies
- [Hugging Face Learn](https://huggingface.co/learn/cookbook/en/index) - Quick implementation guides

---

*ACE Method: Because perfect is the enemy of shipped.*