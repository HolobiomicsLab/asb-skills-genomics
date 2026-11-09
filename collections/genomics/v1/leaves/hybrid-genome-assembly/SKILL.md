---
name: hybrid-genome-assembly
description: Use when both Illumina short-read FASTQ and Nanopore long-read FASTQ
  from the same sample are available and the goal is a de novo plasmid or genome assembly,
  especially to resolve circular structures or repetitive regions.
when_to_use_negative:
- Only one read type is available (the comparison — and the hybrid assembly itself
  — requires both short and long reads)
- The goal is only metagenomic or AMR analysis rather than de novo plasmid/genome
  assembly
- The user needs quantitative assembly benchmarks; validation here is qualitative
  graph comparison with no stated numeric threshold
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Unicycler
  role: Performs the hybrid assembly combining trimmed Illumina short reads and filtered
    Nanopore long reads
- name: Trim Galore
  role: Quality- and adapter-trims the Illumina short reads before assembly
- name: Porechop
  role: Adapter-trims the Nanopore long reads
- name: Filtlong
  role: Filters long reads to retain high-quality, high-length reads for assembly
- name: Bandage
  role: Renders the three assembly graphs to inspect contig structure and circularization
- name: NanoGalaxy
  role: Galaxy-based toolkit hosting the integrated workflow for long-read (and hybrid)
    assembly
  repo: mmiladi/galaxy-nanopore
- name: Galaxy
  role: User-friendly platform in which the trimming, filtering, assembly, and comparison
    steps are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:4013161f0c844b2adb3693143ea2be4c6ad25f2db20ec195a311fb63d0773b77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mmiladi/galaxy-nanopore
  tools:
  - Bandage
  - Filtlong
  - Galaxy
  - NanoGalaxy
  - Porechop
  - Trim Galore
  - Unicycler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# hybrid-genome-assembly

## Summary

A Unicycler-based hybrid assembly workflow in NanoGalaxy that combines Illumina short reads and Oxford Nanopore long reads to produce clearer plasmid/genome assemblies than either read type alone, visualized with Bandage.

## When to use

Use when both Illumina short-read FASTQ and Nanopore long-read FASTQ from the same sample are available and the goal is a de novo plasmid or genome assembly, especially to resolve circular structures or repetitive regions.

## When NOT to use

- Only one read type is available (the comparison — and the hybrid assembly itself — requires both short and long reads)
- The goal is only metagenomic or AMR analysis rather than de novo plasmid/genome assembly
- The user needs quantitative assembly benchmarks; validation here is qualitative graph comparison with no stated numeric threshold

## Inputs

- Illumina short-read FASTQ files (150-300 bp reads)
- Oxford Nanopore long-read FASTQ files (>10 kb average reads)
- GigaDB worked-example plasmid datasets accompanying the paper

## Outputs

- Three assembly graphs (short-read-only, long-read-only, Unicycler hybrid)
- Bandage-rendered visualizations of contig structure and circularization
- Hybrid assembly showing a clear circular plasmid structure

## How to apply

Download the worked-example Illumina and Nanopore plasmid datasets from the GigaDB deposit. Adapter- and quality-trim the Illumina reads with Trim Galore and adapter-trim the Nanopore reads with Porechop, then filter the long reads with Filtlong to retain high-quality, high-length reads. Run three assemblies in Galaxy: short-read-only, long-read-only, and Unicycler hybrid combining both read sets. Render each assembly graph with Bandage and compare contig structure and circularization: the Illumina-only graph lacks clear structure, the Nanopore-only graph yields circularized plasmid contigs, and the hybrid graph should give the clearest circular view, consistent with Wick et al. The rationale is that short reads (150-300 bp) alone cannot resolve repeats, while long reads (>10 kb) span them; combining both yields the clearest assembly.

## Related tools

- **Unicycler** (Performs the hybrid assembly combining trimmed Illumina short reads and filtered Nanopore long reads)
- **Trim Galore** (Quality- and adapter-trims the Illumina short reads before assembly)
- **Porechop** (Adapter-trims the Nanopore long reads)
- **Filtlong** (Filters long reads to retain high-quality, high-length reads for assembly)
- **Bandage** (Renders the three assembly graphs to inspect contig structure and circularization)
- **NanoGalaxy** (Galaxy-based toolkit hosting the integrated workflow for long-read (and hybrid) assembly) — mmiladi/galaxy-nanopore
- **Galaxy** (User-friendly platform in which the trimming, filtering, assembly, and comparison steps are executed)

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- The hybrid (Unicycler) assembly graph visually shows the clearest circular plasmid structure among the three conditions — this is the stated acceptance criterion, with no quantitative threshold
- The Illumina-only graph lacks clear structure, serving as the expected negative contrast
- The Nanopore-only graph yields circularized plasmid structures, matching the intermediate expectation
- The overall qualitative pattern matches the results reported by Wick et al.
- All three assembly graphs render in Bandage with inspectable contig connectivity

## Limitations

- Validation is purely qualitative (visual graph comparison); no quantitative threshold is stated in the available text
- Integrated tools in NanoGalaxy were intensively tested only for ONT data; PacBio support is expected but untested
- Other long-read assembly tools exist but were not integrated — the toolkit focuses on the most established, widely used tools

## Evidence

- [abstract] The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly"
- [intro] the short length of the reads obtained (150–300 bp) complicates the assembly of genomes: "However, the short length of the reads obtained (150–300 bp) complicates the assembly of genomes, especially for highly repetitive regions and the detection of structural variation"
- [intro] These platforms generate sequence reads much longer than those of the classic short-read technologies: "These platforms generate sequence reads much longer than those of the classic short-read technologies, including long reads from single DNA molecules and without the need of PCR amplification (>10 kb"
- [discussion] here we have performed intensive testing of the integrated tools for ONT data: "However, here we have performed intensive testing of the integrated tools for ONT data."
- [discussion] Additional supporting data are available from the GigaScience GigaDB database [53]: "Additional supporting data are available from the GigaScience GigaDB database [53]"
- [readme] docker run -d -p 8080:80 mmiladi/galaxy-nanopore: "``docker run -d -p 8080:80 mmiladi/galaxy-nanopore``"
