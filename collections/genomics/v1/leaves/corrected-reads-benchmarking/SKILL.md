---
name: corrected-reads-benchmarking
description: 'Use when comparing long-read error-correction tools by their effect
  on assembly quality: you have raw PacBio/ONT reads, one or more corrected read sets,
  a reference genome, and need NGA50/NGA75-style assembly metrics to rank the correctors.'
when_to_use_negative:
- The goal is to compare correction accuracy on the reads themselves rather than its
  downstream effect on assembly quality — this skill only ranks correctors via QUAST-LG
  assembly metrics.
- No reference genome is available for the dataset, since QUAST-LG evaluation against
  the reference is the sole quality signal.
- Only one read set (raw or corrected) is available, making the raw-vs-corrected NGA50/NGA75
  comparison impossible.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: CONSENT
  role: Self-correction of the raw long reads (--type PB for PacBio, --type ONT for
    ONT) to produce one arm of the raw-vs-corrected benchmark
  repo: https://github.com/morispi/CONSENT
- name: Minimap2
  role: Computes overlaps feeding the Miniasm overlap-layout-consensus assembly of
    raw and corrected reads
- name: Miniasm
  role: Overlap-layout-consensus assembly of raw and each tool's corrected reads for
    the three real datasets
- name: Flye
  role: Assembles raw and corrected H. sapiens chromosome 1 reads to test whether
    CONSENT correction improves Flye assemblies
- name: QUAST-LG
  role: Evaluates every assembly against its reference genome to yield contig counts,
    NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:cee14782afd87d032b3cfb9d08c8f61873901110669662aac3ce41064238a9e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Flye
  - Miniasm
  - Minimap2
  - QUAST-LG
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# corrected-reads-benchmarking

## Summary

Benchmark error-corrected long reads by assembling raw and tool-corrected reads (Minimap2+Miniasm OLC pipeline, plus Flye for the H. sapiens chr 1 dataset) and evaluating each assembly with QUAST-LG against its reference genome. This is how the CONSENT paper demonstrates which correction method genuinely improves downstream assembly quality.

## When to use

Use when comparing long-read error-correction tools by their effect on assembly quality: you have raw PacBio/ONT reads, one or more corrected read sets, a reference genome, and need NGA50/NGA75-style assembly metrics to rank the correctors.

## When NOT to use

- The goal is to compare correction accuracy on the reads themselves rather than its downstream effect on assembly quality — this skill only ranks correctors via QUAST-LG assembly metrics.
- No reference genome is available for the dataset, since QUAST-LG evaluation against the reference is the sole quality signal.
- Only one read set (raw or corrected) is available, making the raw-vs-corrected NGA50/NGA75 comparison impossible.

## Inputs

- Raw long reads per benchmark dataset (FASTA/FASTQ; PacBio or ONT technology)
- Corrected reads from each correction tool being benchmarked (FASTA)
- Reference genome(s) for the benchmark datasets (e.g. H. sapiens chromosome 1)

## Outputs

- Assemblies from raw and corrected reads per dataset (contigs FASTA from Miniasm; Flye assembly for chr 1)
- QUAST-LG evaluation table with contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts (Table 3 analogue)

## How to apply

First run CONSENT-correct on each raw long-read set, setting --type PB for PacBio data and --type ONT for Oxford Nanopore data, to obtain the corrected reads. Assemble the raw reads and each tool's corrected reads for all three real datasets with the Minimap2+Miniasm overlap-layout-consensus pipeline, and additionally assemble the raw and corrected H. sapiens chromosome 1 reads with Flye. Evaluate every assembly against its reference genome with QUAST-LG, then tabulate contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts to reproduce the paper's Table 3. The key decision point is the delta-vs-raw comparison: a corrector is judged beneficial only when its corrected reads yield larger NGA50 and NGA75 than the raw-read assembly; in the paper only CONSENT-corrected reads achieved this for Flye.

## Related tools

- **CONSENT** (Self-correction of the raw long reads (--type PB for PacBio, --type ONT for ONT) to produce one arm of the raw-vs-corrected benchmark) — https://github.com/morispi/CONSENT
- **Minimap2** (Computes overlaps feeding the Miniasm overlap-layout-consensus assembly of raw and corrected reads)
- **Miniasm** (Overlap-layout-consensus assembly of raw and each tool's corrected reads for the three real datasets)
- **Flye** (Assembles raw and corrected H. sapiens chromosome 1 reads to test whether CONSENT correction improves Flye assemblies)
- **QUAST-LG** (Evaluates every assembly against its reference genome to yield contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts)

## Examples

```
./CONSENT-correct --in reads.fasta --out correctedReads.fasta --type ONT
```

## Evaluation signals

- QUAST-LG reports NGA50 and NGA75 for each corrected-read assembly; the corrector is beneficial only if both exceed the raw-read assembly's values — per the paper, only CONSENT's corrected reads did so for Flye.
- Table 3 columns are reproduced: contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts for every raw/corrected-by-tool assembly.
- Assemblies exist for all three real datasets under Minimap2+Miniasm and for the H. sapiens chr 1 dataset under Flye, for both raw and every corrected read set.
- CONSENT-correct completed without errors with the technology flag matching the data (--type PB on PacBio, --type ONT on Oxford Nanopore), producing corrected reads in FASTA.

## Limitations

- On the human dataset, assembling the raw data and polishing the assembly is less resource consuming than correcting and then assembling the reads, while providing better results.
- CONSENT's correction step runtime tends to be higher than that of other state-of-the-art methods, and correction of a full human dataset needs about 10 days and roughly 5 TB of disk space, driven by the overlaps file.
- Assembly-based comparison cannot assess correctors on metrics other than assembly quality; Table 3 covers contig counts, NGA50/NGA75, coverage, errors per 100 kbp, and misassemblies only.

## Evidence

- [abstract] Flye comparison rationale: "our experiments show that error correction with CONSENT improves the quality of Flye assemblies."
- [abstract] On a human dataset, assembling raw data and polishing the assembly is less resource consuming than correcting then assembling reads, while providing better results: "on a human dataset, assembling the raw data and polishing the assembly is less resource consuming than correcting and then assembling the reads, while providing better results."
- [readme] CONSENT-correct usage and technology flag: "./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB"
- [readme] readsTechnology flag semantics: "readsTechnology:		Indicate whether the long reads are from PacBio (--type PB) or Oxford Nanopore (--type ONT)"
- [full_text] Benchmark finding on NGA50/NGA75: [evidence span withheld — anchor status: not_found]
