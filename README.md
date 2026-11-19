# Marginalia

A study companion that answers questions from your own uploaded files — with citations pointing back to the exact page — explains concepts at a chosen level, and generates quizzes. If your files don't support an answer, it says so instead of guessing.

**Status:** slice 1 in progress — see [ROADMAP.md](ROADMAP.md).

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

The full plan lives in [ROADMAP.md](ROADMAP.md), as two tracks. **Track A** is the application: retrieval, evaluation, tracing, guardrails. **Track B** adds trained models — a fine-tuned retriever behind the embedder port, its serving and ops — but starts only once the evaluation harness exists, so every model is measured against the API baseline instead of claimed. Slices ship on their own and the project is stoppable at any slice.

## Repo layout

```
apps/web      Next.js frontend: upload, chat with citations, quizzes, feedback
apps/api      Fastify API: ports, adapters, config, LangGraph.js flow, guardrails
evals/        Python: golden dataset, retrieval metrics, judge, CI gate
redteam/      Python: injection PDFs and attack cases
docs/adr/     one short decision record per choice
```

