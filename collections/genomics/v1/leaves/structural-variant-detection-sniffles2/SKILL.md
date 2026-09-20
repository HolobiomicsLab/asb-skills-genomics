---
name: structural-variant-detection-sniffles2
description: Use when you have basecalled, reference-aligned long-read DNA BAMs (e.g.
  from simulated or experimental ONT R10.4.1 data) and need to call SVs and quantify
  SV-calling fidelity against a truth set.
when_to_use_negative:
- Input is direct-RNA (RNA004) data — the workflow specifies Clair3-RNA for RNA variant
  calling, and SV detection is described only for DNA BAMs
- Goal is SNP/small-indel calling — use Clair3 (DNA) or Clair3-RNA (RNA) with RTG
  vcfeval against GIAB v3.3.2 truth sets instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Sniffles2
  role: Detects structural variants from the aligned DNA R10.4.1 sample BAMs
- name: Truvari
  role: Benchmarks Sniffles2 SV calls against the GIAB HG002 SV Draft Benchmark Set
- name: Dorado
  role: Basecalls simulated reads (via NanoSimFormer --basecall) into FASTQ/BAM upstream
    of alignment and SV calling
  repo: github.com/nanoporetech/dorado
- name: NanoSimFormer
  role: Generates the simulated ONT DNA R10.4.1 signals/reads (ont_r1041_dna_5khz
    preset) whose basecalled BAMs feed Sniffles2
  repo: https://github.com/BioinfoSZU/NanoSimFormer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:f7de87edbd987f99cc03a5055e2eb93af3a896ead3eb8cd3bafb4b673b4af231
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dorado
  - NanoSimFormer
  - RTG tools
  - Sniffles2
  - Squigulator
  - Truvari
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# structural-variant-detection-sniffles2

## Summary

Detect structural variants from aligned nanopore DNA BAMs with Sniffles2, then benchmark the resulting SV VCF against the GIAB HG002 SV Draft Benchmark Set using Truvari. In the NanoSimFormer study this step showed that simulated R10.4.1 reads recapitulate experimental SV calling (SV F1 0.8445).

## When to use

Use when you have basecalled, reference-aligned long-read DNA BAMs (e.g. from simulated or experimental ONT R10.4.1 data) and need to call SVs and quantify SV-calling fidelity against a truth set.

## When NOT to use

- Input is direct-RNA (RNA004) data — the workflow specifies Clair3-RNA for RNA variant calling, and SV detection is described only for DNA BAMs
- Goal is SNP/small-indel calling — use Clair3 (DNA) or Clair3-RNA (RNA) with RTG vcfeval against GIAB v3.3.2 truth sets instead

## Inputs

- Reference-aligned DNA nanopore BAM files (Dorado-basecalled simulated or experimental ONT R10.4.1 reads)
- Reference genome FASTA used for alignment
- GIAB HG002 SV Draft Benchmark Set (truth VCF for Truvari benchmarking)

## Outputs

- Sniffles2 structural variant VCF per DNA sample
- Truvari benchmarking report comparing SV calls to GIAB truth set
- Per-sample, per-variant-class precision, recall and F1 entries in variant_calling_f1_scores.csv

## How to apply

After simulating DNA reads with NanoSimFormer (ont_r1041_dna_5khz preset) and basecalling with Dorado (--basecall, optionally --emit-bam), align the basecalled reads to the reference genome and run Sniffles2 on the resulting DNA BAM files to produce an SV VCF. Benchmark the Sniffles2 SV calls against the GIAB HG002 SV Draft Benchmark Set using Truvari, then compute precision, recall and F1 per sample and variant class, writing results to variant_calling_f1_scores.csv. Validation requires the SV F1-score to fall within the reported range (0.8445), which is how NanoSimFormer-simulated reads are judged to faithfully recapitulate experimental SV detection, in contrast to seq2squiggle and Squigulator baselines.

## Related tools

- **Sniffles2** (Detects structural variants from the aligned DNA R10.4.1 sample BAMs)
- **Truvari** (Benchmarks Sniffles2 SV calls against the GIAB HG002 SV Draft Benchmark Set)
- **Dorado** (Basecalls simulated reads (via NanoSimFormer --basecall) into FASTQ/BAM upstream of alignment and SV calling) — github.com/nanoporetech/dorado
- **NanoSimFormer** (Generates the simulated ONT DNA R10.4.1 signals/reads (ont_r1041_dna_5khz preset) whose basecalled BAMs feed Sniffles2) — https://github.com/BioinfoSZU/NanoSimFormer

## Examples

```
python -m nano_signal_simulator simulate --input chr22.fasta --output out --mode Reference --sample-reads 1000 --gpu 0 --preset ont_r1041_dna_5khz --basecall --emit-bam; sniffles --input aligned.bam --output sv.vcf; truvari bench -b giab_hg002_sv.vcf.gz -c sv.vcf
```

## Evaluation signals

- SV F1-score falls within the reported range, i.e. approximately 0.8445
- Truvari benchmark completes against the GIAB HG002 SV Draft Benchmark Set and yields precision/recall/F1 per variant class
- All results are written to variant_calling_f1_scores.csv with per-sample, per-variant-class entries
- Upstream BAMs are properly aligned basecalled reads (Dorado --basecall / --emit-bam output) rather than raw signal

## Limitations

- SV calling performance was validated only for DNA R10.4.1 data (SV F1 0.8445); the RNA004 sample was assessed only for SNP/indel performance
- Fidelity depends on simulation quality — alternative simulators (seq2squiggle, Squigulator) produced much lower SV-relevant variant-calling fidelity than NanoSimFormer

## Evidence

- [other] Detect structural variants from the DNA BAMs with Sniffles2.: [evidence span withheld — anchor status: not_found]
- [other] Truvari benchmarks SV calls against GIAB HG002 SV Draft Benchmark Set: [evidence span withheld — anchor status: not_found]
- [methods] The --basecall option automatically runs Dorado to basecall the simulated reads: [evidence span withheld — anchor status: not_found]
- [readme] README describes simulate subcommand options including preset ont_r1041_dna_5khz and emit-bam: "usage: python -m nano_signal_simulator simulate [-h] --input INPUT --output OUTPUT [--prefix PREFIX] [--basecall] [--emit-bam]"
