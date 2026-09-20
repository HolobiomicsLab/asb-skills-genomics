---
name: assembly-quality-assessment-quast
description: Use when you have long-read assemblies (raw Miniasm/Flye assemblies,
  CONSENT-polished, or RACON-polished) plus a reference genome and need standardized
  metrics (NGA50, NGA75, misassemblies, genome fraction, errors per 100 kbp) to compare
  correction or polishing tools.
when_to_use_negative:
- No reference genome is available — QUAST-LG metrics such as genome fraction and
  NGA50/NGA75 are reference-based and cannot be computed
- Input is raw unassembled long reads rather than contigs — use a self-correction
  skill (CONSENT-correct) instead of assembly assessment
- Evaluating a polishing run solely by runtime/memory without QUAST-LG quality metrics
  — the article's validation requires comparable error-correction quality, not speed
  alone
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST-LG
  role: Evaluates every raw and polished assembly against each reference genome to
    obtain misassemblies, error counts, genome fraction, and NGAx metrics
- name: CONSENT
  role: Produces polished assemblies (CONSENT-polish) whose QUAST-LG metrics are compared
    against RACON; also supplies corrected reads assembled for the raw-vs-corrected
    comparison
  repo: https://github.com/morispi/CONSENT
- name: Minimap2
  role: Powers the Miniasm overlap step when assembling raw and corrected benchmark
    reads into assemblies to be assessed
- name: Miniasm
  role: De novo OLC assembly of raw and each tool's corrected reads for all real datasets
    prior to QUAST-LG evaluation
- name: Flye
  role: Assembles raw and corrected H. sapiens chromosome 1 reads; its assemblies
    are QUAST-LG-evaluated to test whether correction improves NGA50/NGA75
- name: RACON
  role: State-of-the-art polishing comparator; its polished assemblies are evaluated
    with the same QUAST-LG metrics for quality, runtime, and memory comparison
merged_aliases:
- quast-assembly-quality-assessment
merged_alias_records:
- alias: assembly-quality-assessment-with-quast
  slug: assembly-quality-assessment-with-quast
  jaccard_score: 1.0
  method: token-set-jaccard
  decision: auto
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:5f21353b8f06ca4329b680951be6d2ea269b20c72f6b4caa0f1ec301cacb0610
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Canu
  - Flye
  - Miniasm
  - Minimap2
  - ONT
  - QUAST
  - QUAST-LG
  - RACON
  - VeChat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# assembly-quality-assessment-quast

## Summary

Benchmark long-read assembly quality by evaluating raw, polished (CONSENT-polish vs RACON), and corrected-read-derived assemblies (Miniasm, Flye) with QUAST-LG against reference genomes, comparing NGA50/NGA75, misassemblies, genome fraction, and polishing runtime/memory.

## When to use

Use when you have long-read assemblies (raw Miniasm/Flye assemblies, CONSENT-polished, or RACON-polished) plus a reference genome and need standardized metrics (NGA50, NGA75, misassemblies, genome fraction, errors per 100 kbp) to compare correction or polishing tools.

## When NOT to use

- No reference genome is available — QUAST-LG metrics such as genome fraction and NGA50/NGA75 are reference-based and cannot be computed
- Input is raw unassembled long reads rather than contigs — use a self-correction skill (CONSENT-correct) instead of assembly assessment
- Evaluating a polishing run solely by runtime/memory without QUAST-LG quality metrics — the article's validation requires comparable error-correction quality, not speed alone

## Inputs

- Raw PacBio or ONT long reads in FASTA/FASTQ (run CONSENT-correct with --type PB or --type ONT for corrected-read variants)
- Raw assemblies from Minimap2+Miniasm (all datasets) and Flye (H. sapiens chr 1)
- Polished assemblies from CONSENT-polish and RACON
- Reference genome FASTA for each organism (e.g. E. coli 60x, human chr 1)

## Outputs

- QUAST-LG evaluation report per assembly: contig counts, NGA50, NGA75, genome coverage/fraction, misassemblies, errors per 100 kbp
- Compiled comparison tables reproducing Table 3 (raw vs corrected-read assemblies) and Table 4 (CONSENT-polish vs RACON), e.g. polishing_benchmark_metrics.tsv
- Runtime, core count, and peak-RAM logs per polishing run

## How to apply

Assemble each raw PacBio/ONT long-read dataset (FASTA/FASTQ) with the Minimap2+Miniasm overlap-layout-consensus pipeline, and H. sapiens chromosome 1 reads additionally with Flye; optionally polish raw assemblies with CONSENT-polish (defaults: windowSize 500, minSupport 4, maxSupport 150, merSize 9, solid 4, anchorSupport 8, minAnchors 2, windowOverlap 50) and with RACON as the state-of-the-art comparator. Evaluate every raw and polished assembly against its reference genome with QUAST-LG to obtain misassemblies, error counts, genome fraction, NGA50, NGA75, and contig counts. Record wall-clock time, core count (4 cores where specified, e.g. ~15 sec / 150 MB RAM for a CONSENT-polish run), and peak RAM per run. Compile per-dataset metrics into a comparison table (e.g. polishing_benchmark_metrics.tsv) reproducing the article's Tables 3 and 4. A polisher is judged comparable if QUAST-LG quality metrics match the comparator within expected variation while runtime/memory are lower (CONSENT reported 2–38x faster, up to ~4x less memory than RACON); a correction method is validated if only its corrected reads improve Flye NGA50/NGA75 over the raw-read assembly baseline.

## Related tools

- **QUAST-LG** (Evaluates every raw and polished assembly against each reference genome to obtain misassemblies, error counts, genome fraction, and NGAx metrics)
- **CONSENT** (Produces polished assemblies (CONSENT-polish) whose QUAST-LG metrics are compared against RACON; also supplies corrected reads assembled for the raw-vs-corrected comparison) — https://github.com/morispi/CONSENT
- **Minimap2** (Powers the Miniasm overlap step when assembling raw and corrected benchmark reads into assemblies to be assessed)
- **Miniasm** (De novo OLC assembly of raw and each tool's corrected reads for all real datasets prior to QUAST-LG evaluation)
- **Flye** (Assembles raw and corrected H. sapiens chromosome 1 reads; its assemblies are QUAST-LG-evaluated to test whether correction improves NGA50/NGA75)
- **RACON** (State-of-the-art polishing comparator; its polished assemblies are evaluated with the same QUAST-LG metrics for quality, runtime, and memory comparison)

## Evaluation signals

- QUAST-LG output for every assembly contains the expected metrics: contig counts, NGA50, NGA75, genome fraction, misassemblies, and errors per 100 kbp against the correct reference genome
- CONSENT-polish quality is comparable to RACON within expected variation while runtime is 2–38x faster (e.g. 30 sec vs 5 min 55 sec on E. coli 60x) and memory up to ~4x lower
- In the corrected-reads comparison, only the CONSENT-corrected-read assembly improves Flye NGA50 and NGA75 over the raw-read assembly
- Resource log matches reported conditions: 4 cores, and example CONSENT-polish footprint of ~15 sec wall-clock and at most 150 MB RAM
- Metrics table values reproduce the article's Table 3/4 figures within expected variation across simulated and real datasets

## Limitations

- CONSENT assembly covers a smaller proportion of the reference sequence, and unaligned contigs were not analyzed in detail in the article
- QUAST-LG comparisons are anchored to specific references and datasets (simulated and real PacBio/ONT benchmarks, human chr 1); findings may not transfer to other organisms or assemblers
- Runtime comparisons depend on fixed core counts (4 cores where specified) and will vary with hardware and dataset size (e.g. the full human dataset processing took 10 days)

## Evidence

- [abstract] CONSENT is 2-38x times faster than other polishing tools, while providing comparable results.: "CONSENT is 2-38x times faster than other polishing tools, while providing comparable results."
- [abstract] on a human dataset, assembling the raw data and polishing the assembly is less resource consuming than correcting and then assembling the reads, while providing better results.: "on a human dataset, assembling the raw data and polishing the assembly is less resource consuming than correcting and then assembling the reads, while providing better results."
- [abstract] our experiments show that error correction with CONSENT improves the quality of Flye assemblies.: "our experiments show that error correction with CONSENT improves the quality of Flye assemblies."
- [discussion] CONSENT's assembly polishing feature also offers promising results.: "CONSENT’s assembly polishing feature also offers promising results."
- [readme] This should take about 15 sec and use at most 150 MB of RAM, using 4 cores.: "This should take about 15 sec and use at most 150 MB of RAM, using 4 cores."
