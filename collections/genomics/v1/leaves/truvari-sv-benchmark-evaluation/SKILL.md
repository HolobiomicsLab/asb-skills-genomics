---
name: truvari-sv-benchmark-evaluation
description: Use when SV calls from aligned simulated nanopore BAMs (Sniffles2 output)
  must be quantitatively validated against a GIAB SV truth set, e.g. comparing NanoSimFormer
  vs seq2squiggle vs Squigulator SV fidelity.
when_to_use_negative:
- Input VCFs are SNP/small-indel calls rather than structural variants — these belong
  in the RTG tools vcfeval branch of the benchmark (GIAB v3.3.2 truth sets), not Truvari
  SV benchmarking.
- The source reads are direct-RNA (RNA004) simulations benchmarked with Clair3-RNA
  for SNP/indel performance (SNP 0.92 / indel 0.57), where the SV SV Draft Benchmark
  Set (HG002 DNA) does not apply.
- No matched GIAB truth set exists for the sample being evaluated, making precision/recall/F1
  against a benchmark undefined.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Truvari
  role: Benchmarks called SV VCFs against the GIAB HG002 SV Draft Benchmark Set to
    produce precision/recall/F1
- name: Sniffles2
  role: Detects structural variants from the simulated DNA BAMs prior to Truvari benchmarking
- name: NanoSimFormer
  role: Simulates R10.4.1 DNA signals and basecalls them via Dorado (--basecall, --emit-bam)
    to generate the BAMs whose SV calls are benchmarked
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: RTG tools
  role: vcfeval-based companion benchmark for SNP/small-indel VCFs against GIAB v3.3.2
    truth sets in the same evaluation suite
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:67df92295ced8cdf55de4e4db7ac2383000d22d8f0740c088e35cab9aa9b8297
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - NanoSimFormer
  - RTG tools
  - Sniffles2
  - Squigulator
  - Truvari
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# truvari-sv-benchmark-evaluation

## Summary

Benchmarks structural variant calls derived from simulated nanopore reads against the GIAB HG002 SV Draft Benchmark Set using Truvari, computing precision, recall and F1 per sample and variant class. It is the validation step that determines whether simulated reads (e.g. NanoSimFormer ont_r1041_dna_5khz output) recapitulate real experimental SV detection performance.

## When to use

Use when SV calls from aligned simulated nanopore BAMs (Sniffles2 output) must be quantitatively validated against a GIAB SV truth set, e.g. comparing NanoSimFormer vs seq2squiggle vs Squigulator SV fidelity.

## When NOT to use

- Input VCFs are SNP/small-indel calls rather than structural variants — these belong in the RTG tools vcfeval branch of the benchmark (GIAB v3.3.2 truth sets), not Truvari SV benchmarking.
- The source reads are direct-RNA (RNA004) simulations benchmarked with Clair3-RNA for SNP/indel performance (SNP 0.92 / indel 0.57), where the SV SV Draft Benchmark Set (HG002 DNA) does not apply.
- No matched GIAB truth set exists for the sample being evaluated, making precision/recall/F1 against a benchmark undefined.

## Inputs

- Sniffles2 SV VCF called from aligned basecalled reads of NanoSimFormer-simulated HG001–HG005 DNA R10.4.1 samples
- GIAB HG002 SV Draft Benchmark Set (truth VCF)
- Aligned basecalled BAM files produced by Dorado (--basecall --emit-bam) after NanoSimFormer ont_r1041_dna_5khz simulation
- Reference genome FASTA used for simulation

## Outputs

- Truvari benchmark report comparing SV calls to the GIAB HG002 SV Draft Benchmark Set
- Precision, recall and F1 per sample and variant class written to variant_calling_f1_scores.csv
- Expected SV F1-score range for validation: ~0.8445 (NanoSimFormer) vs 0.1960–0.2419 (Squigulator) and 0.4761–0.5135 small-indel-like ranges for seq2squiggle

## How to apply

First simulate DNA reads with NanoSimFormer using the ont_r1041_dna_5khz preset (e.g. --mode Reference, --seed 42) with --basecall/--emit-bam so Dorado produces basecalled BAMs, then align those reads to the reference and call structural variants from the DNA BAMs with Sniffles2. Benchmark the resulting SV VCFs against the GIAB HG002 SV Draft Benchmark Set using Truvari, and in the same benchmark suite compare small-variant calls via RTG tools vcfeval against the GIAB v3.3.2 truth sets. Compute precision, recall and F1 per sample and variant class, writing all results to variant_calling_f1_scores.csv. The applied-correctly criterion is that the SV F1-score falls within the reported range (SV 0.8445 for NanoSimFormer-simulated reads, far above Squigulator's 0.1960–0.2419 range), so an out-of-range result signals a simulation or calling failure rather than a benchmarking artifact.

## Related tools

- **Truvari** (Benchmarks called SV VCFs against the GIAB HG002 SV Draft Benchmark Set to produce precision/recall/F1)
- **Sniffles2** (Detects structural variants from the simulated DNA BAMs prior to Truvari benchmarking)
- **NanoSimFormer** (Simulates R10.4.1 DNA signals and basecalls them via Dorado (--basecall, --emit-bam) to generate the BAMs whose SV calls are benchmarked) — https://github.com/BioinfoSZU/NanoSimFormer
- **RTG tools** (vcfeval-based companion benchmark for SNP/small-indel VCFs against GIAB v3.3.2 truth sets in the same evaluation suite)

## Evaluation signals

- SV F1-score from Truvari falls within the reported NanoSimFormer range (SV 0.8445), well above the Squigulator range 0.1960–0.2419.
- All per-sample, per-class precision/recall/F1 values are recorded in variant_calling_f1_scores.csv with no missing classes.
- DNA SNP F1-scores from the companion vcfeval step land within 0.9953–0.9973 and small indel F1 within 0.7862–0.8612, confirming the upstream simulation/basecalling chain was correct.
- Simulation used the documented settings (--mode Reference, --preset ont_r1041_dna_5khz, --seed 42) and BAMs were emitted via --basecall --emit-bam, so results are reproducible.

## Limitations

- SV validation values are anchored to a single reported SV F1 (0.8445) and the GIAB HG002 SV Draft Benchmark Set, so benchmarking samples without a GIAB SV truth set cannot be validated the same way.
- Simulator comparisons are chemistry- and preset-specific (ont_r1041_dna_5khz for DNA SV work; RNA004 SV fidelity is not characterized — RNA004 benchmarks report SNP 0.92 / indel 0.57 only).
- The README notes the length distribution model was derived from the HG002 R10.4.1 sample; other samples rely on presets rather than sample-specific length models, which may affect SV call fidelity.

## Evidence

- [methods] Benchmark called SNP/indel VCFs against the GIAB v3.3.2 truth sets using RTG tools vcfeval, and benchmark SV calls against the GIAB HG002 SV Draft Benchmark Set using Truvari.: [evidence span withheld — anchor status: not_found]
- [methods] Validation range for the SV benchmark output.: [evidence span withheld — anchor status: not_found]
- [methods] Downstream computation producing the benchmark table.: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract finding on variant-calling fidelity that the Truvari benchmark quantifies for SVs.: "It faithfully recapitulates experimental variant calling performance across the five human samples, achieving F1-scores of 0.9953–0.9973 and 0.7862–0.8612"
- [readme] README describing how basecalled BAMs fed into the SV-calling chain are produced.: "--emit-bam            basecalling simulated reads are stored in BAM files (default: False)"
