# RLHF-Pipeline-Annotation-Guidelines-Management
A professional reference covering the complete RLHF training pipeline, how annotation work fits within it, and frameworks for managing guideline changes built from real project experience.

## Part 1: The Complete RLHF Pipeline

### Where Human Evaluation Fits

Most evaluators understand their individual task. Fewer understand why that task exists and what happens to their output after they submit it. This understanding is what separates contributors from specialists.

```
Step 1: PROMPT COLLECTION
Raw prompts collected from users or written by prompt engineers.
Purpose: Diverse, realistic inputs that represent real-world use cases.

         ↓

Step 2: RESPONSE GENERATION
The base LLM generates multiple responses to each prompt.
Typically 2–4 responses per prompt for comparison.

         ↓

Step 3: HUMAN EVALUATION (your work)
Evaluators rank responses using structured rubrics.
Output: Preference pairs — (Response A, Response B, Preferred: A)

         ↓

Step 4: REWARD MODEL TRAINING
A separate model is trained on your preference pairs.
It learns to predict which outputs humans prefer.
This is the reward model.

         ↓

Step 5: POLICY OPTIMISATION (RL phase)
The base LLM is updated using the reward model as feedback signal.
The model learns to produce outputs that score higher on the reward model.
Common algorithm: Proximal Policy Optimisation (PPO)

         ↓

Step 6: EVALUATION & ITERATION
The updated model is evaluated against benchmarks and new human ratings.
If quality has improved, the cycle continues with the new model version.
If not, the data or reward model is investigated.

         ↓

Back to Step 1 — with an improved base model.
```

---

### What Your Ranking Actually Does

When you rank Response A over Response B, here is the exact chain of events:

1. Your ranking becomes a data point: `(Prompt X, Response A, Response B) → Preferred: A`
2. The reward model trains on thousands of these data points and learns: *"Responses like A tend to be preferred over responses like B"*
3. The base LLM, optimised against the reward model, adjusts its weights to produce more A-like responses
4. Future users of the model receive outputs shaped by your evaluation decision

**This is why precision matters:**
If you consistently prefer fluent responses over accurate ones, the model learns to prioritise fluency. If you prefer long responses over concise ones, the model learns that length equals quality. Your systematic biases — not just your individual errors — become the model's learned behaviour.

---

### Direct Preference Optimisation (DPO) — The New Pipeline

In DPO pipelines, Step 4 and Step 5 are merged into a single operation. Your preference rankings directly update the base model's weights without an intermediate reward model.

**What changes for evaluators:**

In traditional RLHF: your rankings → reward model (buffer) → base model update
In DPO: your rankings → base model update (direct)

The buffer of the reward model absorbed some noise and inconsistency. In DPO, that buffer is gone. Every ranking has more direct impact. Consistency and accuracy matter even more than in RLHF pipelines.

---

### RLAIF — Where Your Role Evolves

In RLAIF pipelines, an AI evaluator model handles a large portion of standard preference ranking. Human evaluators take on higher-value roles:

**Your new responsibilities in RLAIF:**

| Task | Description |
|------|-------------|
| Rubric Design | Write the criteria the AI evaluator uses to rank responses |
| AI Ranking Audit | Review a sample of AI-generated rankings for accuracy |
| Edge Case Handling | Evaluate cases the AI evaluator flagged as uncertain |
| Bias Detection | Identify systematic errors in AI evaluator behaviour |
| Gold Standard Creation | Create the reference responses the AI evaluator is calibrated against |

The practical implication: understanding RLAIF positions you for senior evaluator and QA lead roles, not just contributor roles.

---

## Part 2: Annotation Guidelines — Management Frameworks

### Why Guidelines Change

Guidelines change for five distinct reasons. Knowing the reason tells you how to update your approach.

| Reason | Trigger | Impact on Evaluator |
|--------|---------|-------------------|
| Safety threshold shift | Model too restrictive or too permissive in production | Re-calibrate all borderline content judgments |
| Quality standard elevation | Model capability improves; bar raised | Previous "good enough" responses now score lower |
| Category redefinition | Researchers discover new edge cases | Existing categories split, merge, or redefine |
| Context-dependency added | Same content now evaluated differently by context | Every evaluation now requires full context read |
| Market/domain expansion | New geography or subject matter added | New domain-specific rules added to base guidelines |

---

### Framework 1: The Full Reset Protocol

**Use this every time a guideline update arrives.**

Most evaluators read only the highlighted changes. This is a mistake. Guidelines are interconnected — a change in Section 3 often affects how Section 7 is applied, even if Section 7 text didn't change.

```
On receiving a guideline update:

□ Stop all active work immediately
□ Do not complete current queue under old guidelines
□ Read the ENTIRE updated document — not just changed sections
□ For each section: compare to your current mental model
□ Note every discrepancy between what you believed and what the document says
□ Pull 3–5 recent completed tasks and re-evaluate under new guidelines
□ Do your verdicts change? Document how and why
□ Resume production only after successful re-evaluation of sample tasks
□ Work at 70% speed for first 50 tasks under new guidelines
□ Return to full speed only after consistency is confirmed
```

---

### Framework 2: The Assumption Audit

**Use this monthly, or whenever you suspect drift from guidelines.**

After months on a project, evaluation becomes automatic. Automatic evaluation means you are applying your internalized version of the guidelines — which may no longer match the current version.

```
The Assumption Audit (15 minutes):

Step 1: Without looking at the guidelines, write down your understanding of:
  - The top 3 most important rules
  - How you handle the most common edge case type you encounter
  - The definition of the most critical scoring dimension

Step 2: Open the current guidelines and compare your written answers

Step 3: For every discrepancy:
  - How long have you been applying the wrong standard?
  - How many tasks might be affected?
  - Flag for review if possible; document if not

Step 4: Correct your mental model before resuming work
```

---

### Framework 3: The Edge Case Stress Test

**Use this after every guideline change.**

New guidelines, written to fix one problem, can create new edge cases. Find them before they corrupt your production data.

```
Edge Case Stress Test:

Step 1: Identify the specific change in the new guideline
  What situation is now evaluated differently than before?

Step 2: Generate 5–10 edge cases near the new boundary
  These are cases where old rule and new rule give different verdicts

Step 3: Apply new guideline to each case
  Does it give a sensible verdict? Document every case where it doesn't.

Step 4: If new rule produces a clearly wrong verdict on an edge case:
  - Do NOT apply the rule and move on
  - Document the case and the apparent flaw
  - Escalate to project lead with specific example
  - Await clarification before applying to production work
```

**Real example of why this matters:**
A guideline update instructed evaluators to prefer responses that cite peer-reviewed research. An evaluator running a stress test immediately found the edge case: what about a response that cites a real paper but misrepresents its findings? Under the new rule, the citation increases the rating. Under correct evaluation, the misrepresentation should decrease it.

The evaluator escalated. The guideline was updated within 48 hours to clarify: citations increase rating only when the cited content is accurately represented. The stress test prevented weeks of miscalibrated evaluation data.

---

### Framework 4: The Calibration Session

**Use this with any team after a guideline change.**

Ten evaluators reading the same guideline will develop ten slightly different interpretations. Without calibration, the team produces ten different standards rather than one consistent standard.

```
Calibration Session Structure:

Preparation (individual, before session):
  - Each evaluator reads the updated guideline independently
  - Each evaluator completes 3–5 selected ambiguous tasks independently
  - No discussion before scores are recorded

During session:
  - Share scores on each task
  - Where all agree → confirmed shared understanding, move on
  - Where evaluators disagree → this is an ambiguity in the guideline

For each disagreement:
  - Each evaluator explains their reasoning with reference to specific guideline text
  - Goal is NOT to find who is right — goal is to find what in the guideline is ambiguous
  - Reach team consensus on how to interpret the ambiguity
  - Document the consensus interpretation

Post-session:
  - Submit consensus interpretations to project lead for confirmation
  - Confirmed interpretations become the team operating standard
  - All ambiguities and their resolutions documented for future reference
```

---

### Real-World Guideline Change Examples

**Example 1: Google Search Quality Rater Guidelines (2022)**
Google added "Experience" to their E-A-T framework, creating E-E-A-T. Raters who had spent years evaluating Expertise, Authoritativeness, and Trustworthiness had to add first-hand experience assessment to every evaluation. A product review written by someone who demonstrably owned the product now scored higher than an equally-written review with no evidence of direct experience. Raters who didn't update their framework missed the new dimension entirely and continued scoring as if Experience didn't exist.

**Example 2: Medical Content Safety Threshold Shift**
An annotation project initially flagged any response discussing medication thresholds as unsafe. After production data showed the model was refusing legitimate clinical queries, the guideline was updated: professional context (clearly established by the user) now unlocked clinical information sharing. Evaluators trained on the original binary rule had to develop a new skill — context recognition — that didn't exist in their original training. Those who didn't adapt kept flagging clinical professionals' legitimate queries as unsafe.

**Example 3: Hate Speech Category Split**
A content moderation project moved from a binary hate speech classification to a five-tier taxonomy (Tier 1 direct dehumanisation, Tier 2 implicit bias, counter-speech, satire, academic discussion). Evaluators who had spent months building a single threshold now had to make sequential distinctions before reaching a verdict. Those who tried to apply the new taxonomy without fully resetting their approach defaulted to their old binary — flagging counter-speech as hate speech and missing Tier 2 content entirely.

**The lesson across all three examples:** Guideline changes are not updates to your existing framework. They are replacements of it. Treat every guideline change as a complete reset, not an amendment.

---

## Part 3: Inter-Annotator Agreement

### What It Is and Why It Matters

Inter-annotator agreement (IAA) is the statistical measurement of how consistently different evaluators apply the same guidelines to the same tasks. It is one of the most important quality metrics on any annotation project.

If two evaluators score the same response as 5/5 and 2/5 respectively, one of them is wrong — or the guideline is ambiguous. Either way, the training signal is corrupted.

**Common IAA metrics:**

| Metric | What it measures | Target |
|--------|-----------------|--------|
| Cohen's Kappa | Agreement adjusted for chance | > 0.8 (excellent) |
| Percent Agreement | Simple % of matching scores | > 85% |
| Correlation Coefficient | How well scores track each other | > 0.9 |

### How Your IAA Is Monitored

On most annotation platforms:

1. **Gold Standard Tasks** — Embedded in your work queue without labeling. Tasks with known correct answers. Your performance on these determines your quality score.

2. **Cross-evaluator Comparison** — The same task is given to multiple evaluators. Agreement rates are calculated automatically.

3. **Drift Monitoring** — Your scores on similar tasks are tracked over time. If your scores shift significantly without a guideline change, it signals drift — you are evaluating differently than before, which means one version was wrong.

**How to maximise your IAA:**
- Always derive your score from the rubric, not from intuition
- When uncertain, re-read the relevant guideline section before scoring
- If your score feels counterintuitive, check whether you're applying the guideline or overriding it with judgment
- Write your rationale before finalising your score — if you can't justify it in writing, you probably can't justify it at all

---

*Portfolio maintained by Akansha Chand — AI Evaluator & RLHF Specialist*
*2+ years evaluating LLMs for Google Gemini (Turing Inc.) and Nvidia NeMo (Spectrum Consultants)*
