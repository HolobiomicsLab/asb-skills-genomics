---
name: long-read-genome-assembly-quality-assessment
description: Use when a set of corrected long reads (e.g., HECIL output LongRead_Corrected.fasta)
  must be benchmarked against baselines using assembly-based metrics (N50 via QUAST,
  dnadiff accuracy) plus k-mer and alignment metrics.
when_to_use_negative:
- Input is raw, uncorrected PacBio/Oxford Nanopore reads — this skill scores corrected
  read sets, so run a hybrid corrector (e.g., HECIL) first
- No reference genome is available for the organism, since the BLASR identity metrics
  and MUMmer dnadiff accuracy assessment both require aligning reads/assemblies to
  a reference
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0080
tools:
- name: Canu
  role: De novo assembly of the corrected long reads, specifically designed for noisy
    long reads
- name: QUAST
  role: Evaluates assembly quality, reporting N50 for each corrected read set's assembly
- name: MUMmer (dnadiff)
  role: Measures assembly accuracy by aligning the assembled genome to the reference
    genome
- name: Jellyfish
  role: K-mer counting tool computing the number of unique k-mers obtained after each
    correction algorithm
- name: BLASR
  role: Aligns corrected long reads to the reference genome to derive PI, percent
    matched bases, and number of aligned reads
- name: HECIL
  role: Produces the corrected long-read sets (quick correction at eta = 0.9 plus
    optimization-based correction) that are then assessed
  repo: https://github.com/NDBL/HECIL
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:b8fb16f50f708de745b0eb37a5a3f5127a76baa313dc3cbcfbe938307f84e2c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BLASR
  - BWA-MEM
  - Canu
  - CoLoRMap
  - HECIL
  - Jellyfish
  - LSC
  - LoRDEC
  - MUMmer (dnadiff)
  - PacBioToCA
  - QUAST
  - proovread
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# long-read-genome-assembly-quality-assessment

## Summary

Benchmarks error-corrected long-read sets by assembling them with Canu and scoring assembly quality (QUAST N50) and accuracy (MUMmer dnadiff), complemented by alignment-based metrics from Jellyfish unique k-mer counts and BLASR. Used in HECIL to show corrected reads outperform LSC, PacBioToCA, LoRDEC, proovread, and CoLoRMap.

## When to use

Use when a set of corrected long reads (e.g., HECIL output LongRead_Corrected.fasta) must be benchmarked against baselines using assembly-based metrics (N50 via QUAST, dnadiff accuracy) plus k-mer and alignment metrics.

## When NOT to use

- Input is raw, uncorrected PacBio/Oxford Nanopore reads — this skill scores corrected read sets, so run a hybrid corrector (e.g., HECIL) first
- No reference genome is available for the organism, since the BLASR identity metrics and MUMmer dnadiff accuracy assessment both require aligning reads/assemblies to a reference

## Inputs

- Corrected long-read FASTA files (e.g., HECIL LongRead_Corrected.fasta for iteration 1 and iteration 5, plus baseline corrector outputs)
- Reference genome FASTA (e.g., E. coli K-12 substr. MG1655, S. cerevisiae S288C 12.2 Mbp, or A. funestus Fumoz GenBank GCA_000349085.1)
- Uncorrected PacBio long reads (E. coli set filtered to reads >= 100 bp: 33,360 reads)

## Outputs

- Canu assemblies of each corrected read set
- QUAST assembly-quality report including N50 per read set
- MUMmer dnadiff accuracy report (assembled genome vs. reference genome)
- Jellyfish unique k-mer counts per corrected read set
- BLASR alignment statistics: percentage identity (PI), percent matched bases, and number of aligned reads
- Benchmark table (e.g., hecil_ecoli_benchmark.csv) comparing HECIL iterations 1 and 5 to the five baselines

## How to apply

Assemble each corrected long-read set de novo with Canu, the assembler specifically designed for noisy long reads, then evaluate assembly quality with QUAST (report N50). Measure assembly accuracy by aligning the assembled genome back to the reference genome (e.g., E. coli K-12 MG1655, S. cerevisiae S288C, or A. funestus GCA_000349085.1) with MUMmer's dnadiff tool. Complement the assembly-based evaluation with alignment-based metrics: count unique k-mers per corrected set with Jellyfish and align the corrected long reads to the reference genome with BLASR to record percentage identity, percent matched bases, and the number of aligned reads. Tabulate every metric for HECIL (iteration 1 and iteration 5 corrected read sets) alongside LSC, PacBioToCA, LoRDEC, proovread, and CoLoRMap so that 'lowest unique k-mer count' and 'highest identity/N50' identify the best corrector. For E. coli, evaluate only after filtering long reads shorter than 100 bp, which leaves 33,360 reads paired with 22,720,100 Illumina short reads.

## Related tools

- **Canu** (De novo assembly of the corrected long reads, specifically designed for noisy long reads)
- **QUAST** (Evaluates assembly quality, reporting N50 for each corrected read set's assembly)
- **MUMmer (dnadiff)** (Measures assembly accuracy by aligning the assembled genome to the reference genome)
- **Jellyfish** (K-mer counting tool computing the number of unique k-mers obtained after each correction algorithm)
- **BLASR** (Aligns corrected long reads to the reference genome to derive PI, percent matched bases, and number of aligned reads)
- **HECIL** (Produces the corrected long-read sets (quick correction at eta = 0.9 plus optimization-based correction) that are then assessed) — https://github.com/NDBL/HECIL

## Evaluation signals

- Corrected sets should yield a lower Jellyfish unique k-mer count than the uncorrected reads and the competing baselines; HECIL's core algorithm (iteration 1) produced the lowest number of unique k-mers for the E. coli and S. cerevisiae datasets
- BLASR percentage identity and percent matched bases should increase relative to uncorrected reads, with the number of aligned reads reported per set
- QUAST N50 of the Canu assembly from corrected reads should exceed that of assemblies from raw or baseline-corrected reads
- MUMmer dnadiff alignment of the assembled genome to the reference genome should show high accuracy (fewer mismatches/gaps) for the winning corrector
- Read-set sanity check: for E. coli, the filtered long-read input must contain 33,360 reads (reads < 100 bp excluded)

## Limitations

- Iterative improvement in HECIL is only a conjecture with no mathematical proof; the article notes a rigorous framework 'likely cannot be made without making (possibly unrealistic) assumptions on stochastic' behavior
- Saturation behavior of the iterative procedure is unvalidated speculation (performance is expected to improve 'until saturation' but this was not proven)
- Experiments were run on Dell PowerEdge R815 servers with baselines (proovread, LoRDEC, CoLoRMap) run with 16 threads and HECIL split into 16 concurrent tasks; multi-threaded speedup inside HECIL itself was not implemented
- The supplied HECIL repository lists dependencies as BWA 0.7.15, Samtools 0.1.18, and Python 2.7.8, so the correction step feeding this assessment must run in that environment

## Evidence

- [results] de novo genome assembly of corrected long reads with Canu: "we use the assembler Canu22, specifically designed for noisy long reads"
- [results] QUAST evaluates assembly quality: "We then use QUAST23 to evaluate assembly quality"
- [results] MUMmer dnadiff measures assembly accuracy: [evidence span withheld — anchor status: not_found]
- [results] Jellyfish unique k-mer counts per correction: "We employ the widely-used k-mer counting tool Jellyfish19 to compute the number of unique k-mers obtained after each correction algorithm"
- [readme] HECIL output file for downstream assessment: "The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta"
- [results] BLASR alignment of corrected reads to reference: "we align corrected long reads to its reference genome using BLASR21"
