---
name: minimizer-sketch-parameter-configuration
description: Use when a downstream application (e.g. Tigmint-long correction or misassembly
  detection) needs long-read-to-contig mappings from a draft FASTA and long-read FASTQ,
  and alignment-based mapping (e.g. minimap2) is unnecessary or too heavyweight.
when_to_use_negative:
- The goal is de novo scaffolding with overlap detection, gap-filling, or in-code
  iterations (Basic Protocols 1-3) rather than producing standalone read-to-contig
  mappings.
- The assembly draft needs correction with Tigmint-long but no ntLink mapping file
  has been generated yet — Alternate Protocol 2 consumes these mappings rather than
  producing them.
- Short paired-end read data is the input; ntLink's mapping/scaffolding modes are
  designed for long-read sequencing data.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntLink
  role: Computes ordered minimizer sketches and maps long reads onto target assembly
    contigs in pair mode with paf=True output
  repo: https://github.com/bcgsc/ntLink
- name: ABySS
  role: Source of the example draft assembly (celegans-abyss.fa) used as the mapping
    target
  repo: https://github.com/BirolLab/abyss
- name: Tigmint-long
  role: Downstream consumer of the ntLink PAF mappings for genome assembly correction
    (Alternate Protocol 2)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:64f4a201e229b55813a821b6436b46d2d797c4a4978d4e9c7a064e3094fee23c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - ABySS
  - Tigmint-long
  - ntLink
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# minimizer-sketch-parameter-configuration

## Summary

Configure and run ntLink's minimizer-based mapping mode so that ordered minimizer sketches of a draft assembly and long reads are computed and used to map reads to contigs without full read alignment, emitting standard PAF-formatted mappings.

## When to use

Use when a downstream application (e.g. Tigmint-long correction or misassembly detection) needs long-read-to-contig mappings from a draft FASTA and long-read FASTQ, and alignment-based mapping (e.g. minimap2) is unnecessary or too heavyweight.

## When NOT to use

- The goal is de novo scaffolding with overlap detection, gap-filling, or in-code iterations (Basic Protocols 1-3) rather than producing standalone read-to-contig mappings.
- The assembly draft needs correction with Tigmint-long but no ntLink mapping file has been generated yet — Alternate Protocol 2 consumes these mappings rather than producing them.
- Short paired-end read data is the input; ntLink's mapping/scaffolding modes are designed for long-read sequencing data.

## Inputs

- Draft assembly FASTA (e.g. celegans-abyss.fa, ABySS C. elegans draft assembly, Zenodo 7526395)
- Long-read sequencing file (e.g. SRR10028109.fastq.gz, FASTQ, gzipped)

## Outputs

- PAF-formatted long-read to contig mapping file named <target>.k<k>.w<w>.z<z>.paf (e.g. celegans-abyss.fa.k32.w100.z1000.paf)

## How to apply

Place the draft assembly (e.g. the ABySS C. elegans assembly celegans-abyss.fa from Zenodo 7526395) and the long-read file (e.g. SRR10028109.fastq.gz) in the current working directory, as ntLink requires. Run ntLink in pair (mapping) mode with explicit sketch parameters: k=32, w=100, z=1000 (window size and minimizer density control the sketch granularity and mapping sensitivity), threads t=5, and sensitive=True to enable the sensitive mapping stage; set paf=True so mappings are written in PAF-like format. ntLink first computes ordered minimizer sketches of the input target assembly and long reads, then uses the minimizers to map the long reads to the target assembly contigs. The output is a file named with the default intermediate-file prefix, e.g. celegans-abyss.fa.k32.w100.z1000.paf, where each line carries the query read name, target contig name, and mapping coordinates. Validate that the file exists, is PAF-like, and contains mappings for the input reads against the assembly contigs before feeding it to downstream tools such as Tigmint-long.

## Related tools

- **ntLink** (Computes ordered minimizer sketches and maps long reads onto target assembly contigs in pair mode with paf=True output) — https://github.com/bcgsc/ntLink
- **ABySS** (Source of the example draft assembly (celegans-abyss.fa) used as the mapping target) — https://github.com/BirolLab/abyss
- **Tigmint-long** (Downstream consumer of the ntLink PAF mappings for genome assembly correction (Alternate Protocol 2))

## Examples

```
ntLink pair target=celegans-abyss.fa reads=SRR10028109.fastq.gz k=32 w=100 z=1000 t=5 sensitive=True paf=True
```

## Evaluation signals

- The output file celegans-abyss.fa.k32.w100.z1000.paf exists in the working directory after the run.
- Each line of the output contains PAF-like fields: query read name, target contig name, and mapping coordinates.
- Mappings reference reads from SRR10028109 and contigs from celegans-abyss.fa, confirming the correct input pairing.
- The output filename prefix matches the target and the sketch parameters (k32.w100.z1000), confirming the parameters were applied as intended.
- The mapping file is in a format usable by downstream applications such as Tigmint-long misassembly correction.

## Limitations

- ntLink mappings are minimizer-based, not full read alignments; they are suited to scaffolding and lightweight downstream uses such as misassembly detection, not base-pair-precise alignment analyses.
- Both the target assembly and the reads must be placed in the current working directory as required by ntLink.
- Output file formats are documented only in the project wiki, so format details may require consulting https://github.com/BirolLab/ntLink/wiki.

## Evidence

- [other] Running ntLink in pair mode on the celegans-abyss.fa draft assembly with the SRR10028109 long reads (t=5, sensitive=True, paf=True) yields the PAF-formatted mapping file celegans-abyss.fa.k32.w100.z1000.paf in the working directory.: [evidence span withheld — anchor status: not_found]
- [intro] In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool, and, as an example of using ntLink mappings in a different downstream application, showcase how these mappings can: "In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool"
- [abstract] Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds: "ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [methods] 1. Compute ordered minimizer sketches of the input target assembly and long reads: "1. Compute ordered minimizer sketches of the input target assembly and long reads"
- [abstract] the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection: "the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection"
- [abstract] ntLink is an open-source project and is freely available from https://github.com/bcgsc/ntLink: "ntLink is an open-source project and is freely available from https://github.com/bcgsc/ntLink"
