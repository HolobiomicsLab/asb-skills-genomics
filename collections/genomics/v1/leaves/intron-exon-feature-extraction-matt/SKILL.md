---
name: intron-exon-feature-extraction-matt
description: Use when you have per-exon AISO quantification (upfi/dofi counts or Fupfi)
  from Insplico plus genome FASTA/GTF annotations and need to identify which genomic
  features correlate with splicing order.
when_to_use_negative:
- If you only need AISO quantification itself without relating it to genomic features
  — use Insplico alone
- If the goal is splicing-order information beyond the introns directly flanking the
  input exon — short reads barely provide such information
- If exons exceed the length quantifiable by Insplico with short reads, which is limited
  by read length (75/150 nt) and paired-end fragment size (250/500 bp)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Insplico
  role: Quantifies adjacent-intron splicing order (AISO) around user-specified exons,
    producing upfi/dofi counts and Fupfi per exon from short or long RNA-seq reads
  repo: gitlab.com/aghr/insplico
- name: Matt
  role: cmpr_features extracts the 55 intron-exon features around each exon (splice-site
    strengths, intron/exon lengths, branch point-to-AG distances)
- name: dplyr
  role: Binning exons into five equal subsets per feature and computing mean Fupfi
    per bin in R
- name: tidyr
  role: Reshaping the binned feature-Fupfi data for plotting
- name: ggplot2
  role: Plotting mean Fupfi per bin for each of the 55 features across the six species
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:2fb7f1b5b842826978b55e2dd2f4b5a896b63c7194692dacfec5c98f77e67f1d
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# intron-exon-feature-extraction-matt

## Summary

Extracts 55 intron-exon features (splice-site strengths, intron/exon lengths, branch point-to-AG distances) around user-specified exons using Matt's cmpr_features, then relates them to Insplico-derived Fupfi values by binning exons into five equal subsets per feature and plotting mean Fupfi per bin across species.

## When to use

Use when you have per-exon AISO quantification (upfi/dofi counts or Fupfi) from Insplico plus genome FASTA/GTF annotations and need to identify which genomic features correlate with splicing order.

## When NOT to use

- If you only need AISO quantification itself without relating it to genomic features — use Insplico alone
- If the goal is splicing-order information beyond the introns directly flanking the input exon — short reads barely provide such information
- If exons exceed the length quantifiable by Insplico with short reads, which is limited by read length (75/150 nt) and paired-end fragment size (250/500 bp)

## Inputs

- Per-exon upfi/dofi counts and Fupfi values from Insplico on short-read RNA-seq data
- Species-specific genome FASTA files (e.g. chromosome FASTA files chr1-chrY in a chroms folder)
- GTF gene annotation files (e.g. Hsa38_only_transcript_support_level_1.gtf)
- User-specified exons of interest

## Outputs

- Feature table of 55 intron-exon features per exon (splice-site strengths, intron/exon lengths, branch point-to-AG distance)
- Binned mean-Fupfi-per-bin values for each of the 55 features (five equal bins per feature)
- ggplot2 plots of mean Fupfi per feature bin, aggregated across six species to show consensus AISO-feature associations

## How to apply

Run Insplico on short-read RNA-seq datasets (e.g. GEO GSE207459) to obtain per-exon upfi/dofi counts and the fraction of transcripts with the upstream intron spliced first (Fupfi). Extract the 55 intron-exon features around each exon with Matt's cmpr_features tool, covering upstream/downstream 5' and 3' splice-site strengths, intron and exon lengths, and branch point-to-3'ss(AG) distances, using the species-specific genome FASTA and GTF annotation. In R (dplyr, tidyr), bin exons into five equal subsets per feature and compute the mean Fupfi per bin. Plot mean Fupfi per bin for each feature with ggplot2, and aggregate the binned patterns across species (human, mouse, fruitfly, round worm, Arabidopsis, rice) to derive consensus associations.

## Related tools

- **Insplico** (Quantifies adjacent-intron splicing order (AISO) around user-specified exons, producing upfi/dofi counts and Fupfi per exon from short or long RNA-seq reads) — gitlab.com/aghr/insplico
- **Matt** (cmpr_features extracts the 55 intron-exon features around each exon (splice-site strengths, intron/exon lengths, branch point-to-AG distances))
- **dplyr** (Binning exons into five equal subsets per feature and computing mean Fupfi per bin in R)
- **tidyr** (Reshaping the binned feature-Fupfi data for plotting)
- **ggplot2** (Plotting mean Fupfi per bin for each of the 55 features across the six species)

## Examples

```
gtf_file<-"Hsa38_only_transcript_support_level_1.gtf"; my_obj <- import(gtf_file); numexo_rand<-read.table("Transcript_Exon_Count.tab", col.names=c("Transcript","GeneName","GeneID","Nexo")) %>% as_tibble() %>% filter(Nexo>2)
```

## Evaluation signals

- Consensus binned plots show mean Fupfi rising with upstream 5' and 3' splice-site strength
- Mean Fupfi falls with increasing upstream intron length and rises with increasing downstream intron length
- Mean Fupfi varies with branch point-to-AG distance, with branch points close to the 3' splice site (AG) characterizing upfi exons
- The consensus patterns reproduce across all six species (human, mouse, fruitfly, round worm, Arabidopsis, rice), confirming the universal AISO feature associations
- Feature extraction covers all 55 intron-exon features per exon with no missing values for well-annotated exons

## Limitations

- Short reads limit the maximum length of exons quantifiable by Insplico (bounded by read length and paired-end fragment size)
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon, unlike long reads
- Long-read AISO estimates require correcting 5' and 3' biases in sequencing and library preparation to harmonize with short-read results
- Full mechanistic understanding of the universal AISO code would require integrating mathematical modelling of co-transcriptional splicing, RBP binding, chromatin features and spliceosomal kinetics, which is not yet done

## Evidence

- [other] 55 intron-exon features around each exon with Matt's cmpr_features, covering splice-site strengths, intron/exon lengths, and branch point-to-AG distances: [evidence span withheld — anchor status: not_found]
- [other] Bin exons into five equal subsets per feature and compute mean Fupfi per bin: [evidence span withheld — anchor status: not_found]
- [abstract] establish a set of universal features associated with AISO patterns across various animal and plant species: "We also establish a set of universal features associated with AISO patterns across various animal and plant species"
- [discussion] key measurements to investigate pre-mRNA processing around user-specified exons (upfi, dofi, bus and bos counts) and estimates the fraction of transcripts: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts) and estimates the fraction of transcripts"
- [discussion] the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads: "the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads"
- [readme] Readlength and fragment-size simulation parameters from the README: "readlengths<-c(75,150)
fragsizes<-c(250,500)
nsim<-1000
nsamples<-20"
