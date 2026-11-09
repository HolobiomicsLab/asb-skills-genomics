---
name: paf-alignment-format-handling
description: Use when long reads must be mapped to draft assembly contigs in PAF format
  via minimizer sketches instead of a full read aligner, e.g. as input for Tigmint-long
  correction or misassembly detection.
when_to_use_negative:
- The goal is full scaffolding (scaffold graph, ordering/orientation of contigs) rather
  than standalone read-to-contig mappings — use ntLink's basic scaffolding protocols
  instead.
- You need base-level aligned alignments with mismatch/CIGAR detail rather than minimizer-based
  mappings — an aligner like minimap2 is more appropriate.
- Input assemblies or reads are not present in the current working directory, since
  ntLink's pair/mapping stage expects both target and reads locally.
edam_operation: http://edamontology.org/operation_0292
edam_topics: []
tools:
- name: ntLink
  role: Run in pair/mapping mode with paf=True to produce minimizer-based long-read
    to contig mappings in PAF format
  repo: https://github.com/bcgsc/ntLink
- name: minimap2
  role: Alternative PAF-producing read-to-assembly aligner listed among the article's
    associated repositories
  repo: https://github.com/lh3/minimap2
- name: ABySS
  role: Source of the draft assembly input (celegans-abyss.fa) used in the mapping
    protocol
  repo: https://github.com/BirolLab/abyss
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:88715e0d5113481ecd83a8627ae1ee77f0632c177f029ba67404e387905943ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - ABySS
  - minimap2
  - ntLink
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# paf-alignment-format-handling

## Summary

Generating and handling PAF-format long-read to contig mappings using ntLink's standalone minimizer-based mapping mode (Alternate Protocol 1), which reuses the scaffolder's lightweight mapping machinery without full alignment.

## When to use

Use when long reads must be mapped to draft assembly contigs in PAF format via minimizer sketches instead of a full read aligner, e.g. as input for Tigmint-long correction or misassembly detection.

## When NOT to use

- The goal is full scaffolding (scaffold graph, ordering/orientation of contigs) rather than standalone read-to-contig mappings — use ntLink's basic scaffolding protocols instead.
- You need base-level aligned alignments with mismatch/CIGAR detail rather than minimizer-based mappings — an aligner like minimap2 is more appropriate.
- Input assemblies or reads are not present in the current working directory, since ntLink's pair/mapping stage expects both target and reads locally.

## Inputs

- Draft assembly FASTA file (e.g. celegans-abyss.fa, ABySS C. elegns draft assembly, Zenodo 7526395)
- Long-read sequencing file (e.g. SRR10028109.fastq.gz, FASTQ/FASTA)

## Outputs

- PAF-formatted long-read to contig mapping file named <target>.k<k>.w<w>.z<z>.paf (e.g. celegans-abyss.fa.k32.w100.z1000.paf)

## How to apply

Place the draft assembly (e.g. the ABySS C. elegans assembly celegans-abyss.fa, Zenodo 7526395) and the long-read file (e.g. SRR10028109.fastq.gz) in the current working directory, as ntLink requires both inputs locally. Run ntLink in pair/mapping mode with the `pair` target and set `paf=True` to request PAF-formatted output; typical parameters from the protocol are k=32, w=100, z=1000, t=5, sensitive=True. ntLink computes ordered minimizer sketches of the target assembly and reads, then uses the minimizers to map the long reads to the target assembly contigs, emitting the file celegans-abyss.fa.k32.w100.z1000.paf under the default intermediate-file prefix. Validate that the output is PAF-like (query read name, target contig name, and mapping coordinates per line) and that mappings correspond to the input reads against the assembly contigs.

## Related tools

- **ntLink** (Run in pair/mapping mode with paf=True to produce minimizer-based long-read to contig mappings in PAF format) — https://github.com/bcgsc/ntLink
- **minimap2** (Alternative PAF-producing read-to-assembly aligner listed among the article's associated repositories) — https://github.com/lh3/minimap2
- **ABySS** (Source of the draft assembly input (celegans-abyss.fa) used in the mapping protocol) — https://github.com/BirolLab/abyss

## Examples

```
ntLink pair target=celegans-abyss.fa reads=SRR10028109.fastq.gz k=32 w=100 z=1000 t=5 sensitive=True paf=True
```

## Evaluation signals

- The output file celegans-abyss.fa.k32.w100.z1000.paf exists in the working directory after the ntLink pair run.
- Each line of the .paf file contains PAF-like fields: query read name, target contig name, and mapping coordinates.
- Mapped query names in the PAF correspond to SRR10028109 long reads and target names to celegans-abyss.fa contigs.
- The file prefix matches the ntLink parameterization (k=32, w=100, z=1000), confirming the expected intermediate-file naming.
- Mappings can be consumed downstream (e.g. by Tigmint-long) as PAF-format input without reformatting.

## Limitations

- ntLink mappings are minimizer-based, not full read alignments, so they lack base-level alignment detail (mismatches, CIGARs).
- Both the target assembly and the reads must be present in the current working directory for ntLink mapping mode.
- Output file formats and naming are governed by the tool's defaults (prefix <target>.k<w>.w<z>.z<z>.paf); consult the ntLink wiki for format details.

## Evidence

- [intro] In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool: "In Alternate Protocols 1 and 2, we highlight the usage of ntLink as a mapping tool"
- [abstract] the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection: "the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection"
- [methods] 2. Use minimizers to map the long reads to the target assembly contigs: "2. Use minimizers to map the long reads to the target assembly contigs"
- [readme] See our [wiki](https://github.com/BirolLab/ntLink/wiki) for more information about output file formats.: "See our [wiki](https://github.com/BirolLab/ntLink/wiki) for more information about output file formats."
