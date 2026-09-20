---
name: arg-containing-read-extraction-annotation
description: Use when you have quality-controlled long reads (Nanopore/PacBio FASTQ)
  from complex metagenomes and need to subset and annotate reads carrying antibiotic
  resistance genes before species-level host assignment.
when_to_use_negative:
- Input is already an annotated ARG table or assembled contigs with gene calls rather
  than raw long reads
- The goal is de novo identification of novel ARGs or point-mutation-derived ARGs
  — SARG+ excludes ARGs arising from point mutations because SNPs cannot be distinguished
  from sequencing errors in read-based profiling
- Short-read data requiring per-read species typing of highly similar ARG alleles,
  where the SARG+ clustering of near-identical genes (e.g., blaOXA-1/blaOXA-1042 as
  blaOXA) lowers resolution
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0797
tools:
- name: DIAMOND
  role: Frameshift-aware blastx alignment of long reads against SARG+ to extract ARG-containing
    reads (--evalue 1e-15 --range-culling --frameshift 15)
- name: SARG+
  role: Curated ARG reference/extension database used as the alignment target; built
    from NDARO, CARD, SARG, and evidence-matched RefSeq proteins
  repo: https://github.com/xinehc/sarg-curation
- name: Argo
  role: Downstream profiler that consumes the ARG annotations and assigns species
    hosts via read-overlapping clusters and set cover
  repo: https://github.com/xinehc/argo
- name: minimap2
  role: Provides base-level alignment (-x map-ont) of ARG-containing reads against
    GTDB for raw species assignments consolidated by Argo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:4ac5ea1175b5d5840fcaab9e1166d7bedbedc7d045c289b6c3560d888f88884a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/sarg-curation
  tools:
  - Argo
  - Centrifuger
  - DIAMOND
  - Kraken2
  - MEGAN-LR
  - MetaMaps
  - NanoSim
  - SARG+
  - minimap2
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ARG-containing read extraction and annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Isolates ARG-carrying long reads from metagenomic read sets using DIAMOND frameshift-aware blastx against the SARG+ database, so that downstream host-identification classifiers operate only on resistance-relevant reads. This filtering step underpins species-resolved ARG profiling with Argo and its benchmark comparisons.

## When to use

Use when you have quality-controlled long reads (Nanopore/PacBio FASTQ) from complex metagenomes and need to subset and annotate reads carrying antibiotic resistance genes before species-level host assignment.

## When NOT to use

- Input is already an annotated ARG table or assembled contigs with gene calls rather than raw long reads
- The goal is de novo identification of novel ARGs or point-mutation-derived ARGs — SARG+ excludes ARGs arising from point mutations because SNPs cannot be distinguished from sequencing errors in read-based profiling
- Short-read data requiring per-read species typing of highly similar ARG alleles, where the SARG+ clustering of near-identical genes (e.g., blaOXA-1/blaOXA-1042 as blaOXA) lowers resolution

## Inputs

- Quality-controlled long reads (ONT or PacBio) in FASTQ from complex metagenomes
- SARG+ protein database (sarg.fa combining sarg_ref.fa and sarg_ext.fa)
- DIAMOND database built from sarg.fa (diamond makedb --in database/sarg.fa --db database/sarg)

## Outputs

- Subset of ARG-containing reads passed to taxonomic/host classifiers
- ARG annotation table with predicted ARG types and subtypes per read (consumed by Argo for host and abundance calling)
- Species-level ARG profile with estimated abundances in ARG copies per genome (cpg)

## How to apply

Run DIAMOND in frameshift-aware blastx mode against the curated SARG+ database using the parameter set --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0; frameshift awareness is essential because raw ONT reads carry indel errors that would otherwise break gene alignment. The reported hits define which reads are ARG-containing; SARG+ extends experimentally validated sequences (NDARO, CARD, SARG) with RefSeq proteins sharing the same PGAP annotation evidence (BlastRules or HMMs), enabling stringent cutoffs while preserving sensitivity. The extracted ARG read subset then feeds host-identification classifiers (e.g., minimap2-based read overlapping in Argo, Kraken2, Centrifuger) or Argo's cluster-based taxonomic consolidation. Correct application yields a per-read annotation table of ARG types/subtypes that reproduces known ARG content on mock communities.

## Related tools

- **DIAMOND** (Frameshift-aware blastx alignment of long reads against SARG+ to extract ARG-containing reads (--evalue 1e-15 --range-culling --frameshift 15))
- **SARG+** (Curated ARG reference/extension database used as the alignment target; built from NDARO, CARD, SARG, and evidence-matched RefSeq proteins) — https://github.com/xinehc/sarg-curation
- **Argo** (Downstream profiler that consumes the ARG annotations and assigns species hosts via read-overlapping clusters and set cover) — https://github.com/xinehc/argo
- **minimap2** (Provides base-level alignment (-x map-ont) of ARG-containing reads against GTDB for raw species assignments consolidated by Argo)

## Examples

```
diamond blastx --db database/sarg --query reads.fq --out arg_hits.tsv --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0
```

## Evaluation signals

- DIAMOND hits meet the article's thresholds: e-value ≤ 1e-15 with range culling, frameshift 15, range cover 25
- On simulated mock communities, extracted ARG reads classified with minimap2+RO yield overall misclassification rates of ~0.19% (HQ) and ~0.09% (LQ)
- SARG+ outputs (sarg.fa, sarg_ref.fa, sarg_ext.fa) pass the notebook pipeline's duplicate/cross-mapping clustering check (n4-remove-dups.ipynb)
- Argo reports ARGs only for species with ≥1× genome copy; species below threshold are labeled 'unclassified' rather than misassigned
- Resulting ARG tables list predicted types/subtypes, hosts, and cpg abundances consistent with known mock-community composition

## Limitations

- Read-based profiling cannot reliably distinguish SNPs from sequencing errors, so ARGs originating from point mutations are excluded from the SARG+ database
- Highly similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 as blaOXA), lowering resolution for ORF-based annotation or detailed sequence typing
- Argo/SARG+ is reference-based and may misclassify unseen species or novel plasmid syntenies; spurious alignments from local sequence homology can inflate false-positive ARG predictions without the integration step

## Evidence

- [other] DIAMOND frameshift-aware blastx against SARG+: [evidence span withheld — anchor status: not_found]
- [readme] SARG+ composition: [evidence span withheld — anchor status: not_found]
- [readme] SARG+ design rationale: [evidence span withheld — anchor status: external_doc]
