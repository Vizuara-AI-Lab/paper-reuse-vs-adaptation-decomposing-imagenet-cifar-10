# Peer Review (final, post-revision): Reuse versus Adaptation: Decomposing the ImageNet-to-CIFAR-10 Transfer Win for ResNet-18

**Reviewer:** balanced / deep
**Recommendation:** accept (workshop track)
**Confidence:** 3
**Score:** 8

## Summary of contributions

The paper takes the everyday "ImageNet pretraining helps on CIFAR-10" observation and decomposes the resulting accuracy gain into a feature-reuse term (linear probe over a frozen ImageNet-pretrained ResNet-18) and a feature-adaptation term (full end-to-end fine-tuning of the same checkpoint). All three conditions share architecture, schedule, batch size, input pipeline, and seed budget; only initialisation and parameter freedom vary. The headline result is a strict per-seed ordering Scratch (0.7830 +- 0.0244) < Linear Probe (0.8674 +- 0.0016) < Fine-tune (0.9119 +- 0.0025), with the linear-probe step buying +8.44 pp and the fine-tune step buying an additional +4.45 pp.

## Strengths

1. **Cleanly controlled experimental design.** The three conditions are matched on every variable except initialisation and parameter freedom; the BatchNorm-pinned-to-eval detail in Section 3.2 handles a subtlety that quietly invalidates many probe studies.
2. **Honest per-seed reporting.** The strict ordering is verified on every individual seed, not only on the mean. The order-of-magnitude std gap between pretrained (1e-3 to 3e-3) and from-scratch (2.4e-2) is itself a useful secondary finding.
3. **Sober scoping.** The 5-epoch budget, the 32-to-224 upsample confound, and the choice to omit external CIFAR-10 baselines are explicit; the paper does not let any of these become hidden footnotes.
4. **Reproducibility commitment.** The Reproducibility subsection added in revision lists the exact seeds, the torchvision weights tag, the HuggingFace dataset revision, and the single `python experiment.py` command that reruns the full sweep.

## Weaknesses (post-revision, all low severity)

1. **No external CIFAR-10 baseline.** Scope-of-work choice; the paper is explicit about why and what the trade-off is.
2. **Single backbone (ResNet-18) and source domain (ImageNet-1K).** Acknowledged as future work in Section 6; the headline numbers are reported for one backbone with that limitation stated up front.
3. **No epoch-budget sweep.** Acknowledged as future work; the 5-epoch numbers should be read as "how quickly each strategy reaches a competent classifier under a tight budget" rather than asymptotic accuracy.

## Recommendation justification

This is a tightly scoped, honestly reported, single-GPU reproducibility study that does exactly what its title promises. The decomposition into feature reuse versus feature adaptation is not a novel concept but the clean execution, the per-seed agreement check, the explicit acknowledgement of scope, and the post-revision Reproducibility subsection together make this a genuinely useful workshop contribution and a high-quality pedagogical artefact. The remaining medium-from-iteration-1 weaknesses are all genuine scope-of-work choices that the paper itself flags as limitations and future work, and on a fair workshop bar they reduce to low severity. I recommend accept for the workshop track.
