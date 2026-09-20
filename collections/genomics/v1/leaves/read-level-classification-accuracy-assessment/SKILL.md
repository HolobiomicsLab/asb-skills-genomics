---
name: read-level-classification-accuracy-assessment
description: Use when per-read 5mC calls from Nanopore data (e.g., DeepSignal-plant
  vs Megalodon) must be validated against bisulfite sequencing across CG/CHG/CHH contexts
  using Pearson correlation, RMSE, and read-level accuracy.
when_to_use_negative:
- Only bisulfite sequencing data is available with no native Nanopore reads — there
  are no per-read modification calls to assess against the gold standard.
- The goal is only to profile methylation frequencies, not to benchmark a caller's
  per-read classification against bisulfite.
- Cytosines of interest are covered by fewer than 5 Nanopore reads or fewer than 10
  bisulfite reads — these are excluded by the evaluation filter and cannot support
  the metrics.
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DeepSignal-plant
  role: Produces the per-read 5mC probabilities (call_mods) and per-site frequencies
    (call_freq) being evaluated against bisulfite
  repo: https://github.com/PengNi/deepsignal-plant
- name: Megalodon
  role: Alternative basecalling-based modified-base caller, original and re-trained,
    evaluated head-to-head on the same reads
- name: Tombo
  role: Re-squiggles raw Nanopore signals to genome reference bases (version 1.5.1),
    a prerequisite for per-read signal features
  repo: https://github.com/nanoporetech/tombo
- name: Guppy
  role: Basecalls the Nanopore reads (version >=3.6.1) prior to modification calling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:2921ec6589cb11d6336a3e81879e522596b6eff3f86d870f623900a2c4bd6cc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Guppy
  - Megalodon
  - Tombo
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# read-level-classification-accuracy-assessment

## Summary

Assess per-read 5mC classification accuracy of Nanopore modification callers (DeepSignal-plant vs Megalodon) against a bisulfite-sequencing gold standard, computing per-context (CG/CHG/CHH) Pearson correlation, RMSE, and read-level accuracy in plants.

## When to use

Use when per-read 5mC calls from Nanopore data (e.g., DeepSignal-plant vs Megalodon) must be validated against bisulfite sequencing across CG/CHG/CHH contexts using Pearson correlation, RMSE, and read-level accuracy.

## When NOT to use

- Only bisulfite sequencing data is available with no native Nanopore reads — there are no per-read modification calls to assess against the gold standard.
- The goal is only to profile methylation frequencies, not to benchmark a caller's per-read classification against bisulfite.
- Cytosines of interest are covered by fewer than 5 Nanopore reads or fewer than 10 bisulfite reads — these are excluded by the evaluation filter and cannot support the metrics.

## Inputs

- Raw Nanopore FAST5 reads from plants (e.g., A. thaliana R9.4 reads)
- Per-read 5mC probability calls from deepsignal_plant call_mods (arabnrice2-1 checkpoint)
- Megalodon modified-base calls on the same reads (original or re-trained on the combined A. thaliana/O. sativa dataset)
- Bisulfite sequencing methylation calls as the gold standard
- Reference genome FASTA (e.g., GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana)

## Outputs

- Per-site methylation frequency file from deepsignal_plant call_freq (optionally bedMethyl via --bed --sort)
- Context-wise (CG/CHG/CHH) Pearson correlation with bisulfite methylation levels per tool
- Context-wise RMSE against bisulfite methylation levels
- Read-level classification accuracy per tool
- Evaluation site list restricted to cytosines with >=5 Nanopore reads and >=10 bisulfite reads

## How to apply

Basecall raw Nanopore reads with Guppy (version >=3.6.1) and re-squiggle the raw FAST5 signals against the reference genome with tombo 1.5.1, writing to the RawGenomeCorrected_000 group. Run `deepsignal_plant call_mods` with the pre-trained arabnrice2-1 checkpoint (motifs C) to obtain per-read 5mC probabilities, then aggregate to per-site methylation frequencies with `deepsignal_plant call_freq`; run Megalodon's modified-base calling on the same reads. Restrict the evaluation set to cytosines covered by at least 5 mapped Nanopore reads AND at least 10 mapped bisulfite reads, since only sites observed by both platforms can be assessed. Split metrics by sequence context (CG/CHG/CHH) because plant methylation, especially CHH, is highly asymmetric, and compute Pearson correlation with bisulfite methylation levels, RMSE, and read-level accuracy per tool. Accept the assessment only when recomputed context-wise metrics reproduce the published values (e.g., CHG RMSE 0.1355 and CG read-level accuracy 0.9257 for B. nigra) within a small tolerance.

## Related tools

- **DeepSignal-plant** (Produces the per-read 5mC probabilities (call_mods) and per-site frequencies (call_freq) being evaluated against bisulfite) — https://github.com/PengNi/deepsignal-plant
- **Megalodon** (Alternative basecalling-based modified-base caller, original and re-trained, evaluated head-to-head on the same reads)
- **Tombo** (Re-squiggles raw Nanopore signals to genome reference bases (version 1.5.1), a prerequisite for per-read signal features) — https://github.com/nanoporetech/tombo
- **Guppy** (Basecalls the Nanopore reads (version >=3.6.1) prior to modification calling)

## Examples

```
tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna
```

## Evaluation signals

- Evaluation restricted to cytosines with >=5 mapped Nanopore reads and >=10 mapped bisulfite reads; metrics must be reported per context (CG/CHG/CHH).
- Recomputed CHG RMSE and CG read-level accuracy reproduce the published B. nigra values (0.1355 and 0.9257) within a small tolerance.
- DeepSignal-plant should outperform both original and re-trained Megalodon, especially in the CHH context.
- Per-site frequencies from call_freq are consistent with the call_mods per-read output (counts of methylated reads over mapped reads).
- The resquiggle step succeeded — all FAST5s should not fail (a VBZ HDF5 plugin path error would cause failures in tombo resquiggle and call_mods).

## Limitations

- The gold standard is bisulfite sequencing, so model performance is bounded by bisulfite quality; short reads miss repetitive regions and incomplete conversion/DNA degradation introduce errors.
- CHG/CHH high-confidence methylated sites for A. thaliana use the union of three replicates (lower stringency), a potential label-noise source in the evaluation labels.
- Megalodon's training procedure is described only in Supplementary Note 5, limiting reproducibility of the head-to-head comparison.
- Tombo is deprecated (its repository recommends Remora), so re-squiggling depends on unsupported software.

## Evidence

- [discussion] evaluation filter: [evidence span withheld — anchor status: not_found]
- [discussion] gold-standard label criteria: "cytosines covered with at least five reads and had at least 0.9 methylation frequency as high-confidence methylated sites"
- [methods] frequency aggregation: "A modification-frequency file can be generated by `call_freq` function with the call_mods file as input"
- [discussion] signal alignment prerequisite: "We use Tombo21 (version 1.5.1) to map raw signals of reads to contiguous bases in the genome reference"
- [readme] tool description: "deepsignal-plant applies BiLSTM to detect methylation from Nanopore reads. It is built on **Python3** and **PyTorch**."
- [readme] known failure mode: "Please try adding ont-vbz-hdf-plugin to your environment as follows when all fast5s failed in `tombo resquiggle` and/or `deepsignal_plant call_mods`"
