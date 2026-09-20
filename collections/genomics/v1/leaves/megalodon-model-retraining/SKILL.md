---
name: megalodon-model-retraining
description: Use when benchmarking a signal-based 5mC caller like DeepSignal-plant
  against Megalodon and the pretrained Megalodon model must be re-trained on the same
  combined plant (A. thaliana + O. sativa) training samples to remove training-dataset
  bias.
when_to_use_negative:
- When only CpG (CG) context detection is needed with a pretrained model — the retraining
  is motivated by plant CHG/CHH contexts that pretrained Megalodon models cannot handle
- When reads are R10.4.1 POD5/BAM data handled by Dorado — this workflow depends on
  the tombo 1.5.1 resquiggling path for R9.4 FAST5 data
- When no bisulfite sequencing gold standard is available, since high-confidence methylated/unmethylated
  training labels and evaluation both depend on bisulfite methylation frequencies
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Megalodon
  role: Basecalling-based modified-base caller that is re-trained on the combined
    A. thaliana/O. sativa dataset and used to call 5mC on the same reads as DeepSignal-plant
- name: Tombo
  role: Re-squiggles raw Nanopore signals to genome reference bases (RawGenomeCorrected_000
    group) before modification calling; version 1.5.1 required
  repo: https://github.com/nanoporetech/tombo
- name: Guppy
  role: Basecalls the Nanopore reads (version >=3.6.1) prior to re-squiggling and
    modified-base calling
- name: DeepSignal-plant
  role: The deep-learning comparator against which the re-trained Megalodon is evaluated,
    run with the arabnrice2-1 checkpoint
  repo: https://github.com/PengNi/deepsignal-plant
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:1923c86352b770b919f816675d45c896dde6284945a45a5d6427e3c00dd849ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - DeepSignal-plant
  - Guppy
  - Megalodon
  - Tombo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# megalodon-model-retraining

## Summary

Re-trains the Megalodon basecalling-based modified-base caller on a combined A. thaliana/O. sativa Nanopore training set (~500x reads, at most 20 million subsampled samples per motif, half methylated and half unmethylated) so it can be fairly compared with DeepSignal-plant for plant 5mC detection across CG, CHG, and CHH contexts.

## When to use

Use when benchmarking a signal-based 5mC caller like DeepSignal-plant against Megalodon and the pretrained Megalodon model must be re-trained on the same combined plant (A. thaliana + O. sativa) training samples to remove training-dataset bias.

## When NOT to use

- When only CpG (CG) context detection is needed with a pretrained model — the retraining is motivated by plant CHG/CHH contexts that pretrained Megalodon models cannot handle
- When reads are R10.4.1 POD5/BAM data handled by Dorado — this workflow depends on the tombo 1.5.1 resquiggling path for R9.4 FAST5 data
- When no bisulfite sequencing gold standard is available, since high-confidence methylated/unmethylated training labels and evaluation both depend on bisulfite methylation frequencies

## Inputs

- Raw Nanopore FAST5 reads (~500x combined A. thaliana and O. sativa training samples; remaining ~100x held out for evaluation)
- Genome reference FASTA (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana; IRGSP-1.0/GCA_001433935.1 for O. sativa)
- Bisulfite sequencing methylation calls used as gold standard for training labels and evaluation
- Pre-trained or re-trained Megalodon model checkpoint

## Outputs

- Per-read 5mC modification calls from the re-trained Megalodon model
- Per-site methylation frequencies for CG, CHG, and CHH contexts
- Context-wise evaluation table: Pearson correlation vs. bisulfite, RMSE, and read-level accuracy

## How to apply

Re-train Megalodon on the combined A. thaliana/O. sativa training samples (~500x Nanopore reads), randomly subsampling at most 20 million samples per motif with half positive (methylated) and half negative (unmethylated) labels, using the procedure detailed in Supplementary Note 5. Prior to calling, re-squiggle raw Nanopore FAST5 signals against the reference genome (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana) with tombo 1.5.1, writing to the RawGenomeCorrected_000 group, and basecall reads with Guppy (>=3.6.1). Run Megalodon's modified-base calling on the same reads processed identically to DeepSignal-plant's deepsignal_plant call_mods pathway. Evaluate only cytosines covered by at least 5 mapped Nanopore reads AND at least 10 mapped bisulfite reads, computing per-context (CG/CHG/CHH) Pearson correlation with bisulfite methylation levels, RMSE, and read-level accuracy. The detailed Megalodon training description lives in Supplementary Note 5 / Supplementary Fig. 4b, so consult those materials before retraining.

## Related tools

- **Megalodon** (Basecalling-based modified-base caller that is re-trained on the combined A. thaliana/O. sativa dataset and used to call 5mC on the same reads as DeepSignal-plant)
- **Tombo** (Re-squiggles raw Nanopore signals to genome reference bases (RawGenomeCorrected_000 group) before modification calling; version 1.5.1 required) — https://github.com/nanoporetech/tombo
- **Guppy** (Basecalls the Nanopore reads (version >=3.6.1) prior to re-squiggling and modified-base calling)
- **DeepSignal-plant** (The deep-learning comparator against which the re-trained Megalodon is evaluated, run with the arabnrice2-1 checkpoint) — https://github.com/PengNi/deepsignal-plant

## Examples

```
tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna
```

## Evaluation signals

- Recomputed context-wise metrics reproduce the published values (e.g. CHG RMSE 0.1355 and CG read-level accuracy 0.9257 for B. nigra) within a small tolerance
- Training subsample respects the cap of at most 20 million samples per motif, split half positive / half negative
- Evaluation cytosines satisfy the dual coverage cutoff: >=5 mapped Nanopore reads and >=10 mapped bisulfite reads
- Resquiggled FAST5 files land in the RawGenomeCorrected_000 tombo group
- DeepSignal-plant outperforms both original and re-trained Megalodon, with the largest gap in the CHH context (as in Fig. 2)

## Limitations

- The Megalodon training procedure is deferred to Supplementary Note 5, limiting reproducibility of the comparison
- Training labels derive from bisulfite sequencing, so model performance is bounded by bisulfite quality
- For CHG/CHH in A. thaliana, high-confidence methylated sites use a union of three replicates (lower stringency), a potential label-noise source

## Evidence

- [discussion] training subsample cap: "we randomly subsample at most 20 million (half positive and half negative) samples as the training dataset for each motif"
- [discussion] Megalodon training reproducibility gap: "A detailed description of training models of Megalodon is shown in Supplementary Note 5 and Supplementary Fig. 4b"
- [discussion] evaluation coverage cutoff: [evidence span withheld — anchor status: not_found]
- [discussion] training/evaluation read split: "we randomly selected ~500× Nanopore reads for training. The remaining ~100× reads were used for evaluation"
- [readme] tombo requirement: "[tombo (version 1.5.1)](https://github.com/nanoporetech/tombo) is required to be installed"
