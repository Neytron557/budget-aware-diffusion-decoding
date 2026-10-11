# Contributing

Please open an issue explaining the problem or experiment before proposing a substantial change. Small fixes can be submitted directly as pull requests.

1. Set up the environment described in [the README](README.md).
2. Keep changes focused and update the relevant documentation.
3. Run the CPU test suite with `python -m pytest tests -q`. For changes to generation, also describe any GPU validation performed, including the model, configuration, seed, and hardware.
4. Include a short explanation of the change and its validation in the pull request.

When adding experiments, keep training, validation, and evaluation examples separate. Ground-truth answers belong in rewards and evaluation, never in inference-time policy features. Distinguish continuation forward passes from total inference cost, and include raw metrics with any performance claim.

Retain existing copyright and attribution notices in source files. The supplied upstream [LICENSE](LICENSE) and [ACKNOWLEDGEMENTS](ACKNOWLEDGEMENTS) remain part of this distribution; do not replace them with an unrelated license. Only submit material you have the right to share.

The project contributors are listed in [CONTRIBUTORS.md](CONTRIBUTORS.md).
