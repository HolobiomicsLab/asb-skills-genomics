---
name: linked-read-simulation-benchmarking
description: Use when you need to quantitatively compare NRS insertion callers on
  linked-read data where no truth set exists, by simulating a ground-truth genome
  with known insertions and scoring calls.
when_to_use_negative:
- You have a real (non-simulated) cohort with an independent truth set (e.g. GIAB
  HG002 or T2T CHM13) — benchmark directly against that instead of simulating.
- The caller under test cannot consume barcoded linked-reads and you have no means
  to generate matched short-read data (as ART does for Pamir).
- You only care about insertions shorter than 300 bp — the Novel-X analysis explicitly
  restricts to insertions longer than 300 bp with anchor sums exceeding 300 bp, so
  small-insertion recall will be structurally limited.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LRSIM
  role: Simulates 65x-coverage barcoded synthetic long (linked) reads from the ground-truth
    genome with 2000 implanted insertions
- name: ART
  role: Simulates matched standard short-read data at the same coverage for the Pamir
    baseline, which cannot consume linked-reads
- name: Novel-X
  role: Primary NRS insertion caller run on the LongRanger-aligned linked-read BAM
  repo: github.com/1dayac/Novel-X
- name: NUI
  role: Synthetic long-read NRS insertion baseline run on the same linked-read data
- name: PopIns2
  role: Short-read insertion-calling baseline run on the same linked-read data
- name: Pamir
  role: Short-read NRS baseline run on ART-simulated short reads, including contamination
    removal as in Pamir
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:fb7ff43853b21ea0a40607bc5fe3b57c46b378cfc1bdc0d1464ec77f02614f68
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - LRSIM
  - NUI
  - Novel-X
  - Pamir
  - PopIns2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# linked-read-simulation-benchmarking

## Summary

Simulate barcoded synthetic long-read (linked-read) benchmarks with planted non-reference sequence (NRS) insertions and compute precision/recall for insertion callers such as Novel-X, NUI, PopIns2, and Pamir, including size-bin breakdowns.

## When to use

Use when you need to quantitatively compare NRS insertion callers on linked-read data where no truth set exists, by simulating a ground-truth genome with known insertions and scoring calls.

## When NOT to use

- You have a real (non-simulated) cohort with an independent truth set (e.g. GIAB HG002 or T2T CHM13) — benchmark directly against that instead of simulating.
- The caller under test cannot consume barcoded linked-reads and you have no means to generate matched short-read data (as ART does for Pamir).
- You only care about insertions shorter than 300 bp — the Novel-X analysis explicitly restricts to insertions longer than 300 bp with anchor sums exceeding 300 bp, so small-insertion recall will be structurally limited.

## Inputs

- Reference genome FASTA (e.g. GRCh38)
- Randomly generated insertion sequences of 50-6000 bp (2000 events) implanted into the reference to form the ground-truth genome
- LRSIM configuration for 65x barcoded synthetic long-read simulation
- ART configuration for matched short-read simulation (for short-read-only baselines like Pamir)
- LongRanger-aligned linked-read BAM with barcode tags

## Outputs

- benchmark_results.csv with precision and recall per tool (Novel-X, NUI, PopIns2, Pamir), including insertion-size-bin breakdowns
- VCF files of each caller's insertion calls (analogous to the 'simulated' VCFs in the novel_insertions_supplementary repository)
- Match tables of predicted vs. 2000 ground-truth insertions under the 100 bp breakpoint / 5% length criterion

## How to apply

Insert 2000 randomly generated DNA sequences of 50-6000 bp into a reference genome (e.g. GRCh38) to create a simulated ground-truth genome, then use LRSIM to simulate 65x-coverage barcoded synthetic long reads from that genome. For baselines that cannot consume linked-reads (e.g. Pamir), simulate matched short-read data at the same coverage with ART. Align the linked reads with LongRanger and run each caller on its appropriate input (Novel-X, NUI, and PopIns2 on the linked-read BAM; Pamir on the ART short-read data). Match each tool's calls to the ground-truth insertions using the criterion that breakpoint positions differ by no more than 100 bp and insertion lengths differ by no more than 5%, then compute precision and recall per tool, including breakdowns by insertion-size bins. For read-level filtering consistency, apply caller-specific thresholds such as average base quality > 20 and discarding contigs recruiting fewer than t=5 barcodes.

## Related tools

- **LRSIM** (Simulates 65x-coverage barcoded synthetic long (linked) reads from the ground-truth genome with 2000 implanted insertions)
- **ART** (Simulates matched standard short-read data at the same coverage for the Pamir baseline, which cannot consume linked-reads)
- **Novel-X** (Primary NRS insertion caller run on the LongRanger-aligned linked-read BAM) — github.com/1dayac/Novel-X
- **NUI** (Synthetic long-read NRS insertion baseline run on the same linked-read data)
- **PopIns2** (Short-read insertion-calling baseline run on the same linked-read data)
- **Pamir** (Short-read NRS baseline run on ART-simulated short reads, including contamination removal as in Pamir)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git && longranger align --fastqs simulated_lrsim.fastq.gz --reference=GRCh38 --sample=sim65x
```

## Evaluation signals

- Reproduced precision values match the reported numbers: Novel-X 0.999, NUI 0.997, PopIns2 0.999, Pamir 0.817.
- Novel-X shows the best overall recall and precision among the compared tools on the simulated linked-read set.
- Every reported call can be paired to one of the 2000 ground-truth insertions with breakpoint difference <= 100 bp and length difference <= 5%.
- Precision/recall are additionally stratified by insertion size bins and remain internally consistent (precision + false-discovery accounting sums correctly per bin).
- Downsampled variants (e.g. removing 80/60/40/20 percent of reads as in the supplementary simulated VCFs) show monotonically decreasing recall with coverage.

## Limitations

- Simulated data do not capture all real-genome artifacts; conclusions should be cross-checked on real haploid (CHM1, CHM13) and diploid (NA12878, HG002) linked-read datasets.
- Novel-X needs a substantial number of distinct barcodes per insertion; small insertions usually have too few associated barcodes and insufficient coverage for anchor assembly, depressing small-bin recall.
- Downstream matching depends on assembly quality of the underlying contigs; filtering steps such as the QUAST-like best-alignment-subset selection and discarding orphan contigs recruiting fewer than t=5 barcodes affect sensitivity.
- The simulated design tests non-reference insertions only; other classes of complex structural variants are not covered by this benchmark.

## Evidence

- [abstract] We propose a novel integrated alignment-based and local assembly-based algorithm, Novel-X, that uses the barcode information encoded in synthetic long reads: "We propose a novel integrated alignment-based and local assembly-based algorithm, Novel-X, that uses the barcode information encoded in synthetic long reads"
- [intro] Here we evaluated our method on several cohorts of synthetic long reads sequenced on 10x Genomics, TELL-Seq and stLFR platforms.: "Here we evaluated our method on several cohorts of synthetic long reads sequenced on 10x Genomics, TELL-Seq and stLFR platforms."
- [methods] Each contig that recruits less than t barcodes is discarded ... we set the parameter t to 5 as default: [evidence span withheld — anchor status: not_found]
- [methods] a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs: "a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs"
- [readme] Results folder contains all vcf files that we used for benchmarks that generated Figures 4-5 and Tables S1-S3 from the article.: [evidence span withheld — anchor status: external_doc]
- [readme] Folder simulated contains different VCFs: * No postfix - simple simulated dataset created using LRsim.: [evidence span withheld — anchor status: external_doc]
