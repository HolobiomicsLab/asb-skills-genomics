---
name: snv-phase-block-connection-verification
description: Reference for benchmarking haplotype phasing accuracy when a method (e.g.,
  MethPhaser) joins SNV phase blocks and you must validate each connection against
  a trio-phased truth VCF before reporting N50 or switch-error improvements.
when_to_use_negative:
- Phasing sex chromosomes — random X inactivation in females produces inconsistent
  haplotype patterns, and the README states MethPhaser 'won't work on sex chromosomes'
- Samples lacking a trio-phased truth set — per-connection verification against GIAB
  trio-phased VCF is not possible for non-benchmark genomes
- Coverage below 30X ONT — SNV phasing methods like HapCUT2 and WhatsHap only work
  reasonably above that coverage, so the baseline being verified is unreliable
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: MethPhaser
  role: Produces the methylation-extended phase blocks whose connections are verified
    against the trio-phased truth VCF
  repo: https://github.com/treangenlab/methphaser
- name: WhatsHap
  role: haplotag module tags reads from HapCUT2 SNV phasing; compare module computes
    flip and switch errors on HG002
- name: HapCUT2
  role: Produces the SNV-only phasing baseline (extractHAIRS --ont 1; hapcut2 --outvcf
    1) against which corrected N50 is compared
- name: meth_phaser_post_processing
  role: Emits the modified reads and output VCF that are evaluated for connection
    correctness
  repo: https://github.com/treangenlab/methphaser
- name: SAMtools
  role: Filters BAM to primary alignments only (samtools view -bF 2304) to avoid pysam
    MM tag reading errors
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:b14e4aa014b6bcdbf19c9b1310f45879f991541246d8d94833c31a323fb89fc1
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
  - meth_phaser_post_processing
  - minimap2
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# snv-phase-block-connection-verification

## Summary

Verify the correctness of SNV phase block connections established by a phasing method (e.g., MethPhaser) by comparing each block connection against the GIAB trio-phased truth VCF for HG002, then compute corrected phase block N50 and switch-error metrics. This is how the MethPhaser paper separates genuine phasing gains from incorrectly joined blocks.

## When to use

Reference for benchmarking haplotype phasing accuracy when a method (e.g., MethPhaser) joins SNV phase blocks and you must validate each connection against a trio-phased truth VCF before reporting N50 or switch-error improvements.

## When NOT to use

- Phasing sex chromosomes — random X inactivation in females produces inconsistent haplotype patterns, and the README states MethPhaser 'won't work on sex chromosomes'
- Samples lacking a trio-phased truth set — per-connection verification against GIAB trio-phased VCF is not possible for non-benchmark genomes
- Coverage below 30X ONT — SNV phasing methods like HapCUT2 and WhatsHap only work reasonably above that coverage, so the baseline being verified is unreliable

## Inputs

- Haplotagged BAM (WhatsHap haplotag output, primary alignments only via samtools view -bF 2304)
- HapCUT2 phased SNV VCF
- MethPhaser output VCF (meth_phaser_post_processing -ov)
- GIAB trio-phased truth VCF for HG002
- GIAB high-confidence autosomal region BED (X and Y excluded)

## Outputs

- Per-connection correctness labels for MethPhaser block joins relative to GIAB trio truth
- Corrected phase block N50 (after removing incorrectly connected larger blocks)
- Switch-error and flip-error rates from WhatsHap compare
- Per-connection switch-error rate (new switches / number of block connections established)
- Benchmark metrics table (e.g., methphaser_hg002_r9_benchmark.csv)

## How to apply

Run the SNV-based phasing baseline (HapCUT2 with extractHAIRS --ont 1 and hapcut2 --outvcf 1, followed by WhatsHap haplotag) and the methylation-based extension (MethPhaser with meth_phaser_parallel then meth_phaser_post_processing) on methylation-tagged ONT reads aligned with minimap2 v2.24 preserving MM and ML tags. For every new phase block connection introduced by the extension method, check whether it agrees with the GIAB trio-phased truth VCF restricted to high-confidence autosomal regions (X and Y excluded); remove larger phase blocks that were incorrectly connected before computing N50. Define N50 as the minimum block length at which blocks of that size or larger span at least 50% of total phased length. Assess phasing accuracy with WhatsHap compare (flip and switch errors on HG002), and report the per-connection switch-error rate by dividing new switches introduced by the method by the number of phase block connections it established.

## Related tools

- **MethPhaser** (Produces the methylation-extended phase blocks whose connections are verified against the trio-phased truth VCF) — https://github.com/treangenlab/methphaser
- **WhatsHap** (haplotag module tags reads from HapCUT2 SNV phasing; compare module computes flip and switch errors on HG002)
- **HapCUT2** (Produces the SNV-only phasing baseline (extractHAIRS --ont 1; hapcut2 --outvcf 1) against which corrected N50 is compared)
- **meth_phaser_post_processing** (Emits the modified reads and output VCF that are evaluated for connection correctness) — https://github.com/treangenlab/methphaser
- **SAMtools** (Filters BAM to primary alignments only (samtools view -bF 2304) to avoid pysam MM tag reading errors)

## Examples

```
./meth_phaser_post_processing -ib HG002.haplotagged.bam -if work/ -ov output.vcf -vc HapCUT2.phased.vcf.gz
```

## Evaluation signals

- After removing incorrectly connected larger blocks, HG002 R9 N50 fold improvements should reproduce ~1.78x at 30X, ~2.34x at 60X, and ~2.53x at 80X versus SNV-only phasing
- Switch error after block joining should stay minimal, within the reported 0.0010%–0.0152% range (Supplementary Data2)
- Per-connection error rate (new switches divided by number of established block connections) should be computed and reported rather than only genome-wide switch error
- All metrics restricted to GIAB high-confidence autosomal regions with X and Y excluded; variant calls filtered to AF > 0.25
- N50 defined as minimum block length whose blocks of that size or greater span >= 50% of total phased length

## Limitations

- MethPhaser depends on initial SNV phasing accuracy; high SNV phasing errors (e.g., very high hamming distance in R9 80X results) negatively impact MethPhaser's accuracy
- Results depend on accuracy of methylation detection, overall read length, and a certain minimum coverage of the dataset
- Concrete checks and thresholds MethPhaser uses to uphold accurate phasing are not detailed in the discussion, leaving robustness to high SNV phasing errors to future versions

## Evidence

- [methods] we can easily know the correctness of MethPhaser’s each SNV phase block connection with our relationship output and further get the correct N50 on the HG002 sample: "we can easily know the correctness of MethPhaser’s each SNV phase block connection with our relationship output and further get the correct N50 on the HG002 sample"
- [methods] we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB.: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB."
- [discussion] we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error): "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
- [methods] limited our interested region to “high confidence” regions provided by GIAB to remove potential Clair3’s false positive calls.: "limited our interested region to “high confidence” regions provided by GIAB to remove potential Clair3’s false positive calls."
- [readme] MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes.: "MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes."
