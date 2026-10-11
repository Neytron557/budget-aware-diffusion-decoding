# Method

This project studies whether a small controller can decide when to spend additional inference compute on a frozen diffusion language model. Its current continuation-policy workflow trains and evaluates that decision **offline**, using paired base and continued generations. The learned checkpoints are not connected to an online generation gate in this repository.

The code builds on [the upstream RL-dLLM repository](https://github.com/apple-aiml-research/ml-rl-dllm), associated with [Learning Unmasking Policies for Diffusion Language Models](https://arxiv.org/abs/2512.09106). The supplied proposal explored a broader sequential RL controller. That proposal is preserved in [the research archive](research/README.md); the continuation workflow is the primary documented path here.

## Offline decision problem

For each GSM8K example, the analysis script records a base answer and an answer-region continuation. The exported row combines base-generation features, a configured continuation action, both correctness outcomes, and the continuation cost.

```text
Frozen LLaDA + configured unmasking policy
                    |
             Base generation
                    |
          Paired continuation record
                    |
       Export features + outcome labels
                    |
          Train continuation controller
                    |
      Offline evaluation on paired outcomes
```

The controller chooses between:

- **Stop:** retain the base answer.
- **Continue:** use the logged answer-region continuation.

The primary analysis configuration requests `K=4` continuation steps. Actual additional forward passes can be fewer, so training and evaluation distinguish planned steps from recorded cost.

The base model weights remain frozen. The analysis script freezes the configured unmasking policy when `freeze_policy: true`; the small continuation controller is trained separately.

## Features and leakage boundaries

[`export_continuation_gate_dataset.py`](../scripts/export_continuation_gate_dataset.py) defines an explicit feature allowlist. [`continuation_gate_features.py`](../scripts/continuation_gate_features.py) flattens nested confidence and entropy statistics and handles missing values.

| Feature group | Included fields |
|---|---|
| Budget | `target_budget`, `base_nfe`, `remaining_budget` |
| Verifier | `base_parse_ok`, `base_format_ok`, `base_arithmetic_ok`, `base_verifier_score` |
| Answer and output signals | `base_reward_answer_source`, `base_repetition_penalty`, `base_visible_generated_char_count`, `base_answer_span_confidence` |
| Uncertainty | `base_entropy_stats`, `base_confidence_stats` |
| Planned continuation | `continuation_k`, `continuation_remasked_token_count`, `continuation_over_budget` |

The `base_reward_answer_source` field describes the extraction format/source rather than the answer's correctness. Planned continuation fields describe the action being considered; a future online gate must reproduce them before deciding to continue.

Correctness, ground truth, rewards, generated text, and continuation gains are excluded from model inputs. The recorded extra-step count is used to construct rewards and compute metrics, not as an input feature. The exporter and trainers check feature keys for known leakage fields. These checks protect the documented schema; they do not prove that an arbitrary external dataset is free of leakage.

The math verifier uses generated text to extract a final answer and check simple arithmetic and formatting. A successful verifier check does not establish that the answer solves the original problem.

## Reward

For the default correctness reward of 1, the two logged action rewards are:

```text
r_stop     = base_correct
r_continue = continued_correct
             - cost_lambda * actual_extra_steps
             - harm_penalty * I[base_correct and not continued_correct]
```

`correctness_reward` can scale the correctness terms. The utility oracle chooses continuation when `r_continue > r_stop`, with ties assigned to stop. It uses both observed outcomes and is an evaluation reference, not an inference-time policy.

The step cost uses `continued_extra_steps` when present, then `continuation_extra_steps`, then the planned `continuation_k` as a fallback. Preserve actual cost fields in new experiment exports to avoid relying on that fallback.

## Neural controller and objective

[`train_neural_continuation_bandit.py`](../scripts/train_neural_continuation_bandit.py) uses a small MLP with two hidden layers of width 64 by default. Each hidden layer uses ReLU and dropout of 0.1; the output is a single continuation logit. The feature preprocessing pipeline combines a dictionary vectorizer, constant imputation, and standardization, fitted on training rows.

Let `p = sigmoid(logit)` and `delta = r_continue - r_stop`. The training loss is:

```text
expected_utility = p * r_continue + (1 - p) * r_stop
loss = -mean(expected_utility)
       - entropy_coef * mean(Bernoulli entropy(p))
       + aux_bce_weight * weighted_BCE(logit, I[delta > 0])
```

The auxiliary BCE weights each row by `max(abs(delta), 0.01)`. Since both action outcomes are recorded, this is a full-information contextual-bandit objective rather than online exploration or base-model fine-tuning.

The scripts use explicit `mode: train` and `mode: eval` rows when both exist. Otherwise, they fall back to a row split. Within training data, validation splitting groups rows by `example_id` when IDs are available. For credible experiments, supply explicit modes and stable IDs, and check that prompts do not overlap across splits.

A threshold selected on validation data converts continuation probabilities into decisions. The trainer saves the network, feature preprocessor, feature names, evaluation predictions, JSON results, and a Markdown report.

## Cost-conditioned extension

[`train_budget_conditioned_neural_bandit.py`](../scripts/train_budget_conditioned_neural_bandit.py) expands training rows over a cost grid and adds `requested_cost_lambda` to the input. It recomputes the rewards for each requested cost.

The default grid is `0, 0.0025, 0.005, 0.01, 0.02`. Two evaluation modes are available:

- `per_lambda_validation`: select a separate decision threshold for each cost on validation data.
- `fixed_0_5`: use threshold 0.5 at every cost, allowing examination of the conditioned model's scores under a common threshold.

The implementation does not impose a monotonicity constraint on continuation decisions as cost increases. Historical figures showing a trend at selected costs do not guarantee that trend on other datasets or runs.

## Supporting sequential decoding code

The repository also retains the upstream token-level unmasking policy and GRPO training path, budget features, configurable decoding rewards, math verifier, answer-span finder, and remasking strategies. These are separate from the offline continuation-bandit objective above.

Historical configurations include experiments with shaped rewards, checkpoint warm starts, different extraction rules, and optional oracle analysis fields. Read their values directly: file names alone do not establish the effective configuration. Some checkpoints referenced by these configurations are absent from the supplied archive.

## Compute and evidence limits

An NFE is a frozen-model forward evaluation. The continuation-policy reports compare **additional continuation NFEs**, excluding the base-generation cost. A percentage reduction in these extra steps is not the same as a reduction in total inference time, total NFEs, or FLOPs.

The source archive includes implementation and tests but no paired experiment dataset, raw evaluation logs, or trained checkpoints. [Historical results](results.md) are preserved with that evidence limitation. See [the source audit](source-audit.md) and [the architecture map](CODEMAPS/INDEX.md) for provenance and entry points.
