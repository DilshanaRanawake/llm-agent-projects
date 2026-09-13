# llm-agent-projects

Five small projects I built to learn different agent/RAG patterns, using local LLMs
(Llama 3.2 3B via LM Studio) wherever possible so the whole series cost nothing to run.
Four of them share the same underlying data (a corpus of Sinhala NLP papers); one is a
separate experiment in a different pattern. Not one big pipeline - more like a set of
related experiments, each with its own repo, real output, and (where relevant) real bugs
I hit and fixed.

## How they relate

| # | Project | Corpus | Builds on |
|---|---|---|---|
| 1 | [research-mcp-server](https://github.com/DilshanaRanawake/research-mcp-server) | Sinhala NLP papers | - |
| 2 | [reasoning-rag](https://github.com/DilshanaRanawake/reasoning-rag) | same corpus as #1 | reuses #1's chroma vector store directly |
| 3 | [hierarchical-research-agent](https://github.com/DilshanaRanawake/hierarchical-research-agent) | arXiv gesture-recognition papers (different topic) | standalone, not connected to the others |
| 4 | [live-data-vs-static-agent](https://github.com/DilshanaRanawake/live-data-vs-static-agent) | same corpus as #1 | reuses #1's chroma vector store directly |
| 5 | [EDD retrofit](https://github.com/DilshanaRanawake/research-mcp-server) | - | added directly into #1's repo as a CI eval suite |

Worth being upfront about: #1, #2, #4, and #5 form a real throughline - same paper
corpus, each one building on the last. #3 is a separate exercise in a different pattern
(parallel multi-agent orchestration vs. retrieval reasoning) using a different topic
entirely. They don't run together as one system, and I'm not claiming they do -
positioning it that way would fall apart the moment someone asked to see it demoed live.

## 1. research-mcp-server

Semantic search over my own research papers (a thesis + several Sinhala NLP papers),
exposed as an MCP server so Claude Desktop can query them directly in conversation.

- `sentence-transformers` embeddings + `chromadb` vector store
- Three MCP tools: `search_papers`, `list_papers`, `get_paper_chunks`
- Hit a real timeout bug getting it running in Claude Desktop (eager model loading vs.
  the ~60s `initialize` handshake) - documented in the repo's README with the fix
- Later retrofitted with an automated eval suite + CI (see #5)

[repo](https://github.com/DilshanaRanawake/research-mcp-server)

## 2. reasoning-rag

Same corpus as #1, but instead of one fixed retrieve-then-answer pass, a small
LangGraph state machine lets the model decide for itself whether it has enough
information yet, or needs to search again (up to 3 rounds).

- Decide -> retrieve -> answer loop, not a fixed pipeline
- Real example in the README where it retrieves the right numbers but reasons about
  them incorrectly - a useful reminder that retrieval and reasoning can fail
  independently, even on the same small model

[repo](https://github.com/DilshanaRanawake/reasoning-rag)

## 3. hierarchical-research-agent

A different pattern entirely: Searcher -> Analyst (run in parallel) -> Writer, applied
to live arXiv search + a weekly digest generator, not the Sinhala paper corpus.

- Measured a real 1.21x speedup from parallelizing the Analyst step - modest, and
  documented exactly why (LM Studio's own concurrency limit was the bottleneck, not the
  orchestration code)

[repo](https://github.com/DilshanaRanawake/hierarchical-research-agent)

## 4. live-data-vs-static-agent

Same corpus as #1 again, this time comparing an agent that can only see the static
papers against one that can also search the live web, scored on 10 real questions
against my own researched ground truth.

- Live search helped exactly once, on the one question it was built for (a genuinely
  current fact the static papers couldn't have)
- Also introduced a new failure mode the static agent didn't have - overrode a correct
  retrieved answer with a hallucinated one. Spent a few rounds trying to prompt-fix it,
  documented where I decided to stop chasing it instead of pretending it was solved

[repo](https://github.com/DilshanaRanawake/live-data-vs-static-agent)

## 5. EDD retrofit (inside research-mcp-server)

Went back and added an actual automated eval suite + CI to #1, instead of just
eyeballing whether search results looked right.

- 12 eval cases, keyword-based checks on retrieval quality
- Proved it catches real regressions by deliberately breaking `n_results=3` -> `1` and
  watching a case fail for the right reason
- First CI run failed for a reason worth documenting: the vector store was gitignored,
  so a fresh checkout had nothing to query. Fixed by rebuilding the index from a
  (newly-tracked) intermediate JSON file instead of committing the binary DB itself

[same repo as #1](https://github.com/DilshanaRanawake/research-mcp-server)

## What I'd do differently next time

- Name repos consistently from the start - #1's local folder is called
  `research-mcp-server` everywhere in code and docs, but the actual GitHub repo is
  `multimodal-rag-agent` from an early naming decision. Harmless functionally, mildly
  confusing for anyone browsing.
- Start the eval suite (project #5) alongside the first project instead of retrofitting
  it after three other projects were already built on the same untested retrieval code.
