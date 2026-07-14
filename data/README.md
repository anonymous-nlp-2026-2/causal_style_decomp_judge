# Experiment Data

This directory contains the counterfactual pair datasets, raw judge scoring outputs, and NLI gate results for reproducing the main analyses.

## Directory Structure

### `counterfactual_pairs/`
Style-transformed text pairs with NLI verification scores. Each JSONL file contains one record per text pair.

| File | Axis | Benchmark | Judge/Rewriter | n |
|------|------|-----------|----------------|---|
| `formality_llmbar_qwen32b_n100.jsonl` | Formality | LLMBar | Qwen3-32B | 100 |
| `formality_llmbar_llama70b_n88.jsonl` | Formality | LLMBar | Llama-70B (scored) | 88 |
| `verbosity_llmbar_llama70b_n86.jsonl` | Verbosity | LLMBar | Llama-70B (scored) | 86 |
| `register_llmbar_llama70b_n94.jsonl` | Register | LLMBar | Llama-70B (scored) | 94 |

Key fields: `original_text`, `counterfactual_text`, `nli_cf` (NLI entailment score), `passed_cf` (gate pass/fail), `original_formality` (original style label), `score_original`/`score_counterfactual` (Likert scores where available).

### `judge_scores/`
Raw pairwise comparison outputs from target judges.

| File | Axis | Judge | Notes |
|------|------|-------|-------|
| `pairwise_formality_llmbar_qwen32b.jsonl` | Formality | Qwen3-32B | Primary |
| `pairwise_formality_llmbar_llama70b.jsonl` | Formality | Llama-70B | Cross-judge |
| `pairwise_verbosity_llmbar_llama70b.jsonl` | Verbosity | Llama-70B | Cross-judge |
| `pairwise_register_llmbar_llama70b.jsonl` | Register | Llama-70B | Exploratory |
| `pairwise_debiased_*` | Form./Verb. | Llama-70B | Debiased prompting control |

Key fields: `trial1_choice`, `trial2_choice` (A/B selections in position-swapped trials), `consistent` (same choice both orderings), `formal_wins` / equivalent field (style-direction preference).

### `nli_gate/`
NLI threshold sensitivity analysis results.

- `nli_threshold_sweep.json` — BT win probabilities across 11 NLI thresholds (Table in Appendix F)

### `human_annotation/`
Human annotation data for NLI gate validation (Appendix V).

- `annotation_sheet.csv` — 100 pairs (50 formality, 50 verbosity) with annotator judgments
- `annotator1_results.csv` — Annotator 1 binary labels

## Data Format
- `.jsonl`: One JSON object per line
- `.csv`: Standard CSV with header row
- `.json`: Single JSON object

## Reproduction
See `analysis/` and `csd_pipeline/` directories for the analysis scripts that consume these data files.
