---
name: giab-truth-vcf-block-relationship-inference
description: Use when benchmarking MethPhaser or SNV-only phasing against HG002, where
  a GIAB trio-phased truth VCF exists and phase-block connections must be checked
  for correctness before computing N50 or CMRG coverage.
when_to_use_negative:
- The sample has no trio-phased truth VCF (no GIAB benchmark available), so block
  relationships cannot be verified.
- The analysis targets sex chromosomes — random X inactivation in females leads to
  inconsistent haplotype patterns and MethPhaser does not support sex chromosomes.
- Input BAM contains secondary or supplementary alignments, which trigger pysam MM
  tag reading errors and invalidate upstream haplotagged input.
edam_operation: http://edamontology.org/operation_3745
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: MethPhaser
  role: Produces the methylation-based block connections and relationship output that
    are validated against the GIAB truth VCF
  repo: https://github.com/treangenlab/methphaser
- name: HapCUT2
  role: Provides the baseline SNV-only phased VCF whose blocks are assigned relationships
    in the truth VCF
- name: WhatsHap
  role: compare mode quantifies flip and switch errors introduced by block connection
- name: SAMtools
  role: Filters the input BAM to primary alignments only (samtools view -bF 2304)
    so the benchmark inputs are valid
- name: meth_phaser_parallel
  role: Runs MethPhaser with voting confidence > 0.5 and best-accuracy minimum read
    number to generate block relationships
  repo: https://github.com/treangenlab/methphaser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:8f0f2657e4d310d8e31f0485ef951196364977ee2845ee3e7435ea5474edeb39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/treangenlab/methphaser
  tools:
  - HapCUT2
  - MethPhaser
  - SAMtools
  - WhatsHap
  - meth_phaser_parallel
  edam_operation: http://edamontology.org/operation_3745
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# giab-truth-vcf-block-relationship-inference

## Summary

Assign each SNV phase block's haplotype relationship to the GIAB trio-phased truth VCF so that block connections (e.g., those made by MethPhaser) can be validated and incorrectly connected larger blocks removed before N50-style comparisons on HG002.

## When to use

Use when benchmarking MethPhaser or SNV-only phasing against HG002, where a GIAB trio-phased truth VCF exists and phase-block connections must be checked for correctness before computing N50 or CMRG coverage.

## When NOT to use

- The sample has no trio-phased truth VCF (no GIAB benchmark available), so block relationships cannot be verified.
- The analysis targets sex chromosomes — random X inactivation in females leads to inconsistent haplotype patterns and MethPhaser does not support sex chromosomes.
- Input BAM contains secondary or supplementary alignments, which trigger pysam MM tag reading errors and invalidate upstream haplotagged input.

## Inputs

- GIAB trio-phased truth VCF for HG002
- HapCUT2 SNV-only phased VCF for HG002
- MethPhaser output VCF (meth_phaser_parallel + meth_phaser_post_processing, default best-accuracy parameters)
- MethPhaser relationship output describing per-block haplotype connections

## Outputs

- Per-phase-block haplotype relationship assignments relative to the GIAB truth VCF
- List of incorrectly connected larger phase blocks to remove prior to N50 calculation
- Corrected phase-block N50 and flip/switch error statistics (WhatsHap compare, Supplementary Data2)
- Counts of phased CMRG genes and genome-wide phase blocks per method

## How to apply

Load the GIAB trio-phased VCF and assign the true haplotype relationship of each SNV phase block from the HapCUT2/MethPhaser output, treating two neighboring blocks as sharing the same haplotype when their hap1/hap2 relationships to the truth VCF match. Cross-check MethPhaser's proposed block connections with its relationship output, applying the default voting-confidence filter (votes_hap1 - votes_hap2)/total_votes > 0.5 and the best-accuracy minimum read number of read coverage/(genome-wide block number/1000). Remove larger phase blocks that were incorrectly connected before any N50-style comparison, and restrict the benchmark to autosomes since random X inactivation makes sex chromosomes unreliable. Finally, quantify flip and switch errors introduced by block connection with WhatsHap compare and record the new-switch/established-connection rate (Supplementary Data2).

## Related tools

- **MethPhaser** (Produces the methylation-based block connections and relationship output that are validated against the GIAB truth VCF) — https://github.com/treangenlab/methphaser
- **HapCUT2** (Provides the baseline SNV-only phased VCF whose blocks are assigned relationships in the truth VCF)
- **WhatsHap** (compare mode quantifies flip and switch errors introduced by block connection)
- **SAMtools** (Filters the input BAM to primary alignments only (samtools view -bF 2304) so the benchmark inputs are valid)
- **meth_phaser_parallel** (Runs MethPhaser with voting confidence > 0.5 and best-accuracy minimum read number to generate block relationships) — https://github.com/treangenlab/methphaser

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -vc test_data/HLA.R10.phased.vcf.gz -vt truth.vcf -o test_data/work
```

## Evaluation signals

- The CMRG table reproduces the reported results: 265/273 (97.1%) genes phased by MethPhaser vs 258 (94.51%) by SNV-only phasing, with phase blocks reduced from 160 to 140 on HG002.
- Every retained block connection matches the hap1/hap2 relationship to the GIAB trio-phased VCF; neighboring blocks are only merged when relationships agree.
- Flip and switch error figures from WhatsHap compare are consistent with Supplementary Data2 (new-switch/established-connection rate shows only a minimal increase in switch error).
- No sex chromosomes (X/Y) are present in the benchmarked block set — only autosomes.
- Incorrectly connected larger phase blocks are removed before N50 is computed, so the corrected N50 differs from the raw MethPhaser N50.

## Limitations

- Sex chromosomes cannot be improved or benchmarked in clinical samples due to random deactivation of chromosome X in females, producing inconsistent haplotype patterns.
- MethPhaser depends on initial SNV phasing accuracy; high SNV phasing errors (e.g., very high hamming distance in R9 80x data) negatively impact accuracy.
- Results depend on the accuracy of methylation detection, overall read length, and a minimum data coverage (README recommends at least 30X ONT coverage).

## Evidence

- [methods] Removal of incorrectly connected larger phase blocks before N50 calculation: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB."
- [methods] Voting confidence filter for block relationships: "(Votes for one haplotype-votes for the other one)/total votes > 0.5"
- [discussion] Restricting benchmark to autosomes: "Thus, we exclude the X and Y chromosomes from our benchmark"
- [discussion] Minimal switch error from joining blocks: "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
- [readme] Sex chromosome exclusion in README: "MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes."
