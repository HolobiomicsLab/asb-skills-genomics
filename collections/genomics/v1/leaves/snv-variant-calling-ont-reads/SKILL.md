---
name: snv-variant-calling-ont-reads
description: Use when raw ONT R9/R10 long reads (e.g., HG002) have been basecalled
  with methylation tags and aligned, and phased SNV calls in VCF form are needed before
  haplotype phasing.
when_to_use_negative:
- When the input data are short reads (e.g., Illumina) rather than ONT long reads
  — Clair3 was configured here 'with the consideration of different read types' for
  ONT data
- When the input BAM still contains secondary or supplementary alignments (flag 2304)
  — this triggers pysam MM tag reading errors downstream
- When the goal is benchmarking phasing on sex chromosomes — the workflow excludes
  X and Y from the benchmark due to random X inactivation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0199
tools:
- name: Clair3
  role: Variant caller used on the ONT-aligned BAM, version v0.1-r12, with read-type-specific
    model configuration; its calls are filtered to AF > 0.25 before phasing
- name: minimap2
  role: Aligns basecalled ONT reads to the GRCh38 no-alt reference while preserving
    the MM and ML methylation tags required by the downstream workflow
- name: SAMtools
  role: Filters the aligned BAM to primary alignments only (samtools view -bF 2304)
    before variant calling and methylation-based phasing
- name: MethPhaser
  role: The downstream consumer of the SNV calls — the filtered Clair3 VCF is phased
    with HapCUT2/WhatsHap and haplotagged for methylation-based block chaining
  repo: https://github.com/treangenlab/methphaser
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:c7d081a3d68896abf088b54f033f296af1582fb9dd78aa280ee20ff32c1f11ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/treangenlab/methphaser
  tools:
  - Clair3
  - HapCUT2
  - MethPhaser
  - SAMtools
  - WhatsHap
  - meth_phaser_parallel
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# snv-variant-calling-ont-reads

## Summary

Calls SNVs from methylation-annotated Oxford Nanopore reads with Clair3 v0.1-r12 as the prerequisite step for downstream HapCUT2/WhatsHap phasing in the MethPhaser workflow, applying an AF > 0.25 filter and restricting to GIAB high-confidence regions.

## When to use

Use when raw ONT R9/R10 long reads (e.g., HG002) have been basecalled with methylation tags and aligned, and phased SNV calls in VCF form are needed before haplotype phasing.

## When NOT to use

- When the input data are short reads (e.g., Illumina) rather than ONT long reads — Clair3 was configured here 'with the consideration of different read types' for ONT data
- When the input BAM still contains secondary or supplementary alignments (flag 2304) — this triggers pysam MM tag reading errors downstream
- When the goal is benchmarking phasing on sex chromosomes — the workflow excludes X and Y from the benchmark due to random X inactivation

## Inputs

- Raw ONT (R9 or R10) FAST5/POD5 or methylation-aware basecalled FASTQ
- Reference genome FASTA (GCA_000001405.15_GRCh38_no_alt_analysis_set)
- BAM of reads aligned with minimap2 retaining MM and ML tags, filtered to primary alignments only
- GIAB high-confidence BED regions

## Outputs

- Clair3 VCF of SNV calls
- Allele-frequency-filtered (AF > 0.25) variant set restricted to GIAB high-confidence regions, formatted as input to HapCUT2/WhatsHap phasing

## How to apply

Basecall the raw ONT reads with a methylation-aware ONT basecaller so MM and ML tags are present, then align to the GCA_000001405.15_GRCh38_no_alt_analysis_set reference with minimap2 v2.24 retaining MM and ML tags. Filter the BAM to primary alignments only with 'samtools view -bF 2304', because secondary/supplementary alignments trigger pysam MM tag reading errors in the downstream MethPhaser steps. Call variants with Clair3 v0.1-r12, configured for the specific ONT read type (R9 vs R10). Filter calls to allele frequency > 0.25 and restrict the region of interest to the 'high confidence' regions provided by GIAB to remove potential Clair3 false positive calls. The resulting filtered VCF feeds HapCUT2 (extractHAIRS --ont 1) or WhatsHap for SNV phasing, so permissive variant sets will degrade phasing accuracy downstream.

## Related tools

- **Clair3** (Variant caller used on the ONT-aligned BAM, version v0.1-r12, with read-type-specific model configuration; its calls are filtered to AF > 0.25 before phasing)
- **minimap2** (Aligns basecalled ONT reads to the GRCh38 no-alt reference while preserving the MM and ML methylation tags required by the downstream workflow)
- **SAMtools** (Filters the aligned BAM to primary alignments only (samtools view -bF 2304) before variant calling and methylation-based phasing)
- **MethPhaser** (The downstream consumer of the SNV calls — the filtered Clair3 VCF is phased with HapCUT2/WhatsHap and haplotagged for methylation-based block chaining) — https://github.com/treangenlab/methphaser

## Examples

```
samtools view -bF 2304 -o output.bam input.bam
```

## Evaluation signals

- The aligned BAM retains MM and ML tags after minimap2 alignment and contains only primary alignments (samtools view -bF 2304)
- Every emitted SNV call has allele frequency > 0.25, matching the paper's filter
- Variant calls are intersected with GIAB 'high confidence' regions and fall outside X/Y for benchmarking
- The filtered VCF is accepted by HapCUT2 extractHAIRS (--ont 1) without coordinate or format errors
- On HG002, phased output benchmarks against the GIAB trio-phased truth VCF with WhatsHap compare at expected accuracy (e.g., corrected phase-block N50 of 3,034,229 for 60X SNV-only phasing)

## Limitations

- Clair3 false positive calls remain a risk — the paper explicitly limits analysis to GIAB high-confidence regions 'to remove potential Clair3's false positive calls'
- SNV phasing methods (HapCUT2, WhatsHap) only work reasonably at approximately 30X or higher ONT coverage, so low-coverage input limits the value of the calls
- Performance of the downstream phasing depends on the accuracy of this SNV calling step; high SNV phasing errors (e.g., very high hamming distance in R9 80X results) negatively impact MethPhaser accuracy

## Evidence

- [methods] Clair3 v0.1-r1263 was used for variant calling with the consideration of different read types: "Clair3 v0.1-r1263 was used for variant calling with the consideration of different read types"
- [methods] AF filter: "We also filtered the allele frequency (AF > 0.25)"
- [methods] GIAB region restriction: "limited our interested region to “high confidence” regions provided by GIAB to remove potential Clair3’s false positive calls."
- [methods] Alignment preserving methylation tags: "We then used minimap2 v2.2462 to align reads to the reference and keep the MM and ML tags."
- [readme] Primary-alignment requirement: "The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error."
- [readme] Coverage recommendation: "We recommend using MethPhaser for at least 30X coverage ONT reads since most of the SNV phasing methods (HapCUT2, Whatshap) only work reasonably above that coverage"
