---
name: assembly-graph-visualization
description: Use when an assembly step (e.g. Unicycler hybrid, short-read-only, or
  Nanopore long-read-only assembly of plasmid reads) has produced a graph/FASTG-type
  assembly output that must be visually inspected for contig structure and circular
  plasmid structure.
when_to_use_negative:
- The input is raw FASTQ reads that have not yet been assembled — graph visualization
  requires an assembly/graph product
- A quantitative assembly metric (e.g. N50, identity threshold) is required — the
  article validates this step only qualitatively via the visual graph pattern
- The goal is read quality control rather than assembly inspection — trimming/filtering
  tools (Trim Galore, Porechop, Filtlong) are the appropriate stage
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Bandage
  role: Renders the three assembly graphs for visual inspection of contig structure
    and circularization; also used to confirm PlasFlow plasmid contigs
- name: NanoGalaxy
  role: Galaxy-based toolkit environment in which the assemblies and graph visualization
    workflows are run
  repo: https://github.com/mmiladi/galaxy-nanopore
- name: Galaxy
  role: Platform providing the user-friendly interface through which Bandage visualization
    is executed without command-line skills
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:bf215b851875d2a0ed733bd99792255b7ca7af3df246a8e27437428f74667e45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mmiladi/galaxy-nanopore
  tools:
  - Bandage
  - Galaxy
  - NanoGalaxy
  - PlasFlow
  - Racon
  - miniasm
  - minimap2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# assembly-graph-visualization

## Summary

Render and inspect de novo assembly graphs (short-read-only, long-read-only, or hybrid) as interactive visualizations inside the NanoGalaxy/Galaxy environment, primarily with Bandage, to assess contig structure and circularization of plasmid or genomic assemblies.

## When to use

Use when an assembly step (e.g. Unicycler hybrid, short-read-only, or Nanopore long-read-only assembly of plasmid reads) has produced a graph/FASTG-type assembly output that must be visually inspected for contig structure and circular plasmid structure.

## When NOT to use

- The input is raw FASTQ reads that have not yet been assembled — graph visualization requires an assembly/graph product
- A quantitative assembly metric (e.g. N50, identity threshold) is required — the article validates this step only qualitatively via the visual graph pattern
- The goal is read quality control rather than assembly inspection — trimming/filtering tools (Trim Galore, Porechop, Filtlong) are the appropriate stage

## Inputs

- Assembly graph output from short-read-only assembly (Illumina plasmid reads)
- Assembly graph output from Nanopore long-read-only assembly (miniasm/Racon draft)
- Assembly graph output from Unicycler hybrid assembly (short + long reads)
- PlasFlow plasmid-classified contigs to confirm visually

## Outputs

- Rendered interactive assembly graph images per condition
- Qualitative comparison of contig structure and circularization across the three assembly conditions
- Visual confirmation of plasmid-derived contigs in plasmid_recovery_results.csv workflow

## How to apply

After running assemblies in NanoGalaxy — e.g. Illumina short-read-only assembly, Nanopore long-read-only assembly, and Unicycler hybrid assembly on GigaDB-deposited plasmid datasets (Illumina reads trimmed with Trim Galore, Nanopore reads adapter-trimmed with Porechop and filtered with Filtlong) — load each assembly output into Bandage within Galaxy to render the graph. Inspect contig counts, connectivity, and whether contigs close into circular plasmid structures. In the hybrid-vs-single-technology comparison, the Illumina-only graph is expected to lack clear structure, the Nanopore-only graph to yield circularized plasmid structures, and the hybrid graph to give the clearest circular view consistent with Wick et al. In the AMR long-read workflow (minimap2 + miniasm + Racon), Bandage is used to inspect/confirm contigs that PlasFlow classified as plasmid-derived. The qualitative visual comparison — hybrid graph showing the clearest circular plasmid structure among the three conditions — is the acceptance criterion; no quantitative threshold is stated in the available text.

## Related tools

- **Bandage** (Renders the three assembly graphs for visual inspection of contig structure and circularization; also used to confirm PlasFlow plasmid contigs)
- **NanoGalaxy** (Galaxy-based toolkit environment in which the assemblies and graph visualization workflows are run) — https://github.com/mmiladi/galaxy-nanopore
- **Galaxy** (Platform providing the user-friendly interface through which Bandage visualization is executed without command-line skills)

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- The Illumina-only graph visually lacks clear structure relative to the other two conditions
- The Nanopore-only graph shows circularized plasmid structures
- The Unicycler hybrid graph shows the clearest circular plasmid structure among the three conditions, matching the Wick et al. qualitative pattern
- Bandage rendering of the AMR long-read workflow matches the PlasFlow-classified plasmid contigs (19 of 21 plasmids recovered, 97.76% mean identity against the Li et al. reference set)

## Limitations

- Validation is purely qualitative/visual; no quantitative threshold is stated in the available text
- Graph clarity depends on upstream read quality — Nanopore reads must be adapter-trimmed (Porechop) and filtered (Filtlong) before assembly, or the graph may be fragmented
- NanoGalaxy focused on the most established long-read tools; other tools developed or under development were not integrated, so visualization options within the toolkit are limited to the integrated set

## Evidence

- [abstract] NanoGalaxy toolkit introduction: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly"
- [discussion] Community availability: "we expect that the toolkit will be further extended by the community because NanoGalaxy is part of the open Galaxy platform and Galaxy community"
- [readme] README deployment: "NanoGalaxy: Galaxy toolkit and workflows for long-read Oxford Nanopore data"
- [readme] README docker invocation: "docker run -d -p 8080:80 mmiladi/galaxy-nanopore"
