# Causal Style Decomposition for LLM-as-Judge Evaluation

Code and paper for EMNLP 2026 submission: *The Likert Blindspot: Measurement-Mode Divergence in LLM Judge Style Evaluation*.

## Overview

This repository provides the **CSD** (Controlled Style Decomposition) pipeline for measuring style bias in LLM-as-Judge systems. CSD is a black-box diagnostic that combines NLI-gated content control, per-axis style decomposition, and position-controlled pairwise comparison to reveal measurement-mode divergence: statistically significant pairwise sensitivity to style that is absent from Likert scoring on the same data (the *Likert Blindspot*). Across four target judges, two style axes, and three benchmarks, the divergence is confirmed by seven statistical methods and shown to be structural via scaled replication.

## Repository Structure

```
csd_pipeline/          # Core pipeline modules
  data_prep.py         # Data preparation from LLMBar/AlpacaEval/MT-Bench
  counterfactual_gen.py # Style-flipping counterfactual generation
  content_verification.py # NLI + BERTScore semantic preservation gating
  judge_scoring.py     # Likert scoring with LLM judges
  pairwise_scoring.py  # Position-controlled pairwise comparison
  ate_analysis.py      # ATE computation, BT model, GEE decomposition

experiments/           # Experiment scripts (Table 1 analyses)
  exp010_binarized_likert.py  # Binarized Likert analysis
  exp011_nli_sensitivity.py   # NLI threshold sensitivity
  exp012_consistency_bias.py  # Position consistency analysis
  exp014_logistic_decomposition.py  # GEE logistic decomposition

analysis/              # Extended analysis and robustness checks
  bootstrap_10k_holm.py       # 10K bootstrap with Holm correction
  gee_diagnostics.py          # GEE model diagnostics
  verbosity_gee.py            # Verbosity-specific GEE analysis
  verbosity_length_confound.py # Length confound controls
  r15/fk_length_control.py    # FK readability covariate analysis
  r16/construct_divergence.py # Construct divergence adjudication

scoring/               # Judge-specific scoring scripts
  alpacaeval_qwen32b_scoring.py
  claude_judge_scoring.py
  gpt4o_crossbenchmark_scoring.py
  llama70b_scoring.py

figures/               # Figure generation scripts
  gen_power_curve.py
  gen_forest_plot.py
  gen_gee_decomposition.py
  cross_benchmark_heatmap.py

paper/                 # LaTeX source
  main.tex             # Main document
  sections/            # Paper sections
  figures/             # Pre-generated figure PDFs
  references.bib       # Bibliography
```

## Requirements

```bash
pip install -r requirements.txt
```

Key dependencies:
- Python >= 3.10
- PyTorch >= 2.0
- transformers >= 4.40
- vLLM >= 0.4 (for local model serving)
- choix (Bradley-Terry model)
- statsmodels (GEE analysis)
- scipy, numpy, pandas

## Reproducing Results

### Step 1: Serve local models

```bash
# Serve the rewriter/judge model
python -m vllm.entrypoints.openai.api_server \
  --model Qwen/Qwen3-32B \
  --tensor-parallel-size 2 \
  --port 8000

# Serve NLI verification model
# (DeBERTa-v3-large-mnli runs on CPU or single GPU)
```

### Step 2: Run the pipeline

```bash
# 1. Prepare data
python -m csd_pipeline.data_prep --dataset llmbar --output data/

# 2. Generate counterfactuals
python -m csd_pipeline.counterfactual_gen \
  --input data/prepared.jsonl \
  --axis formality \
  --base-url http://localhost:8000/v1

# 3. Verify semantic preservation
python -m csd_pipeline.content_verification \
  --input data/counterfactuals.jsonl \
  --nli-threshold 0.90

# 4. Score with LLM judge (Likert)
python -m csd_pipeline.judge_scoring \
  --input data/verified.jsonl \
  --base-url http://localhost:8000/v1

# 5. Pairwise comparison (position-controlled)
python -m csd_pipeline.pairwise_scoring \
  --input data/verified.jsonl \
  --base-url http://localhost:8000/v1

# 6. Analyze results
python -m csd_pipeline.ate_analysis \
  --scored data/scored.jsonl \
  --pairwise data/pairwise.jsonl
```

### Step 3: Run experiments

```bash
# Table 1: Main results across axes and judges
python experiments/exp014_logistic_decomposition.py

# Power analysis (Figure 3)
python figures/gen_power_curve.py

# Cross-benchmark heatmap (Figure 2)
python figures/cross_benchmark_heatmap.py
```

## Axes and Judges

**Style Axes**: Formality (formal/casual), Verbosity (verbose/concise), Register (academic/conversational)

**LLM Judges**: Qwen3-32B, Llama-3.3-70B-Instruct, GPT-4o, Gemini-2.5-Flash, Claude Sonnet 4

**Benchmarks**: LLMBar, AlpacaEval, MT-Bench

## Paper

Compile the paper:

```bash
cd paper
pdflatex main
bibtex main
pdflatex main
pdflatex main
```

## License

This code is released for research purposes under the MIT License.
