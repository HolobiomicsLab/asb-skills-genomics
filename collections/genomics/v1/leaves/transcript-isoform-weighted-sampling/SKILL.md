---
name: transcript-isoform-weighted-sampling
description: Use when building simulated RNA-seq transcriptomes where the splicing
  order of adjacent introns (UPFI vs DOFI) must vary per transcript while gene/transcript
  selection reflects transcript exon-count complexity, to validate a splicing-order
  quantification tool like Insplico.
when_to_use_negative:
- The goal is quantifying AISO from real RNA-seq alignments rather than generating
  simulated ground-truth transcript copies for validation.
- The transcriptome of interest includes transcripts with 2 or fewer exons, since
  middle exons (with both flanking introns) cannot be selected (Nexo>2 filter).
- A non-uniform, biology-driven UPFI/DOFI distribution is required — this skill deliberately
  assigns a uniform runif split, which does not model skewed AISO.
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
tools:
- name: Insplico
  role: AISO quantification tool whose UPFI/DOFI recovery is validated against the
    simulated isoform copies
  repo: gitlab.com/aghr/insplico
- name: R
  role: environment for the weighted sampling and isoform construction workflow
- name: rtracklayer
  role: imports the transcript-support-level-1 GTF (import(gtf_file)) for exon filtering
- name: plyranges
  role: filters exons by exon_number (random, upstream, downstream) and filter_by_non_overlaps
    when building UPFI/DOFI exons
- name: dplyr
  role: group_by/slice_sample weighted gene and transcript sampling and UPFI/DOFI
    copy-number assignment
- name: polyester
  role: simulates short reads from the constructed UPFI/DOFI transcript FASTA/GTF
- name: STAR
  role: aligns the simulated reads (twopassMode Basic, chimSegmentMin 20, alignSJoverhangMin
    8) for Insplico quantification
- name: seq_gtf
  role: extracts transcript sequences from simulations_<i>.gtf against the chroms/
    FASTA folder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:a580ca19575065d7c9124ae87457ba04e3bb7f717fdaaeeeb34de9707bdc134d
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
  - seq_gtf
  - tibble
  - tidyr
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5238095238095238
n_claims: 21
---

# transcript-isoform-weighted-sampling

## Summary

Simulate AISO-aware transcript isoforms by sampling genes and transcripts with weights derived from exon counts, then splitting each transcript's reads between UPFI and DOFI variants with a uniform runif fraction and 10-20 total copies, as done in the Insplico simulation pipeline (polyester reads from a transcript-support-level-1 GTF).

## When to use

Use when building simulated RNA-seq transcriptomes where the splicing order of adjacent introns (UPFI vs DOFI) must vary per transcript while gene/transcript selection reflects transcript exon-count complexity, to validate a splicing-order quantification tool like Insplico.

## When NOT to use

- The goal is quantifying AISO from real RNA-seq alignments rather than generating simulated ground-truth transcript copies for validation.
- The transcriptome of interest includes transcripts with 2 or fewer exons, since middle exons (with both flanking introns) cannot be selected (Nexo>2 filter).
- A non-uniform, biology-driven UPFI/DOFI distribution is required — this skill deliberately assigns a uniform runif split, which does not model skewed AISO.

## Inputs

- Hsa38_only_transcript_support_level_1.gtf (GTF annotation restricted to transcript_support_level 1)
- Transcript_Exon_Count.tab (tab-delimited Transcript/GeneName/GeneID/Nexo table built with grep/awk from the GTF)
- Per-chromosome FASTA files decompressed into a chroms/ folder (chr1.fa-chrY.fa from GRCh38.p7 Primary_Assembly)
- Simulation parameters: readlengths c(75,150), fragsizes c(250,500), nsim=1000, nsamples=20, seed=20221107

## Outputs

- numexo_rand_subset table with per-transcript UPFI, DOFI, Ncopies, Ncopies_UPFI, Ncopies_DOFI and exon/intron lengths
- per-sample GTF of UPFI and DOFI transcripts (simulations_<i>.gtf) written with write_gff
- per-sample transcript FASTA (simulations_<i>.fa) extracted with seq_gtf for polyester read simulation

## How to apply

From a transcript-support-level-1 GTF (e.g. Hsa38_only_transcript_support_level_1.gtf imported with rtracklayer) and a Transcript_Exon_Count.tab table, first filter transcripts with Nexo>2 and pick one random middle exon per transcript via RandomExo=round(runif(1,1.5,Nexo-0.5)). For each of 20 simulated samples (seed 20221107+i), sample 1000 genes weighted by the median number of selectable exons (medianExo/sum(medianExo)), then select one transcript per gene weighted by (Nexo-2)/sum(Nexo-2) so exon-rich transcripts are more likely. Assign UPFI=round(runif(nsim),1) with DOFI=1-UPFI (a uniform AISO split) and Ncopies=round(runif(nsim,10.5,20.5)) with Ncopies_UPFI/Ncopies_DOFI proportional to the split. Build UPFI and DOFI transcripts by merging the selected exon with the downstream or upstream intron-plus-exon respectively, removing the original exons and sorting by strand, then write simulations_<i>.gtf and extract sequences with seq_gtf from the chroms/ FASTA folder for downstream polyester read simulation (read lengths 75/150, fragment sizes 250/500).

## Related tools

- **Insplico** (AISO quantification tool whose UPFI/DOFI recovery is validated against the simulated isoform copies) — gitlab.com/aghr/insplico
- **R** (environment for the weighted sampling and isoform construction workflow)
- **rtracklayer** (imports the transcript-support-level-1 GTF (import(gtf_file)) for exon filtering)
- **plyranges** (filters exons by exon_number (random, upstream, downstream) and filter_by_non_overlaps when building UPFI/DOFI exons)
- **dplyr** (group_by/slice_sample weighted gene and transcript sampling and UPFI/DOFI copy-number assignment)
- **polyester** (simulates short reads from the constructed UPFI/DOFI transcript FASTA/GTF)
- **STAR** (aligns the simulated reads (twopassMode Basic, chimSegmentMin 20, alignSJoverhangMin 8) for Insplico quantification)
- **seq_gtf** (extracts transcript sequences from simulations_<i>.gtf against the chroms/ FASTA folder)

## Examples

```
set.seed(seed+i); numexo_rand_subset <- numexo_rand %>% filter(Nexo>2) %>% mutate(UPFI=round(runif(nsim),1), DOFI=1-UPFI, Ncopies=round(runif(nsim,min=10.5,max=20.5),0), Ncopies_UPFI=round(UPFI*Ncopies,0), Ncopies_DOFI=round(DOFI*Ncopies,0))
```

## Evaluation signals

- Insplico-recovered UPFI/DOFI read fractions per transcript must reflect the simulated Ncopies_UPFI vs Ncopies_DOFI split (validation criterion stated in the task card).
- Per-transcript Ncopies must fall within 10-20 (round(runif(nsim,min=10.5,max=20.5))) and Ncopies_UPFI + Ncopies_DOFI must equal Ncopies.
- UPFI fraction distribution across the 1000 simulated transcripts per sample must be uniform (runif split), reproducible under set.seed(20221107+i).
- Constructed UPFI/DOFI transcripts must be strand-sorted: ascending start on '+' strand, descending on '-' strand, after inserting the merged exon-intron-exon.
- Only transcripts with Nexo>2 and exon_number strictly between 1 and Nexo (a middle exon with both flanking introns) may be selected.

## Limitations

- Maximum quantifiable exon length with short reads is limited by read length and paired-end fragment size, so very long simulated exons are not informative.
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon.
- Software version numbers for the R packages and Insplico are not stated in the methods/README, hampering exact reproducibility.

## Evidence

- [readme] Weighted gene sampling by exon count: "genes are weighted based on the median number of possible selected 
    # exons in their transcripts"
- [readme] UPFI/DOFI copy assignment: "mutate(UPFI=round(runif(nsim),1), DOFI=1-UPFI,
                             Ncopies=round(runif(nsim,min = 10.5, max=20.5),0),"
- [readme] Nexo filter: "as_tibble()  %>% filter(Nexo>2) %>"
- [readme] Isoform construction rationale: "Random selected exons are substituted by new exons, which are defined by merging
    # the up and down stream intron + exon of the exon of interest, emulating UPFI and DOFI"
- [abstract] Insplico purpose: "we present Insplico, the first standalone software for quantifying AISO that works with both short and long read sequencing technologies"
- [discussion] Validation approach: "demonstrated its effectiveness using simulated reads and by comparing it with two published studies of AISO"
