---
name: minimizer-based-read-mapping
description: Use when you have a draft assembly FASTA (e.g. celegans-abyss.fa) and
  long reads (e.g. SRR10028109 FASTQ.GZ) and need read-to-contig mappings faster than
  minimap2 alignments, in PAF format, for scaffolding or correction.
when_to_use_negative:
- You need base-level alignment detail (CIGAR strings, per-base alignment scores)
  rather than minimizer-based contig-level mappings
- Your goal is scaffolding itself rather than standalone mappings — run the full ntLink
  scaffold pipeline instead of pair/mapping mode
- The read or assembly data are not present in the current working directory, which
  ntLink requires (use soft links)
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: ntLink
  role: Runs the minimizer-based mapping (pair) mode that sketches assembly and reads
    with btllib ntHash2 hashing and outputs PAF-like read-to-contig mappings
  repo: https://github.com/bcgsc/ntLink
- name: btllib
  role: Provides ntHash2-style recursive spaced-seed hashing used for the ordered
    minimizer sketches (btllib 1.7.1 or higher)
  repo: https://github.com/BirolLab/btllib
- name: Tigmint-long
  role: Downstream consumer of ntLink mappings (mapping=ntLink) that detects mapping-discrepancy
    windows and cuts putative misassemblies
  repo: https://github.com/BirolLab/tigmint
- name: ncbi/sra-tools
  role: Downloads the long-read input (e.g. SRR10028109) from SRA
  repo: https://github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_008
  - task_007
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:5a48a2b234d9a5fc3c1d419538b842334a50ebb9bf2de38e7765065dcbc3db16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - Tigmint-long
  - btllib
  - minimap2
  - ncbi/sra-tools
  - ntLink
  - sra-tools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# minimizer-based-read-mapping

## Summary

Use ntLink's lightweight minimizer-based mapping mode (Alternate Protocol 1) to align long reads to draft assembly contigs without full read alignments, outputting PAF-like mappings that can power scaffolding, misassembly detection (Tigmint-long), or other downstream applications.

## When to use

Use when you have a draft assembly FASTA (e.g. celegans-abyss.fa) and long reads (e.g. SRR10028109 FASTQ.GZ) and need read-to-contig mappings faster than minimap2 alignments, in PAF format, for scaffolding or correction.

## When NOT to use

- You need base-level alignment detail (CIGAR strings, per-base alignment scores) rather than minimizer-based contig-level mappings
- Your goal is scaffolding itself rather than standalone mappings — run the full ntLink scaffold pipeline instead of pair/mapping mode
- The read or assembly data are not present in the current working directory, which ntLink requires (use soft links)

## Inputs

- Draft assembly FASTA (e.g. celegans-abyss.fa or celegans_flye.fa)
- Long-read FASTA or FASTQ, gzipped or plain (e.g. SRR10028109.fastq.gz ONT reads)
- ntLink parameters: k=32, w=100, z=1000, t=5, sensitive=True, paf=True

## Outputs

- PAF-like mapping file named <target>.k<w>.w<w>.z<z>.paf (e.g. celegans-abyss.fa.k32.w100.z1000.paf) with per-line query read name, target contig name, and mapping coordinates

## How to apply

Place the target assembly and long-read files (gzipped FASTA/FASTQ allowed) in the working directory, soft-linking if necessary. Run ntLink in pair/mapping mode with the minimizer parameters k=32, w=100, z=1000, threads t=5, optionally enabling sensitive mode for improved mapping and paf=True to emit PAF-like output. ntLink computes ordered minimizer sketches of both the assembly and reads using btllib ntHash2-style recursive spaced-seed hashing, then maps read minimizers to contigs to produce the mapping file named <target>.k<w>.w<w>.z<z>.paf. Validate that each PAF line contains query read name, target contig name, and mapping coordinates, and that mappings cover the reads against the target contigs. These same mappings are what ntLink uses internally to infer contig joins and gap sizes, and what Tigmint-long consumes (mapping=ntLink, t=5, span=2) to cut putative misassemblies.

## Related tools

- **ntLink** (Runs the minimizer-based mapping (pair) mode that sketches assembly and reads with btllib ntHash2 hashing and outputs PAF-like read-to-contig mappings) — https://github.com/bcgsc/ntLink
- **btllib** (Provides ntHash2-style recursive spaced-seed hashing used for the ordered minimizer sketches (btllib 1.7.1 or higher)) — https://github.com/BirolLab/btllib
- **Tigmint-long** (Downstream consumer of ntLink mappings (mapping=ntLink) that detects mapping-discrepancy windows and cuts putative misassemblies) — https://github.com/BirolLab/tigmint
- **ncbi/sra-tools** (Downloads the long-read input (e.g. SRR10028109) from SRA) — https://github.com/ncbi/sra-tools

## Examples

```
ntLink pair target=celegans-abyss.fa reads=SRR10028109.fastq.gz k=32 w=100 z=1000 t=5 sensitive=True paf=True
```

## Evaluation signals

- Output file exists with the expected prefix scheme <target>.k<w>.w<w>.z<z>.paf, e.g. celegans-abyss.fa.k32.w100.z1000.paf
- Each PAF line contains the required fields: query read name, target contig name, and mapping coordinates
- Mappings reference the SRR10028109 reads and the celegans-abyss.fa contigs specifically
- Mappings can be regenerated reproducibly and every downstream scaffold-graph edge is supported by at least the minimum number (a=1) of anchored long-read mappings
- When fed to Tigmint-long, the mappings yield a corrected FASTA (e.g. celegans-abyss.cut500.tigmint.fa) with QUAST metrics consistent with improved contiguity/correctness vs. the baseline

## Limitations

- Mappings are minimizer-based and lightweight rather than full alignments, trading sensitivity/alignment detail for computational efficiency
- ntLink requires all assembly and read files in the current working directory (soft links if necessary)
- For very repetitive or low-diversity regions, default w=100 may miss mappings; sensitive mode can improve mapping at additional cost

## Evidence

- [results] Running ntLink in pair mode on the celegans-abyss.fa draft assembly with the SRR10028109 long reads (t=5, sensitive=True, paf=True) yields the PAF-formatted mapping file celegans-abyss.fa.k32.w100.z1000.paf in the working directory.: [evidence span withheld — anchor status: not_found]
- [abstract] ntLink scaffolds genomes using minimizer-based mappings rather than read alignments to infer ordering and orientation of input sequences.: "Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [methods] The core mechanism maps long-read minimizers to contigs using ntHash2-style hashing with specific k/w/z parameters.: [evidence span withheld — anchor status: not_found]
- [abstract] ntLink's mappings are used downstream by Tigmint-long for misassembly detection and cutting.: "the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection"
