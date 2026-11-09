---
name: assembly-polishing-workflow-execution
description: Use when a study requires polishing raw long-read assemblies (e.g. Miniasm
  outputs from PacBio/ONT reads) and comparing polisher quality, runtime, and memory
  against RACON.
when_to_use_negative:
- The goal is self-correction of reads before assembly rather than correcting an existing
  assembly — CONSENT-correct is the appropriate mode, and the paper notes correction-then-assembly
  is more resource-intensive than polish-then-assemble on human data
- The assembly was not generated from raw long reads (CONSENT polishing targets assemblies
  generated from raw long reads)
- Reads are short reads (Illumina) — the skill is scoped to third-generation PacBio/ONT
  long reads
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: CONSENT
  role: Polishing tool under test; corrects contigs via MSA of overlapping reads plus
    local de Bruijn graph polishing (CONSENT-polish)
  repo: https://github.com/morispi/CONSENT
- name: RACON
  role: State-of-the-art polishing comparator run under equivalent conditions on the
    same raw assemblies
- name: Miniasm
  role: Produces the raw de novo assemblies from PacBio/ONT long reads that are subsequently
    polished
- name: QUAST-LG
  role: Evaluates raw and polished assemblies against reference genomes for misassemblies,
    errors, genome fraction, and NGAx
- name: Minimap2
  role: Required dependency providing read-to-contig overlaps for CONSENT polishing
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:34f88391c2d7b78bc86e967f8245ca253df8833af4246c9bc2315141a44460ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Miniasm
  - Minimap2
  - ONT
  - QUAST-LG
  - RACON
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# assembly-polishing-workflow-execution

## Summary

Benchmark CONSENT's assembly polishing feature against RACON on Miniasm raw assemblies built from PacBio/ONT long reads, measuring correction quality (QUAST-LG misassemblies, errors, genome fraction, NGAx) and resource usage (wall-clock time, peak RAM) to demonstrate CONSENT's 2–38x speed advantage with comparable quality.

## When to use

Use when a study requires polishing raw long-read assemblies (e.g. Miniasm outputs from PacBio/ONT reads) and comparing polisher quality, runtime, and memory against RACON.

## When NOT to use

- The goal is self-correction of reads before assembly rather than correcting an existing assembly — CONSENT-correct is the appropriate mode, and the paper notes correction-then-assembly is more resource-intensive than polish-then-assemble on human data
- The assembly was not generated from raw long reads (CONSENT polishing targets assemblies generated from raw long reads)
- Reads are short reads (Illumina) — the skill is scoped to third-generation PacBio/ONT long reads

## Inputs

- Raw long-read dataset in FASTA or FASTQ format (PacBio --type PB or ONT)
- Raw de novo assembly of the same reads (e.g. Miniasm contigs in FASTA)
- Reference genome for QUAST-LG evaluation
- Optionally simulated read ground truth for error-rate validation

## Outputs

- Polished assembly FASTA per dataset and tool (CONSENT-polish and RACON outputs)
- QUAST-LG quality metrics per assembly: misassemblies, error counts, genome fraction, NGAx
- Per-run resource metrics: wall-clock time, number of cores (4 where specified), peak RAM
- Compiled CONSENT-vs-RACON benchmark table (polishing_benchmark_metrics.tsv)

## How to apply

First assemble each simulated and real long-read dataset (PacBio fasta/fastq or ONT fasta/fastq inputs) de novo with Miniasm to produce raw assemblies per organism and coverage condition. Then run CONSENT in polishing mode (`CONSENT-polish --contigs <raw assembly> --reads <long reads> --out <polished assembly>`) with default parameters: windowSize 500, minSupport 4, maxSupport 150, merSize k=9, solid 4, anchorSupport 8, minAnchors 2, windowOverlap 50, 4 cores. Polish the identical raw assemblies with RACON under equivalent conditions as the state-of-the-art comparator. Evaluate every raw and polished assembly against each reference genome with QUAST-LG to extract misassembly counts, error counts, genome fraction, and NGAx, and log wall-clock time and peak RAM per run. The expected outcome is polishing quality highly comparable to RACON while CONSENT runs ~2x to ~38x faster (e.g. ~30 sec vs 5 min 55 sec on E. coli 60x) and uses up to ~4x less memory; a README reference run takes ~15 sec and 150 MB RAM on 4 cores.

## Related tools

- **CONSENT** (Polishing tool under test; corrects contigs via MSA of overlapping reads plus local de Bruijn graph polishing (CONSENT-polish)) — https://github.com/morispi/CONSENT
- **RACON** (State-of-the-art polishing comparator run under equivalent conditions on the same raw assemblies)
- **Miniasm** (Produces the raw de novo assemblies from PacBio/ONT long reads that are subsequently polished)
- **QUAST-LG** (Evaluates raw and polished assemblies against reference genomes for misassemblies, errors, genome fraction, and NGAx)
- **Minimap2** (Required dependency providing read-to-contig overlaps for CONSENT polishing)

## Examples

```
./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta
```

## Evaluation signals

- QUAST-LG on polished vs raw assemblies shows reduced error counts and improved genome fraction/NGAx for both polishers
- CONSENT-polish quality metrics fall within expected variation of RACON's on every dataset (comparable misassembly and error profiles)
- CONSENT-polish runtime is ~2x to ~38x faster than RACON (e.g. ~30 sec vs 5 min 55 sec on E. coli 60x) and peak RAM up to ~4x lower
- Reference example run completes in ~15 sec using at most 150 MB RAM on 4 cores
- Compiled polishing_benchmark_metrics.tsv reproduces the article's polishing table within expected variation

## Limitations

- CONSENT's correction step (as opposed to polishing) tends to be slower than other state-of-the-art methods
- The strategy of selecting only the N highest identity overlaps may prove inefficient when repeated regions grow long
- Window size and minimum number of anchors strongly affect runtime; an optimal runtime/quality parameter compromise has not been established
- CONSENT assemblies can cover a smaller proportion of the reference; unaligned contigs were not analyzed in detail
- Developed and tested only on x86-64 GNU/Linux; other platforms untested

## Evidence

- [abstract] CONSENT is 2-38x times faster than other polishing tools, while providing comparable results.: "CONSENT is 2-38x times faster than other polishing tools, while providing comparable results."
- [abstract] CONSENT implements a polishing feature, allowing to correct raw assemblies.: "CONSENT implements a polishing feature, allowing to correct raw assemblies."
- [intro] These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate.: "These consensus sequences are then polished with local de Bruijn graphs, in order to correct remaining errors, and further reduce the final error rate."
- [methods] --minSupport INT, -s INT:      Minimum support to consider a window for correction. (default: 4): "--minSupport INT, -s INT:      Minimum support to consider a window for correction. (default: 4)"
- [readme] ./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta: "./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta"
- [readme] This should take about 15 sec and use at most 150 MB of RAM, using 4 cores.: "This should take about 15 sec and use at most 150 MB of RAM, using 4 cores."
