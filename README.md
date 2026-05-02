# Reuse versus Adaptation: Decomposing the ImageNet-to-CIFAR-10 Transfer Win for ResNet-18

A controlled three-condition decomposition of the ImageNet-to-CIFAR-10 transfer-learning win for ResNet-18.

## Paper

- **PDF**: [tex/main.pdf](tex/main.pdf)
- **Source**: [tex/main.tex](tex/main.tex) — compile with `tectonic -X compile tex/main.tex`
- **Bibliography**: [tex/references.bib](tex/references.bib)

## Primary result

Across three random seeds, the predicted strict ordering Scratch < Linear Probe < Fine-tune holds on every individual seed and in the mean.

| Condition | Trainable params | Test top-1 accuracy |
|-----------|------------------|----------------------|
| ResNet-18 from scratch                    | ~11.2 M | **0.7830** ± 0.0244 |
| Linear probe (frozen ImageNet ResNet-18)  | 5,130   | **0.8674** ± 0.0016 |
| Fine-tune (ImageNet ResNet-18)            | ~11.2 M | **0.9119** ± 0.0025 |

## How to reproduce

```bash
# Single command on a machine with a CUDA-capable GPU (16 GB+ recommended).
# The script bootstraps its own venv on /workspace and installs torch,
# torchvision, numpy, tqdm, datasets, and Pillow.
python experiments/01-transfer-baselines/experiment.py
```

The script trains nine ResNet-18 models (three transfer-learning conditions × three seeds) on CIFAR-10 (loaded via the `uoft-cs/cifar10` HuggingFace release), writes per-seed metrics to `summary.json` and `metrics.jsonl`, and a CSV table to `tables/transfer_results.csv`. Total wall-clock per seed averages ~478 seconds on an NVIDIA RTX A4000 (16 GB).

## Figures

| File | Title |
|------|-------|
| `fig-method-overview.png` | Three transfer-learning configurations under study |
| `fig-data-pipeline.png` | CIFAR-10 input pipeline used in every condition |
| `fig-main-bar.png` | Test top-1 accuracy by transfer-learning strategy |
| `fig-test-curves.png` | Per-epoch test top-1 accuracy |
| `fig-loss-curves.png` | Per-epoch training cross-entropy loss |

## Recommended venues

- **CVPR Workshop on Computer Vision in the Wild** — Top-tier vision venue with multiple workshops on transfer learning, robustness, and small-data settings; the controlled ...
- **ICLR Tiny Papers / ICLR Reproducibility Workshop** — The Tiny Papers track explicitly targets workshop-grade contributions of exactly this size: 2-4 page papers with a singl...
- **WACV** — Mid-tier vision conference with a strong "applied / practical" disposition; values reproducibility and well-controlled e...
- **TMLR** — TMLR's explicit policy of accepting methodologically clean work without a "novelty above SOTA" bar is a strong match: th...
- **JOSS** — JOSS publishes short papers (~1000 words) accompanying open-source research software releases; the single-script + per-s...

## Authors

Vikash Chandra Mishra

## Provenance

Session id: `20260502-094010-faa2`. See [log.md](log.md) and [state.json](state.json) for the full pipeline trace.
