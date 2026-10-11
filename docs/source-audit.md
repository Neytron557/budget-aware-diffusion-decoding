# Source audit

Reviewed: 2026-10-12. This repository was prepared from the two project files supplied in the workspace. The original files were left intact; the publication folder is a separate copy.

## Input files

| Input | Size | Contents | SHA-256 |
|---|---:|---|---|
| `RL-main.zip` | 289,168 bytes | 104 files under `RL-main/`; 122 ZIP entries including directories. | `036d1552cb1ab4e69edcd259a567608f04a664f751ad226d2e6c82b5774ab3df` |
| `RL_Plan.pdf` | 185,618 bytes | 11-page project proposal for verifier-aware budget-conditioned frozen diffusion-LLM decoding. | `94a71f321179fe0ec0cbf844a5bea0d614ca2bd53295d5b5075004c724ffb215` |

The workspace's `.DS_Store` is macOS folder metadata and is excluded. Neither original input file is duplicated wholesale at the repository root. The proposal is preserved as `docs/research/RL_Plan.pdf`; the ZIP's relevant contents are unpacked into this repository.

## Archive inventory and disposition

All 104 archive files are accounted for below.

| Original area | Files | Publication treatment |
|---|---:|---|
| `common/` | 25 | Retained model, sampling, parsing, verifier, remasking, budget, reward, config, and storage modules, with the limited preparation fixes listed below. |
| `configs/` | 28 | Retained experiment and Accelerate configurations; historical dependency limits are documented. |
| `data/` | 8 | Retained dataset adapters and data utilities. |
| `eval/` | 4 | Retained evaluation and aggregation modules. |
| `scripts/` | 10 | Retained continuation analysis, export, feature, training, and consistency tools. |
| `tests/` | 12 | Retained CPU test files and added adjacent-equation regression cases to the math-verifier tests. |
| `train/` | 4 | Retained sequential policy training modules. |
| `docs/` | 7 | Three research notes retained under `docs/research/`; four obsolete prompt/template files omitted. |
| Root files | 6 | README, contribution guide, ignore rules, and package metadata updated; LICENSE and ACKNOWLEDGEMENTS retained unchanged. |

The count describes the supplied archive, not the final repository's file count; preparation adds documentation and validation configuration.

## Publication changes

| Original file | Treatment and reason |
|---|---|
| `README.md` | Superseded by a concise project README and focused method/reproduction documentation. All three original numeric result tables are retained in [results.md](results.md) with explicit evidence limits. |
| `CONTRIBUTING.md` | Replaced with instructions for this project; removed inherited statements that the project is an official paper companion and that contributors submit their work to Apple. |
| `.gitignore` | Replaced with publication ignore rules for generated outputs, model artifacts, local environments, caches, and secrets. |
| `pyproject.toml` | Updated package identity and installation metadata for this derived project. |
| `docs/project_plan_verifier_budget_remasking.md` | Preserved unchanged as a historical research note. |
| `docs/reward_design_and_tests.md` | Preserved unchanged as a historical reward specification. |
| `docs/paper_notes_learning_unmasking_policies.md` | Preserved unchanged as supplied reference-paper notes. |
| `docs/README_context_bundle.md` | Omitted because it describes installing an AI context bundle rather than using the project. |
| `docs/codex_implementation_prompt.md` | Omitted because it is an old implementation prompt with phase-specific instructions and hypothetical APIs. |
| `docs/server_run_commands_template.md` | Omitted because its proposed commands and analysis modules do not match the actual implementation. |
| `docs/papers/README.md` | Omitted because it is a placeholder for optional PDFs, not a supplied paper artifact. |

The proposal PDF is retained without edits. Its hypotheses and planned methods are distinguished from the implemented continuation workflow in [the research archive](research/README.md).

The publication copy also adds contributor documentation, a sample local environment file, a package manifest, and a GitHub Actions CPU validation workflow. Package metadata now separates the small offline installation from the optional GPU research dependencies and includes the `scripts` package.

## Limited source corrections

Repository preparation found an arithmetic-verifier bug when multiple equations appear next to each other. Expression extraction could combine the end of one equation with the start of the next and fail to check them separately. The publication copy contains these focused changes:

| File | Change |
|---|---|
| [`common/verifiers/math_verifier.py`](../common/verifiers/math_verifier.py) | Split adjacent operands separated by whitespace when identifying equation boundaries, allowing consecutive arithmetic equations to be checked separately. |
| [`tests/test_math_verifier.py`](../tests/test_math_verifier.py) | Added regression cases for adjacent equations, including valid and invalid outcomes, newlines, parentheses, expression-valued right sides, and negative results. |
| [`common/parsing/parser_utils.py`](../common/parsing/parser_utils.py) | Made the existing literal backslash-percent escape explicit to remove a Python invalid-escape warning. |
| [`common/parsing/parsers.py`](../common/parsing/parsers.py) | Applied the same warning cleanup to the equivalent parser expression. |

The parser escape cleanup preserves the intended literal value. Modified source files carry a short change notice and retain their existing copyright and upstream attribution. These corrections are separate from any claim about empirical model performance.

## Implementation findings

The supplied code contains a frozen LLaDA backend, token-level unmasking policies and GRPO training, label-free arithmetic/format verification, answer-span lookup and remasking strategies, budget and reward configuration, paired continuation analysis, offline classical and neural continuation policies, and tests.

The primary documented workflow is GSM8K continuation analysis followed by offline stop/continue training. The learned continuation checkpoints are evaluated against logged paired outcomes; the supplied code does not connect these checkpoints to an online generation gate.

Several configurations are historical experiments rather than standalone runnable examples. They include unavailable checkpoint paths and file names that do not fully describe the values inside the YAML. The `fresh` continuation configuration sets `policy_checkpoint_path: null`, so it avoids a missing policy-checkpoint dependency; it still requires downloading model weights and dataset data.

The inherited storage setup in `common/s3.py` and `eval/pipeline.py` is deliberately stripped. The legacy server command template advertised a different CLI and absent analysis modules, which is why it was omitted.

The original README's Python 3.10 setup conflicted with the archive's Python 3.12 package requirement. The prepared project documentation aligns environment guidance with its package metadata.

## Result evidence

No experiment output directory, exported continuation dataset, raw result logs, evaluation prediction JSONLs, or trained policy checkpoints were included in the ZIP. The original numerical claims therefore remain unverified historical figures. Test validation of code does not verify those experiment results.

Reported savings compare extra continuation NFEs with always-continuation. They do not establish savings in total generation cost, wall-clock time, or FLOPs. See [results.md](results.md) for the transcribed tables and required evidence for future reports.

## Licensing and provenance

The supplied `LICENSE` is the upstream Apple license, and `ACKNOWLEDGEMENTS` contains the supplied d1, NVIDIA Fast-dLLM, and LLaDA license notices. Both files are retained byte-for-byte, along with existing source copyright notices. This preparation does not replace those terms with another license.

| Preserved file | SHA-256 |
|---|---|
| [LICENSE](../LICENSE) | `d365a1bc1f719572673c473cf91ac63aa18750f149694566ed87a98adc3e1983` |
| [ACKNOWLEDGEMENTS](../ACKNOWLEDGEMENTS) | `e9ae459a94a0403f63bc81bac3dd2455f1c34d61ec827116e559c27f4efc95f3` |

The supplied package metadata and paper notes identify the implementation as derived from [Apple's RL-dLLM project](https://github.com/apple-aiml-research/ml-rl-dllm), accompanying [Learning Unmasking Policies for Diffusion Language Models](https://arxiv.org/abs/2512.09106). The ZIP does not contain Git history or an upstream commit identifier, so an exact upstream revision cannot be recovered from these inputs alone.

Project contributors are listed separately in [CONTRIBUTORS.md](../CONTRIBUTORS.md). Upstream copyright and scholarly references are attribution, not additions to that project contributor list.
