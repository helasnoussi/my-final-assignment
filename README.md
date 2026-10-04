# my-final-assignment

An autonomous local research assistant built for the Dev3Pack capstone to answer developer questions strictly from a versioned document corpus with verified citations and robust safety defenses.

## Overview

This project is a grounded QA system that answers questions using a small, controlled corpus instead of relying on unverified model memory. It is designed to:

- answer only from the provided source documents,
- cite the document used for each answer,
- refuse unsafe or out-of-domain requests,
- guard against prompt-injection attempts and unsupported claims.

## The problem

Developers and technical teams often need precise answers drawn exclusively from internal documentation without hallucinated facts or unverified claims. In real-world settings, naive LLMs may fabricate answers, answer questions outside the allowed corpus, or follow malicious instructions hidden in user input.

This project aims to keep the agent reliable by enforcing a strict retrieval-and-verification workflow.

## Demo

The commands below are examples from the project’s trace output. `trace` prints each step taken by the agent before returning the answer.

### Supported answer

```bash
uv run bootcamp capstone trace "How does chunking work in RAG?"
```

Example output:

```text
[retrieve] top_k=3 -> [('rag-basics', 0), ('evaluation-basics', 0), ('rag-basics', 1)]
[llm_call] attempt 1: 260 chars
[decision] answered with citations ['rag-basics']

answer: Chunking splits documents into passages small enough to be individually relevant — respecting paragraph boundaries beats cutting at a fixed character count mid-sentence.
citations: ['rag-basics']
confidence: 1.0
needs_human_review: False
```

### Refusal example

```bash
uv run bootcamp capstone trace "What is the capital city of Mongolia?"
```

Example output:

```text
[retrieve] top_k=3 -> []
[decision] no relevant chunks; refusing without an LLM call

answer: I don't know based on the provided corpus.
citations: []
confidence: 0.0
needs_human_review: True
```

## Architecture

The agent follows a controlled linear pipeline:

1. Incoming queries are screened for prompt-injection or unsafe patterns.
2. Relevant passages are retrieved from the local corpus using a small retrieval step.
3. The model evaluates only the retrieved evidence.
4. The system either answers with citations or refuses when the corpus does not support the claim.

This behavior is documented in [docs/adr/0001-run-shape.md](docs/adr/0001-run-shape.md).

## Measured results

The numbers below come from commands run on this commit. When no API key is configured, the project falls back to the offline fake model for CI-safe checks.

| Check | Command | Model | Result |
| --- | --- | --- | --- |
| Contract tests | `uv run pytest` | fake / ollama | 7 passed, 2 skipped |
| Practice grader | `uv run bootcamp final grade` | ollama (`qwen2.5:7b-instruct`) | score: 10/10 (100%) — PASSED |
| Evaluation before/after | see [docs/EVAL_REPORT.md](docs/EVAL_REPORT.md) | ollama (`qwen2.5:7b-instruct`) | Historical comparison; results can vary because local generation is nondeterministic |

## Honest limitation

Local LLM generation is nondeterministic, so individual grounded cases can vary
between practice runs. The agent reduces this risk with source-based extraction,
canonical citation validation, and refusal handling for unsupported claims.

The ranked issue list is tracked in [docs/ISSUES.md](docs/ISSUES.md).

## Getting started

```bash
git clone https://github.com/helasnoussi/my-final-assignment.git
cd my-final-assignment
uv sync
uv run pytest
```

### Optional model setup

No key is required to run the project in offline/fake mode. If you want to use a real model, copy the example env file and configure your provider:

```bash
cp .env.example .env
# fill in your provider configuration
```

Then install the relevant extra if needed:

```bash
uv sync --extra anthropic
# or
uv sync --extra openai
```

### Final submission flow

To submit the assignment:

```bash
uv run bootcamp capstone submit --github helasnoussi
```

This runs the practice set first, then answers the final questions and opens the pull request. The `--dry-run` flag shows the bundle without submitting anything.

## Repository layout

| Path | What it is |
| --- | --- |
| [agent.py](agent.py) | The main agent implementation (`YourAgent`) |
| [tests/test_contract.py](tests/test_contract.py) | Contract tests for the capstone behavior |
| [data/corpus/](data/corpus/) | The six source documents used as the trusted corpus |
| [docs/EVAL_REPORT.md](docs/EVAL_REPORT.md) | Evaluation results before and after changes |
| [docs/SKILL.md](docs/SKILL.md) | A reusable skill another assistant can load |
| [docs/RETENTION.md](docs/RETENTION.md) | Information about retention and refusal behavior |
| [docs/ISSUES.md](docs/ISSUES.md) | Ranked issue tracker |
| [docs/adr/0001-run-shape.md](docs/adr/0001-run-shape.md) | Architecture decision record for the run shape |

## Built with

This project was built during the Dev3Pack AI Engineering bootcamp, based on the course package at:

https://github.com/Gecko-Academy/dev3pack-cohort-2026-09

Commit:

`85ad371e3e6354fc18edb4522b1fd66ac6223f62`
