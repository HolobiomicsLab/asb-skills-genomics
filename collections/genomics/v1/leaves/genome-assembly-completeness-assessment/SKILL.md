---
name: genome-assembly-completeness-assessment
description: Use when comparing assemblies of the same organism produced by different
  assemblers (e.g., HASLR vs Canu on E. coli, yeast, C. elegans) and you need an ortholog-based
  completeness metric to supplement contiguity (NGA50) and misassembly counts.
when_to_use_negative:
- The goal is to measure contiguity or misassembly counts rather than gene completeness
  — use QUAST-like metrics (NGA50, genome fraction, misassemblies) instead.
- The organism has no close-matching BUSCO odb10 lineage database, so conserved ortholog
  classification would be unreliable.
- Assessment is of raw long reads or intermediate backbone graphs (e.g., backbone.05.superbubble.gfa)
  rather than a final polished assembly FASTA.
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0797
tools:
- name: BUSCO
  role: Genome-mode assessment of conserved single-copy orthologs against lineage-specific
    odb10 databases to classify complete/fragmented/missing genes
- name: HASLR
  role: Hybrid assembler whose output assemblies (from PacBio/ONT long reads plus
    Illumina short reads) are the subject of the completeness comparison
  repo: https://github.com/vpc-ccg/haslr
- name: MaSuRCA
  role: Competing assembler included in the real-dataset completeness comparison
- name: PBcR
  role: Competing assembler included in the real-dataset completeness comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:3dde0cf6e6828fb212a72644ef358bd57205b2f60f22be1d2e96ca54ecfc7c7e
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
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# genome-assembly-completeness-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark gene completeness of hybrid-assembled genomes by running BUSCO in genome mode against lineage-specific odb10 databases, extracting the five percentage categories (complete, single-copy, duplicated, fragmented, missing) to compare assemblers such as HASLR against Canu, MaSuRCA, and PBcR.

## When to use

Use when comparing assemblies of the same organism produced by different assemblers (e.g., HASLR vs Canu on E. coli, yeast, C. elegans) and you need an ortholog-based completeness metric to supplement contiguity (NGA50) and misassembly counts.

## When NOT to use

- The goal is to measure contiguity or misassembly counts rather than gene completeness — use QUAST-like metrics (NGA50, genome fraction, misassemblies) instead.
- The organism has no close-matching BUSCO odb10 lineage database, so conserved ortholog classification would be unreliable.
- Assessment is of raw long reads or intermediate backbone graphs (e.g., backbone.05.superbubble.gfa) rather than a final polished assembly FASTA.

## Inputs

- Final assembly FASTA files (e.g., asm.final.fa) from each benchmarked assembler for E. coli, yeast, and C. elegans
- BUSCO odb10 lineage databases: enterobacterales_odb10, saccharomycetes_odb10, nematoda_odb10

## Outputs

- BUSCO short_summary files per assembly
- Comparative table of complete, single-copy, duplicated, fragmented, and missing percentages, one row per assembler per dataset (reproducing Table 4)

## How to apply

Collect the final assembly FASTA files (e.g., asm.final.fa) from each benchmarked assembler — HASLR plus competitors like Canu, MaSuRCA, and PBcR — for the real E. coli, yeast, and C. elegans datasets. Select the matching BUSCO odb10 lineage per organism: enterobacterales_odb10 for E. coli, saccharomycetes_odb10 for yeast, and nematoda_odb10 for C. elegans. Run BUSCO in genome mode on each assembly against its lineage-specific gene set, then parse the short_summary output to extract the five percentage categories (complete, single-copy, duplicated, fragmented, missing) per assembler and organism. Assemble these percentages into one comparative table with one row per assembler per dataset (e.g., Table 4 in the HASLR paper), and interpret completeness jointly with other metrics: the paper notes HASLR trades slightly lower completeness/genome fraction for speed and fewer misassemblies.

## Related tools

- **BUSCO** (Genome-mode assessment of conserved single-copy orthologs against lineage-specific odb10 databases to classify complete/fragmented/missing genes)
- **HASLR** (Hybrid assembler whose output assemblies (from PacBio/ONT long reads plus Illumina short reads) are the subject of the completeness comparison) — https://github.com/vpc-ccg/haslr
- **MaSuRCA** (Competing assembler included in the real-dataset completeness comparison)
- **PBcR** (Competing assembler included in the real-dataset completeness comparison)

## Examples

```
busco -i asm.final.fa -l nematoda_odb10 -m genome -o haslr_celegans --out_path busco_results
```

## Evaluation signals

- Reproduced complete/fragmented/missing percentages match the paper's real-dataset assembler comparison (e.g., C. elegans: 97.1% complete for HASLR vs 97.4% for Canu), allowing for HASLR's slightly lower completeness/genome fraction trade-off.
- Each assembly is assessed against its organism-correct lineage database (enterobacterales_odb10 for E. coli, saccharomycetes_odb10 for yeast, nematoda_odb10 for C. elegans), not a mismatched lineage.
- The five percentage categories (complete, single-copy, duplicated, fragmented, missing) are extracted for every assembler x dataset combination and sum consistently to 100% of orthologs.
- The final comparative table has exactly one row per assembler per dataset across the three organisms.

## Limitations

- HASLR trades slightly lower gene completeness and genome fraction for speed and fewer misassemblies, so completeness alone does not rank assemblers.
- Compared with other tools, HASLR usually has a higher indel rate, which can affect downstream gene-based assessment.
- HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools, especially versus Canu on the human genome — conclusions may not transfer to large genomes.

## Evidence

- [results] HASLR is on par with other tools in BUSCO gene completeness on real datasets: [evidence span withheld — anchor status: not_found]
