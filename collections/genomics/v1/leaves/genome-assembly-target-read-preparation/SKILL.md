---
name: genome-assembly-target-read-preparation
description: Use when a draft genome assembly (e.g., a FASTA contig set like the ABySS
  C. elegans assembly celegans-abyss.fa) and long reads (e.g., SRR10028109 FASTQ)
  must be turned into read-to-contig PAF mappings for scaffolding or misassembly detection
  without running a full scaffolding protocol.
when_to_use_negative:
- A complete scaffolded assembly is the goal — use ntLink's full scaffolding protocols
  (Basic Protocols 1–3 with overlap detection, gap-filling, or in-code iterations)
  rather than mapping-only pair mode
- The input mappings are already available in PAF format (e.g., from a prior ntLink
  run) — proceed directly to the downstream step such as Tigmint-long correction (Alternate
  Protocol 2)
- High-accuracy base-level alignment is required rather than lightweight minimizer-based
  mappings, e.g., for polishing workflows where full read alignments are preferred
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntLink
  role: Run in pair mode with k=32 w=100 z=1000 t=5 sensitive=True paf=True to compute
    minimizer sketches and map long reads onto target assembly contigs, emitting a
    PAF mapping file
  repo: https://github.com/bcgsc/ntLink
- name: Tigmint-long
  role: Downstream consumer of the ntLink PAF mappings for genome assembly correction
    / misassembly detection (Alternate Protocol 2)
  repo: https://github.com/BirolLab/tigmint
- name: ABySS
  role: Assembler used to produce the draft target assembly (celegans-abyss.fa) used
    as mapping input
  repo: https://github.com/BirolLab/abyss
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:01acaa0061c590a29da0fec8b8c4558f8fb25fb9ed66b257e5bbb54c4729ab7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - ABySS
  - Tigmint-long
  - ntLink
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# genome-assembly-target-read-preparation

## Summary

Prepares a draft genome assembly target and long-read sequencing data for ntLink-based read-to-contig mapping, leveraging ntLink's minimizer-based mapping mode as a standalone alternative to alignment tools like minimap2, producing PAF-formatted mappings for downstream applications such as Tigmint-long misassembly correction.

## When to use

Use when a draft genome assembly (e.g., a FASTA contig set like the ABySS C. elegans assembly celegans-abyss.fa) and long reads (e.g., SRR10028109 FASTQ) must be turned into read-to-contig PAF mappings for scaffolding or misassembly detection without running a full scaffolding protocol.

## When NOT to use

- A complete scaffolded assembly is the goal — use ntLink's full scaffolding protocols (Basic Protocols 1–3 with overlap detection, gap-filling, or in-code iterations) rather than mapping-only pair mode
- The input mappings are already available in PAF format (e.g., from a prior ntLink run) — proceed directly to the downstream step such as Tigmint-long correction (Alternate Protocol 2)
- High-accuracy base-level alignment is required rather than lightweight minimizer-based mappings, e.g., for polishing workflows where full read alignments are preferred

## Inputs

- Draft genome assembly FASTA (e.g., celegans-abyss.fa, ABySS C. elegans draft assembly, Zenodo 7526395)
- Long-read sequencing file (e.g., SRR10028109.fastq.gz) placed in the current working directory

## Outputs

- PAF-formatted long-read to contig mapping file (e.g., celegans-abyss.fa.k32.w100.z1000.paf) with per-line query read name, target contig name, and mapping coordinates

## How to apply

Obtain the target assembly (e.g., celegans-abyss.fa from Zenodo 7526395, an ABySS-built C. elegans draft) and the long-read FASTQ (e.g., SRR10028109.fastq.gz) and place both in the current working directory, as ntLink requires. Run ntLink in pair mode, which exercises the first two steps of the scaffolding workflow — computing ordered minimizer sketches of the target assembly and reads, then using minimizers to map the long reads to the target assembly contigs — instead of proceeding to join inference and scaffold graph output. Key parameters in the documented run are k=32, w=100, z=1000, threads t=5, sensitive=True for increased mapping sensitivity, and paf=True to emit standard PAF-formatted output. The rationale is that ntLink's lightweight minimizer-based mappings infer how sequences relate without costly full alignments, making them suitable for downstream applications such as Tigmint-long assembly correction. Validate that the output file (named under the default prefix, e.g., celegans-abyss.fa.k32.w100.z1000.paf) exists and each line carries query read name, target contig name, and mapping coordinates as expected for PAF.

## Related tools

- **ntLink** (Run in pair mode with k=32 w=100 z=1000 t=5 sensitive=True paf=True to compute minimizer sketches and map long reads onto target assembly contigs, emitting a PAF mapping file) — https://github.com/bcgsc/ntLink
- **Tigmint-long** (Downstream consumer of the ntLink PAF mappings for genome assembly correction / misassembly detection (Alternate Protocol 2)) — https://github.com/BirolLab/tigmint
- **ABySS** (Assembler used to produce the draft target assembly (celegans-abyss.fa) used as mapping input) — https://github.com/BirolLab/abyss

## Examples

```
ntLink pair target=celegans-abyss.fa reads=SRR10028109.fastq.gz k=32 w=100 z=1000 t=5 sensitive=True paf=True
```

## Evaluation signals

- The output file celegans-abyss.fa.k32.w100.z1000.paf exists in the working directory under the default intermediate-file prefix
- Each line of the PAF output contains the expected PAF-like fields: query read name, target contig name, and mapping coordinates
- Mapped queries correspond to the SRR10028109 long reads and targets to the celegans-abyss.fa contigs, not to other sequences
- The input files celegans-abyss.fa and SRR10028109.fastq.gz were present in the current working directory before running, as ntLink requires
- Mappings are sufficient to support downstream use (e.g., Tigmint-long misassembly detection) as described for Alternate Protocol 2

## Limitations

- ntLink's minimizer-based mappings are lightweight and alignment-free rather than full read alignments, so they may not substitute for base-level alignments in all downstream analyses
- Input files must be located in the current working directory as required by ntLink; running from another location can fail
- Running ntLink pair mode emits intermediate files under the default prefix (e.g., celegans-abyss.fa.k32.w100.z1000), so working-directory naming collisions can occur if reusing prefixes

## Evidence

- [methods] Running ntLink in pair mode on the celegans-abyss.fa draft assembly with the SRR10028109 long reads (t=5, sensitive=True, paf=True) yields the PAF-formatted mapping file celegans-abyss.fa.k32.w100.z1000.paf in the working directory.: [evidence span withheld — anchor status: not_found]
- [methods] ntLink pairing stage invocation with minimizer-based mapping to PAF output: [evidence span withheld — anchor status: not_found]
- [abstract] ntLink minimizer-based mappings as foundation of scaffolding and downstream use: "Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [abstract] ntLink lightweight mappings usable for downstream applications such as misassembly detection: "the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection"
- [readme] ntLink is freely available on GitHub: "ntLink is an open-source project and is freely available from https://github.com/bcgsc/ntLink"
