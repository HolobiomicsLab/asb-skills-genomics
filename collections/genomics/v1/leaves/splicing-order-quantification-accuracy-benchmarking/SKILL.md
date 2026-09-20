---
name: splicing-order-quantification-accuracy-benchmarking
description: Use when validating an AISO/Fupfi quantification tool with simulated
  short RNA-seq reads carrying known UPFI/DOFI copy numbers, or when assessing whether
  exon and flanking intron lengths bias splicing-order quantification.
when_to_use_negative:
- The goal is quantifying real biological AISO patterns from actual RNA-seq data rather
  than validating quantification accuracy with known ground truth.
- Long-read sequencing data is the target — the article's short-read simulation cannot
  emulate the 5'/3' long-read biases that Insplico corrects with its optional module.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Insplico
  role: 'Tool under test: computes upfi/dofi/bus/bos counts and Fupfi around the simulated
    exons of interest'
  repo: gitlab.com/aghr/insplico
- name: polyester
  role: R package used to simulate short reads from the synthetic UPFI/DOFI transcripts
    via simulate_experiment_countmat
- name: STAR
  role: Aligns simulated reads in --twopassMode Basic against hg38_reference for exon-detection
    and Fupfi accuracy assessment
- name: rtracklayer
  role: Imports the GTF annotation (import) and writes the simulation GTFs
- name: plyranges
  role: Filters exons by exon_number and removes/inserts merged exons when building
    UPFI/DOFI transcripts
- name: ggplot2
  role: Plots the detection curves and Fupfi accuracy plots reproducing Figure 2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:ad2d8515a9858fa0889376f264740dceacd936f1d0e5580b170c429b9ddee087
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# splicing-order-quantification-accuracy-benchmarking

## Summary

Benchmark the accuracy of Insplico's Fupfi/Fdofi quantification and exon-detection limits by simulating RNA-seq reads (polyester, 75/150-nt, SE and PE 250/500) from synthetic UPFI/DOFI transcript pairs with known copy numbers, then comparing STAR-aligned results against ground truth (Figure 2 of the Insplico paper).

## When to use

Use when validating an AISO/Fupfi quantification tool with simulated short RNA-seq reads carrying known UPFI/DOFI copy numbers, or when assessing whether exon and flanking intron lengths bias splicing-order quantification.

## When NOT to use

- The goal is quantifying real biological AISO patterns from actual RNA-seq data rather than validating quantification accuracy with known ground truth.
- Long-read sequencing data is the target — the article's short-read simulation cannot emulate the 5'/3' long-read biases that Insplico corrects with its optional module.

## Inputs

- hg38/GRCh38 chromosome FASTA files (chr1–chrY) in a 'chroms' folder
- Hsa38_only_transcript_support_level_1.gtf (Ensembl release-85 GTF, chr-prefixed, transcript_support_level 1)
- Transcript_Exon_Count.tab (transcript, gene name, gene ID, Nexo)
- Simulation parameters: nsim=1000, nsamples=20, seed=20221107, readlengths c(75,150), fragsizes c(250,500)

## Outputs

- simulations_{i}.gtf and simulations_{i}.fa per simulated sample (write_gff / seq_gtf)
- Ground-truth count matrices R_{readl}_countmat.txt per simulation and read length
- polyester-simulated FASTQ/FASTA reads (20 simulations x 4 library layouts)
- STAR BAM alignments and Figure 2-style detection curves and Fupfi accuracy plots (ggplot2)

## How to apply

Build 20 simulations (nsim=1000, seed=20221107): filter Hsa38_only_transcript_support_level_1.gtf transcripts with Nexo>2, draw one random middle exon per transcript, sample 1000 genes weighted by median selectable exons (Nexo-2), pick one transcript per gene weighted by exon count, and assign UPFI~runif(0,1), DOFI=1-UPFI, total copies~runif(10,20). Construct UPFI/DOFI 'transcripts' by merging the exon of interest with its upstream or downstream intron-plus-exon, sort exons strand-aware (descending start on minus strand), write simulations_{i}.gtf with write_gff, and extract sequences with seq_gtf. Simulate reads with polyester simulate_experiment_countmat using expected counts round((transcript_length/read_length)*copies) for 75- and 150-nt reads, SE plus PE with fraglen 250/500, fragsd 25, error model illumina5 (uniform if readlen>102), error_rate=0.005, gzip=TRUE, seed=seed+i. Align all 20x4 libraries with STAR --twopassMode Basic against hg38_reference (chimSegmentMin 20, alignSJoverhangMin 8, alignIntronMin 20, alignIntronMax 200000, outFilterMultimapNmax 100), then score exons as detected when N(upfi+dofi)>=1 and compare Fupfi estimates to simulated UPFI/DOFI copies stratified by exon and flanking intron lengths.

## Related tools

- **Insplico** (Tool under test: computes upfi/dofi/bus/bos counts and Fupfi around the simulated exons of interest) — gitlab.com/aghr/insplico
- **polyester** (R package used to simulate short reads from the synthetic UPFI/DOFI transcripts via simulate_experiment_countmat)
- **STAR** (Aligns simulated reads in --twopassMode Basic against hg38_reference for exon-detection and Fupfi accuracy assessment)
- **rtracklayer** (Imports the GTF annotation (import) and writes the simulation GTFs)
- **plyranges** (Filters exons by exon_number and removes/inserts merged exons when building UPFI/DOFI transcripts)
- **ggplot2** (Plots the detection curves and Fupfi accuracy plots reproducing Figure 2)

## Examples

```
numexo_rand<-read.table("Transcript_Exon_Count.tab", col.names=c("Transcript","GeneName","GeneID","Nexo")) %>% filter(Nexo>2) %>% rowwise() %>% mutate(RandomExo=round(runif(1,min=1.5,max=Nexo-0.5),0))
```

## Evaluation signals

- Fupfi quantification is unbiased: measured Fupfi distributions match simulated ground-truth UPFI/DOFI values regardless of exon, upstream intron, or downstream intron length.
- Exons are scored as detected when at least one upfi or dofi read is observed (N(upfi+dofi)>=1); detection curves stratified by exon and flanking intron lengths reproduce Figure 2 short-read behavior.
- Each simulation's count matrix R_{readl}_countmat.txt equals round((transcript_length/read_length)*copies) for 75- and 150-nt reads.
- 20 simulations x 4 library layouts (75/150 nt SE, 250/500 bp PE) complete STAR alignment without failure; strand-sorted UPFI/DOFI GTFs load cleanly with rtracklayer.
- Expected copies of each UPFI/DOFI transcript lie in the simulated range (UPFI+DOFI between 10 and 20 total copies).

## Limitations

- Read length and paired-end fragment size limit the maximum exon length quantifiable by Insplico with short reads.
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon, unlike long reads.
- Simulation building aborts per transcript if strand is not consistent (if(length(unique(strand(res)))!=1){break}), so mixed-strand entries drop out.
- Software versions for R packages are not stated in the methods.

## Evidence

- [methods] mutate(UPFI=round(runif(nsim),1), DOFI=1-UPFI: "mutate(UPFI=round(runif(nsim),1), DOFI=1-UPFI,
                             Ncopies=round(runif(nsim,min = 10.5, max=20.5),0)"
- [methods] as_tibble()  %>% filter(Nexo>2): "as_tibble()  %>% filter(Nexo>2) %>
      rowwise() %>% mutate(RandomExo=round(runif(1, min = 1.5, max=Nexo-0.5),0))"
- [readme] random selected exons are substituted: "Random selected exons are substituted by new exons, which are defined by merging
    # the up and down stream intron + exon of the exon of interest, emulating UPFI and DOFI"
- [readme] select one random middle exon: "# Select one random middle exon from transcripts with more than 2 exons."
- [readme] Initial parameters: "readlengths<-c(75,150)
fragsizes<-c(250,500)
nsim<-1000
nsamples<-20
seed<-20221107"
- [discussion] short read length limitation: "the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads"
