---
name: pearson-correlation-benchmark-against-simulator-baselines
description: Use when evaluating whether simulated nanopore reads (DNA metagenomic
  or direct-RNA) preserve known strain or transcript-isoform relative abundances and
  when comparing fidelity against other simulators.
when_to_use_negative:
- Input is real experimental sequencing data rather than simulator output — there
  is no simulator baseline to benchmark and correlation against ground truth is trivially
  the data itself.
- The reference abundances of the community/transcripts are unknown, making a correlation
  between estimated and known abundances undefined.
- The analysis goal is basecalling accuracy or variant-calling fidelity (SNP/indel
  F1), not abundance preservation.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: NanoSimFormer
  role: Generates synthetic nanopore signals from fungal references (--mode Reference,
    ont_r1041_dna_5khz) and HG002 direct-RNA transcripts (ont_rna004_4khz) whose abundance
    preservation is benchmarked.
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls simulated signals into FASTQ via NanoSimFormer's --basecall option,
    producing reads for abundance estimation.
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Aligns basecalled reads (ONT mapping mode for DNA; transcriptome alignment
    for RNA) to assign reads to strains/isoforms.
- name: ESPRESSO
  role: Quantifies transcript isoform abundances from aligned HG002-RNA004 reads for
    the transcriptomic Pearson correlation.
- name: seq2squiggle
  role: Baseline simulator whose metagenomic abundance correlation (0.9577) benchmarks
    NanoSimFormer's performance.
  repo: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:eefd4a46f7348fd6918db49e62d1f828ecc1e9eb03045ec52452527cd0b1ce51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Docker
  - Dorado
  - ESPRESSO
  - NanoSimFormer
  - minimap2
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pearson-correlation-benchmark-against-simulator-baselines

## Summary

Quantify how faithfully simulator-generated nanopore reads preserve known relative abundances by computing Pearson correlations between estimated and ground-truth abundances, and benchmarking against baseline simulators (seq2squiggle, Squigulator).

## When to use

Use when evaluating whether simulated nanopore reads (DNA metagenomic or direct-RNA) preserve known strain or transcript-isoform relative abundances and when comparing fidelity against other simulators.

## When NOT to use

- Input is real experimental sequencing data rather than simulator output — there is no simulator baseline to benchmark and correlation against ground truth is trivially the data itself.
- The reference abundances of the community/transcripts are unknown, making a correlation between estimated and known abundances undefined.
- The analysis goal is basecalling accuracy or variant-calling fidelity (SNP/indel F1), not abundance preservation.

## Inputs

- Ten MSA-1010 fungal strain reference genomes (FASTA)
- HG002 direct-RNA transcriptome reference (FASTA)
- NanoSimFormer-simulated POD5 signals basecalled to FASTQ via Dorado
- Optional --trans-profile TSV with per-transcript trunc_start/trunc_end
- Ground-truth relative abundances for strains/isoforms

## Outputs

- Per-strain read counts and estimated relative abundances (from minimap2 alignment to the multi-strain reference)
- Transcript isoform abundances quantified by ESPRESSO
- Pearson correlation values (target ~0.9932 metagenomic, ~0.996 transcriptomic)
- abundance_correlations.tsv

## How to apply

Simulate reads from the ten MSA-1010 fungal strain reference genomes with NanoSimFormer in --mode Reference using the ont_r1041_dna_5khz preset, basecalling simulated signals with the built-in Dorado --basecall option to produce FASTQ. Align basecalled reads to the multi-strain reference with minimap2 in ONT read-mapping mode and count reads per strain to estimate relative abundances. For the transcriptomic arm, simulate HG002 direct-RNA reads with the ont_rna004_4khz preset (optionally with a --trans-profile TSV giving per-transcript trunc_start/trunc_end), basecall with Dorado, align to the HG002 transcriptome with minimap2, and quantify isoform abundances with ESPRESSO. Compute Pearson correlations between estimated and known abundances for both analyses; the reference targets are ~0.9932 (metagenomic) and ~0.996 (transcriptomic), against metagenomic baselines of 0.9577 (seq2squiggle) and 0.9378 (Squigulator).

## Related tools

- **NanoSimFormer** (Generates synthetic nanopore signals from fungal references (--mode Reference, ont_r1041_dna_5khz) and HG002 direct-RNA transcripts (ont_rna004_4khz) whose abundance preservation is benchmarked.) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls simulated signals into FASTQ via NanoSimFormer's --basecall option, producing reads for abundance estimation.) — https://github.com/nanoporetech/dorado
- **minimap2** (Aligns basecalled reads (ONT mapping mode for DNA; transcriptome alignment for RNA) to assign reads to strains/isoforms.)
- **ESPRESSO** (Quantifies transcript isoform abundances from aligned HG002-RNA004 reads for the transcriptomic Pearson correlation.)
- **seq2squiggle** (Baseline simulator whose metagenomic abundance correlation (0.9577) benchmarks NanoSimFormer's performance.) — https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle

## Examples

```
nanosimformer simulate --mode Reference --input msa1010_strains.fna --preset ont_r1041_dna_5khz --basecall -o sim_pod5/  # then minimap2 -ax map-ont multi_strain_ref.fna basecalls.fastq | count per-strain reads; compute Pearson r vs known abundances → abundance_correlations.tsv
```

## Evaluation signals

- Metagenomic Pearson correlation of estimated vs ground-truth strain abundances reaches approximately 0.9932.
- Transcriptomic isoform Pearson correlation reaches approximately 0.996.
- Both correlations exceed the seq2squiggle baseline (0.9577) and the Squigulator baseline (0.9378).
- All correlation values are written to abundance_correlations.tsv for reproducibility.
- Every simulated read set is basecalled (FASTQ present) and aligned with minimap2 before abundance counting, so correlations are computed on matched pipelines across simulators.

## Limitations

- Correlation captures linear agreement in relative abundance structure; it does not measure absolute depth calibration or per-strain bias magnitude.
- The benchmark is tied to specific presets (ont_r1041_dna_5khz for DNA, ont_rna004_4khz for RNA) and the MSA-1010 ten-strain community; other chemistries or communities may shift values.
- Squigulator is mentioned only as a baseline value (0.9378); no workflow details for reproducing it are provided in the context.

## Evidence

- [results] NanoSimFormer showed the strongest agreement with real experimental data in community structure preservation on the MSA-1010 fungal metagenome, achieving a Pearson correlation of 0.9932, versus lower correlations of 0.9577 for seq2squiggle and 0.9378 for Squigulator.: [evidence span withheld — anchor status: not_found]
- [methods] The --basecall option automatically runs Dorado to basecall the simulated reads into a FASTQ file after signal simulation.: [evidence span withheld — anchor status: not_found]
- [abstract] NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp and maintain high correlations with experimental abundance: "NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp and maintain high correlations with experimental abundance"
- [readme] It supports both ONT DNA (R10.4.1) and direct-RNA (RNA004) signal simulation, enabling users to generate synthetic POD5 files from references or existing basecalled reads.: "It supports both ONT DNA (R10.4.1) and direct-RNA (RNA004) signal simulation, enabling users to generate synthetic POD5 files from references or existing basecalled reads."
