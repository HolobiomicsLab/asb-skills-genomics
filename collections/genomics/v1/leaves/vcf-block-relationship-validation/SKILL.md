---
name: vcf-block-relationship-validation
description: Use when a methylation-based phasing tool has chained SNV phase blocks
  in a VCF and you need to verify block connections against GIAB trio-phased truth,
  computing corrected N50 and switch-error cost.
when_to_use_negative:
- Phasing sex chromosomes in clinical samples — random X inactivation in females produces
  inconsistent haplotype patterns, and MethPhaser/README explicitly excludes sex chromosomes
- No trio-phased truth set is available for the sample, making corrected-N50 and connection-level
  validation impossible
- Initial SNV phasing has very high error (e.g., very high hamming distance in R9
  80x results), which negatively impacts MethPhaser accuracy and invalidates the connection
  judgments
edam_operation: http://edamontology.org/operation_3199
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: MethPhaser
  role: Produces the methylation-based SNV phase-block connections being validated;
    accepts the truth VCF via -vt for benchmarking
  repo: https://github.com/treangenlab/methphaser
- name: meth_phaser_parallel
  role: Runs the methylation-based block chaining with optional -vt/--vcf_truth benchmarking
    input
  repo: https://github.com/treangenlab/methphaser
- name: WhatsHap
  role: Provides the haplotag module and stats module used to generate the phase-block
    GTF and benchmark phased VCFs against truth
- name: HapCUT2
  role: Produces the SNV-only phasing baseline and input VCF whose block connections
    MethPhaser extends
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:600936e33688eea8676a434b42cea90cb1a709e87dfd7465e710fc8516adc969
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/treangenlab/methphaser
  tools:
  - Clair3
  - HapCUT2
  - MethPhaser
  - WhatsHap
  - meth_phaser_parallel
  - minimap2
  edam_operation: http://edamontology.org/operation_3199
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# vcf-block-relationship-validation

## Summary

Validates methylation-chained SNV phase-block relationships (e.g., from MethPhaser) against a trio-phased truth VCF, quantifying how many block connections are correct and the switch-error cost of joining blocks. This distinguishes genuine N50 gains from incorrectly connected blocks on benchmarks such as HG002.

## When to use

Use when a methylation-based phasing tool has chained SNV phase blocks in a VCF and you need to verify block connections against GIAB trio-phased truth, computing corrected N50 and switch-error cost.

## When NOT to use

- Phasing sex chromosomes in clinical samples — random X inactivation in females produces inconsistent haplotype patterns, and MethPhaser/README explicitly excludes sex chromosomes
- No trio-phased truth set is available for the sample, making corrected-N50 and connection-level validation impossible
- Initial SNV phasing has very high error (e.g., very high hamming distance in R9 80x results), which negatively impacts MethPhaser accuracy and invalidates the connection judgments

## Inputs

- HapCUT2-phased SNV VCF from ONT R10 HG002 reads (Clair3 calls, AF > 0.25, GIAB high-confidence regions)
- MethPhaser methylation-phased VCF (output.vcf from meth_phaser_post_processing)
- GIAB trio-phased truth VCF for HG002
- WhatsHap phase-block GTF generated from HapCUT2 results

## Outputs

- Corrected phase-block N50 after removal of incorrectly connected blocks
- Switch-error increase (new switches / number of MethPhaser-established connections)
- Block relationship correctness labels for each methylation-based block connection

## How to apply

Supply the GIAB trio-phased truth VCF via the -vt/--vcf_truth option of meth_phaser_parallel, or benchmark the methylation-phased output VCF with WhatsHap compare against the truth set while restricting to GIAB 'high confidence' regions and excluding X/Y chromosomes. For each MethPhaser-established block connection, determine correctness by comparing the SNV-based phasing against the trio-phased VCF; remove larger phase blocks that were incorrectly connected before recomputing the corrected phase-block N50. Quantify the switch-error cost as the number of new switches introduced divided by the number of MethPhaser-established connections. On HG002 R10 data this validation framework confirmed a corrected N50 of 3,152,506 at 30X (exceeding 3,034,229 for 60X SNV-only phasing) with only a 0.0532% switch-error increase, consistent with majority-vote decisions using voting confidence > 0.5 and the best-accuracy minimum read number (read coverage/(genome-wide block number/1000)).

## Related tools

- **MethPhaser** (Produces the methylation-based SNV phase-block connections being validated; accepts the truth VCF via -vt for benchmarking) — https://github.com/treangenlab/methphaser
- **meth_phaser_parallel** (Runs the methylation-based block chaining with optional -vt/--vcf_truth benchmarking input) — https://github.com/treangenlab/methphaser
- **WhatsHap** (Provides the haplotag module and stats module used to generate the phase-block GTF and benchmark phased VCFs against truth)
- **HapCUT2** (Produces the SNV-only phasing baseline and input VCF whose block connections MethPhaser extends)

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -g test_data/LSK.filtered.gtf -vc test_data/HLA.R10.phased.vcf.gz -vt truth.vcf -o test_data/work
```

## Evaluation signals

- Corrected phase-block N50 on HG002 R10 exceeds the 60X SNV-only N50 (3,152,506 vs 3,034,229), an approximately 1.5x N50 increase
- Switch-error increase is ~0.0532%, computed as new switches introduced divided by the number of MethPhaser-established connections
- All benchmark comparisons restricted to GIAB 'high confidence' regions with X and Y chromosomes excluded
- Every retained block relationship satisfies the voting confidence threshold (Votes for one haplotype - votes for the other)/total votes > 0.5
- Corrected N50 matches the value reported in the R10 results and Fig. 2c after removal of incorrectly connected larger blocks

## Limitations

- Cannot improve phasing for human sex chromosomes in clinical samples due to random X inactivation in females
- Validation quality depends on initial SNV phasing accuracy; high SNV phasing errors negatively impact MethPhaser accuracy
- Results depend on methylation detection accuracy, overall read length, and minimum data coverage; tissue-specific methylation signals make cross-tissue performance hard to predict

## Evidence

- [methods] we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB.: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB."
- [methods] (Votes for one haplotype-votes for the other one)/total votes > 0.5: "(Votes for one haplotype-votes for the other one)/total votes > 0.5"
- [discussion] we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error): "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
- [readme] -vt , --vcf_truth     Truth vcf file for benchmarking: "-vt , --vcf_truth     Truth vcf file for benchmarking"
