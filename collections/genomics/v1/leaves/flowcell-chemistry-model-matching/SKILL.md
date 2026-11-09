---
name: flowcell-chemistry-model-matching
description: Use when an ONT dataset (FAST5 or POD5) must be basecalled and methylation-called
  but the flowcell chemistry (R9.4.1 vs R10.4.1) dictates which Guppy/Dorado model
  and DeepMod2 BiLSTM model must be applied.
when_to_use_negative:
- The signal files' chemistry is unknown and cannot be determined from metadata —
  guessing a model risks systematically wrong methylation calls
- You already have Dorado/Guppy modBAMs with MM/ML tags and only need bedMethyl conversion
  (use modkit directly instead)
- Detecting modifications other than 5mCpG (e.g. 5hmC, 6mA) — DeepMod2 currently only
  provides 5mCpG models
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0654
tools:
- name: DeepMod2
  role: Runs chemistry-matched BiLSTM models to call per-read/per-site 5mC from FAST5/POD5
    signal given an aligned BAM
  repo: https://github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecalls POD5/FAST5 with --emit-moves using a chemistry-specific model to
    produce move-table BAMs
  repo: https://github.com/nanoporetech/dorado
- name: Guppy
  role: Alternative basecaller with chemistry-specific configs (--bam_out --moves_out)
    whose pass/fail BAMs feed DeepMod2
- name: modkit
  role: Post-processing of modBAM (MM/ML) outputs into bedMethyl summaries
  repo: https://github.com/nanoporetech/modkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:9773e6a49eec377e7fdfbc6967241224328c6a90ba4d18b1b26ccf8b803d017c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - Guppy
  - minimap2
  - modkit
  - samtools
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# flowcell-chemistry-model-matching

## Summary

Select the chemistry-matched basecaller configuration and DeepMod2 methylation model (R9.4.1 vs R10.4.1) so that 5mC detection from POD5/FAST5 ONT signal is compatible with the flowcell that generated the data, since signal characteristics and trained models differ per chemistry.

## When to use

Use when an ONT dataset (FAST5 or POD5) must be basecalled and methylation-called but the flowcell chemistry (R9.4.1 vs R10.4.1) dictates which Guppy/Dorado model and DeepMod2 BiLSTM model must be applied.

## When NOT to use

- The signal files' chemistry is unknown and cannot be determined from metadata — guessing a model risks systematically wrong methylation calls
- You already have Dorado/Guppy modBAMs with MM/ML tags and only need bedMethyl conversion (use modkit directly instead)
- Detecting modifications other than 5mCpG (e.g. 5hmC, 6mA) — DeepMod2 currently only provides 5mCpG models

## Inputs

- POD5 signal files (R10.4.1 flowcell datasets)
- FAST5 signal files (R9.4.1 flowcell datasets)
- Reference FASTA (GRCh38 analysis set) with .fai index
- Aligned BAM carrying move tables (MM/ML-capable)

## Outputs

- Basecalled BAM with move tables per sample/chemistry
- Methylation-tagged output BAM with MM/ML 5mC tags
- Per-read and per-site methylation BED/TSV outputs

## How to apply

First identify the flowcell chemistry of the signal files (POD5 for R10.4.1, FAST5 for R9.4.1 in the Ashkenazim trio workflow). Basecall with a kit/chemistry-matched model: Dorado with --emit-moves using e.g. dna_r10.4.1_e8.2_400bps_hac@v4.3.0, or Guppy with the matching config (e.g. dna_r10.4.1_e8.2_400bps_5khz_hac.cfg) with --bam_out --moves_out, because move tables are required for signal-to-basecall alignment. Then run DeepMod2 detect with a chemistry-matched BiLSTM model such as bilstm_r10.4.1_5khz_v4.3, passing the aligned BAM, signal directory, and GRCh38 reference. The rationale is that DeepMod2 provides separate models per flowcell chemistry, and mismatched models would misinterpret ionic current levels, degrading F1 relative to the paper's 95.7–97.1% per-read range.

## Related tools

- **DeepMod2** (Runs chemistry-matched BiLSTM models to call per-read/per-site 5mC from FAST5/POD5 signal given an aligned BAM) — https://github.com/WGLab/DeepMod2
- **Dorado** (Basecalls POD5/FAST5 with --emit-moves using a chemistry-specific model to produce move-table BAMs) — https://github.com/nanoporetech/dorado
- **Guppy** (Alternative basecaller with chemistry-specific configs (--bam_out --moves_out) whose pass/fail BAMs feed DeepMod2)
- **modkit** (Post-processing of modBAM (MM/ML) outputs into bedMethyl summaries) — https://github.com/nanoporetech/modkit

## Examples

```
dorado basecaller dna_r10.4.1_e8.2_400bps_hac@v4.3.0 pod5s/ --emit-moves > basecall.bam && deepmod2 detect --bam aligned.bam --input pod5s/ --model bilstm_r10.4.1_5khz_v4.3 --file_type pod5 --threads 8 --ref GRCh38.fa --output MOD_CALLS
```

## Evaluation signals

- All six per-sample/per-flowcell sorted BAMs are valid, indexed, and carry MM/ML 5mC modification tags
- Downstream per-read F1 falls within the paper-reported 95.7–97.1% range for the R9.4.1 Ashkenazim trio benchmark
- The basecaller model/config string (e.g. dna_r10.4.1_e8.2_400bps_hac@v4.3.0) matches the flowcell chemistry in the dataset metadata
- Move tables (mv tags) are preserved through minimap2 alignment (-ax map-ont -y), verifiable via samtools view tag inspection
- DeepMod2 per-read predictions on the same sample are consistent across reruns and site-level methylation frequencies correlate with expected CpG patterns

## Limitations

- Move tables only approximately demarcate signal-to-base boundaries, which can yield uninformative signal summary statistics for some reads
- Nanopolish, DeepMP, DeepSignal, Tombo, methBERT and Rockfish supply models only for R9.4 flowcells, so chemistry matching constrains tool choice
- DeepMod2 currently provides models only for 5mCpG because datasets for other methylation types are lacking

## Evidence

- [abstract] DeepMod2 implements both a bidirectional long short-term memory (BiLSTM) model and a Transformer model and can analyze POD5 and FAST5 signal files generated on R9 and R10 flowcells: "DeepMod2 implements both a bidirectional long short-term memory (BiLSTM) model and a Transformer model and can analyze POD5 and FAST5 signal files generated on R9 and R10 flowcells"
- [results] move tables required: "It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls"
- [results] R9.4.1 F1 range: "For R9.4.1 datasets of the Ashkenazim trio HG002, HG003 and HG004, the F1-score of DeepMod2 models is in the 95.7–97.1% range"
- [readme] readme basecalling: [evidence span withheld — anchor status: external_doc]
- [intro] R9.4-only tools: "Nanopolish, DeepMP, DeepSignal, DeepMod, Tombo, methBERT and Rockfish provide models only for R9.4 flowcells"
