# Ranked issues

**Filled by:** session 9 (the first list, `cap01-e5`), kept current until
session 14, which fixes rank 1 and adds its regression test.

At least three rows. Ranks 1, 2, 3... with no gap and no tie: two issues ranked
1 is a list nobody prioritised. The impact is what orders it.

The columns are the three fields `cap01-e5` reads.

| rank | issue | impact |
|---:|---|---|
| 1 | Local LLM citation drift under strict automated evaluation gates | Hurts the overall evaluation score on grounded questions when the local qwen model formats citations slightly differently than expected. |
| 2 | Language sensitivity on adversarial refusals and out-of-domain queries | Can occasionally bypass standard refusal language triggers if non-English inputs are not caught early enough by the prompt guard. |
| 3 | Increased inference latency during sequential local model execution | Slows down the end-to-end response time for multi-step reasoning loops when running locally via Ollama. |

## Rank 1, in progress

- The fix: Refined prompt formatting instructions and structured output parsing to enforce strict passage identifier returns.
- The regression test: `tests/test_agent.py::test_citation_recall_regression`
- Before and after: see [EVAL_REPORT.md](EVAL_REPORT.md).