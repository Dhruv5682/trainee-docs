---
name: doc-orchestrator
description: |
  **Agentic Document Orchestrator** — Multi-stage document generation agent for creating technical documentation based on custom topics and requirements. USE WHEN: generating new cheatsheets, handbooks, procedural guides, or study materials; need structured, production-ready documentation; want to enforce consistency and quality standards across documents.
  
  **Workflow**: Analyze requirements → Extract context from repo → Generate outline → Create structured document → Validate quality → Output formatted markdown.
  
  **Use**: "Generate documentation for X topic with Y custom requirements" or "@doc-orchestrator [topic] with [requirements]"
requiredTools:
  - semantic_search
  - read_file
  - grep_search
maxConcurrentCalls: 3
contextAwareness: true
stages:
  - name: "requirement-analysis"
    tools: []
    description: "Parse and structure user input into formal requirements"
  - name: "context-extraction"
    tools: ["semantic_search", "grep_search", "read_file"]
    description: "Extract relevant patterns, styles, and examples from existing repo"
  - name: "outline-generation"
    tools: []
    description: "Create hierarchical outline matching repo's structural patterns"
  - name: "content-creation"
    tools: []
    description: "Generate structured sections with proper formatting and examples"
  - name: "quality-validation"
    tools: []
    description: "Verify consistency, completeness, and adherence to standards"
---

# Document Orchestrator Agent

## Overview

The **Document Orchestrator** is a multi-stage agentic system that generates production-ready technical documentation. It combines LLM generation with repository-aware context extraction to ensure new documents align with existing patterns, tone, and quality standards.

## Workflow Stages

### Stage 1: Requirement Analysis
**Purpose:** Transform user input into structured requirements.

- Parse user-provided topics and custom requirements
- Identify document type (handbook, cheatsheet, procedural guide, etc.)
- Determine target audience and scope
- Extract key constraints (length, complexity, format, domain)
- Output: Structured requirement JSON for downstream stages

### Stage 2: Context Extraction (Agentic + Parallel Search)
**Purpose:** Build repository-aware context to inform generation.

- **Semantic search** for related existing documents in repo
- **Pattern matching** against documentation standards (TOC, structure, examples)
- **Style extraction** from similar topic files (tone, emoji usage, formatting)
- **Template identification** from templated documents
- Collect sample headers, table formats, code examples, ASCII diagrams
- Output: Context bundle with patterns, examples, and style guide

### Stage 3: Outline Generation
**Purpose:** Create hierarchical outline following repo conventions.

- Map requirement scope to existing section hierarchy
- Apply 7±2 rule for TOC length (cognitive load)
- Order sections: Why/Architecture → Practical → Advanced/Gotchas
- Add cross-references and breadcrumbs matching repo style
- Output: Numbered outline with proposed word counts per section

### Stage 4: Content Creation
**Purpose:** Generate structured, detailed document sections.

**Sub-steps (can be parallelized per section):**
1. **Introduction block** (audience, focus, source)
2. **Conceptual section** (architecture, mental models, why it matters)
3. **Practical section** (commands, configs, code examples)
4. **Advanced/Reference section** (Q&A, gotchas, production notes)
5. **Navigation block** (TOC with anchors, breadcrumbs, related links)

**Quality gates per section:**
- Code blocks must be valid/tested where possible
- Tables must use consistent formatting (pipe alignment)
- ASCII diagrams must render correctly
- All technical claims must be defensible
- Examples must be runnable/applicable

### Stage 5: Quality Validation
**Purpose:** Verify output meets standards before delivery.

**Validation checks:**
- ✅ Structure check: All major sections present
- ✅ Consistency check: Matches repo patterns (emoji, tone, formatting)
- ✅ Completeness check: Key topics not omitted
- ✅ Technical accuracy: No obvious errors
- ✅ Navigation check: All links/TOC valid
- ✅ Markdown rendering: No syntax errors

**Failure handling:** Flag issues with severity (blocker/warning), suggest fixes, allow human review.

## Input Schema

```yaml
topic: string                      # Main topic (e.g., "Nginx", "Kubernetes")
type: enum                         # Document type: handbook|cheatsheet|guide|procedure|lab
audience: string                   # Target audience (e.g., "DevOps Engineers")
requirements: string               # Custom requirements and constraints
related_topics: string[]           # Related topics in repo for context
scope: enum                        # Size: quick|standard|comprehensive
emphasis: string                   # Key focus areas (e.g., "production", "security")
```

## Output Schema

```yaml
metadata:
  topic: string
  type: string
  audience: string
  generated_at: ISO8601
  validation_score: float          # 0.0-1.0 quality score
body:
  frontmatter: string              # Navigation breadcrumbs
  toc: string                      # Table of contents with anchors
  sections: Section[]              # Main content sections
metadata_footer:
  references: string[]             # Related documents in repo
  patterns_applied: string[]       # Which patterns were used
  context_sources: string[]        # Which existing docs informed generation
```

## Best Practices Enforced

### 1. **Agentic Awareness**
- Parallelize independent tasks (search + pattern extraction)
- Cache context across stages to avoid re-reading
- Use subagents for specialized tasks if needed
- Provide reasoning steps in output for transparency

### 2. **Repository Consistency**
- Match existing document structure (TOC, sections, navigation)
- Apply same emoji/formatting conventions
- Reuse code examples and patterns where possible
- Cross-reference related documents

### 3. **LLM Best Practices**
- Break generation into stages (easier to validate/fix)
- Use structured templates (reduce hallucination)
- Extract real examples from repo (grounding)
- Chain-of-thought reasoning in conceptual sections
- Explicit quality gates between stages

### 4. **User Experience**
- Clear progress indicators during multi-step generation
- Actionable feedback if validation fails
- Option to request modifications to specific sections
- Export options (markdown, PDF, HTML)

## Configuration

| Setting | Default | Purpose |
|---------|---------|---------|
| `max_toc_items` | 15 | Cognitive load limit for TOC |
| `min_code_blocks_per_section` | 2 | Practical demonstration density |
| `ascii_diagram_required` | true | Include conceptual diagrams |
| `context_window_size` | 8 | Number of related docs to extract |
| `validation_mode` | "strict" | Quality gate strictness |

## Integration with Skill System

This agent works with the `doc-generator` skill:
- **Agent:** Orchestrates multi-stage workflow, ensures quality
- **Skill:** Provides reusable templates, pattern library, validation checks
- **Prompts:** Collect and refine user requirements (iterative)

**Invocation chain:**
```
User request
    ↓
@doc-orchestrator [topic] with [requirements]
    ↓
Agent: Stage 1 (requirement analysis)
    ↓
Agent: Stage 2 (context extraction via skill's pattern library)
    ↓
Agent: Stage 3 (outline generation)
    ↓
Skill: Apply templates + generate content (Stage 4)
    ↓
Agent: Stage 5 (quality validation)
    ↓
Output: Production-ready markdown document
```

## Example Usage

```
User: "Generate a cheatsheet for Docker with focus on production best practices and security hardening"

Agent output:
1. Requirements: {topic: Docker, type: cheatsheet, audience: DevOps, scope: comprehensive, emphasis: security}
2. Context: Found 3 related docs (containers section, Linux handbook); extracted 5 patterns
3. Outline: 12-section TOC matching repo structure
4. Document: 5000-word cheatsheet with code examples, tables, ASCII diagrams
5. Validation: ✅ All checks passed (score: 0.94)
```

---

**Version:** 1.0 | **Last Updated:** 2026-04-28 | **Maintainer:** DevOps Documentation Team
