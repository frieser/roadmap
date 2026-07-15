---
---

## Git Core (01–06)

| Concept | Command / Fact |
|---------|----------------|
| Init/Clone/Commit | `git init`, `clone`, `add`, `commit -m`, `--amend` |
| Branch/Merge | `switch -c`, `merge`, `branch -d`. FF, `--no-ff`, rebase (never shared), squash, cherry-pick |
| Remote | `fetch` (download), `pull`=fetch+merge, `push`. Fork=server copy. |
| Conflicts | `git mergetool`. Marked `<<<<<<`. `git config --global user.name/email`. |

## Undo & Diff (11, 13, 14)

| Command | Effect |
|---------|--------|
| `git revert <sha>` | Safe inverse commit |
| `git reset --soft/mixed/hard` | HEAD / +index / +working (DESTRUCTIVE) |
| `git reflog` / `git bisect` | Recovery / find bug commit |
| `git diff` / `--staged` / `A..B` | Unstaged / staged / branch |
| `git filter-repo` / `--force-with-lease` | History rewrite (modern) / safer force push |

## GitHub Platform (05, 07, 12, 18, 19, 24, 25)

| Feature | Key Facts |
|---------|-----------|
| CLI (`gh`) | `gh repo/pr/issue/run`. Automation. |
| PRs & Issues | Labels, assignees, milestones, `@mention`. |
| Actions | YAML. `on: push/pull_request/schedule`. `${{ secrets.X }}`. `cache@v4`. |
| Projects | Kanban, roadmaps, automations. |
| Security | Dependabot, code/secret scanning, advisories. |
| Pages/API | Jekyll/Hugo. REST+GraphQL (PAT, OAuth). Copilot, Gists, Codespaces. |

## Advanced Git (15, 16, 17, 20)

| Topic | Key Point |
|-------|-----------|
| Submodules | `add/update`. Nested repos. |
| Hooks | `.git/hooks/`. pre-commit, commit-msg, pre-push. |
| Tags/Releases | `tag -a v1.0`. GitHub Releases = annotated tags. |
| LFS / Worktree | Pointer files for binaries. Multiple branches checked out. |

## Pre-trained Models (26) + Prompt Engineering (27) + Embeddings (28)

| Concept | Detail |
|---------|--------|
| OpenAI GPT-4o | General, vision, function calling |
| Claude 3.5 Sonnet | Safety, 200K context |
| Gemini 1.5 Pro | Multimodal, 1M context |
| OSS (Llama/Mistral/DeepSeek) | 7B–405B, self-hostable |
| Prompting | Zero-shot, few-shot (2–5ex), CoT ("step-by-step"), system prompts |
| Embeddings | Dense vectors (768–3072d). Cosine similarity. |

## Vector DBs (29) + RAG (30)

| DB | Style |
|----|-------|
| Pinecone | Managed, serverless |
| Milvus | OSS, GPU-accel |
| ChromaDB | OSS, lightweight |
| Weaviate | Hybrid, GraphQL |
| Qdrant | Rust, fast filter |

RAG: Ingest → Chunk (overlapping) → Embed → Top-K search → Context inject → Generate. Lost in Middle: best chunks at start+end.

## AI Agents (31)

| Component | Detail |
|-----------|--------|
| Loop | Perception → Reason → Act → Observe |
| ReAct | Think then Act. JSON Schema tool defs. |
| Frameworks | LangGraph (cyclic), CrewAI (roles), AutoGPT |
| Planning | CoT, Tree of Thoughts, Self-Reflection, Plan-Execute |
| Orchestration | Sequential, Hierarchical (manager/worker), Swarm |

## Fine-tuning (32)

| Concept | Rule |
|---------|------|
| RAG-first | RAG for facts, FT for style/format. Never FT volatile knowledge. |
| Dataset | Chat format (messages). Dedup + diversity + synthetic data. |
| LoRA | Train <1% params. Adapters = few MB. QLoRA + 4-bit = consumer GPU. |
| SFT | Base→Instruct via labeled pairs. Early stop on val loss. Alignment: RLHF/DPO. |

## Evaluation (33) + Deployment (34)

| Topic | Key |
|-------|-----|
| BLEU/ROUGE | N-gram overlap. Fast, shallow. |
| LLM-as-Judge | Semantic. Biases: positional, verbosity. Validate with Human-AI agreement. |
| Ragas | Faithfulness, Answer Relevancy, Context Recall/Precision. |
| Benchmarks | MMLU (knowledge), GSM8K (math), HumanEval (code), LMSYS Arena (ELO). |
| API deploy | OpenAI/Bedrock/Vertex. TPM+RPM. Exponential backoff+jitter. |
| Self-host | vLLM (PagedAttention), TGI, Ollama. A100/H100. K8s. |
| Quantization | GGUF (Mac), AWQ (server). 4-bit sweet spot. 4x less VRAM. |

## Responsible AI (35) + Dev Tools (36)

| Domain | Key |
|--------|-----|
| Bias | Selection/Label/Algorithmic. Demographic Parity, Equal Opportunity. |
| Privacy | PII masking, differential privacy. |
| Security | Prompt injection (direct+indirect), jailbreaking, model inversion. |
| Safety | Guardrails (Llama Guard, NeMo), Constitutional AI. EU AI Act, NIST RMF. |
| LangChain | Orchestration, LCEL (`pipe`), agents, memory. |
| LlamaIndex | Ingestion, LlamaHub, reranking, router queries. |
| LangSmith | Tracing, eval datasets, monitoring, regression testing. |

## Rules

1. RAG for facts, FT for style. Never FT volatile knowledge.
2. PII masking BEFORE vector DB or LLM.
3. `git fetch` → diff → merge. `--force-with-lease` > `--force`.
4. LLM-as-Judge must be stronger than eval target. Validate with Human-AI agreement.
5. 4-bit quantization sweet spot. GGUF for Mac, AWQ for GPU server.
6. Guardrails are independent layer. Never rely solely on alignment.
7. Lost in the Middle → best chunks at start+end of prompt.
8. LoRA adapters swappable at runtime. Train once per task.
9. Never rebase shared branches. Revert > reset on public history.
10. `gh` CLI for scripts, REST for integrations, GraphQL for complex queries.
