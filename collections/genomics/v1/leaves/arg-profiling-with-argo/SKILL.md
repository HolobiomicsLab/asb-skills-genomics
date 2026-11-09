---
name: arg-profiling-with-argo
description: Use when profiling ARGs at species resolution in quality-controlled Nanopore/PacBio
  long-read metagenomes (FASTA/FASTQ), especially when linking ARGs to microbial hosts
  and distinguishing chromosomal vs plasmid carriers is required.
when_to_use_negative:
- Input reads are short (second-generation) reads — Argo relies on long-read overlapping
  and requires Nanopore or PacBio data
- Goal is de novo identification of ARG genomic contexts and hosts in new, understudied
  environments — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 +
  GTDB-Tk) methods may be better suited
- Application requires ORF-based annotation or detailed sequence typing of highly
  similar ARGs (e.g., distinguishing blaOXA-1 from blaOXA-1042), which Argo's clustered
  subtypes cannot resolve
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_3697
tools:
- name: Argo
  role: 'Core long-read ARG profiler: overlaps reads, clusters them, assigns species-level
    lineage via set covering, and quantifies ARGs in cpg'
  repo: github.com/xinehc/argo
- name: SARG+
  role: Manually curated ARG database (reference + RefSeq-derived extension) used
    as Argo's default '-d database'
  repo: github.com/xinehc/sarg-curation
- name: Porechop
  role: Adapter trimming of Nanopore reads with '--discard_middle' during quality
    control
- name: nanoq
  role: Read filtering with '--min-qual 10 --min-len 1000' during quality control
- name: DIAMOND
  role: Indexes SARG+ ('diamond makedb') and computes expected ARG abundances via
    blastx of reference genomes with stringent HSP filters
- name: minimap2
  role: Indexing of database FASTA files with '-x map-ont -d' for ONT read alignment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:eade46c93a31819393c0293a4ea014d3fe7b17c1e1f485cbd6f43cc34bed7fac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  tools:
  - Argo
  - DIAMOND
  - Porechop
  - SARG+
  - minimap2
  - nanoq
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# arg-profiling-with-argo

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Species-resolved profiling of antibiotic resistance genes (ARGs) in long-read metagenomes using Argo, which assigns taxonomy collectively to read clusters via long-read overlapping and set covering, producing ARG types/subtypes, carriers, and abundances as copies per genome (cpg).

## When to use

Use when profiling ARGs at species resolution in quality-controlled Nanopore/PacBio long-read metagenomes (FASTA/FASTQ), especially when linking ARGs to microbial hosts and distinguishing chromosomal vs plasmid carriers is required.

## When NOT to use

- Input reads are short (second-generation) reads — Argo relies on long-read overlapping and requires Nanopore or PacBio data
- Goal is de novo identification of ARG genomic contexts and hosts in new, understudied environments — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods may be better suited
- Application requires ORF-based annotation or detailed sequence typing of highly similar ARGs (e.g., distinguishing blaOXA-1 from blaOXA-1042), which Argo's clustered subtypes cannot resolve

## Inputs

- Quality-controlled long reads (Nanopore or PacBio) in FASTA/FASTQ, gzip optional
- SARG+ database (sarg.fa, prot.fa) from Zenodo with DIAMOND and minimap2 (.mmi) indices
- Optional ground truth: supplier reference genomes for expected ARG abundance computation

## Outputs

- Table (*.sarg.tsv) listing predicted ARG types and subtypes, potential host lineages, carrier (chromosome/plasmid), copies, genome copies, and abundance in cpg per species
- JSON file (*.sarg.) with per-read annotation details: hit, plasmid flag, and lineage for ARG-containing reads

## How to apply

Quality-control raw long reads with Porechop ('--discard_middle') and nanoq ('--min-qual 10 --min-len 1000'), removing yeast-mapping and unmapped reads for mock samples. Download SARG+ from Zenodo and index it with 'diamond makedb' and minimap2 ('-x map-ont -d'). Run Argo with '-d database --plasmid', keeping the default adaptive identity cutoff of 90 - 2.5 * 100 * median sequence divergence inferred from read overlaps (or pass a fixed cutoff, e.g. '-i 80 -s 80'). To validate against known truth, compute expected ARG abundances by aligning supplier reference genomes to SARG+ with DIAMOND blastx ('--evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0'), filter HSPs, and count ARG occurrences; then compare estimated vs expected profiles at type and subtype levels with precision, recall, F1-score, and Pearson correlation of cpg values.

## Related tools

- **Argo** (Core long-read ARG profiler: overlaps reads, clusters them, assigns species-level lineage via set covering, and quantifies ARGs in cpg) — github.com/xinehc/argo
- **SARG+** (Manually curated ARG database (reference + RefSeq-derived extension) used as Argo's default '-d database') — github.com/xinehc/sarg-curation
- **Porechop** (Adapter trimming of Nanopore reads with '--discard_middle' during quality control)
- **nanoq** (Read filtering with '--min-qual 10 --min-len 1000' during quality control)
- **DIAMOND** (Indexes SARG+ ('diamond makedb') and computes expected ARG abundances via blastx of reference genomes with stringent HSP filters)
- **minimap2** (Indexing of database FASTA files with '-x map-ont -d' for ONT read alignment)

## Examples

```
argo *.fa -d database -o . --plasmid
```

## Evaluation signals

- Reproduce type-level Pearson correlations against expected profiles: 0.990 (D6300), 0.976 (MSA2006), 0.981 (MSA1003), and 0.672 (D6331)
- Type and subtype precision above 0.9 for every mock community
- Correct plasmid-borne ARG calls recovered, e.g. ant(4')-I, qacG, tet(L) for D6300 and aph(2")-I at 4.256 cpg in Enterococcus faecalis for MSA2006
- Sanity-check adaptive cutoff inference in the log: 'median sequence divergence ... | identity cutoff: 90 - 2.5 * 100 * divergence' line appears during overlapping
- Output schema check: *.sarg.tsv contains lineage, type, subtype, carrier, copy, genome, and abundance columns with abundance = copy/genome

## Limitations

- Argo is inherently reference-based and may misclassify unseen species or novel plasmid syntenies
- ARGs originating from point mutations are excluded from the database because SNPs cannot be reliably distinguished from sequencing errors in read-based profiling
- Highly similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA), lowering resolution for ORF-based annotation or detailed sequence typing
- By default, ARGs of species below 1x genome copy are marked 'unclassified', making Argo less suitable for extremely rare species
- Insufficient coverage leads to unstable normalization and inaccurate taxonomic classification due to inadequate read overlaps and weaker clustering; '--plasmid' results can be confounded by chimeric reads and uncharacterized plasmids

## Evidence

- [results] Mock community performance: [evidence span withheld — anchor status: not_found]
