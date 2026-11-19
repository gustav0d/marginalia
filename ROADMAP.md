# Marginalia — Roadmap

The full project plan, built as thin vertical slices: each ships on its own and the
project is stoppable at any slice.

## Slices

### 1. Ask a question, get a cited answer
- Upload a PDF → ingest (parse, chunk, embed) → ask → answer with page-level citations.
- The answer refuses when retrieved evidence is weak instead of guessing.
- Embeddings from an API, always behind the `Embedder` port so a trained model can
  drop in later.
- Deployed from the start — a live link with real traces beats local polish.

### 2. Measure retrieval
- Golden dataset of 50+ questions written by hand, not generated.
- Retrieval metrics: recall@k and MRR against the baseline.
- Hybrid search (pgvector + Postgres full-text) compared against the baseline in the
  same harness — a real retrieval experiment without training anything.

### 3. Judge and CI gate
- LLM-as-a-judge rubrics: faithfulness, relevance, citation correctness.
- Judge calibrated against ~50 hand-labeled answers; agreement reported (Cohen's
  kappa), including failure cases.
- Judge tested for position bias and verbosity bias; pairwise comparison of prompt
  versions.
- CI gate: a pull request that lowers eval scores fails.

### 4. Tracing
- OpenTelemetry across web, API and the eval harnesses; Langfuse for LLM spans.
- One trace ID follows a question from browser to model call.
- User feedback (thumbs) is tied to trace IDs, so a bad answer leads back to its
  retrieval, prompt and model versions.

### 5. Guardrails and red team
- Indirect prompt injection: a red-team suite of malicious uploaded PDFs, run through
  the same eval harness and in CI.
- PII redaction on upload, output groundedness checks, scope limits, rate limits.
- Weak-evidence refusal (from slice 1) stays under continuous red-team tests.

### 6. Quizzes and explanation modes
- Quiz and flashcard generation as structured output, validated with Zod.
- Explanation levels (simple, exam-level, analogy) built from versioned prompt
  templates.
- Generated content passes through the same eval harness: quizzes get judged too.

Slices 7–9 add trained models. They start only after the evaluation harness exists:
every trained model must beat the off-the-shelf baseline in the same harness —
measured, not claimed.

### 7. Fine-tuned retriever
- Train an embedding model or cross-encoder reranker on question–passage pairs, with
  synthetic questions generated and filtered by the slice-3 judge.
- Measured with recall@k and MRR against the off-the-shelf baseline — same harness,
  same dataset.
- Published with a model card that includes where it fails.
- Data policy: only openly licensed material in any released dataset. Never train on
  copyrighted uploads.
- Plugs into the `Embedder` port with zero app changes.

### 8. Inference service
- Small Python service (FastAPI) that serves the fine-tuned model, since the training
  stack is Python-native.
- Model version pinned; the version is stamped on every trace.

### 9. Operationalization
- Versioned models, a CI gate that blocks a model regression, and tracing that shows
  which model version served each query.
- Drift monitoring on retrieval quality and guardrail trigger rates.

### 10. Stretch: small models
- **Judge distillation** — distill hand-labeled grading data into a small judge
  (LoRA); report its agreement against the calibrated large-model judge.
- **Forgetting predictor** — a small model on review history that predicts which card
  a learner is about to forget; sized honestly (logistic regression / gradient
  boosting, not a transformer).

Throughout: ADRs (`docs/adr/`, one short file per decision), an eval results table,
and honest documentation of what failed and what changed.

## Backlog

Not scheduled, kept visible. Pulled in only after the slices above are done.

- **Study-planner agent** — exam date + syllabus file in, a study plan out; the user
  approves the plan before it commits. Cost and step budgets, loop limits, retries,
  model fallback.
- **Spaced repetition** — quiz results drive review scheduling (pairs naturally with
  the forgetting predictor).
- **Online judging** — sample production traces and judge them, closing the loop
  between live traffic and the eval harness.
- **MCP server** — expose notes and citations to other agents.
- **Blog posts** — written from the eval and training results once real numbers
  exist; the results tables are the source material, the posts are not the deliverable.

## Stopping points

Build in order; stop when time runs out. The eval harness (slice 2) is the cheapest
early win outside the app itself: it makes every later decision — hybrid search,
prompts, trained models — measurable instead of vibes.
