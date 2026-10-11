# Setup and reproduction

Run commands from the repository root with Python 3.12. There are two environments to consider: the small offline controller runs on CPU, while collecting language-model outputs requires the research dependencies and substantial GPU resources.

## CPU installation

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -e '.[dev]'
python -m pytest -q
```

The default dependencies cover PyTorch, numerical utilities, feature preprocessing, YAML configuration, and controller training. The `dev` extra adds pytest and package-build tools. Tests use small synthetic examples; they do not download LLaDA or GSM8K.

```bash
python -m scripts.train_neural_continuation_bandit --help
python -m scripts.train_budget_conditioned_neural_bandit --help
python -m scripts.run_continuation_analysis --help
```

On Windows, activate the environment with `.venv\Scripts\Activate.ps1` in PowerShell.

## GPU research environment

Use a Linux CUDA environment for the full generation and earlier GRPO training paths. Install a PyTorch build compatible with your CUDA environment using the [official installation instructions](https://pytorch.org/get-started/locally/), then install the research extra:

```bash
python -m pip install -e '.[research,dev]'
python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
```

The research extra retains the supplied versions of Transformers, Accelerate, PEFT, TRL, DeepSpeed, datasets, tiktoken, and bitsandbytes. DeepSpeed and bitsandbytes are Linux-only dependencies here. The full GPU stack has not been validated during repository preparation; compatibility depends on the hardware, CUDA version, and resolved PyTorch build.

The main config loads `GSAI-ML/LLaDA-8B-Instruct` in bfloat16 without 4-bit quantization. Model weights and intermediate tensors must fit in available accelerator memory. This repository does not bundle model weights or guarantee a particular memory requirement.

Optional local configuration:

```bash
cp .env.example .env
set -a
source .env
set +a
```

Populate `HF_TOKEN` only if access requires it. `WANDB_MODE=disabled` disables external experiment tracking. `RLDLLM_DATASETS_DIR` points to optional datasets saved with Hugging Face `save_to_disk`; the GSM8K training/collection path falls back to `openai/gsm8k` if no local copy exists. Hugging Face's standard cache variables can also be used.

## 1. Collect paired outcomes

The supplied final configuration requests 3,000 training examples and 512 evaluation examples, uses 192 generated tokens with block length 32, and evaluates a planned K=4 answer-span continuation. It uses a frozen smart-initialized unmasking policy; it does not require a supplied trained policy checkpoint.

Start with a small collection to check your GPU environment:

```bash
python -m scripts.run_continuation_analysis \
  --config configs/experiment_configs/llada_8b_instruct_verifier_budget_gsm8k_continuation_k4_fresh.yaml \
  --gpu 0 \
  --split both \
  --max_train_samples 16 \
  --max_eval_samples 8 \
  --output_dir outputs/continuation_smoke
```

This smoke dataset checks collection only. It is too small for a credible performance comparison.

For the configured collection:

```bash
python -m scripts.run_continuation_analysis \
  --config configs/experiment_configs/llada_8b_instruct_verifier_budget_gsm8k_continuation_k4_fresh.yaml \
  --gpu 0 \
  --split both \
  --output_dir outputs/continuation_k4
```

Expected files include `per_example.jsonl`, `generations.jsonl`, `verifier_results.jsonl`, `continuation_records.jsonl`, and `analysis_config.json`. Both the base and continued outcomes are collected even though a deployed controller would select one action. That makes this a full-information offline experiment.

Use a new output directory for a new experiment. `--resume` and the start/end index options support interrupted collection; check the help output before using them. `--skip_failed_batches` records failures and continues, so it can reduce or change the evaluated sample set. The commands above leave that option off.

## 2. Export controller data

```bash
python -m scripts.export_continuation_gate_dataset \
  --output_dir outputs/continuation_k4
```

This writes `outputs/continuation_k4/continuation_gate_dataset.jsonl`. Each row contains permitted input features, labels for reward/evaluation, and the original train/evaluation mode. Ground-truth correctness labels are not controller input features. The paired dataset is not included in the repository; generate it first or obtain a compatible export separately.

## 3. Train a fixed-cost neural controller

This step can run in the CPU environment:

```bash
python -m scripts.train_neural_continuation_bandit \
  --dataset outputs/continuation_k4/continuation_gate_dataset.jsonl \
  --output_dir outputs/neural_lambda_0 \
  --cost_lambda 0.0 \
  --harm_penalty 0.0 \
  --epochs 300 \
  --patience 40 \
  --bootstrap_samples 100 \
  --device cpu
```

Repeat with separate output directories and `--cost_lambda 0.005`, `0.01`, or `0.02` for a cost sweep. The trainer fits preprocessing on its training portion, uses validation for checkpoint/threshold selection, and evaluates against the logged evaluation rows.

Outputs include `neural_bandit_policy.pt`, `feature_preprocessor.joblib`, feature metadata, `neural_bandit_result.json`, and `neural_bandit_result_pretty.md`.

## 4. Train one budget-conditioned controller

```bash
python -m scripts.train_budget_conditioned_neural_bandit \
  --dataset outputs/continuation_k4/continuation_gate_dataset.jsonl \
  --output_dir outputs/budget_conditioned \
  --train_cost_lambdas '0.0,0.0025,0.005,0.01,0.02' \
  --eval_cost_lambdas '0.0,0.0025,0.005,0.01,0.02' \
  --threshold_mode per_lambda_validation \
  --epochs 300 \
  --patience 40 \
  --bootstrap_samples 100 \
  --device cpu
```

This adds `requested_cost_lambda` to the inputs and recomputes each row's action utilities for the requested cost. Read `budget_conditioned_result.json` and `budget_conditioned_result_pretty.md` in the output directory.

To study cost conditioning with the same threshold at each cost, repeat the command with a new output directory and `--threshold_mode fixed_0_5`.

## 5. Compare and interpret

Compare learned controllers with never-continue, always-continue, combined heuristic, and utility-oracle baselines. The oracle uses both known outcomes and is an analysis upper bound. Report accuracy, utility, trigger rate, average actual extra steps, fixes, and harms. A configured K=4 is a maximum/planned continuation budget; the actual extra steps may be lower.

Saved-step percentages refer to **additional continuation forward evaluations**, relative to always-continue. They do not measure savings in the base generation, total FLOPs, wall-clock time, or latency of verifier/controller overhead.

Keep the paired export, effective configuration, seed, environment versions, reports, and checkpoints together if publishing a new empirical result. The numerical tables in [Historical results](results.md) cannot be independently reproduced from the supplied archive alone.

## Retained research paths and limitations

- `train/train.py` and `train/trainer.py` retain earlier GRPO token-unmasking experiments. Most experiment configs describe that earlier direction rather than the main offline controller workflow.
- Checkpoint warm-start configs reference local checkpoints that are not supplied. Config filenames are historical labels; inspect their sample limits rather than inferring them from a filename.
- The `eval.pipeline` and `common.s3` paths contain deliberately incomplete S3 setup functions.
- The retained `eval.eval` distributed sampler assumes an initialized process group when it infers world size and rank; ordinary single-process evaluation can fail. Its GSM8K few-shot loader also does not populate demonstrations correctly. The main continuation-analysis workflow bypasses those paths.
- The retained MATH-500, HumanEval, and MBPP evaluation loaders require local saved datasets. GSM8K is the documented collection task.
- Offline controller checkpoints are not connected to the online generation loop. The reported policy metrics select between collected base/continued outcomes.

These boundaries are also recorded in the [architecture map](CODEMAPS/INDEX.md) and [source audit](source-audit.md).
