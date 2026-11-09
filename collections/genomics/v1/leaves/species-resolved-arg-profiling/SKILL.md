---
name: species-resolved-arg-profiling
description: Use when quality-controlled Nanopore/PacBio long reads need ARG identification
  and quantification at species level, e.g., linking ARGs to microbial hosts or comparing
  gut ARG loads across populations.
when_to_use_negative:
- 'Short-read metagenomes: Argo is built on long-read overlapping and will not work
  on second-generation reads (use a short-read profiler such as Lynx instead)'
- De novo discovery of ARG genomic contexts or hosts in new, understudied environments
  where assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk)
  approaches are better suited
- Applications needing ORF-level annotation or detailed sequence typing, since highly
  similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_0085
tools:
- name: Argo
  role: 'Core profiler: identifies ARGs via minimap2 alignment with adaptive identity
    cutoff, clusters read overlaps (MCL inflation=2, expansion=2), assigns species
    via set cover, reports cpg'
  repo: https://github.com/xinehc/argo
- name: SARG+
  role: Default curated ARG database (NDARO + CARD + SARG + RefSeq PGAP-evidenced
    homologs) embedded in Argo
  repo: https://github.com/xinehc/sarg-curation
- name: Melon
  role: Estimates genome copies from prokaryotic reads to normalize ARG abundance
    into cpg
  repo: https://github.com/xinehc/argo
- name: Kraken2
  role: PlusPF pre-filtering of non-prokaryotic reads before genome-copy estimation
- name: Porechop
  role: Adapter trimming of Nanopore reads (--discard_middle) as upstream QC
- name: nanoq
  role: Read quality/length filtering (--min-qual 10 --min-len 1000) as upstream QC
- name: argo-supplementary
  role: Instructions and pipeline for building the GTDB-based Argo database
  repo: https://github.com/xinehc/argo-supplementary
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:70a766afa09b89840065cb16b4079c82bdebf4b8da8508e2991eeff8f3d1910b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - Kraken2
  - Melon
  - Porechop
  - R
  - SARG+
  - argo-supplementary
  - ggplot2
  - nanoq
  - vegan
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# species-resolved-arg-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Species-resolved profiling of antibiotic resistance genes (ARGs) in long-read metagenomes by clustering read overlaps and assigning taxonomy per cluster with Argo, using the curated SARG+ database and returning ARG copies per genome (cpg) per species.

## When to use

Use when quality-controlled Nanopore/PacBio long reads need ARG identification and quantification at species level, e.g., linking ARGs to microbial hosts or comparing gut ARG loads across populations.

## When NOT to use

- Short-read metagenomes: Argo is built on long-read overlapping and will not work on second-generation reads (use a short-read profiler such as Lynx instead)
- De novo discovery of ARG genomic contexts or hosts in new, understudied environments where assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) approaches are better suited
- Applications needing ORF-level annotation or detailed sequence typing, since highly similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA)

## Inputs

- Quality-controlled long-read FASTA/FASTQ (Nanopore or PacBio; e.g., nanopore reads after Porechop and nanoq --min-qual 10 --min-len 1000)
- Argo database bundle (SARG+ reference/extension and ARG-tailored GTDB) from Zenodo record 20451539

## Outputs

- *.sarg.tsv table of predicted ARG types and subtypes, hosts (GTDB lineages), carriers (chromosome/plasmid), copies, genomes, and cpg abundance per sample
- *.sarg. with per-read annotation details (hits, plasmid flag, lineage) for ARG-containing reads
- Per-sample genome copy estimates (bacteria/archaea) usable for cpg normalization

## How to apply

Quality-control reads first (Porechop --discard_middle, then nanoq with --min-qual 10 --min-len 1000) since Argo requires QC'd long reads as input. Download the Argo database (SARG+ plus ARG-tailored GTDB) from Zenodo record 20451539 and index it with diamond makedb and minimap2 (map-ont). Run Argo per sample with the SARG+ database; it filters ARG-free reads, identifies ARGs with an adaptive identity cutoff (90 - 2.5 * 100 * median sequence divergence), assigns taxonomy to read clusters via graph clustering (MCL, inflation=2, expansion=2) followed by greedy weighted set cover, and estimates genome copies with Melon (after Kraken2 PlusPF pre-filtering of non-prokaryotic reads). Use --plasmid to split ARGs by carrier (chromosome vs plasmid) when host-tracking plasmid-borne ARGs, but interpret plasmid read assignments cautiously. Aggregate per-sample outputs into total ARG abundance (cpg) and subtype-by-species tables for downstream statistics (e.g., Kruskal-Wallis, PERMANOVA).

## Related tools

- **Argo** (Core profiler: identifies ARGs via minimap2 alignment with adaptive identity cutoff, clusters read overlaps (MCL inflation=2, expansion=2), assigns species via set cover, reports cpg) — https://github.com/xinehc/argo
- **SARG+** (Default curated ARG database (NDARO + CARD + SARG + RefSeq PGAP-evidenced homologs) embedded in Argo) — https://github.com/xinehc/sarg-curation
- **Melon** (Estimates genome copies from prokaryotic reads to normalize ARG abundance into cpg) — https://github.com/xinehc/argo
- **Kraken2** (PlusPF pre-filtering of non-prokaryotic reads before genome-copy estimation)
- **Porechop** (Adapter trimming of Nanopore reads (--discard_middle) as upstream QC)
- **nanoq** (Read quality/length filtering (--min-qual 10 --min-len 1000) as upstream QC)
- **argo-supplementary** (Instructions and pipeline for building the GTDB-based Argo database) — https://github.com/xinehc/argo-supplementary

## Examples

```
argo example.fa.gz -d database -o . --plasmid
```

## Evaluation signals

- Log reports distinct stages completing: genome copies found, taxonomy assignment, graph clustering (expected read-cluster count), and 'Set covering ... ARG copies per genome' with ARG type/subtype/species counts
- Output .sarg.tsv has columns lineage, type, subtype, carrier, copy, genome, abundance with plausible cpg values (genome copies typically around single digits per prokaryote, e.g., Staphylococcus aureus genome 4.125)
- Reproduced total-ARG cpg comparison of the 329-sample human/non-human primate fecal dataset yields Kruskal-Wallis chi-squared = 98.4, adjusted p < 2e-16, with a decreasing CN to NH trend and NH primates lowest
- E. coli sul2 in the CN group reaches 0.705 cpg and all sul/mcr genes are absent from NH primate samples
- Example run reproduces reference behavior: ~579 ARG-containing reads of 10,000, 8 unique species, 138 ARG subtypes, 24.478 cpg total

## Limitations

- Argo is inherently reference-based and may misclassify unseen species or novel plasmid syntenies
- Point-mutation-derived ARGs are excluded from the database because SNPs cannot be reliably distinguished from sequencing errors in read-based profiling
- By default, ARGs of species below 1x genome copy are reported as 'unclassified', making Argo less suitable for extremely rare species
- Insufficient coverage causes unstable normalization and hampers clustering/taxonomic classification; Argo performed consistently from 1x to 32x in simulation
- Plasmid read assignments can have multiple hosts; --plasmid splitting can be confounded by chimeric reads and uncharacterized plasmids

## Evidence

- [intro] By leveraging a dataset comprising 329 human and non-human primate fecal samples, we illustrate Argo's effectiveness in surveilling ARGs within complex environments: [evidence span withheld — anchor status: not_found]
