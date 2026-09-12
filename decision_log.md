# Decision Log

Non-obvious decisions made while building this pipeline, and why. Edit freely — these are starting points based on the actual design; add your own as you iterate.

1. **Used Gemini directly for classification instead of training a classifier on LLM-generated labels.** Training a model on top of LLM labels adds an extra layer of noise and a lot of pipeline complexity (large-scale batch labeling, training, saving) without clearly beating the LLM's own zero/few-shot accuracy. For a small-scope agent, calling the LLM directly is simpler to build, explain, and evaluate.

2. **Fit both baselines on a 70/30 split of the golden set itself, not on a separately weak-labeled training set.** Weak/heuristic labels (e.g. keyword rules) can make a baseline look artificially strong or weak in ways that don't reflect real intent boundaries. Using the same hand-labeled ground truth for both fitting and testing the baselines (on disjoint slices) keeps the comparison to the LLM agent fair.

3. **Followed `in_response_to_tweet_id` into the full dataset, not just the brand's own rows, when reconstructing pairs.** Restricting the lookup to brand rows silently drops or mis-pairs replies whose parent tweet lives elsewhere in the dataset, undercounting valid conversations.

4. **Filtered pairs to only those where the parent tweet's `inbound` flag is `True`.** Without this, some "pairs" would actually be brand-to-brand or bot-to-bot exchanges, which would pollute both the retrieval corpus and the golden set sample.

5. **Deduplicated on cleaned customer text.** The raw Twitter dataset contains many near-identical repeated complaints (e.g. mass outage tweets); without dedup, both the golden set sample and retrieval corpus would overrepresent a handful of incidents.

6. **Subsampled to ~4,000 pairs for the working corpus instead of using the full dataset.** The assignment explicitly expects and rewards a representative subsample rather than full-dataset runs; this keeps iteration fast and stays inside the 15-minute reproduction budget.

7. **Built the retrieval index only from the working subsample, never from the golden set.** Grounding an agent's answer in the same examples used to test it would inflate quality/accuracy numbers without meaning anything.

8. **Made the LLM output a single structured object (intent + confidence + reply + action + reason) in one call**, rather than three separate calls. This is cheaper, faster, and forces internal consistency (e.g., the escalation reason can directly reference the same intent the model just committed to).

9. **Defined explicit escalation triggers in the prompt** (account/security, data loss, safety, billing/legal, low confidence) instead of relying purely on a numeric confidence threshold. A stated, categorical reason is easier for a human reviewer to sanity-check than "confidence < 0.7."

10. **Added an LLM-as-judge step with a required human-agreement check**, rather than trusting the judge's scores at face value. An unchecked LLM judge can systematically over- or under-score in ways that don't match human intuition; the assignment explicitly asks for evidence of agreement.

11. **Used `getpass` for the API key instead of Colab Secrets.** Colab Secrets are per-account and don't transfer if this notebook is opened somewhere else (e.g. by a grader); pasting into a hidden prompt each run trades a small bit of convenience for portability, and the key is never written to a cell's saved output.

12. **Held out a small final test slice from the golden set for headline numbers, rather than using the full golden set for both baseline-fitting and reporting.** Reusing the same rows to fit the LogisticRegression baseline and to report its accuracy would overstate its performance and make the baseline comparison unfair to the LLM agent, which sees no golden-set examples during "training" at all.
