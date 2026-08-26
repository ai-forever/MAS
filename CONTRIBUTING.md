# Contributing

Contributions that improve reproducibility, framework compatibility, documentation, or evaluation coverage are welcome.

## Before opening a pull request

1. Open an issue describing the problem or proposed experiment.
2. Keep changes focused and avoid unrelated formatting updates.
3. Do not commit manuscript images, access tokens, model weights, absolute local paths, or private dataset copies.
4. State the framework and version used to validate configuration changes.
5. For metric changes, include a small deterministic test case and explain whether existing reported numbers are affected.

## Reporting results

When contributing benchmark results, include:

- the exact model identifier or checkpoint;
- training datasets and split names;
- random seed and decoding parameters;
- CER/WER implementation and aggregation method;
- hardware and relevant package versions;
- the complete command or configuration file.

Please use GitHub Issues for questions and bug reports.
