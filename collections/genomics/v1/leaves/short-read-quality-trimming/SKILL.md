---
name: short-read-quality-trimming
description: Use when raw Illumina FASTQ and/or Nanopore long-read FASTQ (e.g. plasmid
  datasets from GigaDB) must be trimmed before assembly with Unicycler, Filtlong,
  or Bandage graph inspection.
when_to_use_negative:
- Reads are already adapter-trimmed and quality-trimmed and the goal is only assembly
  or Bandage graph rendering
- Input is not sequencing read data (e.g. already-assembled contigs or an AMR analysis
  use case)
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Trim Galore
  role: Quality- and adapter-trims the Illumina short reads prior to assembly
- name: Porechop
  role: Adapter-trims the Nanopore long reads before long-read filtering
- name: Filtlong
  role: Filters trimmed long reads to retain high-quality, high-length reads for assembly
- name: NanoGalaxy
  role: Galaxy-based toolkit hosting the trimming and assembly tools without command-line
    skills
  repo: mmiladi/galaxy-nanopore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:e41dba1c823c8c458dc8798c3bf53c9cad1a9a28ab0abd78c574e49c75095897
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mmiladi/galaxy-nanopore
  tools:
  - Filtlong
  - NanoGalaxy
  - Porechop
  - Trim Galore
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# short-read-quality-trimming

## Summary

Quality- and adapter-trim Illumina short reads and adapter-trim Nanopore long reads within the NanoGalaxy hybrid assembly workflow, so that downstream short-only, long-only, and Unicycler hybrid assemblies of plasmid data start from clean reads.

## When to use

Use when raw Illumina FASTQ and/or Nanopore long-read FASTQ (e.g. plasmid datasets from GigaDB) must be trimmed before assembly with Unicycler, Filtlong, or Bandage graph inspection.

## When NOT to use

- Reads are already adapter-trimmed and quality-trimmed and the goal is only assembly or Bandage graph rendering
- Input is not sequencing read data (e.g. already-assembled contigs or an AMR analysis use case)

## Inputs

- Illumina short-read FASTQ files (worked-example plasmid dataset from GigaDB)
- Oxford Nanopore long-read FASTQ files (worked-example plasmid dataset from GigaDB)

## Outputs

- Adapter- and quality-trimmed Illumina FASTQ reads (via Trim Galore)
- Adapter-trimmed Nanopore FASTQ reads (via Porechop)
- Quality/length-filtered long-read FASTQ (via Filtlong), ready for assembly

## How to apply

Obtain the worked-example Illumina short-read and Nanopore long-read plasmid FASTQ datasets from the GigaDB deposit. Trim Galore to quality- and adapter-trim the Illumina reads; Porechop to adapter-trim the Nanopore reads; then Filtlong to filter long reads, retaining high-quality, high-length reads for assembly. The trimmed read sets feed three assembly conditions: Illumina-only, Nanopore-only, and Unicycler hybrid, whose graphs are rendered in Bandage. Run these steps inside the NanoGalaxy Galaxy environment so tool dependencies are handled by the platform; trimming is a prerequisite step whose quality directly determines whether the hybrid graph shows the expected clear circular plasmid structure.

## Related tools

- **Trim Galore** (Quality- and adapter-trims the Illumina short reads prior to assembly)
- **Porechop** (Adapter-trims the Nanopore long reads before long-read filtering)
- **Filtlong** (Filters trimmed long reads to retain high-quality, high-length reads for assembly)
- **NanoGalaxy** (Galaxy-based toolkit hosting the trimming and assembly tools without command-line skills) — mmiladi/galaxy-nanopore

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- Trimmed Illumina FASTQ contains no remaining adapter sequence after Trim Galore
- Porechop output Nanopore FASTQ is adapter-free and survives Filtlong quality/length filtering
- Downstream Unicycler hybrid assembly graph shows the clearest circular plasmid structure among the three conditions
- Illumina-only assembly graph lacks clear structure while Nanopore-only yields circularized plasmid structures, matching Wick et al.

## Limitations

- The validation is qualitative: accept the result when the hybrid graph visually shows the clearest circular plasmid structure; no quantitative threshold is stated in the available text
- Integrated tools were intensively tested for ONT data specifically; other technologies were not equivalently tested

## Evidence

- [methods] Quality- and adapter-trim the Illumina reads with Trim Galore and adapter-trim the Nanopore reads with Porechop.: [evidence span withheld — anchor status: not_found]
- [abstract] The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly"
- [discussion] Additional supporting data are available from the GigaScience GigaDB database [53]: "Additional supporting data are available from the GigaScience GigaDB database [53]"
- [readme] docker run -d -p 8080:80 mmiladi/galaxy-nanopore: "``docker run -d -p 8080:80 mmiladi/galaxy-nanopore``"
