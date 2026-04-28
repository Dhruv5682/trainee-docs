---
name: collect-doc-requirements
description: |
  Interactive prompt for collecting structured documentation generation requirements. USE WHEN: Starting document creation workflow; need to clarify user's intent and constraints; want to ensure all necessary inputs are provided before generation begins.
---

# Documentation Requirements Collector

## Purpose

This prompt guides users through a structured interview to define documentation requirements clearly. It prevents vague requests and ensures the generation agent has all needed context.

## Interview Flow

### Phase 1: Topic & Type

**Q1: What are you documenting?**
- Free text input for topic
- Examples: "Terraform for AWS", "Docker Swarm", "PostgreSQL replication"

**Q2: What type of document?**
- [ ] Handbook (comprehensive, 5,000-10,000 words)
- [ ] Cheatsheet (quick reference, 2,000-3,000 words)
- [ ] Procedural Guide (step-by-step, 3,000-5,000 words)
- [ ] Study Material (learning-focused, varies)
- [ ] Q&A / Troubleshooting (reference, 1,000-2,000 words)

**Q3: Who is the audience?**
- [ ] Beginners / Trainees
- [ ] Intermediate / Practitioners  
- [ ] Advanced / Experts
- [ ] Mixed / All levels
- Custom: [Free text]

### Phase 2: Scope & Focus

**Q4: What's the scope?**
- [ ] Quick (1,000-1,500 words, essential topics only)
- [ ] Standard (3,000-5,000 words, typical use cases)
- [ ] Comprehensive (8,000-15,000 words, exhaustive coverage)
- Custom: [Specify target word count]

**Q5: What's your primary emphasis?**
- [ ] Conceptual / Architecture (mental models first)
- [ ] Practical / How-to (commands and configs)
- [ ] Production Best Practices (security, performance, reliability)
- [ ] Troubleshooting (common issues and fixes)
- [ ] Comparison (X vs Y vs Z)
- Multiple: [Check all that apply]

**Q6: Any specific topics to include?**
- Free text: e.g., "SSL/TLS, performance tuning, backup strategies"

**Q7: Any topics to EXCLUDE?**
- Free text: e.g., "Don't cover Windows-specific details"

### Phase 3: Context & Style

**Q8: Related topics in your repo?**
- Free text: e.g., "Docker" (inform generation with existing patterns)

**Q9: Style preferences?**
- [ ] Follow repo existing patterns (recommended)
- [ ] Formal / Academic tone
- [ ] Casual / Conversational tone
- [ ] Mix (conceptual = formal, practical = casual)

**Q10: Include these elements?**
- [ ] Mental model diagrams (ASCII art)
- [ ] Comparison tables
- [ ] Code examples / command blocks
- [ ] Production gotchas / warnings
- [ ] Quick reference cards
- [ ] Q&A section
- [ ] All of the above (recommended)

### Phase 4: Constraints

**Q11: Any constraints or requirements?**
- [ ] Must align with [regulation/standard] (e.g., HIPAA, NIST)
- [ ] Exclude specific tools/vendors
- [ ] Include specific frameworks/tools
- [ ] Security-focused (minimize credentials in examples)
- None / standard best practices

**Q12: Delivery format?**
- [ ] Markdown (default)
- [ ] PDF
- [ ] HTML
- [ ] Markdown + PDF

### Phase 5: Confirmation

**Summary for confirmation:**
```
📋 Documentation Requirements Summary

Topic:           [From Q1]
Type:            [From Q2]
Audience:        [From Q3]
Scope:           [From Q4] (~[word count] words)
Emphasis:        [From Q5]
Key Topics:      [From Q6]
Excluded:        [From Q7]
Related Docs:    [From Q8]
Style:           [From Q9]
Elements:        [From Q10]
Constraints:     [From Q11]
Format:          [From Q12]
```

**Confirmation:** Approve above → Proceed to generation?

---

## Output Schema

Structured JSON for doc-orchestrator agent:

```json
{
  "requirements": {
    "topic": "string",
    "type": "enum: handbook|cheatsheet|guide|procedure|qa",
    "audience": {
      "primary": "string",
      "level": "enum: beginner|intermediate|advanced|mixed"
    },
    "scope": {
      "size": "enum: quick|standard|comprehensive",
      "estimated_words": "number"
    },
    "emphasis": ["string"],
    "key_topics": ["string"],
    "excluded_topics": ["string"],
    "related_repo_docs": ["string"],
    "style": {
      "tone": "enum: formal|casual|mixed",
      "follow_repo_patterns": "boolean"
    },
    "elements": ["string"],
    "constraints": ["string"],
    "format": "enum: markdown|pdf|html|markdown+pdf"
  },
  "meta": {
    "collected_at": "ISO8601",
    "collection_duration_seconds": "number",
    "user_confidence": "enum: certain|mostly|uncertain"
  }
}
```

## Best Practices

### For the User

1. **Be specific:** Vague topics → generic output. Specific topics → tailored output.
2. **Know your audience:** "DevOps engineers in finance" ≠ "Software developers"
3. **Set realistic scope:** You can't get 5,000-word handbook in "quick" scope
4. **Leverage repo context:** If related docs exist, mention them to align output

### For the Agent

1. **Parse ambiguity:** If user says "all levels", clarify: "Should section 1 be for beginners but section 5 assumes intermediate knowledge?"
2. **Validate constraints:** If user says "production best practices" but "quick" scope, flag: "10 best practices won't fit in 1,500 words — choose 3-4 most critical?"
3. **Confirm before proceeding:** Show summary; get approval before generation

## Iterative Refinement

If user is uncertain about requirements:

**Option A: Quick Prototype**
```
Agent: "Let's generate a quick 1,500-word summary first.
        Review it, then we can expand specific sections."
```

**Option B: Reference Existing Docs**
```
Agent: "I see [related-doc.md] in your repo with these patterns.
        Should we use a similar structure for this new doc?"
```

**Option C: Multi-Stage Generation**
```
Agent: "1️⃣ Generate outline (5 min review)
        2️⃣ You approve/modify
        3️⃣ Generate full content
        4️⃣ Validation & refinement"
```

---

**Prompt Version:** 1.0 | **Last Updated:** 2026-04-28
