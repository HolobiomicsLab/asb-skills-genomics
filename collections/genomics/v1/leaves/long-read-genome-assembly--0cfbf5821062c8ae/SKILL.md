---
name: long-read-genome-assembly
description: Use when raw or corrected PacBio/ONT long reads and a reference genome
  are available and genome assemblies must be generated and compared across correction
  methods, or when polishing raw long-read assemblies is required.
when_to_use_negative:
- Input is already an assembly or feature table rather than raw/corrected long reads
  — correction and assembly are not applicable
- Only short reads (Illumina) are available — the skill targets third-generation PacBio/ONT
  long reads
- The goal is de novo assembly without any reference genome — QUAST-LG evaluation
  and NGA50/NGA75 comparison against a reference cannot be performed
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0769
tools:
- name: CONSENT
  role: Self-correction of long reads (--type PB/ONT) and assembly polishing (CONSENT-polish)
    prior to or after assembly
  repo: https://github.com/morispi/CONSENT
- name: Minimap2
  role: Computes overlaps between long reads as the alignment step of the Miniasm
    overlap-layout-consensus assembly pipeline
- name: Miniasm
  role: De novo assembly of raw and corrected long reads via the overlap-layout-consensus
    pipeline
- name: Flye
  role: Assembles raw and CONSENT-corrected H. sapiens chromosome 1 reads
- name: QUAST-LG
  role: Evaluates every assembly against its reference genome (NGA50, NGA75, misassemblies,
    errors per 100 kbp, genome coverage)
- name: RACON
  role: State-of-the-art assembly polisher used as the comparison baseline to CONSENT-polish
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:faec191ae10552338c3251f4ba510d5a6d7008117b8fbb014c40196d5760dd89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Flye
  - HASLR
  - Miniasm
  - Minimap2
  - ONT
  - Pilon
  - QUAST-LG
  - RACON
  - Racon
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# long-read-genome-assembly

## Summary

Assemble long-read datasets (PacBio/ONT FASTA/FASTQ) using the Minimap2+Miniasm overlap-layout-consensus pipeline, with Flye for the H. sapiens chromosome 1 benchmark, then evaluate assemblies against a reference with QUAST-LG (contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, misassemblies).

## When to use

Use when raw or corrected PacBio/ONT long reads and a reference genome are available and genome assemblies must be generated and compared across correction methods, or when polishing raw long-read assemblies is required.

## When NOT to use

- Input is already an assembly or feature table rather than raw/corrected long reads — correction and assembly are not applicable
- Only short reads (Illumina) are available — the skill targets third-generation PacBio/ONT long reads
- The goal is de novo assembly without any reference genome — QUAST-LG evaluation and NGA50/NGA75 comparison against a reference cannot be performed

## Inputs

- PacBio or ONT long reads in FASTA or FASTQ format
- Corrected long reads produced by CONSENT-correct (output FASTA)
- Reference genome for QUAST-LG evaluation
- Raw de novo assembly contigs (e.g., from Miniasm) in FASTA/FASTQ for polishing

## Outputs

- Genome assemblies from Minimap2+Miniasm (raw and corrected reads) and Flye (H. sapiens chr 1)
- QUAST-LG evaluation table with contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, and misassemblies (e.g., Table 3)
- Polished assembly FASTA from CONSENT-polish and a CONSENT-vs-RACON quality/runtime/memory metrics table (e.g., polishing_benchmark_metrics.tsv)

## How to apply

Run CONSENT-correct on the raw long reads with --type PB for PacBio or --type ONT for Oxford Nanopore data to obtain corrected reads. Assemble the raw reads and each tool's corrected reads with the Minimap2+Miniasm overlap-layout-consensus pipeline; additionally assemble H. sapiens chromosome 1 reads with Flye. To improve assembly quality, polish raw de novo assemblies with CONSENT-polish using default parameters (windowSize 500, minSupport 4, maxSupport 150, merSize k=9, solid 4, anchorSupport 8, minAnchors 2, windowOverlap 50), comparing against RACON under equivalent conditions. Evaluate every assembly against its reference genome with QUAST-LG and tabulate contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts. Validation hinges on whether corrected reads improve assembly metrics: in the article, only CONSENT-corrected reads reached larger NGA50 and NGA75 than raw reads in Flye assemblies.

## Related tools

- **CONSENT** (Self-correction of long reads (--type PB/ONT) and assembly polishing (CONSENT-polish) prior to or after assembly) — https://github.com/morispi/CONSENT
- **Minimap2** (Computes overlaps between long reads as the alignment step of the Miniasm overlap-layout-consensus assembly pipeline)
- **Miniasm** (De novo assembly of raw and corrected long reads via the overlap-layout-consensus pipeline)
- **Flye** (Assembles raw and CONSENT-corrected H. sapiens chromosome 1 reads)
- **QUAST-LG** (Evaluates every assembly against its reference genome (NGA50, NGA75, misassemblies, errors per 100 kbp, genome coverage))
- **RACON** (State-of-the-art assembly polisher used as the comparison baseline to CONSENT-polish)

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB && ./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta
```

## Evaluation signals

- Only CONSENT-corrected reads should yield larger NGA50 and NGA75 than the raw-read Flye assembly; other correctors should not (Table 3 expectation)
- QUAST-LG metrics for each assembly (contigs, NGA50, NGA75, genome coverage, errors per 100 kbp, misassemblies) match the reported Table 3 values within expected variation
- CONSENT-polish produces polishing quality comparable to RACON while being 3x to 38x faster (e.g., ~30 sec vs 5 min 55 sec on E. coli 60x) and using up to ~4x less memory (~150 MB peak RAM for the example run)
- Corrected-read output from CONSENT-correct is valid FASTA and the polishing pipeline exits with default parameters intact (windowSize 500, minSupport 4, maxSupport 150)
- On the human dataset, assemble-then-polish should be less resource consuming than correct-then-assemble while providing comparable or better results

## Limitations

- CONSENT's correction step runtime tends to be higher than other state-of-the-art methods
- Selecting only the N highest identity overlaps per alignment pile may prove inefficient when repeated regions grow longer
- Window size and minimum number of anchors parameters significantly impact runtime; an optimal runtime/quality compromise was not established
- CONSENT assembly covers a smaller proportion of the reference sequence, and unaligned contigs were not analyzed in detail
- Correcting a full human dataset requires ~5 TB of disk space, driven by the overlaps file (which can be stored on an external hard drive)

## Evidence

- [abstract] CONSENT implements a polishing feature, allowing to correct raw assemblies.: "CONSENT implements a polishing feature, allowing to correct raw assemblies."
- [abstract] CONSENT correction improves Flye assemblies: "our experiments show that error correction with CONSENT improves the quality of Flye assemblies."
- [abstract] CONSENT faster than other polishing tools with comparable results: "CONSENT is 2-38x times faster than other polishing tools, while providing comparable results."
- [results] Only CONSENT-corrected reads improve Flye NGA50/NGA75: [evidence span withheld — anchor status: not_found]
- [readme] Polishing parameters and example run: "./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta"
- [readme] Polishing resource profile: "This should take about 15 sec and use at most 150 MB of RAM, using 4 cores."
- [methods] Polishing default parameters: "--windowSize INT, -l INT:      Size of the windows to process. (default: 500)"
