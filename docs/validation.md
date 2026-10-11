# Repository validation

Validated on 2026-10-12 in an isolated macOS ARM64 CPU environment. These checks cover repository preparation and the implemented CPU components; they do not reproduce the historical GPU experiments.

| Check | Result |
| --- | --- |
| Editable installation with the `dev` extra | Passed |
| Full test suite: `python -m pytest -q` | 159 passed |
| Python source syntax | All source files compile without syntax warnings |
| Fixed-cost controller, conditioned controller, and collection `--help` commands | Passed without model/dataset downloads |
| Source distribution and wheel build | Passed |
| Local Markdown links | All destinations exist |
| Publication scan for credential-shaped tokens/private keys | No matches found |
| Original proposal, archived research notes, LICENSE, and ACKNOWLEDGEMENTS | Preserved byte-for-byte |

## Test environment

| Component | Version |
| --- | --- |
| Python | 3.12.15 |
| PyTorch | 2.14.1 |
| NumPy | 2.5.4 |
| scikit-learn | 1.9.1 |
| SciPy | 1.18.1 |
| PyYAML | 6.0.3 |
| pytest | 9.1.1 |
| setuptools | 84.0.0 |

Default dependencies remain broadly specified in `pyproject.toml`; this table records one tested environment rather than a complete cross-platform dependency lock.

## Regression fixed during preparation

The supplied suite initially reported 149 passing tests and one failing arithmetic-verifier test. Whitespace-separated equations were being combined into an invalid expression. The verifier now separates adjacent operands with no intervening operator while preserving its text, context, equation-count, and AST safety limits. Nine additional regression cases cover valid and invalid later equations, line breaks, parentheses, compound expressions, and a negative result.

Two parser string escapes were also corrected to remove Python syntax warnings without changing their runtime values. Existing copyright notices remain intact, and modified inherited source files contain modification notes.

## Validation limits

The full GPU dependency stack, LLaDA generation, dataset collection, GRPO training, and historical performance figures were not run or verified during this preparation. The CPU tests use synthetic fixtures and temporary outputs. Known limitations of the retained evaluation paths are documented in [the reproduction guide](reproduction.md).

The source distribution includes documentation, configurations, and tests. The wheel contains the Python packages and license files; the reproduction workflow assumes a repository checkout for configurations and documentation.
