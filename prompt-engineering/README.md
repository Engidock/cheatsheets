# Prompt Engineering Cheatsheet

## 1. Fundamentals & Setup

### Core Principles

Prompt engineering is the practice of designing and optimizing text prompts to effectively communicate with AI models and achieve desired outputs.

**Core Principle:** The quality of your prompt directly impacts the quality of the AI response.

### The Power Formula

| Component | Purpose | Example |
|---|---|---|
| Role | Define AI perspective | You are a senior architect |
| Context | Provide background | Building a SaaS platform |
| Task | Specific action | Design the database schema |
| Requirements | Must-have specs | Normalized, indexed, scalable |
| Format | Output structure | JSON with relationships |
| Constraints | Limits/boundaries | Max 5 tables, PostgreSQL |

### Basic Prompt Template

```text
Role: You are an expert [DOMAIN] specialist with 20 years of experience.
Context: [SITUATION DESCRIPTION]
Task: [SPECIFIC ACTION]
Requirements: [MUST-HAVE SPECIFICATIONS]
Format: [DESIRED OUTPUT FORMAT]
Tone: [COMMUNICATION STYLE]
Constraints: [LIMITATIONS AND RULES]
```

## 2. Core Techniques

### Chain of Thought (CoT)

Ask the AI to think step-by-step before answering.

```text
❌ BASIC: "What's 17 × 15?"

✓ COT: "What's 17 × 15? Think through step by step."
Response: "17 × 15 = 17 × (10+5) = 170 + 85 = 255"
```

**Impact:** +20-40% accuracy on logic and reasoning tasks.

### Few-Shot Learning

Provide 2-5 examples to teach the pattern.

```text
Example 1: "Happy news!" → Positive
Example 2: "Terrible situation" → Negative
Example 3: "Normal day" → Neutral

Now classify: "Amazing opportunity"
Response: Positive
```

**Impact:** +30-50% accuracy on classification tasks.

### Role Assignment

| Role | Best For | Improvement |
|---|---|---|
| Senior Architect | System design | +20-25% |
| Data Scientist | Analysis | +18-22% |
| Creative Director | Content | +25-30% |
| Security Expert | Security review | +20-28% |

### Constraint-Based Prompting

```text
❌ VAGUE: "Summarize the article"

✓ CONSTRAINED: "Summarize in exactly 3 bullet points:
- Key finding (max 15 words)
- Impact (max 15 words)
- Action (max 15 words)"
```

## 3. Advanced Techniques

### Self-Verification Loop

```text
Step 1: Generate answer
Step 2: Request verification - "Verify your answer"
Step 3: Request error check - "Did you find any errors?"
Step 4: Compare original vs verified
```

### Decomposition Pattern

Break complex problems into smaller parts.

```text
COMPLEX: Evaluate business proposal

DECOMPOSED:
1. Market Analysis (demand?)
2. Financial Analysis (profitable?)
3. Operational Analysis (feasible?)
4. Risk Analysis (what could fail?)
5. Recommendation (proceed?)
```

### Prompt Chaining

```text
Prompt 1: Generate outline
↓ (output becomes input)
Prompt 2: Expand sections
↓ (output becomes input)
Prompt 3: Edit for clarity
↓ (output becomes input)
Prompt 4: Final review
```

## 4. Parameter Tuning

### Temperature Settings

| Range | Behavior | Best For |
|---|---|---|
| 0.0-0.2 | Deterministic | Code, facts, analysis |
| 0.3-0.5 | Balanced | Most general tasks |
| 0.6-0.8 | Creative | Content creation |
| 0.9-1.0+ | Highly creative | Brainstorming, exploration |

### Top-P & Top-K

```text
Top-P: Cumulative probability (0.5 = focused, 0.95 = varied)
Top-K: Token count (10 = focused, 100 = varied)
Frequency Penalty: Reduce word repetition (-2 to 2)
Presence Penalty: Encourage new topics (-2 to 2)
Max Tokens: Output length (500-4000+)
```

## 5. Copy-Paste Ready Patterns

### Analysis Pattern

```text
Analyze [CONTENT] for:

STRENGTHS:
1. [First strength]
2. [Second strength]
3. [Third strength]

WEAKNESSES:
1. [First weakness]
2. [Second weakness]
3. [Third weakness]

OPPORTUNITIES:
1. [First opportunity]
2. [Second opportunity]

RECOMMENDATION: [Action with reasoning]
```

### Content Generation Pattern

```text
Write [TYPE]:
AUDIENCE: [Who is this for?]
GOAL: [What should they do?]
LENGTH: [Word count]
TONE: [Professional/casual/etc]
MUST_INCLUDE: [Elements 1, 2, 3]
AVOID: [Things to exclude]
FORMAT: [Paragraph/bullets/table]
```

### Classification Pattern

```text
Classify [ITEMS] as:

CATEGORIES:
- Category A: [Definition]
- Category B: [Definition]
- Category C: [Definition]

For each item:
1. Which category?
2. Confidence (High/Medium/Low)
3. Reasoning
```

## 6. Performance Metrics

### Key KPIs

| Metric | Target | How to Measure |
|---|---|---|
| Accuracy | 90%+ | Compare to gold standard |
| Consistency | 95%+ | Run 3x, count matches |
| Clarity | 4.5/5 | Human rating |
| Format Compliance | 98%+ | Parse output |
| Latency | <3 sec | Response time |
| Hallucination Rate | <2% | Fact-check outputs |

### Measurement Workflow

```text
1. Define metrics (what are you measuring?)
2. Create test set (20-50 cases)
3. Run baseline (measure current)
4. Document baseline (save reference)
5. Iterate (make one change)
6. Re-test (measure improvement)
7. Accept/reject (keep if +5% improvement)
8. Repeat until optimized
```

## 7. Optimization & Troubleshooting

### Common Problems & Solutions

| Problem | Root Cause | Solution | Result |
|---|---|---|---|
| Vague output | Not specific | Add requirements | +15% |
| Wrong format | Not specified | Specify format | 100% |
| Inconsistent | No constraints | Add rules | +25% |
| Off-topic | Scope undefined | Define scope | Fixed |
| Hallucination | No grounding | Ground in facts | +30% |
| Wrong length | No constraint | Specify length | Fixed |

### 30-Second Quick Wins

| Change | Time | Impact |
|---|---|---|
| Add "Think step by step" | 5 sec | +20% |
| Specify format | 10 sec | +20% |
| Add 2-3 examples | 30 sec | +30% |
| Add role | 5 sec | +20% |

## 8. Golden Rules & Best Practices

### ✓ DO These

- Be specific and detailed
- Provide 2-5 examples
- Specify output format
- Set clear constraints
- Test with 20+ cases
- Measure before/after
- Change one thing at a time
- Document what works

### ✗ DON'T Do These

- Be vague or generic
- Provide zero examples
- Assume format understood
- Forget to state requirements
- Deploy untested
- Guess improvements
- Change multiple things at once
- Ignore what doesn't work

### Professional Tips

| # | Tip | Benefit |
|---|---|---|
| 1 | Always Establish Baseline - Measure before optimizing | Know starting point |
| 2 | Change One Thing - Modify only one element | Isolate impact |
| 3 | Test 20+ Cases - Never just 1-2 examples | Reliable results |
| 4 | Always Format - Specify JSON/bullets/markdown | Predictable outputs |
| 5 | Temperature by Task - 0.1-0.3 analytical, 0.7-0.9 creative | Optimal results |
| 6 | Document Everything - Keep improvement log | Build playbook |
| 7 | Combine Techniques - Role + Context + Examples | +50% improvement |
| 8 | A/B Test - Test two versions on same cases | Data-driven decisions |

## 9. Production Deployment

### Pre-Deployment Checklist

**✓ Required Tests**

- Test with 50+ diverse cases
- Verify accuracy > 90%
- Test edge cases and failures
- Validate format > 98%
- Measure latency < 3 sec
- Security review passed
- Cost estimate approved

### Deployment Stages

```text
1. Stage 1: Internal testing (1 week)
2. Stage 2: Beta 10% users (1 week)
3. Stage 3: Rollout 50% users (1 week)
4. Stage 4: Full production (ongoing)
```

### Monitoring

| Cadence | Focus |
|---|---|
| Daily Metrics | Check for failures, verify no bugs, look for patterns |
| Weekly Analysis | Calculate accuracy, review feedback, identify improvements |
| Monthly Optimization | Comprehensive analysis, A/B test improvements, document learnings |

## 10. Real-World Example: Customer Support

### Workflow

```text
STEP 1: Ticket intake → Extract sentiment, category, priority
STEP 2: Route & analyze → Categorize and prioritize
STEP 3: Generate response → Draft AI response (150-200 words)
STEP 4: Review → Human review (sampling 10%)
STEP 5: Send & monitor → Track satisfaction

RESULTS:
- 93% customer satisfaction
- 2 hours response time (vs 24 hours manual)
- $25,000/month savings
- 500+ tickets/day handled
```

## 11. FAQ & Troubleshooting

**Q: How do I know if my prompt is good?**
A: Measure it! Test on 20+ cases: accuracy >90%, consistency >95%, clarity 4.5/5, format >98%.

**Q: What temperature should I use?**
A: Analytical (0.1-0.3), Balanced (0.4-0.6), Creative (0.7-0.9). Start at 0.5 and adjust.

**Q: How many examples do I need?**
A: 2-5 examples usually works. More helps if the task is complex. Use 100% correct examples.

**Q: How do I prevent hallucinations?**
A: Ground in provided facts, add "Only use provided info", request citations, use lower temperature.

**Q: Why is accuracy inconsistent?**
A: Add constraints, lower temperature, provide examples, be more specific.

**Q: How do I measure ROI?**
A: (Manual cost - AI cost) × Volume = Savings. Example: ($50 - $0.10) × 1000 = $49,900/month.

**Q: What's the best way to optimize?**
A: Baseline first → Change ONE element → Test same dataset → Measure impact → Keep if >5% improvement.

## 12. Quick Reference Commands

| Category | Command | Use Case |
|---|---|---|
| Analysis | Analyze for: strengths, weaknesses, opportunities | SWOT analysis |
| Analysis | Compare X vs Y vs Z | Comparison |
| Analysis | What are root causes of [problem]? | Problem analysis |
| Generation | Write [type] for [audience] | Content creation |
| Generation | Generate 5 ideas for [purpose] | Brainstorming |
| Generation | Create examples of [concept] | Example generation |
| Refinement | Improve for clarity | Quality improvement |
| Refinement | Rewrite for [audience] | Audience adaptation |
| Refinement | Fix errors in [content] | Error correction |
| Verification | Is this accurate? Check | Validation |
| Verification | Did I miss anything? Review | Completeness check |

## 13. Cost Management & ROI

### ROI Example

```text
SCENARIO: Customer support ticket generation
Manual: $50/ticket × 1,000/month = $50,000/month
AI: $0.10/ticket × 1,000/month = $100/month
Savings: $49,900/month
Annual: $599,000/year
Investment: $1,200/year
ROI: 49,650%
Payback: < 1 day
```

### Cost Optimization Strategies

| Strategy | Savings | Effort |
|---|---|---|
| Caching responses | 20-40% | Low |
| Batch processing | 15-25% | Medium |
| Model selection | 30-50% | Medium |
| Prompt optimization | 10-20% | Low |
| Context compression | 20-30% | Medium |

## 14. Architecture Patterns

### Sequential Processing

```text
User Input → Prompt 1 (Clarify) → Prompt 2 (Analyze) →
Prompt 3 (Recommend) → Prompt 4 (Plan) → Output

Benefits: Each step builds on the previous, better error handling
```

### Multi-Agent Architecture

```text
Query → Agent 1 (Optimist)   ┐
      → Agent 2 (Skeptic)    ├→ Synthesis → Balanced Output
      → Agent 3 (Pragmatist) ┤
      → Agent 4 (Visionary)  ┘

Benefits: Multiple perspectives, balanced analysis
```

### Hierarchical Processing

```text
Level 1: Executive summary (3 sentences)
↓
Level 2: Detailed analysis (comprehensive)
↓
Level 3: Implementation plan (actionable steps)

Benefits: Scalable depth, serves different audiences
```

---

*Source: adapted from the Prompt Engineering cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
