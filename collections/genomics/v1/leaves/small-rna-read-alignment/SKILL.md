---
name: small-rna-read-alignment
description: Use when short (14–50 nt) sRNA-seq reads must be assigned to tRNA-derived
  loci — an artificial masked tRNA genome (tRNA ± 50 nt flanks), a BAM from bowtie
  '-v2 --', and FASTQ inputs are available.
when_to_use_negative:
- The goal is tncRNA detection in non-plant organisms with tools like tDRmapper, MINTmap,
  or tRF2Cancer, which are trained and tested on human datasets — this skill's artificial-genome
  index and tRNAscan-SE filtering are plant-genome specific
- Reads are longer than 50 nt or the library is not tRNA-enriched small RNA — the
  pipeline classifies only 14–50 nt reads mapping exclusively to artificial tRNA chromosomes
- The alignment target is a regular transcriptome/miRNA reference rather than a masked
  artificial tRNA genome; ambiguous non-tRNA reads would not be excluded
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: bowtie1
  role: Aligns Trim Galore-processed sRNA reads to the artificial tRNA genome index
    with '-v 2 --best', then re-aligns unique mapped reads to tRNA features with '--norc
    -v 0 -m 50 --best --strata' for zero-mismatch plus-strand subtype assignment
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: samtools
  role: Sorts bowtie SAM output and converts to BAM; extracts mapped reads (flags
    0 & 16) to build the unique-reads FASTA for re-alignment
- name: tncRNA Toolkit
  role: Encapsulates the index build (tncRNAs.py -g) and the two-stage alignment/classification
    workflow; bowtie1 and HAMR are bundled in the util/ tar package
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tRNAscan-SE
  role: Predicts nuclear (eukaryote model) and organellar ('-O') tRNAs feeding the
    masked artificial genome; pseudogenes and score < 50 tRNAs are eliminated before
    indexing
- name: bedtools
  role: maskfasta masks filtered tRNA regions with 50 bp 5'/3' flanks to create the
    artificial genome chromosomes used for bowtie-build
- name: Trim Galore
  role: Pre-alignment quality and adapter trimming of single-end small RNA reads (v0.6.6)
    before bowtie alignment
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_005
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:8975f74bd87525c892ba950be7fd910e052231b79f4ef8c9c63df5edccca1960
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
  - Bowtie1
  - HAMR
  - Trim Galore
  - bedtools
  - biopython
  - bowtie1
  - pandas
  - samtools
  - tRNAscan-SE
  - tRNAscanSE
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# small-rna-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Two-stage alignment scheme for tRNA-derived small RNA (tncRNA) profiling: error-tolerant unique alignment of adapter-trimmed sRNA reads to an artificial tRNA-masked genome index with bowtie '-v2 --best', followed by strict plus-strand zero-mismatch re-alignment of mapped unique reads to mature tRNA features with bowtie1 '--norc -v 0 -m 50 --best --strata'.

## When to use

Use when short (14–50 nt) sRNA-seq reads must be assigned to tRNA-derived loci — an artificial masked tRNA genome (tRNA ± 50 nt flanks), a BAM from bowtie '-v2 --best', and FASTQ inputs are available.

## When NOT to use

- The goal is tncRNA detection in non-plant organisms with tools like tDRmapper, MINTmap, or tRF2Cancer, which are trained and tested on human datasets — this skill's artificial-genome index and tRNAscan-SE filtering are plant-genome specific
- Reads are longer than 50 nt or the library is not tRNA-enriched small RNA — the pipeline classifies only 14–50 nt reads mapping exclusively to artificial tRNA chromosomes
- The alignment target is a regular transcriptome/miRNA reference rather than a masked artificial tRNA genome; ambiguous non-tRNA reads would not be excluded

## Inputs

- Single-end small RNA-seq FASTQ (adapter-trimmed and quality-filtered by Trim Galore)
- Artificial genome bowtie index (tRNA-masked genome plus mature/leader/trailer tRNA chromosomes, built by tncRNAs.py -g)
- Mature tRNA + 50 bp leader/trailer FASTA features for the re-alignment stage
- Genome FASTA with headers formatted as >chr1, >chr2, >chrMt, >chrPt

## Outputs

- SAM/BAM alignments of uniquely mapped reads against the artificial tRNA genome (sorted with samtools)
- Set of uniquely mapped reads remapped to tRNA features with subtype-compatible loci (feeding tRF-5, tRF-3, tRF-1, leader-tRF, 5'tRH, 3'tRH classification)
- tncRNA count/RPM table (e.g. tncRNA_counts.csv) quantified as reads mapped to progenitor tRNA / PMF, where PMF = total mapped reads / 10^6

## How to apply

First predict nuclear tRNAs with tRNAscan-SE and organellar tRNAs with the '-O' option, drop pseudogenes and tRNAs scoring below 50, then build an artificial genome by masking tRNA regions with 50 bp flanks using bedtools maskfasta and add mature/leader/trailer FASTA as extra chromosomes for the bowtie-build index. Align quality-filtered (Trim Galore v0.6.6) single-end reads to this artificial genome index with bowtie '-v 2 --best' so reads tolerate one mismatch (sequencing errors) while remaining uniquely placed, then sort and convert SAM to BAM with samtools. For subtype classification, extract unique mapped reads and re-align them to mature tRNA + 50 bp leader/trailer features using bowtie1 with '--norc -v 0 -m 50 --best --strata', keeping only plus-strand zero-mismatch alignments and discarding reads mapping more than 50 times. The mismatch-tolerant pass enables placement while the zero-mismatch re-alignment pass prevents base misidentification from inflating false negatives in tncRNA subtype assignment.

## Related tools

- **bowtie1** (Aligns Trim Galore-processed sRNA reads to the artificial tRNA genome index with '-v 2 --best', then re-aligns unique mapped reads to tRNA features with '--norc -v 0 -m 50 --best --strata' for zero-mismatch plus-strand subtype assignment) — https://github.com/skbinfo/tncRNA-Toolkit
- **samtools** (Sorts bowtie SAM output and converts to BAM; extracts mapped reads (flags 0 & 16) to build the unique-reads FASTA for re-alignment)
- **tncRNA Toolkit** (Encapsulates the index build (tncRNAs.py -g) and the two-stage alignment/classification workflow; bowtie1 and HAMR are bundled in the util/ tar package) — https://github.com/skbinfo/tncRNA-Toolkit
- **tRNAscan-SE** (Predicts nuclear (eukaryote model) and organellar ('-O') tRNAs feeding the masked artificial genome; pseudogenes and score < 50 tRNAs are eliminated before indexing)
- **bedtools** (maskfasta masks filtered tRNA regions with 50 bp 5'/3' flanks to create the artificial genome chromosomes used for bowtie-build)
- **Trim Galore** (Pre-alignment quality and adapter trimming of single-end small RNA reads (v0.6.6) before bowtie alignment) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
python3 tncRNAs.py -s Arabidopsis -i sample_trimmed.fq -o out/ ; bowtie -v 2 --best lib/indexes/Arabidopsis sample_trimmed.fq | samtools sort -o sample.bam
```

## Evaluation signals

- Every reported tncRNA maps exclusively to the artificial tRNA chromosomes (tRNA ± 50 nt flank) and carries a single subtype assignment
- Reads that align more than 50 times in the zero-mismatch re-alignment are discarded (bowtie1 '-m 50')
- Reported tncRNAs fall within the 14–50 nt size range established from 2,448 sRNA-seq samples across the six angiosperms
- Mapped read counts pass the default read-count cut-off of 10 enforced by the toolkit (-c option)
- RPM quantification is consistent with PMF = total mapped reads / 10^6 across per-species tncRNA_counts.csv outputs

## Limitations

- Allowing a single mismatch to overcome sequencing errors may lead to base misidentification and inflation of false negatives — hence the zero-mismatch re-alignment stage
- Conventional RNA-seq suffers from tRNA modification biases; tRNA-seq would be advantageous for identifying tncRNAs, and HAMR modification calls on RNA-seq reads are indirect
- Hector/header constraints of the toolkit: genome FASTA headers must start with '>chr[Num]' (>chrMt, >chrPt for mitochondria/plastids) for automation, and HAMR-1.2 additionally requires python2
- In-silico calls need experimental validation and characterization to distinguish bona fide tncRNAs

## Evidence
