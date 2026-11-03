# literature-review

A research skill for literature discovery, close reading, citation checks, and evidence synthesis. Version **2.0.2**.

[中文](README.md) · [Skill entry](skills/literature-review/SKILL.md) · [Command reference](skills/literature-review/references/search_and_citation.md)

See [CHANGELOG.md](CHANGELOG.md) for changes. Local offline suite: 54 tests passed on 2026-11-03; this does not certify live services or scientific claims.

Simple searches, reading tasks, and citation checks can produce tables, reading notes, or bibliographies directly. Use the library tools when maintaining multiple sources, tracking updates, or conducting systematic/scoping reviews.

## Capabilities

- Search by research question and record identity, sources, retrieval dates, and coverage limits.
- Read or compare papers with evidence locations, conditions, and limitations; distinguish abstracts from full text.
- Check DOI metadata and BibTeX while treating existence and claim support as separate questions.
- Merge records offline with stable IDs, source observations, and version relationships; record evidence and screening decisions.
- Export selected bibliographies, evidence tables, reading maps, and search logs.

Search results do not establish complete coverage. Scripts do not automatically establish claim support, full-text identity, current retraction status, or study counts, and do not perform meta-analysis.

## Installation

Scripts require **Python 3.10+**; instruction-only reading tasks need no Python packages. Clone this repository or use Code → Download ZIP and extract it.

Copy the **entire** `skills/literature-review` directory into `~/.agents/skills/`, or the project's `.agents/skills/`. If your existing client recognizes a legacy `$CODEX_HOME/skills` or `~/.codex/skills` layout, you may keep using that location; choose one location, not both. Back up an existing same-name directory first. See the [official skill documentation](https://learn.chatgpt.com/docs/build-skills); restart the client if automatic discovery does not show the change.

Crossref/arXiv search and the offline library tools use only the Python standard library. For OpenAlex, PubMed, or other citation clients, install the optional dependency from the **repository root**:

```sh
python -m pip install -r requirements.txt
```

The installed skill also contains the optional `requirements.txt`. Other compatible clients should follow their own instructions; their integration has not been separately verified by this project.

## Usage

After installation, ask for a task directly, for example:

> Use literature-review to find recent research on active noise control, explain the search scope, and describe each paper's relevance.

> Read this paper closely and explain its method, key equations, evidence locations, and limitations. State whether the available material is an abstract or full text.

> Check these references' DOIs and metadata, and list claims that still require reading the original papers.

### Command examples

From the **repository root**, enter the skill directory. All subsequent relative paths use that directory. Use fresh output paths; the searches require network access.

```sh
cd skills/literature-review
python -c "from pathlib import Path; Path('project').mkdir(exist_ok=True)"
python scripts/search_sources.py crossref "active noise control" --limit 20 -o project/crossref.json
python scripts/search_sources.py arxiv "all:acoustic AND all:control" --limit 20 -o project/arxiv.json
python scripts/literature_pipeline.py merge --input project/crossref.json --input project/arxiv.json --output project/library-v1.json
python scripts/literature_pipeline.py export --library project/library-v1.json --output-dir project/delivery --only references.bib review-map.md
```

Crossref/arXiv fetch one page per call, accept `--limit` from 1–100, and always report `complete: false`. These are bounded discovery tools. For a simple search, summarize the retrieved results directly without merge/export.

To record evidence, read the actual `record_id` and `sources.observation_id` from the library, then create `project/evidence-items.json` according to the [data contract](skills/literature-review/references/library_schema.md):

```sh
python scripts/literature_pipeline.py evidence --library project/library-v1.json --input project/evidence-items.json --output project/library-v2.json
python scripts/literature_pipeline.py export --library project/library-v2.json --output-dir project/evidence-delivery --only evidence.csv references.bib
```

Evidence input accepts one object or an array. Full-text evidence requires an actual local file and a locator; support status defaults to `unresolved`. Without `--only`, export writes six files: `records.csv`, `evidence.csv`, `screening.csv`, `review-map.md`, `references.bib`, and `search-log.json`. Existing outputs are refused unless `--overwrite` is explicitly supplied.

See the [offline workflow](skills/literature-review/references/example_workflow.md) for a synthetic example requiring no network access.

### Retained citation tools

Five CLIs remain in `scripts/citation_tools/`:

| Tool | Purpose |
| --- | --- |
| `search_openalex.py` | OpenAlex search |
| `search_pubmed.py` | PubMed search |
| `extract_metadata.py` | Identifier metadata lookup |
| `doi_to_bibtex.py` | DOI to BibTeX |
| `validate_citations.py` | BibTeX checks, with optional online DOI checks |

Compatibility scripts `scripts/verify_citations.py` and `scripts/search_databases.py` are retained. The latter converts local records and does not search online databases. See the [command reference](skills/literature-review/references/search_and_citation.md) or each script's `--help` for parameters and limits.

## Layout and verification

```text
.
├── README.md / README.en.md
├── LICENSE
├── CHANGELOG.md
├── requirements.txt
├── .github/workflows/test.yml
└── skills/literature-review/
    ├── SKILL.md
    ├── THIRD_PARTY_NOTICES.md
    ├── scripts/
    ├── references/
    ├── assets/
    └── tests/
```

Run the offline tests from the **repository root**:

```sh
python -m unittest discover -s skills/literature-review/tests -v
```

GitHub Actions is configured to run the same type of tests. Offline tests do not establish live provider availability or scientific correctness. Check actual command output and the Actions page for run status.

## License and attribution

Released under the [MIT License](LICENSE), with attribution to K-Dense Inc. and other upstream contributors retained. See [THIRD_PARTY_NOTICES.md](skills/literature-review/THIRD_PARTY_NOTICES.md) for retained code, independently rewritten methods, and source notes. Preserve the applicable notices when redistributing.
