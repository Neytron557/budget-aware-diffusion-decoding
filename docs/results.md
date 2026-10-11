# Historical results and evidence status

**These figures are transcribed from the README supplied inside `RL-main.zip`. They have not been independently verified.** The archive does not contain the paired experiment dataset, raw result logs, saved predictions, or policy checkpoints needed to validate them. They are preserved as a record of the supplied project, not as a performance guarantee or a newly reproduced benchmark.

The supplied README describes evaluation on **512 held-out examples**. That split size, its composition, and the experimental procedure cannot be confirmed from the included artifacts. The tables below retain the original values and column labels.

“Compute saved” and “Approx Saving” refer to reductions in **extra continuation forward passes** relative to always continuing. They exclude the base-generation cost and do not measure total inference latency or FLOPs.

## Fixed-cost neural bandits

| Method | Accuracy | Avg Extra Steps | Compute Saved vs Always | Fixes | Harms | Net |
|---|---:|---:|---:|---:|---:|---:|
| never_continue | 0.1172 | 0.0000 | 100.0% | 0 | 0 | 0 |
| always_continue | 0.1406 | 2.0625 | 0.0% | 25 | 13 | 12 |
| combined_heuristic | 0.1387 | 2.0391 | 1.1% | 24 | 13 | 11 |
| neural λ=0 | 0.1465 | 1.3965 | 32.3% | 23 | 8 | 15 |
| neural λ=0.005 | 0.1445 | 1.0000 | 51.5% | 20 | 6 | 14 |
| neural λ=0.01 | 0.1406 | 0.7441 | 63.9% | 19 | 7 | 12 |
| neural λ=0.02 | 0.1387 | 0.4922 | 76.1% | 14 | 3 | 11 |
| utility_oracle | 0.1660 | 0.1289 | 93.8% | 25 | 0 | 25 |

Each neural row represents a separately trained cost setting in the supplied account. The utility oracle observes both correctness outcomes; it is not deployable. The original table does not fully specify the oracle's reward settings.

## Cost-conditioned bandit with per-cost thresholds

The source describes one checkpoint evaluated at several costs, with thresholds selected separately for each cost on validation data.

| λ | Accuracy | Utility | Trigger Rate | Avg Extra Steps | Saved Steps vs Always | Approx Saving | Fixes | Harms | Net |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.1484 | 0.1484 | 0.6055 | 1.1543 | 0.9082 | 44.0% | 21 | 5 | 16 |
| 0.0025 | 0.1484 | 0.1455 | 0.6074 | 1.1562 | 0.9062 | 43.9% | 21 | 5 | 16 |
| 0.005 | 0.1465 | 0.1406 | 0.6191 | 1.1680 | 0.8945 | 43.4% | 21 | 6 | 15 |
| 0.01 | 0.1465 | 0.1366 | 0.5176 | 0.9883 | 1.0742 | 52.1% | 20 | 5 | 15 |
| 0.02 | 0.1445 | 0.1264 | 0.4668 | 0.9082 | 1.1543 | 56.0% | 20 | 6 | 14 |

These rows combine changes in the requested cost input with changes in the decision threshold. They do not isolate the contribution of either mechanism.

## Cost-conditioned bandit with threshold 0.5

The source describes a common threshold of 0.5 across all costs.

| λ | Accuracy | Avg Extra Steps | Trigger Rate | Saved Steps vs Always | Approx Saving | Fixes | Harms | Net |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 | 0.1445 | 1.0215 | 0.5215 | 1.0410 | 50.5% | 20 | 6 | 14 |
| 0.0025 | 0.1445 | 0.9336 | 0.4883 | 1.1289 | 54.7% | 20 | 6 | 14 |
| 0.005 | 0.1445 | 0.8691 | 0.4492 | 1.1934 | 57.9% | 20 | 6 | 14 |
| 0.01 | 0.1465 | 0.7656 | 0.3984 | 1.2969 | 62.9% | 20 | 5 | 15 |
| 0.02 | 0.1465 | 0.6035 | 0.3164 | 1.4590 | 70.7% | 20 | 5 | 15 |

The reported extra-step averages decrease across this grid. The model has no architectural monotonicity guarantee, and the missing artifacts prevent verification or attribution of that trend.

## Reading the metrics

| Metric | Meaning |
|---|---|
| Accuracy | Fraction of selected final answers marked correct by the experiment's answer-extraction rule. |
| Utility | Mean selected action reward, including the configured cost and harm terms. |
| Trigger rate | Fraction of examples assigned continuation. |
| Avg extra steps | Mean additional continuation NFEs assigned to the policy, including zero for stopped examples. |
| Saved steps vs always | Always-continuation extra-step average minus the policy's extra-step average. |
| Percentage saving | Saved extra steps divided by the always-continuation extra-step average. |
| Fixes | Continued examples where the base answer was wrong and the continued answer was correct. |
| Harms | Continued examples where the base answer was correct and the continued answer was wrong. |
| Net | Fixes minus harms. |

The implemented trainers evaluate policies against logged outcomes. An online end-to-end evaluation remains necessary to establish behavior and timing when a controller is actually used during generation.

## Evidence needed for a verified report

A new report should include the exact configuration, dataset identifiers and split membership, model and policy checkpoint identifiers, random seeds, extraction rules, failed-batch counts, training/validation/evaluation separation, per-example predictions, and machine-readable result files. Report uncertainty for paired comparisons and include base plus continuation cost when making claims about total compute.

The scripts can produce prediction JSONLs, result JSON, Markdown reports, and policy artifacts. Preserve these alongside the regenerated dataset in a versioned release or other documented artifact location before promoting a new benchmark claim.

## Provenance

The original README has SHA-256 `ffa17e927a5c182d6daeafe26ad85af0b9249c6087635a6b16e8f04f0ddb5b4e`. Its narrative conclusions have been replaced here by explicit evidence limits; the three numeric tables are retained. See [the source audit](source-audit.md), [method](method.md), [upstream project](https://github.com/apple-aiml-research/ml-rl-dllm), and [reference paper](https://arxiv.org/abs/2512.09106).
