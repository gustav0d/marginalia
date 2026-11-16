# Marginalia

A study companion that answers questions from your own uploaded files — with citations pointing back to the exact page — explains concepts at a chosen level, and generates quizzes. If your files don't support an answer, it says so instead of guessing.

**Status:** slice 1 in progress — see [Roadmap](#roadmap).

## Why

A RAG study companion is the most common portfolio project there is, so the RAG itself is not the point. The point is the engineering around it:

- **Measured retrieval** — a golden dataset scored with recall@k and MRR, hybrid search compared against a baseline.
- **A judge checked against my own labels** — LLM-as-judge rubrics calibrated on hand-labeled answers, with agreement reported and bias tests.
- **Traces that explain a bad answer** — OpenTelemetry across web, API and eval harness, with LLM spans in Langfuse.
- **A guardrail that survives a malicious PDF** — indirect prompt-injection red teaming and weak-evidence refusals.

## Stack

- **TypeScript end to end.** Next.js web app, Fastify API, LangChain.js + LangGraph.js for the question-answering flow (an explicit graph: retrieve → evidence check → answer or refuse).
- **Python only for evals and red teaming.** The harnesses call the app's public HTTP API, so they test what a user gets.
- **Postgres with pgvector** for embeddings, next to the relational data.
- **Ports and adapters.** Chat models, embedders, retrieval and tracing sit behind ports; one typed config file picks the adapter per use case. Graph nodes import ports only — LangChain and Langfuse appear only inside adapters.

## Roadmap

Six slices, in order. Each ships on its own and the project is stoppable at any slice.

1. **Ask a question, get a cited answer.** Upload a PDF, ingest it (parse, chunk, embed), ask, get an answer with page citations. Deployed from the start.
2. **Measure retrieval.** Golden dataset of 50+ questions; recall@k and MRR; hybrid search vs. baseline.
3. **Judge and CI gate.** Faithfulness, relevance and citation-correctness rubrics; judge calibrated against ~50 hand labels; a PR that lowers scores fails CI.
4. **Tracing.** OpenTelemetry + Langfuse; one trace ID follows a question from browser to model call.
5. **Guardrails and red team.** Prompt-injection PDFs, PII redaction, weak-evidence refusals; red-team results in CI.
6. **Quizzes and explanation modes.** Quiz generation as Zod-validated structured output; explanation levels from versioned prompt templates.

## Repo layout

```
apps/web      Next.js frontend: upload, chat with citations, quizzes, feedback
apps/api      Fastify API: ports, adapters, config, LangGraph.js flow, guardrails
evals/        Python: golden dataset, retrieval metrics, judge, CI gate
redteam/      Python: injection PDFs and attack cases
docs/adr/     one short decision record per choice
```

## Honest scope

This project covers the software-engineering-for-AI side of AI engineering — retrieval quality, evaluation, tracing and guardrails. It does **not** cover LLM training or operationalization: a rushed model with weak results hurts more than no model, so embeddings sit behind an interface where a trained model can drop in later.
