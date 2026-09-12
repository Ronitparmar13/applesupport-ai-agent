# Report — AppleSupport AI Support Agent

*Fill in the bracketed numbers/examples from your own notebook run before submitting.*

## 1. Problem framing

**What "good" means for AppleSupport, specifically:**
- Correctly identifies the customer's primary problem (not a keyword mentioned in passing).
- Drafts a reply that is grounded in how AppleSupport has actually resolved similar issues before — not a generic template, and not an invented policy/promise.
- Escalates conservatively: anything touching account security, data loss, safety, or billing/legal goes to a human, with a specific, checkable reason.

**What I chose not to build:**
- Multi-turn conversation handling (the pipeline reasons over the customer's single opening message, not the full back-and-forth thread).
- Multi-language support (English-only).
- A production-grade confidence-calibration layer — confidence is the LLM's self-reported probability, not a calibrated statistical score.
- A full re-ranking retrieval system — retrieval is TF-IDF cosine similarity, not embeddings, chosen for reproducibility/speed over ceiling accuracy.

## 2. Results vs. baselines

| Method | Accuracy | Macro F1 |
|---|---|---|
| Majority class (trivial baseline) | `[fill in]` | `[fill in]` |
| TF-IDF + Logistic Regression (simple baseline) | `[fill in]` | `[fill in]` |
| Gemini agent (this pipeline) | `[fill in]` | `[fill in]` |

Auto/escalate decision accuracy vs. hand-labeled `expected_action`: `[fill in]`

LLM-judge average reply-quality scores (1–5): grounding `[..]`, factual safety `[..]`, tone `[..]`, actionability `[..]`, overall `[..]`

Judge/human agreement on a `[N]`-row sample: exact agreement `[..]`, within ±1 point `[..]`, correlation `[..]`.

## 3. Failure analysis — top 5 failure modes

For each: 1–2 real examples (pulled from `errors`/`low_quality` in the notebook) and your hypothesis for why it happens.

1. **`[failure mode name]`** — example: `"[customer text]"` → predicted `[x]`, true `[y]`. Hypothesis: `[...]`
2. **`[failure mode name]`** — ...
3. **`[failure mode name]`** — ...
4. **`[failure mode name]`** — ...
5. **`[failure mode name]`** — ...

## 4. What is misleading about my headline number

- The final test slice is small (roughly 60 rows from a 200-row golden set split 70/30), so the accuracy/F1 numbers above carry wide confidence intervals — a handful of examples flipping either way would move the headline number noticeably.
- The golden set was sampled from one brand (AppleSupport), one language, and whatever time window the Kaggle dataset happens to cover — it says nothing about how the agent would perform on other brands, other languages, or messages from a different period (e.g., a new iOS version's specific bugs).
- The LLM-as-judge reply-quality scores are a proxy for human judgment, checked against only a small human-labeled sample (`[N]` rows) — they should not be read as equivalent to full human QA.
- "Auto/escalate accuracy" measures agreement with *my own* hand-labeled judgment of what should be escalated, which is itself a judgment call, not an objective ground truth (e.g., Hiver's own support team might draw the line differently).
- Retrieval grounding quality varies by how much similar historical precedent exists in the subsampled corpus — rare intents have fewer past examples to ground against, which likely lowers reply quality for exactly those cases without showing up directly in the intent-accuracy number.

## 5. What I'd do next with one more week

- Expand the golden set past 200 rows and add inter-rater labeling (a second labeler) to get a real inter-annotator-agreement baseline, not just judge/human agreement.
- Add lightweight conversation-history context (prior customer/brand turns) instead of only the opening message.
- Replace TF-IDF retrieval with an embedding-based retriever and measure whether grounding quality (and judge scores) improve.
- Calibrate the LLM's self-reported confidence against actual correctness (a reliability diagram) and use that to set the escalation threshold instead of relying on the model's own judgment alone.
- Test the same pipeline on a second brand to see how much of the taxonomy and prompt design is brand-specific vs. reusable.
