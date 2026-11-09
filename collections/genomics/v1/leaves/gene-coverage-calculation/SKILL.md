---
name: gene-coverage-calculation
description: Use when after calling RNA modifications with Dogme/modkit across replicates,
  you need to know what fraction of annotated genes contain shared modification sites
  per modification type (m6A, m5C, inosine, pseudouridine, Nm).
when_to_use_negative:
- Input is raw POD5 or unbasecalled reads — Dorado basecalling, minimap2 mapping,
  and modkit extraction/filtering must precede any gene-coverage calculation
- Only one replicate is available, so the strict three-way intersection that defines
  the shared site set cannot be formed
- The goal is transcript isoform abundance rather than per-gene modification coverage
  — use the LR-Kallisto quantification workflow instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: modkit
  role: Extracts and filters modified-base calls from modBAM into the BED/bedMethyl
    site sets that are intersected across replicates before gene annotation mapping
  repo: https://github.com/nanoporetech/modkit
- name: Dogme
  role: Nextflow pipeline that orchestrates Dorado basecalling, minimap2 mapping,
    modkit extraction/filtering (filterbed.py) and replicate comparison (report_bed_rep.py)
    feeding the gene-coverage computation
  repo: https://github.com/mortazavilab/dogme
- name: minimap2
  role: Maps Dorado-basecalled direct RNA reads to the genome so modkit site coordinates
    are in the same reference frame as the gene annotations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:575305d99f94fe5ed2b112a8d27f111047976058c8c9f134d753fb9a4e8a62ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/modkit
  tools:
  - Dogme
  - minimap2
  - modkit
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gene-coverage-calculation

## Summary

Quantifies how many genes carry each nanopore RNA modification by intersecting the replicate-shared modkit modification sites (BED) with gene annotations, yielding per-modification gene counts and coverage percentages (e.g., 12179 genes, 88.2% for m6A in the C2C12 dRNA replicates).

## When to use

Use when, after calling RNA modifications with Dogme/modkit across replicates, you need to know what fraction of annotated genes contain shared modification sites per modification type (m6A, m5C, inosine, pseudouridine, Nm).

## When NOT to use

- Input is raw POD5 or unbasecalled reads — Dorado basecalling, minimap2 mapping, and modkit extraction/filtering must precede any gene-coverage calculation
- Only one replicate is available, so the strict three-way intersection that defines the shared site set cannot be formed
- The goal is transcript isoform abundance rather than per-gene modification coverage — use the LR-Kallisto quantification workflow instead

## Inputs

- Per-replicate filtered modkit BED/bedMethyl modification site files (.bed.gz) from Dorado-basecalled, minimap2-mapped direct RNA reads
- Gene annotation reference (GTF/BED) for the genome used in minimap2 alignment
- List of modification types to evaluate (m6A, m5C, inosine, pseudouridine, Nm)

## Outputs

- Counts of shared modification sites per modification type from the strict three-way replicate intersection
- Per-modification gene coverage: number of distinct genes containing shared sites and coverage percentage of all annotated genes (e.g., 12179 genes / 88.2% m6A; 8547 genes / 74.1% m5C)
- Comparison CSVs per genome/modification (e.g., Compare_GRCh38_m6A.csv style outputs)

## How to apply

Load the per-replicate modkit-extracted and filtered modification BED files (compressed .bed.gz) for each of the three ENCSR160HKZ C2C12 dRNA biological replicates, and intersect genomic site coordinates three ways to retain only sites detected in all replicates. Filter the modkit BED outputs by minimum coverage and per-modification thresholds (params.minCov and params.perMod) before intersecting, since filtering stringency drives both site reproducibility and downstream gene coverage. Map the shared sites to gene annotations (GTF-derived gene coordinates), remembering to merge plus/minus strands per sample so a site on either strand maps to the same gene. Count distinct genes containing at least one shared site per modification and divide by the total annotated genes to obtain coverage, recovering the reported m6A coverage (12179 genes, 88.2%) and m5C coverage (8547 genes, 74.1%). Validate by checking that the underlying shared-site counts match the reference integers 96603 m6A, 43476 m5C, 8829 inosine, 10055 pseudouridine, and 30320 Nm, and that shared m6A equals 63.2% of the average per-replicate total.

## Related tools

- **modkit** (Extracts and filters modified-base calls from modBAM into the BED/bedMethyl site sets that are intersected across replicates before gene annotation mapping) — https://github.com/nanoporetech/modkit
- **Dogme** (Nextflow pipeline that orchestrates Dorado basecalling, minimap2 mapping, modkit extraction/filtering (filterbed.py) and replicate comparison (report_bed_rep.py) feeding the gene-coverage computation) — https://github.com/mortazavilab/dogme
- **minimap2** (Maps Dorado-basecalled direct RNA reads to the genome so modkit site coordinates are in the same reference frame as the gene annotations)

## Evaluation signals

- Shared-site counts match reference integers exactly: 96603 m6A, 43476 m5C, 8829 inosine, 10055 pseudouridine, 30320 Nm
- Shared m6A site count equals 63.2% of the average per-replicate m6A total
- Gene coverage reproduces 12179 genes (88.2%) for m6A and 8547 genes (74.1%) for m5C
- Three-way intersection counts are monotonically less than or equal to any single-replicate site count
- Filtering with params.minCov and params.perMod thresholds is applied before intersection, so lowering stringency should only increase shared-site and gene-coverage numbers

## Limitations

- Strict three-way intersection means coverage depends on modkit filter thresholds (minCov, perMod); different cutoffs will shift both site and gene coverage numbers
- Plus/minus strand handling matters — sites must be merged per sample before gene assignment or strand-splitting can undercount covered genes
- Gene coverage percentages are relative to the genome annotation used for minimap2 alignment; changing annotation versions changes the denominator

## Evidence

- [abstract] We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates: "We detected 96 603 m6A, 43 476 m5C, 8829 inosine, 10 055 pseudouridine, and 30 320 Nm sites in three biological replicates"
- [intro] For RNA and DNA, it will extract modifications using modkit and filter those.: "For RNA and DNA, it will extract modifications using modkit and filter those."
- [readme] Filters modkit bed outputs by minimum coverage and per-mod thresholds (params.minCov and params.perMod), reading and writing compressed `.bed.gz` files: [evidence span withheld — anchor status: external_doc]
- [readme] Compares per-sample modkit BED outputs across folders (replicates), including compressed `.bed.gz` files, merges plus/minus strands per sample: [evidence span withheld — anchor status: external_doc]
