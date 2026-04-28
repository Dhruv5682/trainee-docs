---
name: doc-generator
user-invocable: true
description: |
  **Reusable Document Generation Skill** — Templates, patterns, and validation framework for creating technical documentation at scale. USE WHEN: generating technical cheatsheets, handbooks, procedural guides; need consistent structure and quality; want to leverage extracted repository patterns for new documents.
  
  **Contains**: Pattern library (extracted from repo), templates, validation checklists, formatting standards. Pairs with `doc-orchestrator` agent for multi-stage generation workflows.
---

# Document Generator Skill

## Overview

The Document Generator Skill provides a structured framework for generating production-quality technical documentation. It includes:

- **Pattern Library:** Extracted from existing repository documents
- **Template Library:** Reusable structures for common document types
- **Validation Framework:** Quality checks and consistency rules
- **Formatting Standards:** Style guide and best practices
- **Examples & References:** Real-world samples from the repo

## Quick Start

### For Handbook/Comprehensive Guide
```
@doc-orchestrator Generate a comprehensive handbook for [topic].
Audience: [role]. Depth: handbook.
Apply patterns: comprehensive-structure, mental-models, progressive-difficulty.
Include inline CLI examples, ASCII diagrams, and gotchas per section.
```

### For Cheatsheet
```
@doc-orchestrator Generate a cheatsheet for [topic].
Format: compressed lookup tables only, no explanations.
Apply patterns: quick-reference, command-grouping, syntax-focus.
```

### For Procedural Guide / Lab Pack
```
@doc-orchestrator Generate a practice lab pack for [topic].
Include portal walkthrough, CLI walkthrough, verification, and cleanup per lab.
Apply patterns: step-sequence, error-handling, recovery.
```

## Pattern Library

### Extracted Patterns from Repository

All patterns below were extracted from existing documents (`linux_admin_handbook.md`, `nginx.md`, `windows_server_2022_cheatsheet.md`, task files).

#### 1. Document Architecture Pattern
**Name:** `comprehensive-structure`
**Applies to:** Handbooks, guides
**Components:**
```
[Breadcrumb Navigation]
    ↓
[Title + Description Block]
    - Audience statement
    - Focus/emphasis
    - Source attribution
    ↓
[Table of Contents] (7±2 items)
    ↓
[Section 1: Conceptual/Why]
    - Mental model
    - Architecture diagram
    - Problem statement
    ↓
[Section 2-N: Practical/How]
    - Commands/configs
    - Tables for reference
    - Code examples
    - Inline explanations
    ↓
[Final Section: Q&A/Gotchas]
    - Production pitfalls
    - Common mistakes
    - Comparison matrix (X vs Y)
    ↓
[Revision Card] (optional)
    - Quick lookup table
```

**Example:** [Linux Admin Handbook](../linux/linux_admin_handbook.md) follows this precisely.

---

#### 2. Audience Awareness Pattern
**Name:** `audience-focused`
**Usage:** Set clear context upfront
**Template:**
```markdown
> **Audience:** [Role] | **Focus:** [Key domains] | **Source:** [Origin/training]
> **Tip:** [How to use this document]
```

**Real example from repo:**
```markdown
> **Audience:** DevOps / Cloud Engineers & Trainees | **Focus:** Architecture, Mental Models, Why it Matters  
> **Source:** Complete Linux Training Course to Get Your Dream IT Job 2026  
> **Tip:** Sections are grouped by domain for logical revision flow.
```

**Why it works:** Readers immediately understand if this doc is for them.

---

#### 3. Mental Model + Diagram Pattern
**Name:** `mental-models-with-diagrams`
**Usage:** Make abstract concepts concrete
**Components:**
- Textual explanation (3-5 sentences)
- ASCII architecture diagram or comparison table
- Connection to practical implications

**Real example from repo:**
```markdown
### 1.2 Architecture Diagram
[ASCII box diagram showing NGINX process model]

### 1.3 Why Event-Driven Matters
[Comparison table: Apache vs Nginx]
  Apache: 1 request = 1 thread = 10K connections = 10GB RAM = ❌
  Nginx:  thousands of connections per worker = 50MB RAM = ✅
```

**When to use:** Explaining system design, performance implications, "why X is better than Y"

---

#### 4. Progressive Difficulty Pattern
**Name:** `progressive-difficulty`
**Usage:** Scaffold learning from basic → advanced
**Structure:**
- **Level 1 (Basic):** Foundational concepts, simple commands, "how to get started"
- **Level 2 (Intermediate):** Advanced options, configurations, troubleshooting
- **Level 3 (Expert):** Production patterns, optimization, edge cases

**Real example from repo:** `tasks/linux/level_1.md`, `level_2.md`, `level_3.md`

**Indicators in text:**
- Level 1: "Basic user creation", "Essential commands"
- Level 2: "Advanced options", "Customization"
- Level 3: "Production hardening", "Performance tuning"

---

#### 5. Command/Config Block Pattern
**Name:** `practical-blocks`
**Usage:** Show how-to in bite-sized chunks
**Structure:**
```markdown
### [Subheading — Action Verb]

[1-2 sentence description]

\`\`\`bash
# Basic example (always provide this first)
command arg1 arg2
\`\`\`

**Advanced option:**
\`\`\`bash
# Complex version with flags/options
command --flag1=value --flag2 | additional_pipe
\`\`\`

| Flag | Meaning |
|------|---------|
| `--flag1` | Description of what this does |
| `--flag2` | Description of what this does |
```

**Real example from repo:**
```bash
# Create a New User
# Basic
sudo useradd -m -s /bin/bash johndoe

# Advanced with specific options
sudo useradd -m -s /bin/bash -u 1050 -c "John Doe" -G sudo,docker johndoe

| Flag | Meaning |
| `-m` | Create home directory |
| `-s` | Set login shell |
```

---

#### 6. Reference Table Pattern
**Name:** `quick-reference-tables`
**Usage:** For lookups and comparisons
**Styles:**

**Simple lookup table:**
```markdown
| Command | Purpose | Example |
|---------|---------|---------|
| `cat /etc/passwd` | Show users | `cat /etc/passwd | grep admin` |
| `groups username` | Show user groups | `groups deploy` |
```

**Feature/flag comparison:**
```markdown
| Feature | Nginx | Apache |
|---------|-------|--------|
| Model | Event-driven | Thread-per-request |
| RAM for 10K connections | 50 MB | 10 GB |
| Config complexity | Simpler | More complex |
```

**File/location reference:**
```markdown
| File | Purpose | Edit with |
|------|---------|-----------|
| `/etc/nginx/nginx.conf` | Main config | `visudo` for safety |
| `/etc/nginx/sites-enabled/` | Virtual hosts | Text editor |
```

---

#### 7. "Why It Matters" Pattern
**Name:** `context-and-relevance`
**Usage:** Explain business/technical importance
**Template:**
```markdown
### Why [Topic]?

As a [role], you **will** encounter:
- [Practical scenario 1]
- [Practical scenario 2]
- [Practical scenario 3]

This means:
- [Consequence 1]
- [Consequence 2]
```

**Real example from repo:**
```markdown
### Why Understand the Boot Process?

As a DevOps/Cloud engineer, you **will** encounter:
- VMs stuck in boot loops → need to know WHERE in the chain it fails
- Kernel panics → is it hardware? bootloader? kernel?
- Custom AMIs/images → you control what kernel and init system is used
- Recovery scenarios → GRUB rescue, single-user mode, root password recovery
```

---

#### 8. Production Gotchas Pattern
**Name:** `gotchas-and-warnings`
**Usage:** Share hard-won knowledge
**Template:**
```markdown
### Production Gotcha: [Issue Title]

**The Problem:**
[What goes wrong, why it's subtle]

**Why You'll Hit This:**
[Realistic scenario]

**The Fix:**
\`\`\`bash
# Correct approach
proper_config --safe-flag
\`\`\`

**Prevention:**
[How to avoid in future]
```

**Common sections in existing docs:** "Production Gotchas & Tricky Q&A", "Common Mistakes"

---

#### 9. Breadcrumb Navigation Pattern
**Name:** `hierarchical-navigation`
**Usage:** Provide context and quick navigation
**Style:**
```markdown
[🏠 Home](../README.md) · [Category](README.md) · [Subcategory](related.md)
```

**Real examples from repo:**
```markdown
[🏠 Home](../README.md) · [Linux](README.md)
[🏠 Home](../README.md) · [Servers](README.md)
```

**Why:** Readers always know where they are and can jump between categories.

---

#### 10. Emoji & Visual Hierarchy Pattern
**Name:** `visual-signposting`
**Usage:** Scannable structure with visual cues
**Standard emoji usage in repo:**
```
🏠 Home/navigation
🐧 Linux section
🪟 Windows section
🌐 Servers/web
🧪 Labs/hands-on
⚡ Nginx (speed)
🔒 Security
📚 Documentation/theory
📋 Checklist
🔧 Tools/configuration
☁️ Cloud/Azure section
```

**Usage in headings:**
```markdown
# 🐧 Linux Administration — Comprehensive Handbook
# ☁️ AZ-104 Azure Administrator — Comprehensive Study Handbook
## 🏗️ Repo Structure
### 🔒 Security Hardening
```

---

### Cloud/Certification-Specific Patterns

The following patterns were extracted from the AZ-104 Azure study pack and apply to cloud service documentation and certification study guides.

#### 11. Exam Pattern-Matching Table
**Name:** `exam-pattern-match`
**Usage:** Map question patterns to answer directions for certification study
**Format:**
```markdown
| Question pattern | Answer direction |
|-----------------|------------------|
| "Temporary access to storage" | SAS token |
| "Identity-based access" | Entra ID + data RBAC role |
| "Prevent accidental delete" | Lock (CanNotDelete) |
```
**Source:** `azure/az104_ultra_short_cram_sheet.md`
**Why:** Certification exams use recurring question patterns. This table trains pattern recognition.

#### 12. Cross-Domain Decision Table
**Name:** `decision-table`
**Usage:** Map a need to the best tool/service across a technology domain
**Format:**
```markdown
| Need | Best tool |
|------|-----------|
| HTTP-aware routing with WAF | Application Gateway |
| TCP/UDP distribution | Load Balancer (L4) |
```
**Source:** `azure/az104_azure_administrator_handbook.md` §7
**Why:** Readers need fast answers to "which service do I pick?" questions.

#### 13. Portal + CLI Dual Walkthrough
**Name:** `dual-walkthrough`
**Usage:** Show the same task via portal steps and CLI commands side by side
**Format:**
```markdown
### Portal Walkthrough
1. Open the Azure portal and navigate to...
2. Click... and configure...

### CLI Walkthrough
```bash
az resource create --name ... --resource-group ...
```
```
**Source:** `azure/az104_practice_labs.md`
**Why:** Cloud documentation must serve both GUI-oriented and CLI-oriented readers.

#### 14. Exam Trap Table
**Name:** `exam-trap-table`
**Usage:** List common misconceptions with correct thinking
**Format:**
```markdown
| Trap | Correct thinking | Section |
|------|------------------|---------|
| Policy grants access | Policy = compliance; RBAC = access | §2.8 |
```
**Source:** `azure/az104_azure_administrator_handbook.md` §8
**Why:** Certification candidates fall for the same traps repeatedly. Explicit correction helps.

---

## Template Library

### Template 1: Handbook/Comprehensive Guide

```markdown
[🏠 Home](../README.md) · [Category](README.md)

# [📊 Emoji] [Title] — [Subtitle/Focus]

> **Audience:** [Role] | **Focus:** [Key domains] | **Source:** [Training/reference]
> **Tip:** [How to use this document]

---

## Table of Contents

1. [Section 1 — Concept](#1-section-1--concept)
2. [Section 2 — Practical](#2-section-2--practical)
3. [Section N — Advanced](#n-section-n--advanced)
... (7±2 items total)

---

## 1. Section 1 — Concept & Mental Model

### 1.1 What Is [Topic]?

[2-3 sentence definition]

### 1.2 Architecture / Mental Model

[ASCII diagram or conceptual explanation]

### 1.3 Why [This Topic] Matters

As a [role], you **will** encounter:
- [Scenario 1]
- [Scenario 2]

---

## 2. Section 2 — Practical / How-To

### 2.1 [Action Verb] [Object]

[Brief explanation]

\`\`\`bash
# Basic example
command arg
\`\`\`

### 2.2 [Action Verb] [Advanced Variant]

[Brief explanation]

\`\`\`bash
# Advanced with flags
command --flag1 --flag2=value
\`\`\`

| Flag | Meaning |
|------|---------|

---

## N. Section N — Advanced / Q&A

### N.1 Production Gotcha: [Issue]

[Explanation + fix]

### N.2 X vs Y — Comparison

| Aspect | X | Y |
|--------|---|---|

---

## Quick Revision Card

[Optional: One-page quick reference table]

---

**Version:** 1.0 | **Last Updated:** DATE | **Source:** [Reference]
```

### Template 2: Cheatsheet

```markdown
[🏠 Home](../README.md) · [Category](README.md)

# [Emoji] [Topic] — Quick Reference

> **For:** [Audience] | **Covers:** [Key areas]
> **💡 Tip:** Use TOC to jump to sections.

---

## Table of Contents

1. [Basics](#basics)
2. [Common Tasks](#common-tasks)
3. [Advanced](#advanced)

---

## Basics

### What Is [Topic]?

[1 paragraph definition]

### Installation / Setup

\`\`\`bash
# Installation command
command install package
\`\`\`

### Key Files / Locations

| File | Purpose |
|------|---------|

---

## Common Tasks

### Task 1: [Action Verb] [Object]

\`\`\`bash
# Command
command arg1 arg2
\`\`\`

### Task 2: [Action Verb]

\`\`\`bash
command --flag
\`\`\`

---

## Advanced

### Production Tips

- Tip 1: [Best practice]
- Tip 2: [Best practice]

### Comparison

| Tool | Pros | Cons |
|------|------|------|

---
```

### Template 3: Procedural/Lab Guide

```markdown
[🏠 Home](../README.md) · [Labs](README.md)

# 🧪 [Lab Title] — [Objective]

> **Difficulty:** [Basic/Intermediate/Advanced] | **Time:** [Duration]
> **Prerequisites:** [What you need]

---

## Objective

[Clear 1-2 sentence goal statement]

---

## Prerequisites Checklist

- [ ] [Prerequisite 1]
- [ ] [Prerequisite 2]

---

## Step 1: [Action]

[Explanation]

\`\`\`bash
command
\`\`\`

**Expected output:**
\`\`\`
output here
\`\`\`

---

## Step 2: [Next Action]

[Explanation]

\`\`\`bash
command
\`\`\`

---

## Verification

Use this to confirm success:

\`\`\`bash
verify_command
\`\`\`

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|

---

## Cleanup (Optional)

\`\`\`bash
# Remove test artifacts
rm -rf /tmp/test_dir
\`\`\`

---
```

## Validation Framework

### Pre-Publication Checklist

- [ ] **Structure:** All major sections present
- [ ] **Audience:** Clearly stated; reader can self-identify
- [ ] **Navigation:** TOC with anchors; breadcrumbs; cross-links work
- [ ] **Code:** All bash/config examples are syntactically valid
- [ ] **Tables:** Pipe alignment correct; headers present
- [ ] **Diagrams:** ASCII diagrams render cleanly; easy to read
- [ ] **Tone:** Consistent with repository style (professional, practical, mentor-like)
- [ ] **Completeness:** Key topics not omitted; no "TODO" comments
- [ ] **Examples:** At least 2 runnable examples per concept
- [ ] **Emoji:** Used consistently; not excessive

### Quality Scoring

| Criterion | Weight | Score Method |
|-----------|--------|--------------|
| Completeness | 20% | All sections present; no major gaps |
| Accuracy | 20% | No technical errors; claims defensible |
| Clarity | 20% | Easy to understand; well-organized |
| Consistency | 20% | Matches repo patterns; tone/style aligned |
| Usefulness | 20% | Practical; solves reader's problems |

**Overall Quality Score:** Average of 5 criteria (scale: 0.0-1.0)

**Publication threshold:** ≥ 0.80 (80%)

## Formatting Standards

### Markdown Style Guide

**Headings:**
- Use `#` for main title (1 per document)
- Use `##` for major sections
- Use `###` for subsections
- Maximum nesting: 4 levels (`####`)

**Code blocks:**
- Always specify language: \`\`\`bash, \`\`\`yaml, \`\`\`python
- Prefer \`\`\`bash for shell commands
- Use \`\`\`plaintext for output examples

**Lists:**
- Use `-` for unordered lists
- Use `1.` for ordered lists
- Nest with 2-space indent

**Tables:**
- Use pipes `|` with at least 2 spaces around
- Always include header separator `|---|---|`
- Right-align numbers, left-align text

**Links:**
- Use markdown links: `[text](path)`
- Relative paths in same repo: `../folder/file.md`
- External URLs: Full HTTPS URLs

**Emphasis:**
- Use `**bold**` for important terms
- Use `_italic_` for file paths, code references
- Use backticks `` `code` `` for inline code

### File Naming

- Use lowercase with underscores: `my_document.md` (repo convention)
- Descriptive names: `nginx_ssl_configuration.md`, `az104_azure_administrator_handbook.md`
- Avoid generic names: ✗ `guide.md`, ✓ `docker_production_guide.md`

### Directory Structure

```
repo/
├── topic/
│   ├── README.md          ← Category index
│   ├── handbook.md        ← Comprehensive
│   ├── quick-ref.md       ← Cheatsheet
│   └── labs/
│       └── procedure.md   ← Step-by-step
```

## Integration Examples

### Example 1: Generate Kubernetes Handbook

```
User: "Generate a Kubernetes handbook with focus on deployment patterns and DevOps workflows"

Agent output:
1. ✅ Requirement analysis: {topic: Kubernetes, type: handbook, scope: comprehensive}
2. ✅ Context extraction: Found Docker/container patterns; extracted orchestration diagrams
3. ✅ Outline: 12 sections (Architecture → Deployment → Production → Gotchas)
4. ✅ Content generation: 8000 words with code examples, decision matrices
5. ✅ Validation: Score 0.92 — all checks passed
6. Output: kubernetes-handbook.md (ready to publish)
```

### Example 2: Generate PostgreSQL Cheatsheet

```
User: "Create a PostgreSQL cheatsheet for quick lookups, include common queries and backup procedures"

Agent output:
1. ✅ Requirements: {topic: PostgreSQL, type: cheatsheet, scope: quick}
2. ✅ Context: Found Linux handbook patterns for command structure
3. ✅ Outline: 5 sections (Basics → Admin → Queries → Backup)
4. ✅ Content: 3000 words with command table, query examples
5. ✅ Validation: Score 0.88 — passed all checks
6. Output: postgresql-cheatsheet.md (ready to publish)
```

## Maintenance & Updates

### Version Management

Include in document frontmatter:
```markdown
---
Version: 1.0
Last Updated: 2026-04-28
Status: [Draft/Review/Published]
---
```

### Update Cadence

- **Quarterly review:** Check for deprecated info, new best practices
- **Issue-driven:** Update when bugs/errors reported
- **Pattern extraction:** Improve templates when new good patterns emerge

## References

- **Pattern sources:** Extracted from [linux_admin_handbook.md](../linux/linux_admin_handbook.md), [nginx.md](../servers/nginx.md), [windows_server_2022_cheatsheet.md](../windows/windows_server_2022_cheatsheet.md)
- **Best practices:** LLM agentic workflows, technical writing standards
- **Related docs:** `.github/agents/doc-orchestrator.agent.md`, `.github/prompts/collect-doc-requirements.prompt.md`

---

**Skill Version:** 1.0 | **Last Updated:** 2026-04-28
