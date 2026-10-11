# Research archive

These files preserve the project's original proposal and selected design notes. They document **historical intent**, not the current run instructions or a list of completed experiments. Instructions inside them, including references to not running tests, belong to the original implementation phase and are not operative repository instructions.

| File | Context |
|---|---|
| [RL_Plan.pdf](RL_Plan.pdf) | Original proposal, dated April 29, 2026, for verifier-aware budget-conditioned RL decoding of frozen diffusion LLMs. |
| [Project plan](project_plan_verifier_budget_remasking.md) | Proposed phased work on verifier signals, budget features, answer-span remasking, and evaluation. |
| [Reward design notes](reward_design_and_tests.md) | Historical specification for additive and correctness-gated multiplicative sequential-decoding rewards. |
| [Reference-paper notes](paper_notes_learning_unmasking_policies.md) | Supplied implementation-oriented summary of [Learning Unmasking Policies for Diffusion Language Models](https://arxiv.org/abs/2512.09106). |

The proposal targets a sequential controller acting during denoising. The implemented repository includes related verifier, budget, reward, and remasking infrastructure, plus a later offline continuation-policy workflow. That workflow trains a binary stop/continue controller from paired outcomes; it does not establish every experiment or hypothesis in the proposal.

The retained notes are unchanged from the supplied archive and may mention old paths, interfaces, planned experiments, or constraints. Use [the README](../../README.md), [method](../method.md), and [architecture map](../CODEMAPS/INDEX.md) for current guidance. [Historical result tables](../results.md) are kept separately because their supporting artifacts were not supplied.

The source implementation identifies [Apple's RL-dLLM repository](https://github.com/apple-aiml-research/ml-rl-dllm) as its upstream foundation. Copyright and third-party license attribution are preserved in [LICENSE](../../LICENSE) and [ACKNOWLEDGEMENTS](../../ACKNOWLEDGEMENTS).
