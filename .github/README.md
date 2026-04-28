# 📋 Document Orchestration System

A reusable, agentic system for generating production-quality technical documentation based on custom topics and user requirements. Built on LLM/agentic AI best practices with repository-aware context extraction.

---

## Overview

This system enables intelligent agents to generate documentation that:
- ✅ Aligns with existing repository patterns and conventions
- ✅ Follows technical writing best practices
- ✅ Maintains consistent quality through multi-stage validation
- ✅ Extracts real examples from your documentation repository
- ✅ Supports multiple document types (handbooks, cheatsheets, guides, etc.)

### Key Components

| Component | Purpose | File |
|-----------|---------|------|
| **Doc Orchestrator Agent** | Multi-stage workflow orchestration | `doc-orchestrator.agent.md` |
| **System Instructions** | Operational guardrails & decision logic | `doc-orchestrator.instructions.md` |
| **Doc Generator Skill** | Templates, patterns, validation framework | `SKILL.md` |
| **Pattern Library** | Extracted patterns from existing docs | `patterns.md` |
| **Requirements Prompt** | Structured input collection | `collect-doc-requirements.prompt.md` |

---

## Quick Start

### For Users: Generate Documentation

```
User: "Generate a Kubernetes handbook with focus on deployment patterns and best practices"

Steps:
1. @doc-orchestrator kubernetes handbook deployment-patterns
2. System asks clarifying questions via /collect-doc-requirements
3. Agent extracts context from related repo docs
4. Agent generates 5-stage: analyze → extract → outline → create → validate
5. Output: Production-ready markdown document (kubernetes-handbook.md)
```

### For Developers: Extend the System

**To add a new pattern:**
1. Document it in `patterns.md` with real examples
2. Update pattern frequency/status
3. Add to appropriate template in SKILL.md

**To update templates:**
1. Edit `SKILL.md` → "Template Library" section
2. Test new template with sample generation
3. Document in `patterns.md` if creating new pattern

**To customize workflow stages:**
1. Edit `doc-orchestrator.instructions.md` → "Stage-by-Stage Guardrails"
2. Update corresponding stage logic in agent
3. Test end-to-end generation

---

## Architecture

### 5-Stage Generation Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                   DOCUMENT GENERATION PIPELINE                      │
└─────────────────────────────────────────────────────────────────────┘

User Input (topic + requirements)
    │
    ▼
┌──────────────────────────────────────────────────────────────────┐
│ STAGE 1: REQUIREMENT ANALYSIS (1-2 sec)                          │
│ • Parse user input                                               │
│ • Detect document type                                           │
│ • Identify audience & scope                                      │
│ Output: Structured requirement JSON                              │
└──────────────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────────────┐
│ STAGE 2: CONTEXT EXTRACTION (10-30 sec, PARALLEL)               │
│ • Semantic search for related docs                               │
│ • Pattern extraction from similar files                           │
│ • Style guide extraction                                         │
│ • Template selection                                             │
│ Output: Context bundle (patterns, examples, style)               │
└──────────────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────────────┐
│ STAGE 3: OUTLINE GENERATION (3-5 sec)                            │
│ • Apply section templates                                        │
│ • Generate hierarchical TOC                                      │
│ • Assign word counts per section                                 │
│ • Map to extracted patterns                                      │
│ Output: Detailed outline with anchors                            │
└──────────────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────────────┐
│ STAGE 4: CONTENT GENERATION (30-120 sec)                         │
│ • Generate intro + conceptual sections                           │
│ • Generate practical sections with examples                      │
│ • Generate advanced/Q&A sections                                 │
│ • Apply formatting from context bundle                           │
│ Output: Complete markdown document                               │
└──────────────────────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────────────────────┐
│ STAGE 5: QUALITY VALIDATION (5-10 sec)                           │
│ • Structure check (sections present)                             │
│ • Consistency check (style matches repo)                         │
│ • Completeness check (no gaps)                                   │
│ • Technical check (no errors)                                    │
│ • Usefulness check (solves problem)                              │
│ Output: Validation report + quality score (0.0-1.0)             │
│ Threshold: ≥0.80 for publication                                │
└──────────────────────────────────────────────────────────────────┘
    │
    ▼
Production-ready document OR failure report with suggestions
```

### Component Interaction

```
                    ┌────────────────────────────┐
                    │   doc-orchestrator.agent   │
                    │   (Multi-stage workflow)   │
                    └────────────────┬───────────┘
                                     │
                 ┌───────────────────┼───────────────────┐
                 │                   │                   │
                 ▼                   ▼                   ▼
    ┌─────────────────────┐ ┌──────────────────┐ ┌─────────────────┐
    │   Stage 1: Parse    │ │   Stage 2: Use   │ │ Stage 3-5: Use  │
    │  requirements via   │ │  doc-generator   │ │  skill patterns │
    │      prompt         │ │   skill (SKILL   │ │ & validation    │
    │                     │ │      .md)        │ │                 │
    └─────────────────────┘ └──────────────────┘ └─────────────────┘
                 │                   │                   │
                 │                   │                   │
    /collect-doc-requirements    patterns.md         SKILL.md
         (Prompt)                  (Library)        (Templates)
                                        │
                                        ▼
                             ┌────────────────────┐
                             │  Repository Docs   │
                             │  (for context)     │
                             └────────────────────┘
```

---

## File Structure

```
.github/
├── agents/
│   ├── doc-orchestrator.agent.md           ← Main orchestrator agent
│   └── doc-orchestrator.instructions.md    ← Operational guardrails
│
├── skills/
│   └── doc-generator/
│       ├── SKILL.md                        ← Templates & validation
│       ├── patterns.md                     ← Extracted patterns library
│       └── templates/                      ← Template files (optional)
│           ├── handbook-template.md
│           ├── cheatsheet-template.md
│           └── procedure-template.md
│
└── prompts/
    └── collect-doc-requirements.prompt.md  ← Requirements collection

```

---

## Pattern Library (Extracted)

This system includes **12 extracted patterns** from your existing documentation:

| # | Pattern | Example Docs | Frequency |
|---|---------|--------------|-----------|
| 1 | Breadcrumb Navigation | All long docs | 100% |
| 2 | Audience Awareness | Handbooks, Guides | 95% |
| 3 | Table of Contents | Long-form docs | 85% |
| 4 | Mental Models + Diagrams | Architecture docs | 60% |
| 5 | Progressive Difficulty | Task files, Labs | 50% |
| 6 | Command/Config Blocks | Practical docs | 80% |
| 7 | Reference Tables | Cheatsheets, Advanced | 90% |
| 8 | "Why It Matters" Sections | Handbooks | 70% |
| 9 | Production Gotchas | Advanced sections | 65% |
| 10 | ASCII Architecture Diagrams | System docs | 45% |
| 11 | Emoji Visual Hierarchy | All modern docs | 100% |
| 12 | Cross-References | Integrated docs | 75% |

**See:** `patterns.md` for detailed pattern specifications with real examples.

---

## Document Types Supported

### Type 1: Handbook (Comprehensive)
- **Scope:** 5,000-15,000 words
- **Use case:** Complete guide, reference, learning material
- **Patterns:** Most patterns (10+/12)
- **Time to generate:** 3-5 minutes

**Example templates:**
- Handbook structure (comprehensive-structure pattern)
- Progressive difficulty (optional)
- Mental models + diagrams

### Type 2: Cheatsheet (Quick Reference)
- **Scope:** 1,500-3,000 words
- **Use case:** Quick lookups, command reference
- **Patterns:** Minimal patterns (5-6/12)
- **Time to generate:** 1-2 minutes

**Example templates:**
- Command lookup table
- Quick reference pattern
- Syntax highlighting

### Type 3: Procedural Guide (Step-by-Step)
- **Scope:** 2,000-5,000 words
- **Use case:** Lab guides, how-to, tutorials
- **Patterns:** Procedural focus (6-7/12)
- **Time to generate:** 2-3 minutes

**Example templates:**
- Step sequence
- Error handling
- Verification steps

### Type 4: Study Material (Learning)
- **Scope:** 3,000-8,000 words
- **Use case:** Training, education, certification prep
- **Patterns:** Progressive difficulty + conceptual (8-9/12)
- **Time to generate:** 2-4 minutes

### Type 5: Q&A / Troubleshooting (Reference)
- **Scope:** 1,000-2,000 words
- **Use case:** FAQs, gotchas, production issues
- **Patterns:** Problem-solution focus (5-6/12)
- **Time to generate:** 1-2 minutes

---

## How to Use This System

### Scenario 1: Generate a New Handbook

```bash
User: Generate a comprehensive Docker handbook with focus on production deployments

Agent:
  1. Parse: {type: handbook, topic: Docker, focus: production, scope: comprehensive}
  2. Extract: Find containerization patterns in linux_admin_handbook.md
  3. Outline: 12-section structure (Why → Concepts → Practical → Advanced)
  4. Generate: 8,000-word handbook with mental models, commands, gotchas
  5. Validate: Score 0.92 ✅ → Publish
```

### Scenario 2: Add Quick Reference Cheatsheet

```bash
User: Create a PostgreSQL cheatsheet for SQL commands and backup procedures

Agent:
  1. Parse: {type: cheatsheet, topic: PostgreSQL, scope: quick}
  2. Extract: Find database/command patterns in existing docs
  3. Outline: 5 main sections (Quick Ref → Common Tasks → Admin)
  4. Generate: 2,500-word cheatsheet with command tables, examples
  5. Validate: Score 0.88 ✅ → Publish
```

### Scenario 3: Create Lab Guide for Training

```bash
User: Design a hands-on lab for Kubernetes networking with progressive difficulty

Agent:
  1. Parse: {type: procedure, topic: K8s networking, scope: standard, difficulty: progressive}
  2. Extract: Find lab patterns from linux_etc_disaster_recovery.md
  3. Outline: 3 difficulty levels (Basic → Intermediate → Advanced)
  4. Generate: Level 1: basic services, Level 2: ingress, Level 3: network policies
  5. Validate: Score 0.85 ✅ → Publish
```

---

## Quality Standards

### Validation Checklist

- **Structure:** ✅ All major sections present, TOC valid, no missing parts
- **Consistency:** ✅ Emoji/tone/formatting match repo style
- **Completeness:** ✅ Key topics included, no generic/placeholder text
- **Technical:** ✅ Code examples valid, markdown renders, links work
- **Usefulness:** ✅ Solves stated problem, appropriate depth for audience

### Quality Scoring

Each criterion weighted 20%:
- **Completeness (20%):** All sections present; no major gaps
- **Accuracy (20%):** No technical errors; claims defensible
- **Clarity (20%):** Easy to understand; well-organized
- **Consistency (20%):** Matches repo patterns; tone/style aligned
- **Usefulness (20%):** Practical; solves reader's problems

**Publication threshold:** ≥ 0.80 (80%)

---

## Advanced Usage

### Customizing Patterns

To create a new organization-specific pattern:

1. **Document the pattern:** Add to `patterns.md`
   ```markdown
   ### Pattern N: [Name]
   
   **Frequency:** X% of docs
   **Purpose:** Why this matters
   **Format:** [Example]
   ```

2. **Add examples:** Include real references from your repo

3. **Update templates:** Add to SKILL.md template library if it affects generation

4. **Test:** Generate sample doc using new pattern; validate

### Extending Templates

To create a domain-specific template (e.g., for security docs):

1. **Create template:** `.github/skills/doc-generator/templates/security-template.md`
   ```markdown
   # [Topic] — Security Guide
   
   > **Scope:** [Security domains covered]
   > **Audience:** [Security roles]
   
   ## Threat Model
   [What can go wrong]
   
   ## Mitigations
   [How to protect]
   ...
   ```

2. **Update SKILL.md:** Add to "Template Library" section

3. **Update instructions:** Reference new template in `doc-orchestrator.instructions.md`

4. **Test:** Generate security document using new template

---

## Integration with Version Control

### Recommended Workflow

```
1. Generate draft via /doc-orchestrator [topic]
2. Review generated document (validation score, content, examples)
3. Make manual edits if needed (refine, add context)
4. Commit to feature branch: git commit -m "docs: add [topic] handbook"
5. Create PR with documentation generated by agent
6. Team reviews, suggests improvements
7. Merge to main when ready
```

### CI/CD Integration (Optional)

Add to your CI pipeline:
```bash
# Validate all docs match patterns
python3 .github/scripts/validate_docs.py --patterns .github/skills/doc-generator/patterns.md

# Auto-check: broken links, missing anchors
markdownlint --config .github/markdownlint.json docs/
```

---

## Troubleshooting

### Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Generated doc doesn't match repo style | Patterns not extracted correctly | Check context bundle in Stage 2; retry with similar topic |
| Validation score < 0.80 | Quality issues in generated content | Review failures; regenerate specific sections |
| TOC anchors don't work | Anchor format incorrect | Verify anchor format matches GitHub markdown standards |
| Code examples have syntax errors | LLM generation error | Use `/collect-doc-requirements` to request syntax validation |

### Getting Better Results

1. **Be specific with requirements:** "Kubernetes deployment patterns" > "Kubernetes"
2. **Reference related docs:** "Similar to [existing-doc.md]" helps context extraction
3. **Set scope explicitly:** "Quick" vs "comprehensive" changes generation drastically
4. **Iterate:** Validate output, request fixes to specific sections

---

## Best Practices

### For Generating High-Quality Docs

1. **Provide context:** Mention related existing documents (helps pattern extraction)
2. **Set realistic scope:** Don't ask for "quick" handbook or "comprehensive" cheatsheet
3. **Use clarifying prompts:** Let agent ask via `/collect-doc-requirements` before generating
4. **Review and iterate:** No doc is perfect on first generation; expect to refine
5. **Test examples:** Run any code examples before publishing

### For Maintaining Pattern Library

1. **Keep patterns DRY:** Don't duplicate patterns in multiple sections of SKILL.md
2. **Update frequency:** Quarterly review of pattern applicability
3. **Add new patterns:** Only when pattern appears in 3+ documents
4. **Remove deprecated:** If pattern no longer matches repo style, mark as "legacy"
5. **Document sources:** Always reference which docs exemplify each pattern

### For Extending the System

1. **Don't skip validation:** Always include Stage 5 quality checks
2. **Test end-to-end:** Generate sample doc, validate, commit to test repo
3. **Document changes:** Update relevant .md files when modifying system
4. **Get feedback:** Run new patterns/templates by team before publishing
5. **Version your work:** Tag releases of major system updates

---

## System Performance

### Generation Time Estimates

| Document Type | Scope | Estimated Time |
|---------------|-------|-----------------|
| Cheatsheet | Quick (1.5k words) | 1-2 min |
| Guide | Standard (3-5k words) | 2-3 min |
| Handbook | Comprehensive (8-15k words) | 3-5 min |
| Study Material | Progressive difficulty | 2-4 min |
| Lab Guide | Step-by-step (3k words) | 2-3 min |

### Context Window Usage

- **Stage 1:** ~1KB (requirement JSON)
- **Stage 2:** ~50KB (extracted patterns + examples)
- **Stage 3:** ~5KB (outline)
- **Stage 4:** Variable by scope (30KB-300KB for generated content)
- **Stage 5:** ~10KB (validation report)

**Total:** 50-400KB depending on document size

---

## Files Reference

| File | Purpose | When to Edit |
|------|---------|--------------|
| `doc-orchestrator.agent.md` | Agent definition & workflow | Changing generation stages |
| `doc-orchestrator.instructions.md` | Decision logic & guardrails | Refining stage logic |
| `SKILL.md` | Templates & validation framework | Adding new templates |
| `patterns.md` | Pattern library & examples | Adding/updating patterns |
| `collect-doc-requirements.prompt.md` | Input collection interview | Changing requirement fields |

---

## Getting Started

### 1. Install/Setup
No installation needed — system is file-based.

### 2. First Generation
```
@doc-orchestrator [topic] with [custom requirement]
```

Example:
```
@doc-orchestrator Docker with focus on production hardening and security best practices
```

### 3. Review Output
Check:
- Validation score ≥ 0.80 ✅
- Structure matches your expectations
- Examples are relevant
- Cross-links work

### 4. Refine & Publish
- Manual edits if needed
- Commit to repo
- Add to README.md index

---

## Maintenance & Support

**System Version:** 1.0  
**Last Updated:** 2026-04-28  
**Maintained by:** DevOps Documentation Team  
**Review Schedule:** Quarterly

**To report issues or suggest improvements:**
1. Create issue in repo with `[doc-system]` prefix
2. Provide: problem description, example, expected behavior
3. Tag: documentation, agent-system

---

## References

- **Agent Definition:** [doc-orchestrator.agent.md](./agents/doc-orchestrator.agent.md)
- **System Instructions:** [doc-orchestrator.instructions.md](./agents/doc-orchestrator.instructions.md)
- **Skill & Templates:** [SKILL.md](./skills/doc-generator/SKILL.md)
- **Pattern Library:** [patterns.md](./skills/doc-generator/patterns.md)
- **Requirements Prompt:** [collect-doc-requirements.prompt.md](./prompts/collect-doc-requirements.prompt.md)

---

**Questions?** Check the relevant file above or contact the documentation team.
