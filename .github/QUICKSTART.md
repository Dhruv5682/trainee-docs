# Quick Start Guide — Document Orchestrator

Get started generating documentation in 5 minutes.

---

## 1. Invoke the Agent (1 minute)

```bash
# Simple request
@doc-orchestrator Terraform with focus on AWS best practices

# With scope specification  
@doc-orchestrator Kubernetes handbook comprehensive with security and performance focus
```

**What happens:**
1. Agent parses your request
2. Asks clarifying questions (if needed) via `/collect-doc-requirements`
3. Extracts patterns from your repo
4. Generates document in 5 stages

---

## 2. Review Requirements (30 seconds)

Agent shows a summary:
```
📋 Documentation Requirements Summary

Topic:      Terraform
Type:       Handbook
Audience:   DevOps / Cloud Engineers
Scope:      Comprehensive (~8,000 words)
Emphasis:   Production best practices, security
Style:      Follow repo patterns ✓
Elements:   Mental models, examples, gotchas, tables ✓
```

**Approve to proceed.**

---

## 3. Wait for Generation (2-5 minutes)

Agent shows progress:
```
✅ Stage 1: Requirement analysis (2 sec)
✅ Stage 2: Context extraction — found 3 related docs (25 sec)
✅ Stage 3: Outline generation (4 sec)
✅ Stage 4: Content generation — 8,000 words (120 sec)
✅ Stage 5: Quality validation (8 sec)

📊 Validation Score: 0.92 ✅ (Threshold: 0.80)
```

---

## 4. Review Generated Document (1-2 minutes)

Check:
- [ ] Structure looks good (TOC, sections in order)
- [ ] Examples are relevant
- [ ] Tables/diagrams render correctly
- [ ] Links work
- [ ] Tone matches your repo style

---

## 5. Publish (30 seconds)

```bash
# Copy generated markdown to your repo
cp terraform-handbook.md docs/terraform/

# Optional: Make edits if needed
# Then commit

git add docs/terraform/terraform-handbook.md
git commit -m "docs: add Terraform comprehensive handbook"
git push origin feature/tf-handbook
```

---

## Document Types & Estimates

| Type | Words | Generation | Use For |
|------|-------|-----------|---------|
| Cheatsheet | 1,500-3K | 1-2 min | Quick reference, commands |
| Guide | 3-5K | 2-3 min | How-to, procedures |
| Handbook | 8-15K | 3-5 min | Comprehensive, learning |
| Q&A | 1-2K | 1-2 min | FAQs, troubleshooting |
| Lab | 3-5K | 2-3 min | Hands-on training |

---

## Example Requests

### Minimal (takes ~2 min)
```
@doc-orchestrator PostgreSQL cheatsheet
```

### Standard (takes ~3 min)
```
@doc-orchestrator Docker with focus on production security and performance
```

### Detailed (takes ~5 min)
```
@doc-orchestrator Kubernetes handbook comprehensive 
with themes: deployment patterns, networking, security hardening, production gotchas
audience: DevOps engineers with 1+ year experience
related: linux_admin_handbook.md, containers docs
```

---

## Customization

### Request Specific Emphasis

```
@doc-orchestrator Terraform with EMPHASIS ON:
  1. AWS-specific patterns
  2. State management best practices
  3. Team collaboration
  4. Production hardening
```

### Reference Existing Patterns

```
@doc-orchestrator Ansible 
FOLLOW PATTERN OF: linux_admin_handbook.md (structure and tone)
```

### Specify Audience

```
@doc-orchestrator Prometheus monitoring
FOR AUDIENCE: SRE engineers, 2+ years experience
EXCLUDE: Theory, focus on hands-on configuration
```

---

## Quality Checklist

Generated docs meet these standards:

- ✅ **Completeness:** All major topics covered
- ✅ **Accuracy:** No technical errors; claims substantiated
- ✅ **Clarity:** Well-organized; easy to understand
- ✅ **Consistency:** Matches your repo style (emoji, tone, structure)
- ✅ **Usefulness:** Solves practical problems; appropriate depth

**Quality Score:** 0.0-1.0 (1.0 = perfect)  
**Publication Threshold:** ≥ 0.80

---

## Troubleshooting

### "Validation score too low" (< 0.80)

Agent suggests fixes. Options:
- [ ] Auto-apply fixes and regenerate
- [ ] Review specific issues and fix manually
- [ ] Loosen constraints and retry

### "Generated content doesn't match repo style"

**Fix:** Mention related docs to help pattern extraction
```
@doc-orchestrator [Topic]
FOLLOW PATTERN OF: [existing-doc.md]
```

### "Examples are generic or off-topic"

**Fix:** Be more specific in requirements
```
@doc-orchestrator Docker
WITH FOCUS ON: Production deployments, security, scaling
FOR PLATFORM: AWS ECS, not Kubernetes
```

---

## Advanced: Customize Templates

See [System Documentation](./README.md#extending-templates) for:
- Creating domain-specific templates
- Adding new patterns
- Modifying validation rules
- Integrating with CI/CD

---

## System Architecture (Reference)

```
You request → Agent analyzes → Extracts patterns from repo
  ↓
  ├─ Stage 1: Parse requirements (2 sec)
  ├─ Stage 2: Extract context (25 sec)  ← Looks at your docs
  ├─ Stage 3: Generate outline (4 sec)
  ├─ Stage 4: Create content (120 sec)
  └─ Stage 5: Validate quality (8 sec)
  
  Result: Production-ready markdown
```

---

## Files & Documentation

| File | Purpose |
|------|---------|
| [README.md](./README.md) | System overview & architecture |
| [doc-orchestrator.agent.md](./agents/doc-orchestrator.agent.md) | Agent definition |
| [doc-orchestrator.instructions.md](./agents/doc-orchestrator.instructions.md) | Detailed decision logic |
| [SKILL.md](./skills/doc-generator/SKILL.md) | Templates & patterns |
| [patterns.md](./skills/doc-generator/patterns.md) | Pattern specifications |

---

## Tips for Best Results

1. **Be specific** — "Kubernetes deployment security" > "Kubernetes"
2. **Set scope** — "quick" (1.5K words) vs "comprehensive" (10K+ words)
3. **Reference your repo** — "Similar to [existing-doc.md]"
4. **Iterate** — First pass rarely perfect; expect to refine sections
5. **Validate examples** — Test any code before publishing

---

**Need help?** Check the full [System Documentation](./README.md)

**System Version:** 1.0 | **Last Updated:** 2026-04-28
