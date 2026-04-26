# Pre-Deployment Safety Audit Harness

A small, reproducible harness for **pre-deployment safety evaluation of open-weight instruction-tuned LLMs**. Adapts the pattern of financial model-risk pre-deployment review to frontier-model misuse evaluation.

> **Motivation.** Before a credit model is deployed in a regulated lending business, it passes a pre-deployment review: a battery of tests against known failure modes, a documented residual-risk assessment, and explicit monitoring triggers. Frontier LLM deployments deserve the same discipline. This repository is a zero-to-one version of that audit pattern, applied to refusal behavior and jailbreak robustness.

## What it does

1. Runs **N harmful prompts** drawn from [StrongREJECT](https://github.com/alexandrasouly/strongreject) against **K models** under **M jailbreak conditions**.
2. Scores each response using an **LLM-as-judge** (claude-haiku-4-5 by default) to produce a `{compliant, partial, refused}` label plus rationale.
3. Produces a **standardized audit report** (plots + tables + example failures).
4. Maps findings to **deployment decisions** via a governance checklist.

## Quickstart

```bash
git clone https://github.com/<you>/safety-audit
cd safety-audit
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env     # then edit .env with your keys

# Smoke test: 3 prompts × 3 models × 1 condition
python scripts/run_audit.py --smoke

# Full run (as specified in configs/audit.yaml)
python scripts/run_audit.py --config configs/audit.yaml

# Build the report
python scripts/make_report.py --run results/<RUN_ID>
```

## Design choices

- **Hosted inference** (Together AI / Groq) — no GPU needed, total budget ~$10–20 for a full run.
- **LLM-as-judge over a classifier** — faster to iterate on prompt design; rationale is auditable.
- **Stratified prompt sample** from StrongREJECT — keeps the harness small enough to run in minutes while covering the taxonomy.
- **JSONL result format** — every generation stored with the full prompt, response, judge label, judge rationale, and config hash, so a reviewer can audit any decision.

## What this harness is NOT

- Not a capabilities benchmark.
- Not an adversarial-robustness state-of-the-art — it uses well-known, public jailbreak templates only.
- Not a production monitoring system — it is a pre-deployment artifact. The `docs/governance_checklist.md` sketches how production monitoring would look.

## Results

See [`docs/RESULTS.md`](docs/RESULTS.md) for the latest run summary. Raw artifacts are in `results/`.

## Governance framing

See [`docs/governance_checklist.md`](docs/governance_checklist.md) for the deployment-decision mapping.

## Repo layout

```
safety-audit/
├── README.md                  # this file
├── requirements.txt
├── .env.example
├── configs/
│   └── audit.yaml             # models, providers, judge, prompt subset
├── data/
│   └── prompts_sample.jsonl   # stratified StrongREJECT subset
├── src/
│   ├── __init__.py
│   ├── providers.py           # API wrappers: Together, Anthropic, OpenAI
│   ├── runner.py              # main eval loop
│   ├── judge.py               # LLM-as-judge
│   ├── templates.py           # jailbreak prompt templates
│   └── report.py              # aggregation + plots
├── scripts/
│   ├── run_audit.py
│   └── make_report.py
├── results/                   # run outputs (gitignored except .gitkeep)
└── docs/
    ├── methodology.md
    ├── RESULTS.md
    └── governance_checklist.md
```

## License & responsible use

MIT. This harness produces harmful prompts and model responses to those prompts, to measure refusal behavior — treat the `results/` directory accordingly and do not redistribute raw outputs. The audit is intended to support safety-improving deployment decisions, not to produce a "jailbreak recipe." See `docs/methodology.md` for the responsible-use statement.

## Acknowledgements

Uses StrongREJECT prompts and common public jailbreak templates. LLM-as-judge methodology draws on Zheng et al. 2023 and follow-on work.
