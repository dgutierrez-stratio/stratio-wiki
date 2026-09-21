# Stratio Wiki

An LLM-compiled knowledge base documenting the components that make up the Stratio product —
architecture, configuration, operation, and how the pieces fit together.

**This repository does not duplicate documentation that already exists.** Stratio's product docs,
GitOps repos, and other internal sources remain the source of truth. What lives here instead is a
growing set of standardized cards and artifacts — component cards, architecture cards, and
whatever else proves useful — distilled *from* those existing sources so the entire Stratio
product architecture becomes accessible and explorable from many angles at once: by component, by
dependency, by security posture, by deployment topology, and more. It grows through automated work
performed by LLMs using the skills and plugins of the Stratio ecosystem, not through manual
transcription.

The wiki isn't written by hand. Source material (docs, repos, notes, URLs) is ingested into
`raw/`, then an LLM agent compiles it into cross-referenced articles under `wiki/`. The
repository *is* the wiki: everything is committed and versioned like normal code, so the whole
team can grow and query it together.

## Prerequisites

- [Claude Code](https://claude.com/claude-code) CLI.
- The **[`llm-wiki`](https://github.com/nvk/llm-wiki)** plugin (provides the `wiki` plugin and
  its `/wiki:*` commands). This repo was built with plugin version `0.3.3`.
- The **`stratio-live-docs`** plugin (authenticated read access to
  `sso.stratio.com/documentation`) and the **`keos-apps`** plugin (`component-card` /
  `architecture-card` skills, and local checkouts of the `keos-apps`, `keos-system-services`, and
  `keos-fleet` GitOps repos) — only needed to reproduce or extend the raw component-card material
  described in [Building this knowledge base](#building-this-knowledge-base) below, not to query
  or compile the wiki itself.
- Optionally, [Obsidian](https://obsidian.md) to browse the wiki visually — a `.obsidian/`
  vault config is committed, and articles use Obsidian-compatible `[[wikilinks]]` alongside
  standard markdown links.

## Setup

Add the plugin's marketplace and install it:

```bash
claude plugin marketplace add nvk/llm-wiki
claude plugin install wiki@llm-wiki
```

Then, from a clone of this repository, any `/wiki:*` command (e.g. `/wiki:query`,
`/wiki:ingest`, `/wiki:compile`) auto-resolves to this repo as the active wiki — no extra flags
needed. See [How this repo maps to the plugin](#how-this-repo-maps-to-the-plugin) below for why.

## Using the wiki

- `/wiki:query "<question>"` — ask the wiki what it knows, with citations to source articles.
- `/wiki:ingest <url|file|text>` — add a new source into `raw/`.
- `/wiki:research "<topic>"` — parallel-agent web research that ingests and compiles sources
  automatically.
- `/wiki:compile` — synthesize any ingested-but-uncompiled sources into `wiki/` articles.
- `/wiki:lint` — health-check the wiki structure, indexes, and links; `--fix` to auto-repair.
- `/wiki:output "<request>"` — generate a summary, report, study guide, etc. into `output/`.

Full command reference and conventions live in the plugin itself
([`llm-wiki` on GitHub](https://github.com/nvk/llm-wiki)).

## Building this knowledge base

Beyond the generic `/wiki:*` commands above, this wiki is being built up in stages using two
other plugins that know how to talk to Stratio's own systems: `stratio-live-docs` (reads the
official product documentation portal) and `keos-apps` (audits this org's GitOps repos —
`keos-apps`, `keos-system-services`, `keos-fleet` — for how each component is actually deployed).
Each stage adds raw material under `raw/data/`; none of it is compiled into `wiki/` articles yet.

### Stage 1 — Component inventory (done)

Produces one short, external-facing "component card" per product component: what it is, what it
depends on, what depends on it, how it's exposed, and its security posture — grounded in the real
GitOps configuration, not just the docs.

1. List every component in the current release:
   ```
   /stratio-live-docs:browse list-components latest
   ```
2. For each component returned, run the card generator, pointing it at this wiki's raw-data
   folder instead of its own default location:
   ```
   /keos-apps:component-card COMPONENT_NAME raw/data/component-cards
   ```
3. Repeat for every component in the list. Cards land in
   [`raw/data/component-cards/`](raw/data/component-cards/), one file per component.

### Stage 2 — Architecture deep-dive (not yet run)

The natural next step: for each component that turns out to matter, run the companion
`keos-apps:architecture-card` skill to capture internal architecture (tech stack, internal
data/control flow, HA/DR, API contracts) — the ground `component-card` deliberately leaves out.
Same pattern as above, pointed at a sibling folder:

```
/keos-apps:architecture-card COMPONENT_NAME raw/data/architecture-cards
```

This hasn't been run yet — `raw/data/architecture-cards/` doesn't exist. Whoever picks this up
should decide whether to run it for every component or only the ones that came up as heavily
depended-upon in the Stage 1 cards (Stratio Virtualizer, Stratio PostgreSQL, Stratio GoSec, and
Stratio Connectors are the obvious candidates).

### Then: compile

Both stages only populate `raw/`. Nothing here has gone through `/wiki:compile` yet, so there are
no synthesized, cross-referenced articles under `wiki/` yet either.

## Repository layout

```
_index.md, config.md, log.md   # wiki root: stats/nav, scope, activity log
inbox/                         # drop zone for material not yet ingested
raw/{articles,papers,repos,notes,data}/  # immutable ingested sources
wiki/{concepts,topics,references,theses}/  # compiled, synthesized articles
output/                        # generated artifacts (summaries, reports, ...)
.obsidian/                     # Obsidian vault config
```

`raw/` is never hand-edited after ingestion — it's the record of what was ingested and when.
All synthesis happens in `wiki/`. See a directory's own `_index.md` for its contents.

## How this repo maps to the plugin

By default, the plugin nests wiki content under a hidden `.wiki/` folder (or a shared
`~/wiki/` hub) so it stays out of the way of a project's real files. Since this repository's
entire purpose *is* the wiki, we deliberately deviated: content lives directly at the repo
root so it's fully visible when browsing on GitHub, and nothing is gitignored.

A `.wiki -> .` symlink is kept at the root purely as a compatibility shim, so the plugin's
built-in "does the current directory have a `.wiki/`?" resolution rule finds this wiki
automatically. It's not meant to be opened directly — treat the root-level folders as the real
content.

## Contributing

Anyone with the plugin installed can extend this wiki the same way it was built:

1. `/wiki:ingest` a source, or run `/wiki:research` on a topic.
2. `/wiki:compile` to fold new sources into articles.
3. `/wiki:lint` before committing, to catch broken links or stale indexes.
4. Commit and push — `raw/`, `wiki/`, and `output/` are all meant to be tracked in git.
