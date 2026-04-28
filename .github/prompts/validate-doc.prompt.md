# Validate Documentation Quality

> Run this prompt against a generated document to score it against the doc-generator skill standards.
> Usage: Point an agent at this prompt with the target file path.

## Instructions

You are a documentation quality validator. Given a target markdown file, evaluate it against the checklist below and produce a score report.

### Input

- **Target file:** (provide the path to the file to validate)
- **Document type:** (handbook, cheatsheet, cram sheet, lab pack)

### Validation Checklist

Score each item as ✅ Pass, ⚠️ Partial, or ❌ Fail.

#### Structure (25%)
- [ ] Has frontmatter (Version, Last Updated, Status)
- [ ] Has navigation breadcrumbs (`[🏠 Home](../README.md) · [Category](README.md)`)
- [ ] Has a table of contents with working anchors
- [ ] TOC length follows 7±2 rule (5-11 major sections)
- [ ] Has version footer at end of document

#### Content Depth (25%)
- [ ] At least 2 runnable code/CLI examples per major section
- [ ] At least 1 ASCII diagram per major section
- [ ] "Why It Matters" or context section per domain
- [ ] Gotchas/warnings are inline with full format (Problem/Why/Fix)
- [ ] No major exam or topic gaps for the stated scope

#### Formatting (20%)
- [ ] Uses repo file naming convention (lowercase with underscores)
- [ ] Uses repo emoji conventions consistently
- [ ] Tables use consistent pipe alignment
- [ ] Code blocks specify language (bash, yaml, etc.)
- [ ] Bold, italic, and backtick usage follows repo patterns

#### Deduplication (15%)
- [ ] Content is not repeated verbatim across companion files
- [ ] Each file in a pack serves a distinct, documented purpose
- [ ] Cross-references link to the source of truth rather than duplicating

#### Completeness (15%)
- [ ] All stated topic domains are covered
- [ ] Cleanup commands provided for lab/practice content
- [ ] Source attribution is specific (not just "study topics")
- [ ] Related labs or real-world references are linked where applicable

### Output Format

```markdown
## Validation Report: [filename]

**Document type:** [type]
**Validated on:** [date]

| Category | Score | Notes |
|----------|-------|-------|
| Structure | X/5 | ... |
| Content Depth | X/5 | ... |
| Formatting | X/5 | ... |
| Deduplication | X/3 | ... |
| Completeness | X/4 | ... |

**Overall score:** X/22 (X.XX normalized to 0.00-1.00)

### Issues Found
1. [severity] [description]
2. ...

### Recommendations
1. ...
```

### Scoring Guide

| Score range | Meaning |
|------------|---------|
| 0.90-1.00 | Production ready |
| 0.75-0.89 | Good with minor fixes needed |
| 0.60-0.74 | Needs significant revision |
| Below 0.60 | Major rework required |
