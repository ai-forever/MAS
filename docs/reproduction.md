# Reproducing the MAS experiments

This document separates the released, executable workflow from details that were not archived precisely enough for bit-for-bit reproduction.

## Scope

The repository contains:

- MAS data registration and a canonical Qwen2.5-VL-3B LoRA configuration;
- specialized OCR configurations for EasyOCR, MMOCR, and PaddleOCR;
- an optional `lmms-eval` task for independent evaluation;
- the hyperparameters reported in the paper.

The repository does not vendor LLaMA-Factory or `lmms-eval`. Exact commit hashes and complete environment snapshots from the original ICDAR runs were not retained here, so package evolution may cause behavioral differences.

## Data

The Hugging Face release uses config `mas` with `train` and `test` splits:

```python
from datasets import load_dataset

mas = load_dataset("maximazzik/MAS", "mas")
assert len(mas["train"]) == 9472
assert len(mas["test"]) == 2369
```

The published `test` split is the local validation subset used as the held-out evaluation set in the paper. See [dataset.md](dataset.md) for its schema and limitations.

## LVLM fine-tuning

The paper used [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) with LoRA and DeepSpeed ZeRO-3. The canonical released configuration is:

```text
configs/llamafactory/qwen2_5_vl_3b_lora_mas.yaml
```

Copy `configs/llamafactory/` into a compatible LLaMA-Factory checkout, preserving the directory name. From the LLaMA-Factory root:

```bash
FORCE_TORCHRUN=1 DISABLE_VERSION_CHECK=1 \
llamafactory-cli train configs/llamafactory/qwen2_5_vl_3b_lora_mas.yaml
```

The configuration records the paper protocol:

| Parameter | Value |
|---|---|
| Base model | `Qwen/Qwen2.5-VL-3B-Instruct` |
| Fine-tuning | LoRA on all linear layers |
| Vision encoder | trainable through LoRA |
| Multimodal projector | frozen |
| LoRA rank / alpha / dropout | 8 / 16 / 0.1 |
| Optimizer | AdamW |
| Learning rate | `4e-4` |
| Weight decay | `0.02` |
| Schedule | cosine, 5% warmup |
| Epochs | 10 |
| Seed | 42 |
| Precision | bfloat16 |
| Distributed training | DeepSpeed ZeRO-3 |
| Original hardware | 8 GPUs |
| Per-device batch / accumulation | 16 / 4 |
| Effective train batch | 512 |
| Image pixel cap | 262,144 |

Checkpoints are evaluated every 20 steps in the MAS-only configuration, and the checkpoint with minimum `eval_loss` is selected.

The released YAML reconciles the protocol reported in the paper. The archived MAS-only experiment YAML used ZeRO-2, while the paper reports ZeRO-3; this discrepancy prevents a claim of bit-for-bit reconstruction of the historical run.

For MAS + Muharaf and MAS + SARD transfer experiments, keep the optimization settings fixed and register the additional datasets in LLaMA-Factory. The external corpora are not redistributed by this repository; obtain them from their original providers. Training-set sizes reported in the paper are:

| Training data | Examples |
|---|---:|
| MAS | 9,472 |
| MAS + Muharaf | 33,967 |
| SARD only | 100,000 |
| MAS + SARD + Muharaf | 129,691 |

## Specialized OCR systems

The corresponding configurations are under `configs/`:

- `easy_ocr_g1_finetune_on_MAS.yaml`;
- `mmocr_abinet_MAS.py`;
- `paddle_MAS_rec_mobile.yaml`;
- `paddle_MAS_rec_server.yaml`;
- matching `MAS_Muharaf` variants.

These files are intended to be used with their upstream frameworks. Replace local dataset/output placeholders where required. The paper used each framework's default optimization hyperparameters and fully fine-tuned the specialized OCR models where supported.

## Evaluation used in the paper

The original experiments did **not** use `lmms-eval`. Their evaluation pipeline was:

```text
LoRA training
  → merge adapter into the base model
  → greedy inference with calc_metrics_multi.py
  → line-level CER/WER/BLEU CSV
  → aggregation by script and domain
```

Inference used deterministic decoding (`do_sample: false`) and the same diplomatic-transcription instruction across models. CER was computed as character-level Levenshtein distance divided by reference length, WER with `jiwer`, and BLEU-4 with NLTK smoothing method 4. Reported values are arithmetic means over lines.

The historical evaluation scripts depend on the original LLaMA-Factory checkout, local ShareGPT JSON paths, and additional worker files. They have not yet been packaged into a portable standalone evaluator in this repository. Therefore, the released results can be independently checked against the same MAS split, but the exact historical command is not claimed to run unchanged outside the original environment.

## Optional evaluation with lmms-eval

The included task is a portable post-publication evaluation route, not the pipeline that produced the paper tables.

Copy `configs/paper_arabic/` into the `lmms_eval/tasks/` directory of a compatible [lmms-eval](https://github.com/EvolvingLMMs-Lab/lmms-eval) checkout. Evaluate a merged model on one GPU with:

```bash
python -m lmms_eval \
  --model qwen2_5_vl \
  --model_args pretrained=/path/to/merged/model \
  --tasks paper_arabic \
  --batch_size 1 \
  --output_path results/mas \
  --log_samples
```

For multiple GPUs, use the launcher recommended by the installed `lmms-eval` version. The task loads `maximazzik/MAS`, config `mas`, split `test`. It uses each row's released `prompt`, falling back to the prompt in `paper_arabic.yaml`.

The task normalizes newlines and repeated whitespace, then computes edit distance per line:

$$
\mathrm{CER} = \frac{D_{\mathrm{char}}(\mathrm{reference}, \mathrm{prediction})}
{\max(1, |\mathrm{reference}|)}.
$$

WER is computed analogously over words. The task reports the arithmetic mean over valid lines. Results may differ from the paper if model serialization, preprocessing, prompt handling, or dependency versions differ.

## Reproducibility limitations

The following details are not claimed to be exactly recoverable from this repository:

- the LLaMA-Factory commit hash and historical evaluation environment used for every original run;
- a frozen package/CUDA snapshot from the ICDAR experiments;
- every intermediate checkpoint and proprietary-model API revision;
- deterministic equivalence across GPU architectures and distributed kernels.

Consequently, this release supports reproduction of the documented protocol and independent verification on the same MAS test split, but not guaranteed bit-identical model weights or predictions.
