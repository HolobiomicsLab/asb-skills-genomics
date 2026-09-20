---
name: cross-species-aiso-consensus-analysis
description: Use when short-read RNA-seq datasets plus genome FASTA and GTF annotations
  are available for multiple species and the goal is to identify intron-exon features
  (splice-site strength, intron length, branch point-to-AG distance) universally associated
  with AISO patterns.
when_to_use_negative:
- The goal is splicing-order information beyond the introns directly flanking the
  input exon — short reads can barely provide this; long-read data with Insplico's
  bias-correction module would be needed
- Studying cell-type- or condition-specific changes in AISO within a species — the
  article reports AISO around individual exons is remarkably constant across cell
  and tissue types
- Exons exceeding the length quantifiable by short reads, which is limited by read
  length and paired-end fragment size
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3512
tools:
- name: Insplico
  role: Standalone software to quantify adjacent-intron splicing order (upfi/dofi
    counts, Fupfi) around user-specified exons from short-read RNA-seq per species
  repo: gitlab.com/aghr/insplico
- name: Matt
  role: cmpr_features extracts 55 intron-exon features (splice-site strengths, intron/exon
    lengths, branch point-to-AG distances) around each exon
- name: dplyr
  role: Bin exons into five equal subsets per feature and compute mean Fupfi per bin
- name: tidyr
  role: Reshape per-exon feature and Fupfi tables for binned summarization
- name: ggplot2
  role: Plot mean Fupfi per bin for each of the 55 features, aggregated across six
    species
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:f6d912295ff4c3a3805974b0ad4274bf15c3c72ec88fe7514876021071e4d03d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - Matt
  - R
  - dplyr
  - ggplot2
  - tidyr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cross-species-aiso-consensus-analysis

## Summary

Reproduce the universal intron-exon genomic features associated with adjacent-intron splicing order (AISO) across animal and plant species by quantifying per-exon Fupfi with Insplico, extracting 55 intron-exon features with Matt, and aggregating binned feature-Fupfi relationships across species to derive consensus associations.

## When to use

Use when short-read RNA-seq datasets plus genome FASTA and GTF annotations are available for multiple species and the goal is to identify intron-exon features (splice-site strength, intron length, branch point-to-AG distance) universally associated with AISO patterns.

## When NOT to use

- The goal is splicing-order information beyond the introns directly flanking the input exon — short reads can barely provide this; long-read data with Insplico's bias-correction module would be needed
- Studying cell-type- or condition-specific changes in AISO within a species — the article reports AISO around individual exons is remarkably constant across cell and tissue types
- Exons exceeding the length quantifiable by short reads, which is limited by read length and paired-end fragment size

## Inputs

- Short-read nuclear/chromatin RNA-seq datasets (e.g. GEO accession GSE207459)
- Species-specific genome FASTA files (chromosome FASTA, chr1-chrY)
- Species-specific GTF annotations (e.g. Hsa38_only_transcript_support_level_1.gtf)
- User-specified exons of interest

## Outputs

- Per-exon upfi/dofi counts and Fupfi (fraction of transcripts with upstream intron spliced first) from Insplico
- 55 intron-exon features per exon (splice-site strengths, intron/exon lengths, branch point-to-AG distances) from Matt cmpr_features
- Binned mean-Fupfi-per-feature plots (five equal bins per feature) per species and aggregated across species
- Consensus set of universal AISO-associated features across animal and plant species

## How to apply

Obtain short-read nuclear/chromatin RNA-seq datasets (e.g. GEO GSE207459) together with species-specific genome FASTA files and GTF annotations for human, mouse, fruitfly, round worm, Arabidopsis, and rice. Run Insplico on each dataset to quantify adjacent-intron splicing order around user-specified exons, yielding per-exon upfi/dofi counts and the fraction of transcripts with the upstream intron spliced first (Fupfi). Extract 55 intron-exon features around each exon with Matt's cmpr_features, covering splice-site strengths, intron/exon lengths, and branch point-to-AG distances. Bin exons into five equal subsets per feature and compute mean Fupfi per bin in R (dplyr, tidyr), then plot mean Fupfi per bin for each of the 55 features with ggplot2, aggregating across the six species to derive consensus associations.

## Related tools

- **Insplico** (Standalone software to quantify adjacent-intron splicing order (upfi/dofi counts, Fupfi) around user-specified exons from short-read RNA-seq per species) — gitlab.com/aghr/insplico
- **Matt** (cmpr_features extracts 55 intron-exon features (splice-site strengths, intron/exon lengths, branch point-to-AG distances) around each exon)
- **dplyr** (Bin exons into five equal subsets per feature and compute mean Fupfi per bin)
- **tidyr** (Reshape per-exon feature and Fupfi tables for binned summarization)
- **ggplot2** (Plot mean Fupfi per bin for each of the 55 features, aggregated across six species)

## Examples

```
my_obj <- import(gtf_file); numexo_rand <- read.table("Transcript_Exon_Count.tab", col.names = c("Transcript","GeneName","GeneID","Nexo")) %>% as_tibble() %>% filter(Nexo>2)
```

## Evaluation signals

- Mean Fupfi rises with upstream 5' and 3' splice-site strength in the binned plots
- Mean Fupfi falls with upstream intron length and rises with downstream intron length
- Mean Fupfi varies with branch point-to-AG (3' splice site) distance, matching the universal AISO feature patterns reported across animal and plant species
- Consensus binned patterns agree qualitatively across all six species (human, mouse, fruitfly, round worm, Arabidopsis, rice)
- Upfi exons show well-defined upstream introns with branch points close to their 3' splice site (AG)

## Limitations

- Read length and paired-end fragment size limit the maximum exon length quantifiable with short reads
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon
- Full mechanistic understanding of the universal AISO code would require integrating mathematical modelling of co-transcriptional splicing, RNP binding, chromatin features, and spliceosomal kinetics, which has not yet been done

## Evidence

- [abstract] A set of universal features associated with AISO patterns was established across various animal and plant species: "We also establish a set of universal features associated with AISO patterns across various animal and plant species"
- [discussion] Insplico measures pre-mRNA processing metrics around user-specified exons: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts) and estimates the fraction of transcripts"
- [readme] Download chromosome FASTA files and prepare chroms folder: "wget ${FAPATH}chr5.fna.gz && gunzip chr5.fna.gz && mv chr5.fna chroms/chr5.fa"
- [methods] Filter transcripts to those with more than 2 exons before exon selection: "as_tibble()  %>% filter(Nexo>2) %>"
- [discussion] Short reads cannot provide splicing-order information beyond flanking introns: "short reads can barely provide splicing order information beyond them as long reads can do"
