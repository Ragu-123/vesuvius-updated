# agents.md

## Purpose

This document tells a coding agent how to *understand* the repository and the scoring artifacts (logs and metric files) and what actions it is permitted to take. The agent's job is **analysis first**: inspect the score analysis files and related artifacts, reason about possible causes for the current metrics, and produce clear, testable code edits as pull requests if appropriate.

> Note: This document intentionally **does not** prescribe code changes or root-cause diagnoses. The agent should reach conclusions from data and propose edits; humans will review and merge PRs.

---

## Primary responsibilities for the agent

1. **Ingest** the repository and all available scoring artifacts, including but not limited to `score analysis.txt`, validation logs, metric scripts (e.g. `toposcore.py`, `voi.py`), and training logs.
2. **Analyze**: produce a concise analytical report that summarizes the most salient patterns in the score logs and metrics (per-volume and aggregate). Highlight anomalies, distributions (e.g. counts of Betti numbers, component counts, false positive/negative rates), and any correlations you can compute between model outputs and metric failures.
3. **Verify**: run the project's metric scripts and test harnesses (locally within the provided environment) to reproduce reported scores for selected volumes. Record commands, inputs, and exact outputs.
4. **Propose**: if analysis suggests code edits that could improve scores, prepare well-scoped code changes and open pull requests. Each PR must include thorough description, the tests run, and before/after metric numbers on the same sample set used for analysis.
5. **Document**: produce a short README-style summary for maintainers explaining how to reproduce the analysis, run the metrics, and evaluate a candidate PR.

---

## Files and artifacts to prioritize

* `score analysis.txt` (per-volume metrics and breakdowns)
* Validation logs (per-epoch and per-volume logs)
* Metric implementations: `toposcore.py`, `voi.py` (and any auxiliary metric scripts)
* Precomputation logs and artifacts (e.g. `precomputation updated.txt`, `.npy` label formats)
* Model code and training loop files (e.g. the main training script, model definitions such as `vnetupdated.txt` / `vesuvius_vnet_fixed.py`)
* Inference scripts and post-processing pipelines

The agent should treat these artifacts as primary sources of truth for analysis and must run metric scripts where feasible.

---

## Allowed actions (explicit)

The agent is authorized to perform the following actions in the repository environment:

* Read any repository file and any provided logs, `.npy` files, or metric outputs.
* Execute test and analysis scripts present in the repository in the local runtime environment to reproduce metrics and visualize results.
* Modify code and create branches with well-scoped changes.
* Open pull requests with proposed edits. Each PR must contain:

  * A short summary of intent (1–2 sentences).
  * A clear description of what was changed (file list and high-level description). Do **not** claim fixes unless validated.
  * Repro steps to run included tests and metric scripts.
  * Quantitative before/after metrics measured on the same sample set used in analysis.

Agents must not merge PRs themselves (human review is required). PRs should be safe, reversible, and include tests where appropriate.

---

## Constraints and guardrails

* **No prescriptive fixes in the document.** This file must not instruct the agent to change specific model architecture elements, loss weights, or other design choices. The agent may *discover* such candidates through analysis and include them in PRs with experimental evidence.
* **Reproducibility:** every experiment must include the exact commands, seed values, and input file list needed to reproduce the reported metrics.
* **Safety:** do not publish or remove user data. Avoid destructive operations outside the repository workspace. Keep changes minimal and focused.
* **Transparency:** every PR must include the raw output of metric scripts used to claim improvements (e.g., the output of `python toposcore.py --pred ... --gt ...` for the sample set).

---

## What to produce as deliverables

1. **Analytical Report (Markdown)**

   * Short executive summary (3–5 bullet points)
   * Key numerical findings (tables/plots): per-volume breakdown, distributions, correlations
   * Repro steps and commands
   * Candidate hypotheses (phrased as observations) — not prescriptive fixes

2. **If a code edit is proposed:** a pull request that contains

   * Description and motivation (evidence-based)
   * Files changed (diff)
   * Tests runnable by CI/local maintainer
   * Before/after metric evaluation on the same sample set

3. **Repro scripts** that allow maintainers to replicate the analysis (one-line commands preferred).

---

## How to structure a PR from the agent

* Branch name: `analysis/<short-topic>` (e.g. `analysis/topo-inspect-2026-02-05`)
* PR title: short, descriptive (50 characters max)
* PR body:

  1. Summary of what was done (2–3 lines)
  2. Files changed (list)
  3. How to test (commands)
  4. Results: raw metric outputs (before/after) and the script used to compute them
  5. Rationale: bullet list of analytical observations that motivated the change

---

## Minimal commands to reproduce common checks

* Run single-volume metric: `python toposcore.py --pred <pred.tif> --gt <gt.tif>`
* Run batch evaluation: `python evaluate_all.py --pred-dir ./pred --gt-dir ./gt --out results.csv` (if present)
* Run unit tests (if present): `pytest -q`

Include exact command output in PRs.

---

## Final note to the agent

Your first task: **read** `score analysis.txt` and any validation logs, run the project's metric scripts on a handful of example volumes, and produce a short analytical report (Markdown) summarizing what you found and listing candidate areas for code edits. If you choose to prepare code changes, open a PR with the testing and evidence described above.

Do not apply irreversible changes without PR review. Maintain clear, reproducible records for every experiment.
