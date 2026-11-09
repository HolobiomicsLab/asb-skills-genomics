---
name: variant-benchmark-evaluation-rtg-vcfeval
description: Use when assessing how faithfully variant calling on simulated nanopore
  reads (NanoSimFormer, seq2squiggle, Squigulator) recapitulates real experimental
  SNP, small-indel, and structural-variant detection against GIAB truth sets.
when_to_use_negative:
- Input reads are already experimental — the comparison target itself, not a simulator
  to benchmark
- Evaluating simulator fidelity at the signal or basecalling level (median accuracy
  > 99%, Q > 22.8) rather than downstream variant-calling performance
- Long-read data from platforms other than ONT R10.4.1 DNA or RNA004, where the GIAB
  benchmark ranges do not apply
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: NanoSimFormer
  role: Generates high-fidelity simulated POD5 signals and, with --basecall, Dorado-basecalled
    FASTQ/BAM that feed the variant-calling benchmark
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls simulated signals into FASTQ/BAM via the --basecall option before
    alignment and variant calling
  repo: https://github.com/nanoporetech/dorado
- name: Clair3
  role: Small-variant (SNP and indel) calling on aligned DNA R10.4.1 BAMs
- name: Sniffles2
  role: Structural-variant detection from DNA BAMs, benchmarked with Truvari
- name: RTG tools
  role: vcfeval-based benchmarking of SNP/indel VCFs against GIAB v3.3.2 truth sets
    to derive precision/recall/F1
- name: Truvari
  role: Benchmarks Sniffles2 SV calls against the GIAB HG002 SV Draft Benchmark Set
- name: seq2squiggle
  role: Baseline simulator whose variant-calling F1 (0.4761–0.5135) is compared against
    NanoSimFormer
  repo: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:137c5edfda16f99cc34444905d785b7946d02dbcc63edbf2af402c61492d8b90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Clair3
  - Dorado
  - NanoSimFormer
  - RTG tools
  - Sniffles2
  - Squigulator
  - Truvari
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# variant-benchmark-evaluation-rtg-vcfeval

## Summary

Benchmark variant calls from NanoSimFormer-simulated nanopore reads against GIAB truth sets by aligning Dorado-basecalled reads with an aligner, calling variants with Clair3/Clair3-RNA and Sniffles2, then scoring SNP/indel VCFs with RTG tools vcfeval and SV calls with Truvari to compute per-sample, per-class precision, recall, and F1.

## When to use

Use when assessing how faithfully variant calling on simulated nanopore reads (NanoSimFormer, seq2squiggle, Squigulator) recapitulates real experimental SNP, small-indel, and structural-variant detection against GIAB truth sets.

## When NOT to use

- Input reads are already experimental — the comparison target itself, not a simulator to benchmark
- Evaluating simulator fidelity at the signal or basecalling level (median accuracy > 99%, Q > 22.8) rather than downstream variant-calling performance
- Long-read data from platforms other than ONT R10.4.1 DNA or RNA004, where the GIAB benchmark ranges do not apply

## Inputs

- Reference genome FASTA (e.g. HG001–HG005) for R10.4.1 DNA simulation
- Transcriptome FASTA plus 3-column transcript profile TSV (HG002 direct-RNA, RNA004)
- Basecalled BAM/FASTQ from Dorado after signal simulation
- GIAB v3.3.2 small-variant truth VCFs
- GIAB HG002 SV Draft Benchmark Set (VCF)

## Outputs

- Called SNP/small-indel VCFs from Clair3 and Clair3-RNA
- Structural-variant VCFs from Sniffles2
- vcfeval and Truvari benchmark summaries (precision, recall, F1 per sample and variant class)
- variant_calling_f1_scores.csv

## How to apply

Simulate reads with NanoSimFormer (e.g. --mode Reference, --seed 42, --preset ont_r1041_dna_5khz) enabling Dorado basecalling via --basecall to produce POD5 plus basecalled FASTQ/BAM, repeating with seq2squiggle and Squigulator baselines; for direct-RNA use the ont_rna004_4khz preset with a 3-column --trans-profile for transcript abundance and truncation. Align basecalled reads to their references and run Clair3 on DNA R10.4.1 BAMs and Clair3-RNA on the RNA004 BAM, plus Sniffles2 for structural variants from DNA BAMs. Benchmark SNP/indel VCFs against GIAB v3.3.2 truth sets with RTG tools vcfeval and SV calls against the GIAB HG002 SV Draft Benchmark Set with Truvari. Compute precision, recall, and F1 per sample and variant class, writing results to variant_calling_f1_scores.csv. Rationale: simulated reads are trusted only if their variant-calling F1 falls within the reported experimental ranges, validating the simulator's fidelity rather than raw signal statistics alone.

## Related tools

- **NanoSimFormer** (Generates high-fidelity simulated POD5 signals and, with --basecall, Dorado-basecalled FASTQ/BAM that feed the variant-calling benchmark) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls simulated signals into FASTQ/BAM via the --basecall option before alignment and variant calling) — https://github.com/nanoporetech/dorado
- **Clair3** (Small-variant (SNP and indel) calling on aligned DNA R10.4.1 BAMs)
- **Sniffles2** (Structural-variant detection from DNA BAMs, benchmarked with Truvari)
- **RTG tools** (vcfeval-based benchmarking of SNP/indel VCFs against GIAB v3.3.2 truth sets to derive precision/recall/F1)
- **Truvari** (Benchmarks Sniffles2 SV calls against the GIAB HG002 SV Draft Benchmark Set)
- **seq2squiggle** (Baseline simulator whose variant-calling F1 (0.4761–0.5135) is compared against NanoSimFormer) — https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle

## Examples

```
python -m nano_signal_simulator simulate --input ${EXAMPLE_DIR}/DNA_R10.4.1/chr22.fasta --output ${EXAMPLE_DIR}/DNA_R10.4.1/output --mode Reference --sample-reads 1000 --gpu 0 --preset ont_r1041_dna_5khz --basecall --seed 42
```

## Evaluation signals

- DNA SNP F1-scores fall within 0.9953–0.9973 across the five human samples
- DNA small-indel F1-scores fall within 0.7862–0.8612, comparable to experimental baselines of 0.8209–0.8963
- SV benchmarking yields F1 ≈ 0.8445 via Truvari against the GIAB HG002 SV Draft Benchmark Set
- RNA004 sample reaches SNP F1 ≈ 0.92 and indel F1 ≈ 0.57 with Clair3-RNA
- All per-sample/per-class metrics are recorded in variant_calling_f1_scores.csv without missing rows

## Limitations

- seq2squiggle and Squigulator baselines yield markedly lower indel F1 (0.4761–0.5135 and 0.1960–0.2419), so conclusions about simulator fidelity are specific to NanoSimFormer in this benchmark
- Indel F1 has a wider spread (0.7862–0.8612) than SNP F1, so indel-level comparisons across samples are noisier
- Validation thresholds are tied to the specific GIAB v3.3.2 truth sets and HG002 SV Draft Benchmark Set; other truth resources may not reproduce these ranges

## Evidence

- [abstract] Faithfully recapitulates experimental variant calling across five human samples: SNP F1-scores 0.9953–0.9973 and small indel F1-scores 0.7862–0.8612: "It faithfully recapitulates experimental variant calling performance across the five human samples, achieving F1-scores of 0.9953–0.9973 and 0.7862–0.8612 for single-nucleotide polymorphisms and"
- [methods] Dorado basecalling of simulated reads: [evidence span withheld — anchor status: not_found]
- [readme] Simulation presets and transcript profile input: "--preset {ont_r1041_dna_5khz,ont_rna004_4khz}
                        ont platform preset"
- [readme] Direct-RNA transcript abundance simulation: "3-column TSV file for simulating transcripts with specific abundance and truncation (default: None)"
