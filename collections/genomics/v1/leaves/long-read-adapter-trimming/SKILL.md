---
name: long-read-adapter-trimming
description: Use when raw Nanopore long-read FASTQ data is about to enter de novo
  genome or plasmid assembly (e.g. Unicycler hybrid assembly in NanoGalaxy) and native
  or supplied adapter sequences must be removed first.
when_to_use_negative:
- The input is Illumina short-read FASTQ — the workflow specifies Trim Galore for
  quality- and adapter-trimming of Illumina reads, not Porechop.
- The reads are already adapter-trimmed and length/quality filtered and are ready
  to be passed directly to Unicycler hybrid assembly or Bandage graph rendering.
edam_operation: http://edamontology.org/operation_2019
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Porechop
  role: Adapter-trims Nanopore long reads before assembly in the NanoGalaxy workflow
- name: Filtlong
  role: Filters trimmed long reads to retain high-quality, high-length reads for assembly
- name: NanoGalaxy
  role: Galaxy-based toolkit providing the integrated wrappers and workflow in which
    adapter trimming is executed
  repo: mmiladi/galaxy-nanopore
- name: Galaxy
  role: Platform that handles software dependencies (Conda-managed) and provides the
    refined workflow without requiring command-line skills
  repo: https://toolshed.g2.bx.psu.edu/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:4ed5ef8c2c270aea917b525b2c65c1c21c205af4bd2a0d531a478e251ff7c1b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mmiladi/galaxy-nanopore
  tools:
  - Filtlong
  - Galaxy
  - NanoGalaxy
  - Porechop
  edam_operation: http://edamontology.org/operation_2019
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# long-read-adapter-trimming

## Summary

Adapter-trims Oxford Nanopore long reads with Porechop and optionally filters them with Filtrock-quality length selection, as part of the NanoGalaxy long-read assembly workflow, so that contaminated reads do not propagate into genome or plasmid assemblies.

## When to use

Use when raw Nanopore long-read FASTQ data is about to enter de novo genome or plasmid assembly (e.g. Unicycler hybrid assembly in NanoGalaxy) and native or supplied adapter sequences must be removed first.

## When NOT to use

- The input is Illumina short-read FASTQ — the workflow specifies Trim Galore for quality- and adapter-trimming of Illumina reads, not Porechop.
- The reads are already adapter-trimmed and length/quality filtered and are ready to be passed directly to Unicycler hybrid assembly or Bandage graph rendering.

## Inputs

- Raw Oxford Nanopore long-read sequencing FASTQ files (e.g. plasmid datasets from the GigaDB deposit accompanying the paper)

## Outputs

- Adapter-trimmed Nanopore FASTQ reads
- Length/quality-filtered FASTQ (after Filtlong) ready for long-read-only or Unicycler hybrid assembly

## How to apply

In the NanoGalaxy (Galaxy-based) workflow, run Porechop on the Nanopore long-read input to strip adapter sequences before assembly; note that Illumina short reads in the same study are trimmed with a different tool (Trim Galore, which performs quality- and adapter-trimming), so do not interchange them. After adapter-trimming, pass the long reads to Filtlong to retain high-quality, high-length reads for assembly — this two-step (adapter removal then length/quality filtering) precedes short-read-only, long-read-only, and Unicycler hybrid assembly conditions. The trimmed FASTQ is the input to the long-read-only assembly and the hybrid assembly; judge success by whether downstream Bandage-rendered assembly graphs show clean contig structure, with the hybrid graph giving the clearest circular plasmid view consistent with Wick et al. The Galaxy platform handles tool dependencies via Conda, and wrappers are available from the Galaxy Tool Shed, so no command-line environment setup is required by the user.

## Related tools

- **Porechop** (Adapter-trims Nanopore long reads before assembly in the NanoGalaxy workflow)
- **Filtlong** (Filters trimmed long reads to retain high-quality, high-length reads for assembly)
- **NanoGalaxy** (Galaxy-based toolkit providing the integrated wrappers and workflow in which adapter trimming is executed) — mmiladi/galaxy-nanopore
- **Galaxy** (Platform that handles software dependencies (Conda-managed) and provides the refined workflow without requiring command-line skills) — https://toolshed.g2.bx.psu.edu/

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- Downstream Filtlong step runs without failing on residual adapter-containing reads, and retains high-quality, high-length reads as intended.
- Long-read-only and Unicycler hybrid assemblies of the trimmed reads succeed and produce assembly graphs renderable in Bandage.
- The hybrid assembly graph visually shows the clearest circular plasmid structure among the three conditions (short-read-only, long-read-only, hybrid), matching Wick et al.
- The NanoGalaxy environment (docker image mmiladi/galaxy-nanopore) serves the Galaxy interface on port 8080 with the trimming wrappers installed via Conda/Galaxy Tool Shed.

## Limitations

- Only the most established and widely used long-read tools were integrated; other Nanopore analysis tools developed or under development were deliberately excluded from the toolkit.
- Integrated tools were intensively tested only for ONT data; tools supporting other technologies such as Pacific Biosciences are expected to work but are untested ('should also work').
- No software version numbers for the integrated tools are stated, so exact trimming behaviour may vary with the Conda-resolved tool version.

## Evidence

- [other] Quality- and adapter-trim the Illumina reads with Trim Galore and adapter-trim the Nanopore reads with Porechop.: [evidence span withheld — anchor status: not_found]
- [other] Filter the long reads with Filtlong to retain high-quality, high-length reads for assembly.: [evidence span withheld — anchor status: not_found]
- [abstract] NanoGalaxy is a Galaxy-based toolkit for analysing long-read sequencing data, suitable for diverse applications including de novo genome assembly.: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly"
- [readme] README describes the NanoGalaxy docker image used to run the toolkit.: "NanoGalaxy: Galaxy toolkit and workflows for long-read Oxford Nanopore data"
