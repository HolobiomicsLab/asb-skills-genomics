---
name: ngmlr-read-alignment-of-simulated-long-reads
description: Use when simulated long-read FASTQ datasets (e.g. Sim_A/Sim_B at 90x-20x
  coverage) must be converted to coordinate-sorted BAM alignments as input for third-generation
  sequencing SV callers.
when_to_use_negative:
- Input reads are Illumina short reads (~250-300 bp) rather than third-generation
  long reads, since NGMLR targets long-read alignments
- Raw sequencing data are already provided as aligned BAM files (e.g. the GIAB Ashkenazim
  Trio aligned files), making re-alignment unnecessary
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NGMLR
  role: Aligns PaSS-simulated FASTQ long reads to the reference genome to produce
    BAM files for SV calling
- name: PaSS
  role: Simulates PacBio SMRT long reads at 90x/65x/45x/20x coverage, average read
    length 20k bp, ~15% calling error rate
- name: DASV
  role: Downstream deletion SV caller consuming the NGMLR BAM alignments
  repo: https://github.com/deconvolution-w/DASV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:85c44f76288cbb48c9b1d6e0b02cdcb8410ef269a5daf62f3c497ca2e535a094
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  - NGMLR
  - PBSV
  - PaSS
  - SVIM
  - Sniffles
  - cuteSV
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# NGMLR read alignment of simulated long reads

## Summary

Align PaSS-simulated PacBio SMRT long reads (FASTQ) to a reference genome with NGMLR to produce BAM files that feed downstream deletion SV callers such as DASV, cuteSV, SVIM, Sniffles and PBSV.

## When to use

Use when simulated long-read FASTQ datasets (e.g. Sim_A/Sim_B at 90x-20x coverage) must be converted to coordinate-sorted BAM alignments as input for third-generation sequencing SV callers.

## When NOT to use

- Input reads are Illumina short reads (~250-300 bp) rather than third-generation long reads, since NGMLR targets long-read alignments
- Raw sequencing data are already provided as aligned BAM files (e.g. the GIAB Ashkenazim Trio aligned files), making re-alignment unnecessary

## Inputs

- Simulated PacBio SMRT long reads in FASTQ format from PaSS (Sim_A and Sim_B homozygous/heterozygous groups at 90x, 65x, 45x, 20x coverage)
- Reference genome (FASTA)

## Outputs

- One BAM alignment file per simulated dataset (eight BAMs for the four coverage depths x two variant groups)

## How to apply

After simulating PacBio SMRT reads with PaSS at average read length 20k bp and ~15% calling error rate (coverage depths 90x, 65x, 45x, 20x), align each FASTQ file to the reference genome using NGMLR, which is suited to long reads with high error rates. The result is one BAM file per simulated dataset (Sim_A{90X,65X,45X,20X} and Sim_B{90X,65X,45X,20X}). These BAM files are then run identically through DASV and the baseline callers (cuteSV, SVIM, Sniffles, PBSV). Correct application is verified when each of the eight simulated datasets yields a usable BAM and downstream tools produce calls comparable against the simulated truth set for precision, recall and F1 score per coverage depth.

## Related tools

- **NGMLR** (Aligns PaSS-simulated FASTQ long reads to the reference genome to produce BAM files for SV calling)
- **PaSS** (Simulates PacBio SMRT long reads at 90x/65x/45x/20x coverage, average read length 20k bp, ~15% calling error rate)
- **DASV** (Downstream deletion SV caller consuming the NGMLR BAM alignments) — https://github.com/deconvolution-w/DASV

## Examples

```
ngmlr -t 8 -r reference.fa -q sim_A_90x.fastq -o sim_A_90x.bam
```

## Evaluation signals

- Exactly eight BAM files are produced, one per simulated dataset (Sim_A and Sim_B at 90x, 65x, 45x and 20x)
- Downstream DASV quality filtering (removing reads with alignment quality <20 and FLAG 4 or >256) runs on the NGMLR SAM/BAM without schema errors
- SV callers applied to the BAMs produce calls that can be scored against the simulated truth set, yielding per-depth precision/recall/F1 as in Supplementary Tables 7(a&b) and 8(a&b)
- Metrics from aligned-based calling reproduce the reported trend, e.g. DASV recall of at least 82.16% at the 20x low-coverage condition

## Limitations

- Simulated reads carry ~15% calling error rate at 20k bp average read length; coverage-dependent performance means low-depth (20x) conditions are hardest
- Only deletion variant calling is evaluated; insertion and base substitution detection is future work for this pipeline

## Evidence

- [other] Simulated PacBio SMRT long reads with PaSS at coverage depths of 90x, 65x, 45x and 20x for each group, using an average read length of 20k bp and an average calling error rate of 15%, yielding the eight datasets Sim_A{90X,65X,45X,20X} and Sim_B{90X,65X,45X,20X} as FASTQ files. 3. Align the simulated FASTQ reads to the reference genome with NGMLR to obtain BAM files for each simulated dataset.: [evidence span withheld — anchor status: not_found]
- [methods] Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256: "Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256"
- [methods] Sim_A90X,65X,45X,20X and Sim_B90X,65X,45X,20X were simulated datasets of homozygous and heterozygous variant groups: "Sim_A90X,65X,45X,20X and Sim_B90X,65X,45X,20X were simulated datasets of homozygous and heterozygous variant groups"
