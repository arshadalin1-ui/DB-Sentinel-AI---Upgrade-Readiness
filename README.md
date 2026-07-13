DB Sentinel AI
A structured, evidence-based upgrade-readiness assessment framework for heterogeneous database and infrastructure systems.
This repository is the reference implementation accompanying the paper "DB Sentinel AI: A Structured Knowledge-Retrieval Framework for Automated Upgrade-Readiness Assessment Across Heterogeneous Database and Infrastructure Systems." Every table and figure in the paper is reproducible from this repository using the commands in REPRODUCING_PAPER_RESULTS.md.
What this is (and isn't)
DB Sentinel AI is a deterministic, rule-based evidence validator combined with a lexical (keyword/filename) knowledge-retrieval layer. It is not a trained or learned model, and it does not use embeddings, LLMs, or autonomous agents. We describe it this way deliberately and consistently in the accompanying paper — see the paper's Discussion section for why we consider this an honest and appropriate design choice for a system that gates production database upgrades.
Supported engines: SQLite, PostgreSQL, MongoDB, Cassandra, Docker, Kubernetes. MySQL is not currently supported (see Future Work in the paper).
Architecture
Evidence Collection (evidence.json per engine)
        |
        v
Deterministic Validation Rules ---+
        |                         |
        v                         v
Structured Knowledge Retrieval -> Composite Scoring (Baseline -> SRAG -> RAG)
        |                         |
        +-------------------------+--> TVK Reliability Decomposition
                                   |
                                   v
                    Readiness Classification (EXCELLENT/GOOD/CAUTION/HIGH_RISK)
                                   |
                                   v
                    Reports: JSON, CSV, self-contained HTML dashboard
See docs/screenshots/ for real, rendered examples of the HTML dashboard output.
Quick Start
bashpython3 -m venv .venv
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
Knowledge Corpus
The knowledge/ directory contains a small, real, engine-relevant document per engine (release-note/runbook style), authored specifically so the retrieval-augmentation results in the paper are reproducible by anyone cloning this repo — no machine-specific paths required.
Evidence Schema
Each engine's required evidence fields are declared in its validator module under db_sentinel_ai/engines/. Run init-evidence to generate a populated example evidence.json per engine.
Reproducing the Paper's Results
See REPRODUCING_PAPER_RESULTS.md for the exact command mapped to every table/figure — zero-evidence stress test, populated-evidence run, retrieval ablation study, severity-injection sensitivity study.
Repository Structure
db_sentinel_ai/
├── cli.py                  # Command-line entry point
├── core/
│   ├── pipeline.py         # Orchestrates the full per-engine pipeline
│   ├── scoring.py          # Baseline / SRAG / RAG / Risk / TVK formulas
│   ├── knowledge.py        # Lexical knowledge-retrieval layer
│   ├── models.py           # Finding / EngineResult data classes
│   └── io.py               # YAML/JSON I/O helpers
├── engines/                 # Per-engine deterministic validation rules
│   ├── sqlite.py, postgresql.py, mongodb.py,
│   │   cassandra.py, docker.py, kubernetes.py
│   └── registry.py         # Engine name -> class registry
└── reports/writer.py        # JSON / CSV / HTML report generation

config/default.yaml           # Engine list, knowledge paths, scoring thresholds
evidence/real/                # Per-engine evidence.json snapshots
knowledge/                    # Reproducible knowledge corpus
docs/screenshots/              # Rendered HTML dashboard screenshots
scripts/smoke_test.py         # End-to-end smoke test
Citation
If you use this framework, please cite the accompanying paper (see CITATION.cff).
License
MIT — see LICENSE.
