---
title: "Single-Document Retrieval-Augmented Generation System"
date: 2026-06-30
draft: false
projects: ["RAG for Kubernetes Troubleshooting"]
description: "A locally hosted Retrieval-Augmented Generation (RAG) system that ingests Kubernetes troubleshooting documentation, stores embeddings in Qdrant, and generates grounded responses using Ollama — with no external API calls."
tags: ["GenAI", "RAG", "LangChain", "Ollama", "Qdrant", "Python", "Vector Database", "Kubernetes"]
---

## Overview

Kubernetes troubleshooting documentation is dense, scattered, and hard to search by keyword. This project builds a fully local Retrieval-Augmented Generation (RAG) pipeline that lets an engineer ask natural-language questions — *"Why is my pod stuck in CrashLoopBackOff?"* — and get a grounded answer sourced directly from internal troubleshooting docs, with no data ever leaving the machine.

**Tech stack:** Python · LangChain · Qdrant · Sentence Transformers · Ollama (Qwen2.5-Coder)

---

## Why these tools

A few decisions were intentional rather than default choices:

- **Qdrant over alternatives (FAISS, Chroma):** Qdrant runs as a persistent service with a REST/gRPC API, which made it easier to inspect stored vectors during development and would let the system scale to multi-document retrieval without re-architecting storage.
- **Ollama + Qwen2.5-Coder over a hosted API:** Running the LLM locally removes any dependency on external API keys or network calls, which matters for infrastructure tooling that may need to run in air-gapped or security-sensitive environments. Qwen2.5-Coder was chosen specifically because the source documents are technical/config-heavy, where code-tuned models tend to perform better at parsing YAML and log snippets.
- **Sentence Transformers for embeddings:** Lightweight enough to run on CPU, with strong out-of-the-box performance for semantic similarity on technical text — no GPU requirement for this phase of the project.

---

## Architecture

```text
kubernetes_troubleshooting.txt
        │
        ▼
  Document Loader (LangChain TextLoader)
        │
        ▼
  Text Splitter (chunking)
        │
        ▼
  Embedding Model (Sentence Transformers)
        │
        ▼
  Qdrant Vector Store  ◄──────────────┐
        │                              │
        ▼                              │
  User Question ──► Embed ──► Similarity Search
        │
        ▼
  Prompt Template (context + question)
        │
        ▼
  Ollama (Qwen2.5-Coder)
        │
        ▼
  Grounded Answer
```

---

## Implementation highlights

**1. Chunking the source documentation**

Splitting on fixed-size windows with overlap to preserve context across chunk boundaries:

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
)
chunks = splitter.split_documents(documents)
```

**2. Embedding and storing in Qdrant**

```python
from langchain_community.vectorstores import Qdrant
from langchain_community.embeddings import HuggingFaceEmbeddings

embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")

vector_store = Qdrant.from_documents(
    chunks,
    embeddings,
    url="http://localhost:6333",
    collection_name="k8s_troubleshooting",
)
```

**3. Retrieval and grounded generation**

```python
from langchain_community.llms import Ollama
from langchain.prompts import PromptTemplate

retriever = vector_store.as_retriever(search_kwargs={"k": 4})

prompt = PromptTemplate.from_template(
    """Use the context below to answer the Kubernetes troubleshooting question.
    If the answer isn't in the context, say so — do not guess.

    Context:
    {context}

    Question:
    {question}
    """
)

llm = Ollama(model="qwen2.5-coder")

def answer_question(question: str) -> str:
    docs = retriever.get_relevant_documents(question)
    context = "\n\n".join(d.page_content for d in docs)
    return llm.invoke(prompt.format(context=context, question=question))
```

> Snippets above are representative of the implementation pattern used in `app.py`; adjust to match your exact code if you'd like the README to mirror the repo precisely.

---

## Features

- Local document ingestion and chunking pipeline
- Semantic embedding generation via Sentence Transformers
- Persistent vector storage and similarity search via Qdrant
- Context-grounded answer generation via a locally hosted LLM (no external API calls)
- Prompt design that instructs the model to decline rather than hallucinate when context is insufficient

---

## Results

*(Replace with real numbers once you've benchmarked — even rough figures add credibility, e.g.:)*

- Tested against a set of common Kubernetes failure scenarios (CrashLoopBackOff, ImagePullBackOff, OOMKilled, etc.)
- Average end-to-end response time: *Xs* on CPU-only retrieval + local inference
- Retrieval precision spot-checked manually against source doc sections

---

## What's next

This single-document system is the foundation for a broader DevOps AI troubleshooting agent. Planned next steps:

1. **Multi-document RAG** — ingest multiple sources (runbooks, postmortems, official K8s docs) with source-aware retrieval.
2. **Kubernetes tool integration** — let the agent query live cluster state (`kubectl`, metrics-server) rather than relying solely on static docs.
3. **Full DevOps AI troubleshooting agent** — combine RAG, live tooling, and cloud integrations (AWS) into a single agent capable of diagnosing and suggesting remediations for real infrastructure incidents.

---

*Code for this project is available on [GitHub](#).*