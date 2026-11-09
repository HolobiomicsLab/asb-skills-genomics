---
name: trna-derived-fragment-subtype-classification
description: Use when single-end sRNA-seq FASTQ reads (quality-filtered, adapter-trimmed)
  must be classified as tRNA-derived fragments and halves and assigned to tRF/tRH
  subtypes in plant genomes.
when_to_use_negative:
- The goal is profiling miRNAs, piRNAs, or rRNAs rather than tRNA-derived small RNAs
  (an annotation pipeline like SPORTS1.0 covers broader canonical small RNA classes).
- Existing detection tools such as tDRmapper, MINTmap, or tRF2Cancer are a fit only
  for human datasets — this skill is designed for plant tncRNAs; do not expect human-trained
  tooling assumptions to transfer.
- Input is paired-end RNA-seq or long-read data; the pipeline is built for single-end
  small RNA reads fastq files.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: tncRNA Toolkit
  role: In-house pipeline that detects, classifies, and quantifies tncRNAs into tRF-5s,
    tRF-3s, tRF-1, and tRNA halves from genome FASTA + processed sRNA FASTQ
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tRNAscanSE
  role: Predicts nuclear and organellar tRNA genes used to build the masked artificial
    genome; pseudogenes and score<50 tRNAs are eliminated
- name: Trim Galore
  role: Removes low-quality bases and trims adapters from single-end small RNA reads
    before tncRNA alignment (v0.6.6 used)
  repo: https://github.com/FelixKrueger/TrimGalore
- name: HAMR
  role: Predicts modified nucleosides on tncRNAs; bundled with the toolkit (v1.2,
    requires python2)
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: bowtie1
  role: Aligns processed reads to the artificial genome index ('-v2 --best') and performs
    tncRNA realignment ('--norc -v 0 -m 50 --best --strata'); v1.3.0 bundled
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: pandas
  role: Compiles per-species total counts and subtype composition into tncRNA_counts.csv
    (v1.1.0)
- name: biopython
  role: Builds alignment visualizations via pairwise2 (identical=1, non-identical=-1,
    gap-open=-1, gap-extend=-0.5) (v1.77)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:a995c5fb4e0989e20fcf1d3561846be7321d26420b69894bde2c9c2677f83c68
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - HAMR
  - Trim Galore
  - biopython
  - bowtie1
  - pandas
  - tRNAscanSE
  - tncRNA Toolkit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# trna-derived-fragment-subtype-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classification and quantification of plant tRNA-derived non-coding RNAs (tncRNAs, 14–50 nt) into subtypes (tRF-5, tRF-3, tRF-1, leader-tRF, 5'tRH, 3'tRH, other tRF) based on mapped position on mature tRNAs and their 50 bp leader/trailer flanks, as implemented in the tncRNA Toolkit.

## When to use

Use when single-end sRNA-seq FASTQ reads (quality-filtered, adapter-trimmed) must be classified as tRNA-derived fragments and halves and assigned to tRF/tRH subtypes in plant genomes.

## When NOT to use

- The goal is profiling miRNAs, piRNAs, or rRNAs rather than tRNA-derived small RNAs (an annotation pipeline like SPORTS1.0 covers broader canonical small RNA classes).
- Existing detection tools such as tDRmapper, MINTmap, or tRF2Cancer are a fit only for human datasets — this skill is designed for plant tncRNAs; do not expect human-trained tooling assumptions to transfer.
- Input is paired-end RNA-seq or long-read data; the pipeline is built for single-end small RNA reads fastq files.

## Inputs

- Genome FASTA per species (headers starting '>chr[Num]', organellar as '>chrMt' and '>chrPt')
- Single-end small RNA reads FASTQ, quality-filtered and adapter-trimmed (e.g. with Trim Galore v0.6.6)
- Genome assemblies used in the paper: TAIR10.1 (Arabidopsis), SL4.0 nuclear and SL3.0 organellar (tomato)

## Outputs

- Per-read subtype assignments (tRF-5, tRF-3, tRF-1, leader-tRF, 5'tRH, 3'tRH, other tRF) with mapped positions
- tncRNA counts with RPM normalization (PMF = total mapped reads/10^6), compiled per species into tncRNA_counts.csv
- Bowtie index of the artificial (masked + mature/leader/trailer) genome in lib/indexes/<species>

## How to apply

First predict nuclear tRNAs with tRNAscan-SE (eukaryotes model) and organellar tRNAs with the '-O' option, then eliminate pseudogenes and tRNAs scoring below 50. Build an artificial genome by masking filtered tRNA regions plus 50 bp 3'/5' flanks (bedtools maskfasta) and append mature, leader, and trailer tRNA FASTAs as extra chromosomes for the bowtie-build index. Align Trim Galore-processed single-end reads with bowtie '-v2 --best' for error-tolerant, uniquely mapping alignments, then realign unique reads with bowtie1 '--norc -v 0 -m 50 --best --strata' to the + strand with no mismatch, discarding reads mapping >50 times. Assign subtype by mapped position on the mature tRNA and flanks: tRF-5, tRF-3, tRF-1, and tRNA halves cleaved at the anticodon loop (2 nt + 3 nt anticodon + 2 nt = 7 nt); compute RPM via PMF = total mapped reads/10^6.

## Related tools

- **tncRNA Toolkit** (In-house pipeline that detects, classifies, and quantifies tncRNAs into tRF-5s, tRF-3s, tRF-1, and tRNA halves from genome FASTA + processed sRNA FASTQ) — https://github.com/skbinfo/tncRNA-Toolkit
- **tRNAscanSE** (Predicts nuclear and organellar tRNA genes used to build the masked artificial genome; pseudogenes and score<50 tRNAs are eliminated)
- **Trim Galore** (Removes low-quality bases and trims adapters from single-end small RNA reads before tncRNA alignment (v0.6.6 used)) — https://github.com/FelixKrueger/TrimGalore
- **HAMR** (Predicts modified nucleosides on tncRNAs; bundled with the toolkit (v1.2, requires python2)) — https://github.com/skbinfo/tncRNA-Toolkit
- **bowtie1** (Aligns processed reads to the artificial genome index ('-v2 --best') and performs tncRNA realignment ('--norc -v 0 -m 50 --best --strata'); v1.3.0 bundled) — https://github.com/skbinfo/tncRNA-Toolkit
- **pandas** (Compiles per-species total counts and subtype composition into tncRNA_counts.csv (v1.1.0))
- **biopython** (Builds alignment visualizations via pairwise2 (identical=1, non-identical=-1, gap-open=-1, gap-extend=-0.5) (v1.77))

## Examples

```
python3 tncRNAs.py -s Arabidopsis -i processed_smallRNA.fastq -o output_dir
```

## Evaluation signals

- All reported tncRNAs fall within the 14–50 nt size range identified from 2,448 sRNA-seq samples across the six angiosperms.
- All five core subtype classes (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) plus leader-tRF/other tRF are represented in the output table tncRNA_counts.csv.
- Reads are counted as tncRNAs only if they map exclusively to the artificial chromosomes (masked tRNA region + 50 nt leader/trailer), not to the regular genome.
- Multi-mapped reads aligning more than 50 times (bowtie '-m 50') are absent from final tncRNA calls.
- tRNA halves show cleavage at the anticodon loop window of 7 nt (2 nt + 3 nt anticodon + 2 nt).

## Limitations

- Allowing a single mismatch to overcome sequencing errors may cause base misidentification and inflation of false negatives — hence the strict '-v 0' realignment step.
- Conventional RNA-seq suffers from tRNA modification biases; tRNA-seq would be advantageous for tncRNA identification.
- Experimental validation and characterization are vital to distinguish bona fide tncRNAs; the in-silico pipeline alone cannot confirm them.
- The Bowtie -v mismatch parameter should not exceed 3 per the README; HAMR-1.2 requires python2 to run.
- There is no single standardized nomenclature for tncRNA subtypes, so subtype labels may not match other studies' conventions.

## Evidence
