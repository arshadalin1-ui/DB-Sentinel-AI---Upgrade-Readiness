# DB Sentinel AI

**A structured, evidence-based upgrade-readiness assessment framework for heterogeneous database and infrastructure systems.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![Paper](https://img.shields.io/badge/paper-IEEE%20OJ--CS-b31b1b.svg)](./manuscript.tex)
[![Engines](https://img.shields.io/badge/engines-6-brightgreen.svg)](#supported-engines)

This repository is the reference implementation accompanying the paper *"DB Sentinel AI: A Structured Knowledge-Retrieval Framework for Automated Upgrade-Readiness Assessment Across Heterogeneous Database and Infrastructure Systems."* Every table and figure in the paper is reproducible from this repository — see [Reproducing the Paper's Results](#reproducing-the-papers-results).

---

## Table of Contents

- [What This Is (and Isn't)](#what-this-is-and-isnt)
- [Supported Engines](#supported-engines)
- [Architecture](#architecture)
- [Live Dashboard Results](#live-dashboard-results)
- [Headline Results](#headline-results)
- [Quick Start](#quick-start)
- [Repository Structure](#repository-structure)
- [Reproducing the Paper's Results](#reproducing-the-papers-results)
- [Limitations](#limitations-stated-plainly)
- [Citation](#citation)

---

## What This Is (and Isn't)

DB Sentinel AI is a **deterministic, rule-based evidence validator** combined with a **lexical (keyword/filename) knowledge-retrieval layer**. It is **not** a trained or learned model, and does **not** use embeddings, LLMs, or autonomous agents. We describe it this way deliberately and consistently in the accompanying paper's Discussion section — auditable, hand-inspectable rules are a deliberate design choice for a system that gates production database upgrades, not an oversight.

## Supported Engines

| Engine | Status |
|---|---|
| SQLite | ✅ Supported |
| PostgreSQL | ✅ Supported |
| MongoDB | ✅ Supported |
| Cassandra | ✅ Supported |
| Docker | ✅ Supported |
| Kubernetes | ✅ Supported |
| MySQL | 🚧 Planned (see paper's Future Work) |

---

## Architecture

![DB Sentinel AI architecture](figs/fig1_architecture.png)

Evidence-driven deterministic rule validation runs alongside lexical knowledge retrieval over an organisation's release-note and runbook corpus. Both signals feed a staged composite scoring pipeline (**Baseline → SRAG → RAG**) and a four-component **TVK** reliability decomposition (Technical Validation, Version Compatibility, Retrieved Knowledge, Runtime Evidence), which together determine the readiness classification and populate the JSON, CSV, and HTML reports.

---

## Live Dashboard Results

These are **real, unedited outputs** from the deployed pipeline — not mockups. Full interactive HTML files are included in this repository at [`docs/results_html/`](docs/results_html/); screenshots below for quick viewing.

### Populated-Evidence Run (healthy production snapshot)

![Populated evidence dashboard](docs/screenshots/dashboard_populated_evidence.png)

Five of six engines classified **EXCELLENT**; MongoDB correctly downgraded to **GOOD** due to two genuine detected findings (an FCV/target-version mismatch and a missing-index candidate) — not a uniform pass-through.

📄 [View the full interactive HTML report](docs/results_html/populated_evidence_run/index.html) &nbsp;|&nbsp; [Raw JSON](docs/results_html/populated_evidence_run/report.json) &nbsp;|&nbsp; [CSV summary](docs/results_html/populated_evidence_run/engine_summary.csv)

### Zero-Evidence Stress Test (fail-safe behaviour)

![Zero evidence dashboard](docs/screenshots/dashboard_zero_evidence.png)

All six engines correctly and uniformly classified **HIGH_RISK** (Risk score 100) when no runtime evidence or knowledge documents are supplied — the framework never reports a false sense of readiness from missing data.

📄 [View the full interactive HTML report](docs/results_html/zero_evidence_run/index.html) &nbsp;|&nbsp; [Raw JSON](docs/results_html/zero_evidence_run/report.json) &nbsp;|&nbsp; [CSV summary](docs/results_html/zero_evidence_run/engine_summary.csv)

---

## Headline Results

| Engine | Baseline | SRAG | RAG | Risk | Readiness |
|---|---|---|---|---|---|
| SQLite | 92.0 | 95.08 | 99.28 | 0 | EXCELLENT |
| PostgreSQL | 86.0 | 89.22 | 93.82 | 4.55 | EXCELLENT |
| MongoDB | 75.85 | 79.0 | 83.4 | 22.7 | **GOOD** |
| Cassandra | 94.0 | 97.0 | 100 | 0 | EXCELLENT |
| Docker | 92.0 | 94.85 | 98.45 | 0 | EXCELLENT |
| Kubernetes | 94.0 | 97.08 | 100 | 0 | EXCELLENT |

**Ablation study** — removing the knowledge-retrieval corpus entirely collapses Baseline = SRAG = RAG exactly for every engine, confirming the retrieval-adjustment terms contribute a real, bounded effect (+6.0 to +7.8 RAG points) rather than being cosmetic.

**Severity-injection study** — a single CRITICAL-severity finding alone (e.g., a failed `pg_upgrade_check`), with every other evidence field held healthy, is sufficient to drop a classification from EXCELLENT to HIGH_RISK — verified across PostgreSQL, MongoDB, and Kubernetes independently.

Full details, tables, and the reproducibility script for both studies: [`REPRODUCING_PAPER_RESULTS.md`](./REPRODUCING_PAPER_RESULTS.md).

---

## Quick Start

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Populate evidence templates (or write your own evidence.json per engine)
python -m db_sentinel_ai.cli init-evidence --output evidence/real

# Run the full analysis
python -m db_sentinel_ai.cli analyze \
  --config config/default.yaml \
  --evidence evidence/real \
  --output output/my_run

# View the results
open output/my_run/latest/index.html   # macOS
# or: xdg-open output/my_run/latest/index.html   # Linux
```

## Knowledge Corpus

The [`knowledge/`](knowledge/) directory contains a small, real, engine-relevant document per engine (release-note/runbook style), authored specifically so the retrieval-augmentation results in the paper are reproducible by anyone cloning this repo — no machine-specific paths required.

## Evidence Schema

Each engine's `required` evidence fields are declared in its validator module under [`db_sentinel_ai/engines/`](db_sentinel_ai/engines/). Run `init-evidence` to generate a populated example `evidence.json` per engine. Real-world evidence-collection commands per engine (e.g., `psql`, `nodetool`, `kubectl`) are documented in [`docs/REAL_COLLECTION_COMMANDS.md`](docs/REAL_COLLECTION_COMMANDS.md).

---

## Repository Structure

```
db-sentinel-ai/
├── db_sentinel_ai/
│   ├── cli.py                  # Command-line entry point
│   ├── core/
│   │   ├── pipeline.py         # Orchestrates the full per-engine pipeline
│   │   ├── scoring.py          # Baseline / SRAG / RAG / Risk / TVK formulas
│   │   ├── knowledge.py        # Lexical knowledge-retrieval layer
│   │   ├── models.py           # Finding / EngineResult data classes
│   │   └── io.py               # YAML/JSON I/O helpers
│   ├── engines/                 # Per-engine deterministic validation rules
│   │   ├── sqlite.py, postgresql.py, mongodb.py,
│   │   │   cassandra.py, docker.py, kubernetes.py
│   │   └── registry.py         # Engine name -> class registry
│   └── reports/writer.py        # JSON / CSV / HTML report generation
├── config/default.yaml           # Engine list, knowledge paths, scoring thresholds
├── evidence/real/                # Per-engine evidence.json snapshots
├── knowledge/                    # Reproducible knowledge corpus (see above)
├── figs/                         # Architecture diagrams used in the paper
├── docs/
│   ├── screenshots/              # Dashboard screenshots (quick viewing)
│   ├── results_html/              # Full interactive HTML reports (real output)
│   ├── example_results/           # Raw JSON/CSV from both evaluation conditions
│   └── REAL_COLLECTION_COMMANDS.md  # Real evidence-collection commands per engine
├── scripts/
│   ├── smoke_test.py             # End-to-end smoke test
│   └── severity_injection_test.py # Reproduces the severity-injection study
└── REPRODUCING_PAPER_RESULTS.md   # Exact command for every table/figure in the paper
```

---

## Reproducing the Paper's Results

See [`REPRODUCING_PAPER_RESULTS.md`](./REPRODUCING_PAPER_RESULTS.md) for the exact command mapped to every table and figure in the paper: the zero-evidence stress test, the populated-evidence run, the retrieval ablation study, and the cross-engine severity-injection study. All four conditions are independently reproducible from this repository alone — no machine-specific paths or external data required.

> **Note:** if you empty the evidence directory to reproduce the zero-evidence stress test, remember to also point `knowledge_paths` at an empty directory (see `REPRODUCING_PAPER_RESULTS.md` for the exact command) — since this repository ships a real, populated `knowledge/` corpus, a run with only the evidence directory emptied will still register knowledge-retrieval points and will not reproduce Table II's exact figures.

---

## Limitations, Stated Plainly

- **Deterministic rules, not a learned model.** Every finding traces to an explicit, hand-authored conditional check — there is no trained component.
- **Lexical, not embedding-based, retrieval.** Knowledge documents are matched by filename alias and keyword hits, not dense semantic similarity.
- **MySQL is not yet supported.**
- **Scoring weights are domain judgement**, not fitted against a labelled corpus of real upgrade-incident outcomes.

We report these candidly in the paper's Discussion section rather than overclaim capability the system doesn't have.

---

## Citation

If you use this framework, please cite the accompanying paper (see [`CITATION.cff`](./CITATION.cff)).

## License

MIT — see [`LICENSE`](./LICENSE).
