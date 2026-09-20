---
name: long-read-quality-filtering
description: Use when raw Oxford Nanopore long-read FASTQ (and matched Illumina FASTQ)
  needs trimming and quality/length filtering before de novo plasmid or genomic assembly
  in NanoGalaxy.
when_to_use_negative:
- Input reads are already trimmed/filtered or come from platforms other than Oxford
  Nanopore short/long-read plasmid data — the NanoGalaxy tools were intensively tested
  for ONT data only (PacBio tools are untested, 'should also work').
- The goal is basecalling or downstream assembly graph visualization rather than read
  preparation — use NanoGalaxy assembly/Unicycler and Bandage steps instead.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Filtlong
  role: Filters Nanopore long reads to retain high-quality, high-length reads prior
    to assembly
  repo: mmiladi/galaxy-nanopore
- name: Porechop
  role: Removes adapter sequences from Nanopore long reads
  repo: mmiladi/galaxy-nanopore
- name: Trim Galore
  role: Quality- and adapter-trims the paired Illumina short-read FASTQ
- name: NanoGalaxy
  role: Galaxy-based toolkit in which the trimming and filtering wrappers are installed
    and executed (Conda-managed dependencies)
  repo: mmiladi/galaxy-nanopore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:21fdd22ccf229272653dbb6c11efd97b60fa0d50e95bb4316df728de806c47c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mmiladi/galaxy-nanopore
  tools:
  - Filtlong
  - NanoGalaxy
  - Porechop
  - Trim Galore
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Long-read quality filtering

## Summary

Prepares Nanopore long-read sequencing data for assembly inside the NanoGalaxy/Galaxy toolkit by adapter-trimming with Porechop and filtering with Filtlong to retain high-quality, high-length reads, alongside Trim Galore preparation of the paired Illumina short reads.

## When to use

Use when raw Oxford Nanopore long-read FASTQ (and matched Illumina FASTQ) needs trimming and quality/length filtering before de novo plasmid or genomic assembly in NanoGalaxy.

## When NOT to use

- Input reads are already trimmed/filtered or come from platforms other than Oxford Nanopore short/long-read plasmid data — the NanoGalaxy tools were intensively tested for ONT data only (PacBio tools are untested, 'should also work').
- The goal is basecalling or downstream assembly graph visualization rather than read preparation — use NanoGalaxy assembly/Unicycler and Bandage steps instead.

## Inputs

- Raw Oxford Nanopore long-read sequencing FASTQ (e.g. plasmid dataset from GigaDB)
- Paired Illumina short-read sequencing FASTQ from the same sample (GigaDB deposit)
- NanoGalaxy Galaxy instance with Trim Galore, Porechop, and Filtlong wrappers installed

## Outputs

- Adapter- and quality-trimmed Illumina FASTQ (Trim Galore)
- Adapter-trimmed Nanopore FASTQ (Porechop)
- Quality- and length-filtered Nanopore FASTQ (Filtlong) suitable for assembly

## How to apply

Retrieve the worked-example Illumina short-read and Nanopore long-read plasmid datasets from the GigaDB deposit accompanying the NanoGalaxy (GigaScience, giaa105) paper. Quality- and adapter-trim the Illumina reads with Trim Galore, and adapter-trim the Nanopore reads with Porechop. Apply Filtlong to the long reads to retain high-quality, high-length reads for assembly, since long-read platforms such as Oxford Nanopore generate reads >10 kb on average and filter quality materially affects downstream Unicycler hybrid assembly. Run the filtering within the NanoGalaxy Galaxy environment (tools and dependencies are managed via the Conda framework), then feed the filtered FASTQ into short-read-only, long-read-only, and Unicycler hybrid assemblies and compare the resulting Bandage graphs.

## Related tools

- **Filtlong** (Filters Nanopore long reads to retain high-quality, high-length reads prior to assembly) — mmiladi/galaxy-nanopore
- **Porechop** (Removes adapter sequences from Nanopore long reads) — mmiladi/galaxy-nanopore
- **Trim Galore** (Quality- and adapter-trims the paired Illumina short-read FASTQ)
- **NanoGalaxy** (Galaxy-based toolkit in which the trimming and filtering wrappers are installed and executed (Conda-managed dependencies)) — mmiladi/galaxy-nanopore

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- Output FASTQ files contain no residual adapter sequence after Porechop/Trim Galore trimming.
- Filtlong output retains reads that are both high-quality and high-length (the article's stated retention criterion) relative to the input Nanopore FASTQ.
- Downstream long-read-only assembly yields circularized plasmid structures in Bandage, and the hybrid assembly graph visually shows the clearest circular plasmid structure among short-read-only, long-read-only, and hybrid conditions, matching Wick et al.
- Filtered data processed in NanoGalaxy reproduces the qualitative pattern reported in the source card without requiring any command-line skills outside Galaxy.
- Wrapper/tool installation succeeds via the Galaxy Tool Shed with Conda-managed dependencies.

## Limitations

- Integrated NanoGalaxy tools were intensively tested for ONT data only; support for Pacific Biosciences tools inside Galaxy is untested ('should also work').
- Other long-read analysis tools exist or are under development but were not integrated, as the focus was on the most established and widely used tools.
- No software version numbers are stated for the integrated tools; dependencies are managed implicitly via the Conda framework.
- No quantitative filtering threshold is stated in the available text — the validation is qualitative (clearest circular plasmid structure).

## Evidence

- [abstract] The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly"
- [discussion] All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management: "All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management"
- [discussion] Additional supporting data are available from the GigaScience GigaDB database [53]: "Additional supporting data are available from the GigaScience GigaDB database [53]"
- [discussion] here we have performed intensive testing of the integrated tools for ONT data: "However, here we have performed intensive testing of the integrated tools for ONT data."
- [readme] docker run -d -p 8080:80 mmiladi/galaxy-nanopore: "docker run -d -p 8080:80 mmiladi/galaxy-nanopore"
