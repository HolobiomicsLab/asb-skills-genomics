---
name: minimap2-read-alignment-and-strain-abundance-estimation
description: Use when simulated (or real) nanopore FASTQ reads from a mock community
  such as the ten-strain MSA-1010 fungal metagenome must be checked for community-structure
  fidelity against known per-strain abundances.
when_to_use_negative:
- Analysis goal is variant calling or consensus assembly (e.g., HG002 SNPs/indels,
  de novo bacterial assembly) rather than strain-abundance preservation
- Input reads are unaligned metagenomic contigs or taxonomic profiles already summarized
  as an abundance table (alignment-based counting is unnecessary)
- No reference genomes for the community members are available, so per-strain read
  assignment cannot be performed
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: NanoSimFormer
  role: Generates the synthetic nanopore signals (ont_r1041_dna_5khz preset, --mode
    Reference) whose basecalled reads are aligned and quantified
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls simulated POD5 signals into FASTQ via the --basecall option before
    alignment
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Aligns basecalled reads to the multi-strain reference in ONT read-mapping
    mode for per-strain read assignment
- name: ESPRESSO
  role: Quantifies transcript isoform abundances from aligned HG002 RNA004 reads for
    the transcriptomic correlation arm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:984bf0b316991357db17df6b4f42860b92ceb4a7786a780b1b065e3f43603eb2
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# minimap2-read-alignment-and-strain-abundance-estimation

## Summary

Validates that simulated nanopore reads preserve relative strain abundances by basecalling simulated signals with Dorado, aligning the resulting FASTQ to a multi-strain reference with minimap2 in ONT read-mapping mode, counting reads per strain, and comparing estimated vs. ground-truth abundances by Pearson correlation.

## When to use

Use when simulated (or real) nanopore FASTQ reads from a mock community such as the ten-strain MSA-1010 fungal metagenome must be checked for community-structure fidelity against known per-strain abundances.

## When NOT to use

- Analysis goal is variant calling or consensus assembly (e.g., HG002 SNPs/indels, de novo bacterial assembly) rather than strain-abundance preservation
- Input reads are unaligned metagenomic contigs or taxonomic profiles already summarized as an abundance table (alignment-based counting is unnecessary)
- No reference genomes for the community members are available, so per-strain read assignment cannot be performed

## Inputs

- Basecalled nanopore FASTQ reads (from Dorado basecalling of simulated POD5 signals)
- Multi-strain reference genome FASTA (e.g., ten MSA-1010 fungal strain genomes)
- Ground-truth relative abundance table per strain
- Optionally a HG002 transcriptome reference for the RNA004 isoform-abundance arm

## Outputs

- Read-to-reference alignment file (BAM/PAF) from minimap2 ONT mapping
- Per-strain read counts and estimated relative abundances
- abundance_correlations.tsv with Pearson correlation values vs. ground truth and baseline simulators

## How to apply

Simulate reads per strain reference with NanoSimFormer in --mode Reference using the ont_r1041_dna_5khz preset, and enable the --basecall option so Dorado basecalls the simulated signals into a FASTQ file. Align the basecalled FASTQ to the multi-strain reference with minimap2 in ONT read-mapping mode (map-ont), then count reads assigned to each strain to estimate relative abundances. Compute Pearson correlations between estimated and ground-truth strain abundances; for NanoSimFormer-simulated MSA-1010 reads the expected target is r ≈ 0.9932, exceeding seq2squiggle (0.9577) and Squigulator (0.9378) baselines. Record all correlation values in abundance_correlations.tsv so simulator-to-simulator comparisons are directly auditable. Optionally apply the same pattern to direct-RNA data (ont_rna004_4khz preset, HG002 transcriptome) where ESPRESSO quantifies isoform abundances, targeting r ≈ 0.996.

## Related tools

- **NanoSimFormer** (Generates the synthetic nanopore signals (ont_r1041_dna_5khz preset, --mode Reference) whose basecalled reads are aligned and quantified) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls simulated POD5 signals into FASTQ via the --basecall option before alignment) — https://github.com/nanoporetech/dorado
- **minimap2** (Aligns basecalled reads to the multi-strain reference in ONT read-mapping mode for per-strain read assignment)
- **ESPRESSO** (Quantifies transcript isoform abundances from aligned HG002 RNA004 reads for the transcriptomic correlation arm)

## Examples

```
minimap2 -ax map-ont msa1010_refs.fna basecalled.fq | samtools sort -o fungal.bam; then count reads per strain reference
```

## Evaluation signals

- Pearson correlation of estimated vs. ground-truth strain abundances reaches ≈0.9932 for NanoSimFormer-simulated MSA-1010 reads
- Correlation exceeds the seq2squiggle (0.9577) and Squigulator (0.9378) baselines under identical alignment/counting settings
- Dorado --basecall produced a FASTQ file from the simulated signals prior to alignment
- Per-strain read counts are complete (every strain reference is represented) and estimated relative abundances are properly normalized before correlation
- For the RNA004 arm, ESPRESSO-based isoform correlation reaches ≈0.996

## Limitations

- Reported correlations (0.9932 metagenomic, 0.996 transcriptomic) are specific to the MSA-1010 mock community and HG002 RNA004 datasets with the stated presets (ont_r1041_dna_5khz, ont_rna004_4khz)
- Alignment-based counting fidelity depends on simulator/basecaller accuracy; accuracy claims are anchored to median basecalling accuracy >99% and Q-scores >22.8 for R10.4.1 and direct RNA
- GPU with CUDA compute capability ≥ 8.x is recommended for running the NanoSimFormer simulation pipeline that feeds this workflow

## Evidence

- [abstract] High median basecalling accuracy mirroring real data: "NanoSimFormer achieves a median basecalling accuracy exceeding 99% and Q-scores above 22.8 for Oxford Nanopore Technologies' latest DNA R10.4.1 and direct RNA sequencing"
- [abstract] High correlations with experimental abundance: "NanoSimFormer-derived reads enable high-quality de novo bacterial assembly with consensus error rates below one mismatch per 100 kbp and maintain high correlations with experimental abundance"
- [methods] Dorado basecalling of simulated reads into FASTQ: [evidence span withheld — anchor status: not_found]
- [readme] Dorado as the ONT analysis engine: [evidence span withheld — anchor status: external_doc]
