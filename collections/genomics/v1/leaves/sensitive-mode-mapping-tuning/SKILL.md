---
name: sensitive-mode-mapping-tuning
description: Use when you need PAF-formatted long-read to contig mappings (e.g. for
  downstream misassembly detection or Tigmint-long correction) produced from a draft
  assembly FASTA plus long-read FASTQ using minimizer-based mapping.
when_to_use_negative:
- When the goal is to actually scaffold the assembly (infer contig ordering/orientation
  and output a scaffolded FASTA) — use ntLink's Basic Protocols 1–3 instead of pair/mapping
  mode
- When base-level alignment accuracy is required (e.g. variant calling or consensus
  polishing with minimap2/racon) — ntLink mappings are lightweight minimizer-based
  mappings, not full read alignments
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntLink
  role: Runs in pair mode with sensitive=True and paf=True to produce PAF long-read
    to contig mappings via minimizer sketches
  repo: https://github.com/bcgsc/ntLink
- name: btllib
  role: Dependency library (1.7.1 or higher) required to run ntLink
  repo: https://github.com/BirolLab/btllib
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:be6121fd6b4b990c7c59860c323622a24080fe1f7d18ad479d7b191bcea71f56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - btllib
  - ntLink
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# sensitive-mode-mapping-tuning

## Summary

Generates standalone long-read to contig mappings by running ntLink's pairing stage in mapping-only mode (sensitive=True, paf=True), yielding minimizer-based read-to-contig alignments in PAF format without performing scaffolding.

## When to use

Use when you need PAF-formatted long-read to contig mappings (e.g. for downstream misassembly detection or Tigmint-long correction) produced from a draft assembly FASTA plus long-read FASTQ using minimizer-based mapping.

## When NOT to use

- When the goal is to actually scaffold the assembly (infer contig ordering/orientation and output a scaffolded FASTA) — use ntLink's Basic Protocols 1–3 instead of pair/mapping mode
- When base-level alignment accuracy is required (e.g. variant calling or consensus polishing with minimap2/racon) — ntLink mappings are lightweight minimizer-based mappings, not full read alignments

## Inputs

- Draft genome assembly FASTA (e.g. celegans-abyss.fa, ABySS C. elegans draft assembly, Zenodo 7526395)
- Long-read sequencing file in FASTQ (gzip-compressed, e.g. SRR10028109.fastq.gz)

## Outputs

- PAF-formatted long-read to contig mapping file named {target}.k{k}.w{w}.z{z}.paf (e.g. celegans-abyss.fa.k32.w100.z1000.paf) with per-line query read name, target contig name, and mapping coordinates

## How to apply

Place the draft assembly (e.g. the ABySS C. elegans assembly celegans-abyss.fa from Zenodo 7526395) and the long-read file (SRR10028109.fastq.gz) in the current working directory, as ntLink requires both inputs locally. Run the ntLink pairing stage with the `pair` target, setting k=32, w=100, z=1000, t=5 threads, and enabling sensitive=True to broaden minimizer-based matching and paf=True to emit PAF-style output. ntLink computes ordered minimizer sketches of the target assembly and reads, then uses the minimizers to map reads to contigs instead of running full scaffolding. The tool writes the mappings to an intermediate file named with the default prefix encoding the parameters, i.e. celegans-abyss.fa.k32.w100.z1000.paf. Validate that this PAF-like file exists and each line carries the query read name, target contig name, and mapping coordinates, per the ntLink wiki's output file format documentation.

## Related tools

- **ntLink** (Runs in pair mode with sensitive=True and paf=True to produce PAF long-read to contig mappings via minimizer sketches) — https://github.com/bcgsc/ntLink
- **btllib** (Dependency library (1.7.1 or higher) required to run ntLink) — https://github.com/BirolLab/btllib

## Examples

```
ntLink pair target=celegans-abyss.fa reads=SRR10028109.fastq.gz k=32 w=100 z=1000 t=5 sensitive=True paf=True
```

## Evaluation signals

- Output file celegans-abyss.fa.k32.w100.z1000.paf exists in the working directory after the run
- The output is PAF-like: each line contains a query read name, target contig name, and mapping coordinate fields
- Query names in the PAF correspond to SRR10028109 reads and target names to celegans-abyss.fa contigs
- Output filename reflects the chosen parameters ({target}.k{k}.w{w}.z{z}.paf), confirming k=32, w=100, z=1000 were applied
- The file is non-empty, i.e. contains read-to-contig mappings for the supplied long reads

## Limitations

- Mappings are minimizer-based and lightweight, suited to downstream applications like misassembly detection, not a substitute for sensitive aligner output
- ntLink expects the target assembly and reads in the current working directory, constraining where the run can be launched
- Output file format details (PAF-like fields) are documented only on the ntLink wiki, so schema validation may require consulting that external page

## Evidence

- [intro] In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool: "In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool"
- [methods] Use minimizers to map the long reads to the target assembly contigs: "2. Use minimizers to map the long reads to the target assembly contigs"
- [abstract] ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies: "ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies"
- [readme] See our wiki for more information about output file formats.: "See our [wiki](https://github.com/BirolLab/ntLink/wiki) for more information about output file formats."
