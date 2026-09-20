---
name: nanopore-read-basecalling
description: Use when raw nanopore signals simulated from a reference (e.g. BLOW5/POD5
  from seq2squiggle, squigulator, or DeepSimulator under R9.4.1 or R10.4.1 chemistry)
  must be turned into basecalled reads for variant-detection benchmarking.
when_to_use_negative:
- Input is already basecalled FASTQ/FASTA reads — skip basecalling and proceed directly
  to minimap2 alignment
- Goal is simulating signals from existing reads rather than producing reads from
  signals (seq2squiggle supports this via --read-input)
- Evaluation target is raw-signal similarity to real data rather than downstream read-based
  variant detection
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: seq2squiggle
  role: Generates the synthetic nanopore signals (BLOW5/POD5) that are subsequently
    basecalled into reads; produced the highest true-positive SNV recall/precision
    after basecalling in the benchmark
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: DeepSimulator
  role: Alternative signal simulator run in two configurations whose outputs are basecalled
    for comparison
- name: squigulator
  role: Alternative signal simulator whose outputs are basecalled for comparison
- name: minimap2
  role: Aligns the basecalled reads to hg38 before variant calling
- name: Clair3
  role: Calls SNVs and indels from each basecalled-read alignment
- name: RTGtools
  role: Scores Clair3 callsets against NA12878 truth variants to compute recall and
    precision per simulator
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:edd9b586d85f9dfb14d56462d05f6974603b492f71d4aabed1b9f3f3f11dcbba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clair3
  - DeepSimulator
  - RTGtools
  - mamba
  - minimap2
  - seq2squiggle
  - squigulator
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# nanopore-read-basecalling

## Summary

Converts simulated nanopore raw signals (BLOW5/POD5) into basecalled read sequences so downstream alignment and variant detection can be benchmarked. In the seq2squiggle paper this step bridges signal simulation (seq2squiggle, squigulator, DeepSimulator) to read-based evaluation with minimap2, Clair3, and RTGtools.

## When to use

Use when raw nanopore signals simulated from a reference (e.g. BLOW5/POD5 from seq2squiggle, squigulator, or DeepSimulator under R9.4.1 or R10.4.1 chemistry) must be turned into basecalled reads for variant-detection benchmarking.

## When NOT to use

- Input is already basecalled FASTQ/FASTA reads — skip basecalling and proceed directly to minimap2 alignment
- Goal is simulating signals from existing reads rather than producing reads from signals (seq2squiggle supports this via --read-input)
- Evaluation target is raw-signal similarity to real data rather than downstream read-based variant detection

## Inputs

- Simulated raw nanopore signal files in BLOW5/POD5 format (from seq2squiggle reference mode, squigulator, or DeepSimulator)
- Pretrained model weights / chemistry profile matching the simulated pore (e.g. dna_r9_min for R9.4.1, R10.4.1 profile)
- Modified chr22 reference (hg38 with Genome in a Bottle NA12878 variants applied) for downstream alignment

## Outputs

- Basecalled read sequences (FASTQ/FASTA) per simulator per chemistry
- BAM alignments of basecalled reads to hg38 produced with minimap2
- Clair3 variant callsets per simulator
- variant_calling_benchmark.tsv with true positives, false positives, recall, and precision per simulator

## How to apply

Simulate signals first with seq2squiggle in reference mode (e.g. 250,000 R10.4.1 reads and 50,000 R9.4.1 reads from a modified chr22 reference built by applying Genome in a Bottle NA12878 variants to hg38), selecting the matching chemistry profile such as --profile dna_r9_min. Simulate the same reference with squigulator and with DeepSimulator in two configurations so all simulators feed identically structured signal datasets. Basecall each signal dataset (BLOW5/POD5) into read sequences, then align the basecalled reads to hg38 with minimap2. Call variants from each alignment with Clair3 and score each callset against the NA12878 truth variants with RTGtools, compiling true positives, false positives, recall, and precision per simulator into variant_calling_benchmark.tsv.

## Related tools

- **seq2squiggle** (Generates the synthetic nanopore signals (BLOW5/POD5) that are subsequently basecalled into reads; produced the highest true-positive SNV recall/precision after basecalling in the benchmark) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **DeepSimulator** (Alternative signal simulator run in two configurations whose outputs are basecalled for comparison)
- **squigulator** (Alternative signal simulator whose outputs are basecalled for comparison)
- **minimap2** (Aligns the basecalled reads to hg38 before variant calling)
- **Clair3** (Calls SNVs and indels from each basecalled-read alignment)
- **RTGtools** (Scores Clair3 callsets against NA12878 truth variants to compute recall and precision per simulator)

## Evaluation signals

- Basecalled reads align cleanly to hg38 with minimap2, with read counts consistent with the simulated input (250,000 R10.4.1 reads; 50,000 R9.4.1 reads)
- RTGtools output contains nonempty TP/FP counts plus recall and precision for every simulator's callset, written to variant_calling_benchmark.tsv
- Per the paper's Fig. 4 benchmark, the seq2squiggle-derived callset should show the highest number of true-positive SNVs and overall higher recall and precision than squigulator and DeepSimulator
- The chemistry profile/model used for signal simulation and basecalling matches the pore (R9.4.1 vs R10.4.1) for each dataset

## Limitations

- Basecaller selection and configuration are not specified in the article; only the abstract step 'Basecall all simulated signal datasets into read sequences' is given, so the exact basecaller must be chosen by the practitioner
- Downstream variant-detection quality depends on simulator fidelity; static k-mer-based simulators (squigulator, DeepSimulator) yield fewer true-positive SNVs than seq2squiggle
- Signal simulation runs slowly on CPU; a GPU is recommended, especially for high sample counts

## Evidence

- [other] workflow: [evidence span withheld — anchor status: not_found]
- [results] finding: "Reads generated with seq2squiggle showed a higher number of true-positive SNVs, along with an overall higher recall and precision compared to squigulator and DeepSimulator (Fig. 4)."
- [readme] readme: "`seq2squiggle` is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data."
- [readme] readme-model: [evidence span withheld — anchor status: external_doc]
- [readme] readme-gpu: [evidence span withheld — anchor status: external_doc]
