# ACE Method Repository

A comprehensive framework for systematic software development using the Analyze, Create, Evaluate methodology.

## Repository Structure

This repository contains six interconnected documents that work together to guide you through the complete ACE Method:

### Core Framework
1. **[ACE-Method.md](./ACE-Method.md)**: Universal framework overview for all application types
2. **[CLI-ACE-Method.md](./CLI-ACE-Method.md)**: Specialized guide for command line interface development

### Phase-Specific Guides
3. **[01 Analyze.md](./01 Analyze.md)**: Deep technical dive into the Analysis phase
4. **[02 Create.md](./02 Create.md)**: Detailed implementation strategies and patterns
5. **[03 Evaluate.md](./03 Evaluate.md)**: Comprehensive validation and optimization techniques

### Getting Started
6. **README.md** (this file): Navigation and quick start guide

## How to Use This Repository

### For Individual Developers

1. **Choose your starting point**:
   - General applications: Start with ACE-Method.md
   - CLI applications: Start with CLI-ACE-Method.md
2. **Use phase-specific files** (01-03) as deep references during each phase
3. **Copy templates** from each file into your project documentation
4. **Customize checklists** based on your technology stack and application type

### For Teams

1. **Onboard new members** with ACE-Method.md as required reading
2. **Establish phase gates** using the detailed criteria in each phase file
3. **Create project-specific versions** of the templates for consistency
4. **Use as code review criteria** to ensure quality standards

### For LLMs and AI Assistants

When implementing the ACE Method:

1. **Identify application type** first (Web, Mobile, Desktop, CLI, API, etc.)
2. **Reference appropriate framework**:
   - CLI applications: Use CLI-ACE-Method.md
   - All other types: Use ACE-Method.md
3. **Generate phase-specific documentation** using templates from phase files (01-03)
4. **Apply technical details** from phase files for deeper implementation
5. **Validate completeness** against all checklists and criteria

## Quick Start Guide

### New Project Setup

```bash
# 1. Copy this repository structure to your project
cp -r ace-method/ your-project/docs/

# 2. Create project-specific ACE documentation
mkdir your-project/docs/ace-implementation
touch your-project/docs/ace-implementation/analysis.md
touch your-project/docs/ace-implementation/creation.md
touch your-project/docs/ace-implementation/evaluation.md

# 3. Use templates from phase files to populate your docs
```

### Implementation Workflow

**Process Flow:**

1. **Start**: Identify your application type
2. **Choose Framework**:
   - CLI applications: Read CLI-ACE-Method.md
   - Other applications: Read ACE-Method.md
3. **ANALYZE Phase**: Use templates from [01 Analyze.md](./01 Analyze.md)
4. **CREATE Phase**: Apply patterns from [02 Create.md](./02 Create.md)
5. **EVALUATE Phase**: Follow criteria in [03 Evaluate.md](./03 Evaluate.md)
6. **Production**: Deploy and monitor
7. **Iterate**: Return to ANALYZE with learnings

## Phase Navigation

### ANALYZE Phase ([01 Analyze.md](./01 Analyze.md))
- Problem decomposition techniques
- Risk assessment methodologies
- Architecture decision records
- Constraint analysis frameworks
- Stakeholder mapping strategies

### CREATE Phase ([02 Create.md](./02 Create.md))
- Implementation patterns by technology
- Security implementation guides
- Performance optimization techniques
- Testing strategy templates
- CI/CD pipeline configurations

### EVALUATE Phase ([03 Evaluate.md](./03 Evaluate.md))
- Comprehensive testing methodologies
- Performance benchmarking guides
- Security audit procedures
- Monitoring and observability setup
- Production readiness checklists

## Best Practices for Using ACE Method

### 1. Iterative Application
- Complete all three phases for each feature
- Use micro-iterations for rapid feedback
- Document learnings in each phase

### 2. Template Customization
- Start with provided templates
- Adapt to your technology stack
- Maintain consistency across projects

### 3. Continuous Improvement
- Review effectiveness after each project
- Update templates based on learnings
- Share improvements with the community

## Integration with Development Tools

### IDE Integration
```json
// .vscode/settings.json
{
  "aceMethod.templates": "./docs/ace-method/",
  "aceMethod.currentPhase": "analyze",
  "aceMethod.autoChecklistGeneration": true
}
```

### CI/CD Integration
```yaml
# .github/workflows/ace-validation.yml
name: ACE Method Validation
on: [push, pull_request]

jobs:
  validate-ace:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Validate ANALYZE phase completion
        run: ./scripts/validate-analysis.sh
      - name: Validate CREATE phase standards
        run: ./scripts/validate-implementation.sh
      - name: Validate EVALUATE phase criteria
        run: ./scripts/validate-testing.sh
```

## Application-Specific Guidance

### Command Line Interfaces (CLI)
- **Unique Considerations**: Keyboard-first, scriptability, cross-platform compatibility
- **Specialized Guide**: See [CLI-ACE-Method.md](./CLI-ACE-Method.md)
- **Key Differences**: Focus on argument parsing, exit codes, pipe support

### Web Applications
- **Framework Agnostic**: Works with React, Vue, Angular, or vanilla JS
- **Considerations**: Browser compatibility, responsive design, SEO
- **Use**: Standard ACE-Method.md with web-specific templates

### Mobile Applications
- **Platform Coverage**: iOS, Android, React Native, Flutter
- **Considerations**: Platform guidelines, offline support, app store requirements
- **Use**: Standard ACE-Method.md with mobile-specific patterns

### Desktop Applications
- **Technologies**: Electron, native frameworks, cross-platform tools
- **Considerations**: OS integration, file system access, native performance
- **Use**: Standard ACE-Method.md with desktop-specific considerations

## Contributing

We welcome contributions to improve the ACE Method:

1. **Framework Improvements**: Submit PRs to ACE-Method.md or CLI-ACE-Method.md
2. **Phase Enhancements**: Improve specific phase documentation (01-03)
3. **Case Studies**: Add real-world examples to any file
4. **Tool Integration**: Share automation scripts and configurations
5. **New Specializations**: Propose new application-specific guides

## Support and Community

- **Issues**: Report bugs or suggest improvements
- **Discussions**: Share experiences and best practices
- **Wiki**: Find additional resources and case studies

## License

This framework is available under the MIT License. See LICENSE file for details.

---

**Author**: Max Marquardt  
**Site**: [mlot.ai](https://mlot.ai)

**Remember**: The ACE Method is a living framework. It grows stronger with each application and contribution from the community.