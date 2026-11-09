---
name: exon-detection-curve-visualization
description: Use when assessing whether transcript architecture (exon length, flanking
  intron lengths) limits detection or biases Fupfi quantification of adjacent-intron
  splicing order from simulated short RNA-seq reads.
when_to_use_negative:
- Quantifying Fupfi accuracy against simulated UPFI/DOFI copy numbers rather than
  assessing detection rates — that requires the accuracy plots, not detection curves
- Analyzing real RNA-seq data with unknown ground truth — detection limits can only
  be directly measured on simulated data with known exon architecture
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: STAR
  role: Two-pass spliced aligner producing BAMs used to score exon detection from
    simulated reads
- name: polyester
  role: Simulates 75/150-nt reads (SE and PE, fraglen 250/500) from constructed UPFI/DOFI
    transcripts
- name: ggplot2
  role: Plots detection curves of percent exons detected by length strata as in Figure
    2
- name: Insplico
  role: Defines the upfi/dofi read counts whose sum >= 1 scores an exon as detected
  repo: gitlab.com/aghr/insplico
- name: plyranges
  role: Filters exons of interest from the GTF by exon_number to build the simulation
    ground truth
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:5cf832be5ab23cf51c46bd7777d69c51103f90b572f1431fb9ab7ff34d9b679b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Biostrings
  - Insplico
  - R
  - STAR
  - dplyr
  - ggplot2
  - pacman
  - plyranges
  - polyester
  - rtracklayer
  - tibble
  - tidyr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47368421052631576
n_claims: 19
---

# exon-detection-curve-visualization

## Summary

Computes and plots the percentage of simulated exons detected (N(upfi+dofi)>=1) as a function of exon length and flanking upstream/downstream intron lengths, reproducing the short-read detection-limitation curves of Insplico's Figure 2 validation.

## When to use

Use when assessing whether transcript architecture (exon length, flanking intron lengths) limits detection or biases Fupfi quantification of adjacent-intron splicing order from simulated short RNA-seq reads.

## When NOT to use

- Quantifying Fupfi accuracy against simulated UPFI/DOFI copy numbers rather than assessing detection rates — that requires the accuracy plots, not detection curves
- Analyzing real RNA-seq data with unknown ground truth — detection limits can only be directly measured on simulated data with known exon architecture

## Inputs

- STAR alignment BAMs for 20 simulations x 4 library layouts (SE 75/150, PE fraglen 250/500)
- Ground-truth count matrices 'simulations/sim{i}/R_{readl}_countmat.txt'
- Simulation table with per-transcript ExonLength, UpIntronLength, DownIntronLength and Ncopies_UPFI/Ncopies_DOFI
- Simulated GTF/FASTA pairs 'simulations_{i}.gtf' and 'simulations_{i}.fa'

## Outputs

- Per-bin exon detection percentages stratified by exon and flanking intron lengths
- ggplot2 detection curves reproducing Figure 2 (percent exons detected vs exon/intron length, per read length and layout)

## How to apply

For each of the 20 simulated samples (nsim=1000 exons, seed=20221107), align polyester-simulated 75-nt and 150-nt reads (SE and PE with fragment sizes 250/500) with STAR in --twopassMode Basic against hg38_reference, then score each selected middle exon as detected when at least one UPFI or DOFI read is observed (N(upfi+dofi)>=1). Bin exons by their simulated exon length and by UpIntronLength/DownIntronLength and compute the percent detected per bin for each read length and library layout. Plot the resulting detection percentages with ggplot2 as detection curves, mirroring Figure 2, so the onset of non-detection reveals the maximum quantifiable exon length imposed by read length and paired-end fragment size.

## Related tools

- **STAR** (Two-pass spliced aligner producing BAMs used to score exon detection from simulated reads)
- **polyester** (Simulates 75/150-nt reads (SE and PE, fraglen 250/500) from constructed UPFI/DOFI transcripts)
- **ggplot2** (Plots detection curves of percent exons detected by length strata as in Figure 2)
- **Insplico** (Defines the upfi/dofi read counts whose sum >= 1 scores an exon as detected) — gitlab.com/aghr/insplico
- **plyranges** (Filters exons of interest from the GTF by exon_number to build the simulation ground truth)

## Examples

```
STAR --twopassMode Basic --genomeDir hg38_reference --readFilesCommand zcat --outSAMtype BAM SortedByCoordinate --quantMode GeneCounts --readFilesIn simulations/sim1/R_75_1.fq.gz
```

## Evaluation signals

- Every exon is scored detected iff N(upfi+dofi)>=1 in the STAR-aligned counts; a hand-check on one simulation should reproduce this rule exactly
- Detection percentages should approach 100% for short exons well within read/fragment capacity and fall off for long exons, with 150-nt/PE-500 libraries detecting longer exons than 75-nt SE libraries
- Curves stratified by UpIntronLength and DownIntronLength should show no systematic detection dependence on flanking intron length (unbiased architecture)
- Dropping the length-stratified bins or comparing percent detected per bin across all 20 seed-offset simulations (seed+i) should yield consistent curves within sampling noise
- Recomputed Fupfi accuracy on the same detected exons should match the unbiased behavior reported in Figure 2, confirming the detection filter did not distort quantification

## Limitations

- The maximum quantifiable exon length with short reads is limited by read length and paired-end fragment size, so exons longer than these are systematically undetectable by design
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon, so detection curves only capture the immediate flanking introns
- Only transcripts with Nexo>2 and one randomly selected middle exon are sampled, so terminal and single-exon transcripts are excluded from the detection assessment

## Evidence

- [other] Compute the percent of exons detected (N(upfi+dofi)>=1) stratified by exon and flanking intron lengths: [evidence span withheld — anchor status: not_found]
- [discussion] Read/fragment length limits detectable exon length: "the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads"
- [readme] Simulation parameters: nsim, seed, readlengths: "readlengths<-c(75,150)
fragsizes<-c(250,500)
nsim<-1000
nsamples<-20
seed<-20221107"
- [other] Validation criterion: detection and accuracy reproduce Figure 2: [evidence span withheld — anchor status: not_found]
