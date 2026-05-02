# Experiment Plan v1 — Transfer Learning Baselines on CIFAR-10

## Question
How much does ImageNet-pretrained initialisation help on CIFAR-10
classification, and how does the gain split between (a) reusing
backbone features at all (linear probe over a frozen backbone) and
(b) end-to-end fine-tuning of the backbone? We expect a clean
ordering — Scratch < Linear Probe < Fine-tune — but the size of the
linear-probe gap quantifies how transferable raw ImageNet features are
to small natural images, and the fine-tune gap quantifies the value of
adapting those features to CIFAR-10's 10-class statistics.

## Dataset + preprocessing
- **CIFAR-10** (60k 32×32 RGB images; 50k train / 10k test; 10 classes)
  pulled via `torchvision.datasets.CIFAR10` (auto-downloads to
  `/workspace/data` on first run; persists across iterations).
- Train transforms: resize 32→**224**, random horizontal flip, ToTensor,
  ImageNet mean/std normalisation (the same `T.Normalize` baked into
  `ResNet18_Weights.IMAGENET1K_V1.transforms`).
- Test transforms: resize 32→224, ToTensor, ImageNet mean/std.
- The 32→224 upsample lets stock ImageNet weights see a feature map at
  the resolution they were trained on; this is the standard way to
  evaluate ImageNet-trained backbones on CIFAR-10 in transfer-learning
  studies. It costs more memory than 32×32 but the model still fits a
  16 GB A4000 with batch size 128.

## Conditions (three back-to-back trainings per seed)
| Tag | Initialisation | Trainable params | Optimizer |
|-----|----------------|------------------|-----------|
| Scratch  | random | full ResNet-18 (~11 M) | Adam, lr=1e-3 |
| Linear probe | ImageNet1K_V1 | only the new `fc` layer (5 130 params) | SGD, lr=1e-2, momentum=0.9 |
| Fine-tune | ImageNet1K_V1 | full ResNet-18 (~11 M) | Adam, lr=1e-3 |

For the linear probe, BatchNorm layers are pinned to `eval()` so the
frozen feature statistics stay unchanged — without this, BN running
means drift during a forward-only pass and confound the measurement.

## Training protocol
- **Backbone:** `torchvision.models.resnet18`. The final
  `nn.Linear(in_features, 1000)` is replaced by `nn.Linear(in_features, 10)`
  in every condition and is always randomly initialised.
- **Epochs:** 5 per condition.
- **Batch size:** 128.
- **Loss:** cross-entropy.
- **Seeds:** `SEEDS=0,1,2` (3 runs per condition → 9 trainings total).
- **Hardware:** persistent RunPod pod with NVIDIA RTX A4000 (16 GB).

## Evaluation metric(s)
- **Primary (schema field):** `linear_probe_test_acc` — the cleanest
  isolation of "transferred features".
- **Reported alongside:** `scratch_test_acc`, `finetune_test_acc` and
  per-epoch test accuracy / training loss curves.
- We report mean ± std over 3 seeds per condition.

## Expected artifacts
- `data/summary.json` — the structured summary (primary metric +
  per-condition aggregates + config).
- `data/metrics.jsonl` — one JSON line per (seed) with full curves.
- `tables/transfer_results.csv` — flat table for the paper.
- `figures/fig_test_accuracy_bar.png` — bar chart of mean test
  accuracy ± std for the three conditions.
- `figures/fig_test_accuracy_curves.png` — per-epoch test accuracy
  trajectories for the three conditions.
- `figures/fig_training_loss_curves.png` — per-epoch training loss
  trajectories for the three conditions.

## Compute budget
- 9 trainings × ~5 min/training (5 epochs of 50 k images at batch 128
  on an A4000, including the linear-probe runs which are markedly
  faster) ≈ **45–60 min** wall time per iteration. Comfortably inside
  the 4-hour iteration budget.
- Peak VRAM: ResNet-18 at 224×224 with batch 128 ≈ 5–6 GB; the 16 GB
  A4000 has 2-3× headroom.

## Why this is honest, not ambitious
- One backbone (ResNet-18), one dataset (CIFAR-10), three conditions,
  three seeds. No tricks, no "best out of 5" reporting, no schedule
  search — every hyperparameter is stated above and committed in
  source.
- The numbers we report come straight out of `summary.json` written by
  the script — there is no separate analysis pipeline that could drift
  from the source numbers.
