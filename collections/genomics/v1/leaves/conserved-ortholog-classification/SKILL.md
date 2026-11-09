---
name: conserved-ortholog-classification
description: Use when comparing gene completeness of genome assemblies (e.g., hybrid
  assembler outputs like asm.final.fa) across organisms, and you need standardized
  BUSCO percentages on a matched lineage database per species.
when_to_use_negative:
- When the goal is assembly contiguity or error metrics (NGA50, misassemblies) rather
  than gene completeness — those require different evaluation tools
- When no lineage-matched odb10 database exists for the organism, since ortholog classification
  depends on the correct reference gene set
- When comparing assemblies from different organisms — percentages are only comparable
  within a lineage database
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: BUSCO
  role: Runs in genome mode against odb10 lineages to classify conserved orthologs
    and report completeness percentages
- name: HASLR
  role: Produces the hybrid-assembly FASTA being evaluated; its gene completeness
    is benchmarked against competitors
  repo: https://github.com/vpc-ccg/haslr
- name: MaSuRCA
  role: Comparator assembler whose assembly is scored in the same BUSCO table
- name: PBcR
  role: Comparator assembler whose assembly is scored in the same BUSCO table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:b81025bc0e061856766c8a99c15218755eeecf504ef9033121f5838fa83d9bd3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/haslr
  repo_url: https://github.com/vpc-ccg/haslr
  tools:
  - BUSCO
  - HASLR
  - MaSuRCA
  - PBcR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# conserved-ortholog-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify conserved single-copy orthologs in de novo genome assemblies by running BUSCO in genome mode against organism-specific odb10 lineage databases, yielding complete/single-copy/duplicated/fragmented/missing percentages used to compare assembler gene completeness (e.g., HASLR vs Canu, MaSuRCA, PBcR).

## When to use

Use when comparing gene completeness of genome assemblies (e.g., hybrid assembler outputs like asm.final.fa) across organisms, and you need standardized BUSCO percentages on a matched lineage database per species.

## When NOT to use

- When the goal is assembly contiguity or error metrics (NGA50, misassemblies) rather than gene completeness — those require different evaluation tools
- When no lineage-matched odb10 database exists for the organism, since ortholog classification depends on the correct reference gene set
- When comparing assemblies from different organisms — percentages are only comparable within a lineage database

## Inputs

- Final assembly FASTA files (e.g., asm.final.fa) from each benchmarked assembler (HASLR, Canu, MaSuRCA, PBcR)
- Organism-specific BUSCO odb10 lineage databases (enterobacterales_odb10, saccharomycetes_odb10, nematoda_odb10)

## Outputs

- BUSCO short_summary files with complete, single-copy, duplicated, fragmented, and missing percentages per assembly
- Comparative gene-completeness table with one row per assembler per dataset (E. coli, yeast, C. elegans)

## How to apply

Collect the final assembly FASTA files (e.g., asm.final.fa) produced by each benchmarked assembler — HASLR and comparators such as Canu, MaSuRCA, and PBcR — for the real E. coli, yeast, and C. elegans datasets. Choose the lineage-matched odb10 database per organism: enterobacterales_odb10 for E. coli, saccharomycetes_odb10 for yeast, nematoda_odb10 for C. elegans. Run BUSCO in genome mode on each assembly against its lineage-specific gene set so conserved orthologs are classified as complete, single-copy, duplicated, fragmented, or missing. Parse the BUSCO short_summary outputs and extract the five percentage categories per assembler/dataset, then assemble one comparative table row per assembler per organism. Validate by checking the reproduced complete/fragmented/missing percentages against the paper's real-dataset benchmark (e.g., C. elegans HASLR 97.1% complete vs 97.4% for Canu), allowing that HASLR trades slightly lower completeness/genome fraction for speed and fewer misassemblies.

## Related tools

- **BUSCO** (Runs in genome mode against odb10 lineages to classify conserved orthologs and report completeness percentages)
- **HASLR** (Produces the hybrid-assembly FASTA being evaluated; its gene completeness is benchmarked against competitors) — https://github.com/vpc-ccg/haslr
- **MaSuRCA** (Comparator assembler whose assembly is scored in the same BUSCO table)
- **PBcR** (Comparator assembler whose assembly is scored in the same BUSCO table)

## Examples

```
busco -i asm.final.fa -l nematoda_odb10 -m genome --out_path busco_celegans_haslr
```

## Evaluation signals

- Each assembly's five BUSCO percentage categories (complete, single-copy, duplicated, fragmented, missing) parse cleanly from the short_summary output and sum to ~100%
- Reproduced complete/fragmented/missing percentages match the paper's real-dataset assembler-comparison table for E. coli, yeast, and C. elegans
- C. elegans benchmark reproduces the reported gap: HASLR 97.1% complete vs 97.4% for Canu, with HASLR within ~0.3 percentage points
- Lineage database used matches the organism (enterobacterales_odb10 / saccharomycetes_odb10 / nematoda_odb10) for every row of the table
- HASLR's slightly lower completeness relative to some competitors is consistent with the paper's stated trade-off of speed and fewer misassemblies

## Limitations

- HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools, especially Canu, so completeness percentages may trail the best competitor
- Compared with other tools, HASLR usually has a higher indel rate, which can affect downstream gene-level interpretation even when BUSCO completeness is comparable
- HASLR trades slightly lower completeness/genome fraction for speed and fewer misassemblies, so exact percentage parity with competitors is not expected across all datasets

## Evidence

- [results] HASLR is on par with other tools in BUSCO gene completeness on real datasets: [evidence span withheld — anchor status: not_found]
