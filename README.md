# llm-wiki

A skill for building and maintaining persistent, interlinked Markdown wikis from raw sources. The LLM writes and maintains all wiki content; you curate sources, direct analysis, and ask questions.

Inspired by [Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Built for [Obsidian](https://obsidian.md) as the primary client. Works with any LLM.

> Full workflow documentation in Spanish: [`docs/flujo-de-trabajo.md`](docs/flujo-de-trabajo.md)

---

## The problem this solves

RAG re-discovers the same relationships from scratch on every query. Nothing accumulates.

This skill compiles your sources into a wiki once and keeps it current. Concepts get their own pages. Pages link to each other. Every claim is tagged for provenance. When you archive a query answer, future queries use it as context. Your explorations compound.

```
RAG:      query → search chunks → answer → forget
llm-wiki: sources → compile → wiki → query → archive → richer wiki → better answers
```

## Who it's for

Anyone working with multiple documents who wants a structured knowledge base instead of a folder of files. Domain-agnostic: academic research, legal analysis, intelligence work, journalism, personal knowledge management, or any field where sources accumulate and need to be synthesized.

## Requirements

- Any LLM with file read/write capability (Claude, ChatGPT, Gemini, or others)
- [Obsidian](https://obsidian.md) as primary client (optional but assumed by the skill's conventions)
- No executables, no scripts, no external dependencies

## Installation

### Claude (claude.ai / Claude Code)

**Option A — npx skills:**
```bash
npx skills add git@github.com:hypr-lupo/llm-wiki.git
```

**Option B — manual:**
```bash
mkdir -p ~/.claude/skills/llm-wiki
curl -fsSL https://raw.githubusercontent.com/hypr-lupo/llm-wiki/main/SKILL.md \
  > ~/.claude/skills/llm-wiki/SKILL.md
```

### ChatGPT / Gemini / Other LLMs

Paste the contents of `SKILL.md` into your system prompt or custom instructions. The skill is written in Spanish and contains no provider-specific syntax.

## Usage

Trigger the skill in Spanish by telling your LLM:

| Command | Action |
|---|---|
| `inicializar wiki sobre [dominio]` | Create a new wiki for a domain |
| `ingerir [fuente]` | Process and integrate a new source |
| `consultar: [pregunta]` | Query the wiki for a synthesized answer |
| `auditar wiki` | Run a health check on the wiki |
| `cerrar sesión` | Generate a pending-tasks log for the session |

See [`docs/flujo-de-trabajo.md`](docs/flujo-de-trabajo.md) for the full operational workflow.

## Wiki structure

```
<wiki-root>/
├── SCHEMA.md              # Conventions and domain-specific rules
├── sources/               # Raw source documents (immutable)
│   └── assets/
└── wiki/
    ├── indice.md          # Content catalog (MOC)
    ├── bitacora.md        # Chronological operation log
    ├── panorama.md        # High-level synthesis
    ├── pendientes.md      # Pending tasks and open questions
    ├── referencias.md     # Full bibliography (APA 7)
    ├── entidades/         # People, organizations, places
    ├── conceptos/         # Ideas, theories, frameworks
    ├── fuentes/           # One summary page per ingested source
    └── analisis/          # Archived query answers
```

## Provenance system

Every claim in the wiki is tagged for traceability using standard Markdown footnotes — compatible with Pandoc export to DOCX, LaTeX, PDF, and HTML.

Three categories:

| Tag | Meaning |
|---|---|
| `[^ext-N]` | Directly extracted from a source |
| `[^inf-N]` | Inferred by the LLM from one or more sources (with confidence 0.0–1.0) |
| `[^amb-N]` | Ambiguous — sources conflict, flagged for human review |

```markdown
The organization was founded in 1998 [^ext-1].
This was likely a regional strategy [^inf-1].
Founding dates differ across sources [^amb-1].

[^ext-1]: **Extracted** — [[source-a]], p. 45.
[^inf-1]: **Inferred** (confidence 0.7) — combining [[source-a]] and [[source-b]].
[^amb-1]: **Ambiguous** — [[source-a]] says 1998; [[source-c]] says 1999.
```

## Citations

APA 7th edition throughout. Academic export supported via Pandoc.

## Two-phase ingestion pipeline

Ingestion runs in two conceptual phases to eliminate source-order dependency:

1. **Extraction (read-only):** identify all entities, concepts, and claims in the source; classify each as extracted / inferred / ambiguous.
2. **Materialization:** create or update pages, add provenance tags, flag contradictions, update index and log.

A single source typically touches 5–15 wiki pages.

## Inspired by

- [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [atomicmemory/llm-wiki-compiler](https://github.com/atomicmemory/llm-wiki-compiler)
- [safishamsi/graphify](https://github.com/safishamsi/graphify)
- [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills)
- [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills)

## License

MIT
