# Architecture map

Reviewed: 2026-10-12. This map describes the supplied implementation and the publication documentation.

## Main workflow

```text
YAML config + GSM8K loader
          |
scripts/run_continuation_analysis.py
          |
Frozen model + token unmasking policy + verifier
          |
Base/continuation records in outputs/
          |
scripts/export_continuation_gate_dataset.py
          |
continuation_gate_dataset.jsonl
          |
Feature preprocessing + offline policy training
          |
Checkpoints + predictions + evaluation reports
```

The offline trainers choose between already logged outcomes. They do not install their checkpoints as gates in the generation loop.

## Entry points

| File | Purpose |
|---|---|
| [`scripts/run_continuation_analysis.py`](../../scripts/run_continuation_analysis.py) | Load a frozen model and configured unmasking policy, run GSM8K base/continuation analysis, and write paired records. |
| [`scripts/export_continuation_gate_dataset.py`](../../scripts/export_continuation_gate_dataset.py) | Export the feature allowlist, labels, and continuation costs to an offline dataset. |
| [`scripts/train_neural_continuation_bandit.py`](../../scripts/train_neural_continuation_bandit.py) | Train and evaluate a fixed-cost MLP continuation controller. |
| [`scripts/train_budget_conditioned_neural_bandit.py`](../../scripts/train_budget_conditioned_neural_bandit.py) | Train one MLP with a requested-cost feature and evaluate it at several costs. |
| [`scripts/train_contextual_bandit_continuation_policy.py`](../../scripts/train_contextual_bandit_continuation_policy.py) | Train classical continuation-policy models and compare reward-aware decisions. |
| [`scripts/train_continuation_gate.py`](../../scripts/train_continuation_gate.py) | Train supervised continuation gates from exported labels. |
| [`scripts/evaluate_continuation_heuristic_gate.py`](../../scripts/evaluate_continuation_heuristic_gate.py) | Compare heuristic continuation decisions on logged outcomes. |
| [`scripts/check_training_consistency.py`](../../scripts/check_training_consistency.py) | Check consistency between saved training/evaluation artifacts. |
| [`train/train.py`](../../train/train.py) | Retained token-level unmasking-policy training entry point. |
| [`eval/eval.py`](../../eval/eval.py) | Retained generation evaluation entry point. |

## Modules

| Area | Responsibility |
|---|---|
| [`common/models/`](../../common/models/) | LLaDA model implementation and lightweight token-level policies. |
| [`common/generation/`](../../common/generation/) | Sampling, unmasking, budget accounting, verifier/remasking integration, and post-hoc continuation. |
| [`common/verifiers/`](../../common/verifiers/) | Verifier result interface and label-free math checks. |
| [`common/remasking/`](../../common/remasking/) | Final-answer span lookup and configurable remasking strategies. |
| [`common/policy_features.py`](../../common/policy_features.py) | Budget and verifier features for the sequential token policy. |
| [`common/rewards.py`](../../common/rewards.py) | Configurable additive and multiplicative decoding rewards and answer-quality handling. |
| [`common/budgeting.py`](../../common/budgeting.py) | Budget selection helpers. |
| [`common/config.py`](../../common/config.py) | Model, generation, training, verifier, reward, and continuation configuration. |
| [`common/parsing/`](../../common/parsing/) | Answer extraction and task scoring. |
| [`data/loaders/`](../../data/loaders/) | GSM8K, MATH-500, HumanEval, and MBPP dataset adapters. |
| [`scripts/continuation_gate_features.py`](../../scripts/continuation_gate_features.py) | Offline gate feature normalization and flattening. |
| [`train/trainer.py`](../../train/trainer.py) | Retained GRPO trainer for the lightweight token policy. |
| [`tests/`](../../tests/) | CPU tests for verifiers, spans, budgets, rewards, features, and continuation-policy workflows. |
| [`configs/experiment_configs/`](../../configs/experiment_configs/) | Primary continuation configuration and historical experiment settings. |

## Dependencies and external data

The offline neural workflow uses PyTorch, NumPy, and scikit-learn. Model generation additionally uses Transformers, Accelerate, and dataset/model downloads. The retained sequential training path includes TRL, PEFT, and DeepSpeed dependencies. Environment details and installation commands are in [the README](../../README.md).

Model weights, experiment datasets, logs, and trained checkpoints are not bundled. Some historical configurations refer to unavailable policy checkpoints. [`eval/pipeline.py`](../../eval/pipeline.py) and [`common/s3.py`](../../common/s3.py) retain upstream storage interfaces whose internal bucket setup was stripped; they are not ready-to-use hosted evaluation services.

## Documentation

- [Method](../method.md): implemented decision problem, reward, objective, and limitations.
- [Historical results](../results.md): transcribed tables and their evidence status.
- [Research archive](../research/README.md): original proposal and design notes.
- [Source audit](../source-audit.md): input inventory and publication changes.
