# MAS dataset

MAS is a line-level OCR benchmark built from nine authentic Arabic manuscript works. It contains 11,841 expert-transcribed line images covering Naskh, Taliq, and Nastaliq.

The canonical release is hosted at [maximazzik/MAS](https://huggingface.co/datasets/maximazzik/MAS) under the `mas` configuration.

## Splits

| Split | Lines | Share |
|---|---:|---:|
| `train` | 9,472 | 80% |
| `test` | 2,369 | 20% |

The local validation subset used during the experiments is published as `test`, matching its role in the paper. The split is page-disjoint: no page occurs in both subsets. All nine works occur in both subsets, so this is not a manuscript-held-out evaluation.

## Coverage

The manuscripts span copies from the 12th to the 20th century and compositions from the 10th to the early 20th century. They cover six released domain labels:

- advice to rulers (`siyasat`);
- astronomy (`astronomy`);
- history (`history`);
- prayer collections (`prayers`);
- mathematics (`mathematics`);
- Sufi literature (`sufi_lit`).

## Record schema

Each row contains:

| Field | Meaning |
|---|---|
| `image` | Embedded line image |
| `text` | Diplomatic UTF-8 transcription |
| `prompt` | OCR instruction used to construct the sample |
| `messages` | ShareGPT-style user/assistant messages |
| `script` | Naskh, Taliq, or Nastaliq |
| `domain`, `domain_label` | Machine-readable and display domain names |
| `work`, `author`, `author_dates` | Bibliographic metadata |
| `composition_date`, `copy_date` | Work and manuscript chronology |
| `manuscript_id` | Stable identifier for one of the nine works |
| `source_pack`, `line_id` | Source grouping and line-level traceability |
| `split` | `train` or `test` |

## Annotation

The transcriptions are diplomatic: original spelling, punctuation, dots, diacritics, and historical forms are preserved without modernization or correction. Each line was independently transcribed by two trained researchers, with disagreements resolved through cross-validation and discussion.

## Loading

```python
from datasets import load_dataset

dataset = load_dataset("maximazzik/MAS", "mas")
print(dataset)
print(dataset["train"][0].keys())
```

See [reproduction.md](reproduction.md) for training and evaluation notes and the [paper](https://doi.org/10.1007/978-3-032-36033-5_38) for the complete collection and annotation protocol.
