---
name: assembly-summary-statistics-tabulation
description: Use when comparing final assemblies (FASTA) from several assemblers on
  the same organism(s) and you need organism-aware BUSCO completeness percentages
  tabulated per assembler, e.g. to reproduce an assembler-comparison table like the
  HASLR paper's Table 4.
when_to_use_negative:
- Input is raw long/short sequencing reads rather than final assembled FASTA contigs
  — this skill tabulates assembly quality, it does not perform assembly
- You need contiguity or error metrics (NGA50, genome fraction, misassemblies) rather
  than gene completeness — use QUAST/aligner-based evaluation instead of BUSCO
- No lineage-specific BUSCO odb10 database exists for the organism, since completeness
  percentages are only meaningful against a matching clade gene set
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: BUSCO
  role: Run in genome mode against lineage-specific odb10 gene sets to classify conserved
    orthologs and report complete/single-copy/duplicated/fragmented/missing percentages
- name: HASLR
  role: Source of hybrid assemblies (asm.final.fa) being evaluated alongside competitor
    assemblers
  repo: https://github.com/vpc-ccg/haslr
- name: MaSuRCA
  role: Competitor assembler whose final assembly is tabulated in the comparison
- name: PBcR
  role: Competitor assembler whose final assembly is tabulated in the comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:7e92477076dff310b44aa8f75da41886b7e83056e828435137791fcbab806663
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
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# assembly-summary-statistics-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulates genome assembly quality statistics — specifically BUSCO gene completeness categories (complete, single-copy, duplicated, fragmented, missing) — across multiple assemblers and organisms into a single comparative table, as done for HASLR vs. competitor hybrid/long-read assemblers on E. coli, yeast, and C. elegans (Table 4).

## When to use

Use when comparing final assemblies (FASTA) from several assemblers on the same organism(s) and you need organism-aware BUSCO completeness percentages tabulated per assembler, e.g. to reproduce an assembler-comparison table like the HASLR paper's Table 4.

## When NOT to use

- Input is raw long/short sequencing reads rather than final assembled FASTA contigs — this skill tabulates assembly quality, it does not perform assembly
- You need contiguity or error metrics (NGA50, genome fraction, misassemblies) rather than gene completeness — use QUAST/aligner-based evaluation instead of BUSCO
- No lineage-specific BUSCO odb10 database exists for the organism, since completeness percentages are only meaningful against a matching clade gene set

## Inputs

- Final assembly FASTA files (e.g., asm.final.fa) from each benchmarked assembler (HASLR, Canu, MaSuRCA, PBcR, etc.)
- Organism identifiers to map assemblies to BUSCO odb10 lineages (enterobacterales_odb10, saccharomycetes_odb10, nematoda_odb10)
- BUSCO short_summary output files from genome-mode runs

## Outputs

- Comparative table with one row per assembler per dataset, containing complete, single-copy, duplicated, fragmented, and missing BUSCO percentages
- Per-assembly BUSCO short_summary reports

## How to apply

Collect the final assembly FASTA files (e.g., asm.final.fa) produced by each benchmarked assembler, including HASLR, per organism. Select the lineage-matched BUSCO odb10 database per organism — enterobacterales_odb10 for E. coli, saccharomycetes_odb10 for yeast, nematoda_odb10 for C. elegans — because completeness percentages are only meaningful against the correct clade gene set. Run BUSCO in genome mode on each assembly and parse the short_summary output to extract the five percentage categories (complete, single-copy, duplicated, fragmented, missing). Assemble the parsed values into one table with a row per assembler per dataset; interpret results allowing that HASLR trades slightly lower completeness/genome fraction for speed and fewer misassemblies.

## Related tools

- **BUSCO** (Run in genome mode against lineage-specific odb10 gene sets to classify conserved orthologs and report complete/single-copy/duplicated/fragmented/missing percentages)
- **HASLR** (Source of hybrid assemblies (asm.final.fa) being evaluated alongside competitor assemblers) — https://github.com/vpc-ccg/haslr
- **MaSuRCA** (Competitor assembler whose final assembly is tabulated in the comparison)
- **PBcR** (Competitor assembler whose final assembly is tabulated in the comparison)

## Examples

```
busco -i asm.final.fa -l nematoda_odb10 -m genome -o haslr_celegans --out_path busco_results
```

## Evaluation signals

- Reproduced complete/fragmented/missing percentages match the assembler-comparison values reported in the paper's real-dataset benchmark (e.g., C. elegans: 97.1% complete for HASLR vs 97.4% for Canu), allowing small deviations
- Each assembly was scored against the correct lineage database (enterobacterales_odb10, saccharomycetes_odb10, nematoda_odb10) — cross-check lineage assignment per organism
- The five percentage categories per assembly sum to 100% (complete + fragmented + missing = 100; single-copy + duplicated = complete)
- Table contains one row per assembler per organism covering all three real datasets (E. coli, yeast, C. elegans)
- Parsed values originate from BUSCO short_summary files, not hand-transcribed numbers

## Limitations

- HASLR trades slightly lower completeness and/or genome fraction for speed and fewer misassemblies, so completeness differences alone do not capture overall assembly quality
- High-heterozygosity regions may not be resolved because Minia short read contigs are fragmented and filtered out, which can depress BUSCO completeness for such genomes
- HASLR sometimes generates assemblies with lower genome fraction and/or NGA50 compared with other tools (especially Canu on human), so the comparison table should be read alongside contiguity and error metrics

## Evidence

- [results] HASLR is on par with other tools in BUSCO gene completeness on real datasets (e.g., C. elegans: 97.1% complete vs 97.4% for Canu): [evidence span withheld — anchor status: not_found]
