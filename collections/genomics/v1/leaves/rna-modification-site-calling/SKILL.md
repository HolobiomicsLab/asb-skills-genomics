---
name: rna-modification-site-calling
description: Use when you have Nanopore direct RNA FAST5 signals (RNA002) and need
  transcriptome-wide per-read and per-site m6A or m5C calls without sequence-context
  constraints, including stoichiometry at transcript reference sites.
when_to_use_negative:
- Data are from RNA004 chemistry or you need pseudouridine detection — the README
  directs those cases to SWARM instead of CHEUI
- The goal is differential m6A/m5C between two conditions (CHEUI-diff territory, not
  single-condition site calling)
- A site has coverage ≤20 aligned reads — Model 2 site-level prediction is not applicable
  at that depth
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3295
tools:
- name: CHEUI
  role: Two-stage neural network (Model 1 per-read, Model 2 per-site) that predicts
    m6A and m5C probabilities from DRS ionic current signals
  repo: https://github.com/comprna/CHEUI
- name: Nanopolish
  role: eventalign resquiggling of read signals to transcript references with event
    rescaling before CHEUI preprocessing
  repo: https://nanopolish.readthedocs.io/en/latest/
- name: Guppy
  role: Basecalling of FAST5 raw signal files (template_rna_r9.4.1_70bps model, v4.0.11+)
  repo: https://community.nanoporetech.com/downloads/guppy/
- name: Minimap2
  role: Alignment of basecalled FASTQ to the reference transcriptome before signal
    alignment
- name: R2Dtool
  role: Converts CHEUI calls to bed-like format (cheui_to_bed.sh), annotates them
    (R2_annotate) and lifts to genomic coordinates (R2_lift)
  repo: https://github.com/comprna/R2Dtool
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:72d28531cfb329ba8c6519793e4fa98bfe593bfad8bd1c668e1b4202b6d10a6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Guppy
  - Minimap2
  - Nanocompore
  - Nanopolish
  - R2Dtool
  - Xpore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# rna-modification-site-calling

## Summary

Two-stage deep-learning calling of m6A and m5C RNA modifications from Oxford Nanopore direct RNA sequencing (DRS) ionic-current signals using CHEUI: CHEUI-solo Model 1 scores per-read modification probabilities and Model 2 scores transcript-site-level modification probability and stoichiometry.

## When to use

Use when you have Nanopore direct RNA FAST5 signals (RNA002) and need transcriptome-wide per-read and per-site m6A or m5C calls without sequence-context constraints, including stoichiometry at transcript reference sites.

## When NOT to use

- Data are from RNA004 chemistry or you need pseudouridine detection — the README directs those cases to SWARM instead of CHEUI
- The goal is differential m6A/m5C between two conditions (CHEUI-diff territory, not single-condition site calling)
- A site has coverage ≤20 aligned reads — Model 2 site-level prediction is not applicable at that depth

## Inputs

- Nanopore direct RNA sequencing FAST5 raw signal files (RNA002 chemistry)
- Guppy basecalled FASTQ reads
- Reference transcriptome FASTA (e.g. Gencode v38 / GRCh38 for human)
- CHEUI k-mer expected-signal model (kmer_models/model_kmer.csv)
- Trained CHEUI model files (CHEUI_m6A_model1.h5/model2.h5, CHEUI_m5C_model1.h5/model2.h5)

## Outputs

- Per-read modification probability table from CHEUI-solo Model 1 (per transcript site)
- Per-site modification probability and stoichiometry from CHEUI-solo Model 2 for sites with >20 reads
- Significant m5C/m6A site lists at Model 2 probability ≥0.9999 (e.g. 3,167 WT HeLa m5C sites)
- CHEUI methylation calls convertible to bed-like format via R2Dtool cheui_to_bed.sh

## How to apply

Preprocess raw FAST5 with Guppy 4.0.11+ (template_rna_r9.4.1_70bps basecaller model), align basecalled FASTQ to a reference transcriptome with minimap2 keeping primary positive-strand alignments (samtools -F 232457), and resquiggle with Nanopolish eventalign using --scale-events --signal-index --samples --print-read-names. Run CHEUI_preprocess_m6A.py or CHEUI_preprocess_m5C.py (separate scripts per modification) on the eventalign output with the expected k-mer model, then CHEUI-solo Model 1 to get per-read m6A/m5C probabilities, sorting predictions to group reads by site. Apply CHEUI-solo Model 2 only to sites with coverage >20 reads; on cell transcriptomes use the strict probability cutoff ≥0.9999 established by shuffling read signals across sites (FDR = 0.000384 for m5C). Validate by running both models on unmodified IVT controls to estimate the false positive rate (e.g. m5C FPR of 0.0006 at Model 2 probability >0.9999), since strict cutoffs trade sensitivity for low FDR.

## Related tools

- **CHEUI** (Two-stage neural network (Model 1 per-read, Model 2 per-site) that predicts m6A and m5C probabilities from DRS ionic current signals) — https://github.com/comprna/CHEUI
- **Nanopolish** (eventalign resquiggling of read signals to transcript references with event rescaling before CHEUI preprocessing) — https://nanopolish.readthedocs.io/en/latest/
- **Guppy** (Basecalling of FAST5 raw signal files (template_rna_r9.4.1_70bps model, v4.0.11+)) — https://community.nanoporetech.com/downloads/guppy/
- **Minimap2** (Alignment of basecalled FASTQ to the reference transcriptome before signal alignment)
- **R2Dtool** (Converts CHEUI calls to bed-like format (cheui_to_bed.sh), annotates them (R2_annotate) and lifts to genomic coordinates (R2_lift)) — https://github.com/comprna/R2Dtool

## Examples

```
nanopolish eventalign -t 48 --reads reads.fq --bam sorted.bam --genome transcriptome.fa --scale-events --signal-index --samples --print-read-names > nanopolish_out.txt
```

## Evaluation signals

- WT HeLa DRS yields ~3,167 significant m5C sites at Model 2 probability ≥0.9999
- Unmodified HeLa IVT control yields an m5C false positive rate of 0.0006 (sites with Model 2 probability >0.9999)
- Site-level ROC-AUC on held-out IVT mixtures is ~0.92 for m6A and ~0.953 for m5C
- Per-site FPR at P>0.99 on fully unmodified sites is ~0.00074 (m6A) and ~0.00034 (m5C)
- FDR estimated by shuffling read signals across transcriptomic sites matches ~0.000384 for m5C

## Limitations

- Strict cutoffs (Model 2 probability ≥0.9999) reduce CHEUI sensitivity; many orthogonally supported sites have high but sub-threshold CHEUI probabilities
- Nanopore signal variability means modified vs non-modified read differences can be comparable to variation among non-modified reads, limiting detection
- Sites detected only from nanopore reads require independent validation experiments; detection-rate biases exist across technologies
- Signal interference at distances <5 nt would be exacerbated in densely modified molecules such as rRNA
- m6A and m5C require separate preprocessing scripts and separate trained model files (--DL_model flag)

## Evidence

- [readme] CHEUI uses a two-stage deep learning method to detect m6A and m5C transcriptome-wide at single-read and single-site resolution in any sequence context: [evidence span withheld — anchor status: external_doc]
- [readme] Basecalled sequences should be aligned to a reference transcriptome using minimap2 and primary, positive strand alignments should be selected: [evidence span withheld — anchor status: external_doc]
- [methods] Nanopolish eventalign used to align read signals to matched transcript references: "eventalign was then used to align the read signals to the matched transcript references"
- [methods] FDR estimation by shuffling read signals across transcriptomic sites: "we shuffled all read signals across all transcriptomic sites, maintaining the same number of transcriptomic sites"
- [discussion] Strict cutoffs used on cell transcriptome sites to maintain low expected FDR, reducing sensitivity: "we used very strict cut-offs to maintain a low expected false discovery rate (FDR). While these strict cutoffs resulted in a reduction of CHEUI sensitivity"
