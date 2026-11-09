---
name: long-read-haplotype-phasing
description: Use when ONT long reads carry methylation tags (MM/ML) and existing SNV-only
  phase blocks from HapCUT2 or WhatsHap need to be joined into longer autosomal phase
  blocks.
when_to_use_negative:
- Phasing sex chromosomes (X/Y) — random X inactivation in females produces inconsistent
  methylation haplotype patterns; restrict to autosomes
- Coverage below ~30X ONT, since SNV phasing methods (HapCUT2, WhatsHap) only work
  reasonably above that coverage
- A BAM containing secondary or supplementary alignments, which triggers a pysam MM
  tag reading error — filter with `samtools view -bF 2304` first
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: MethPhaser
  role: Performs methylation-based read phasing and SNV-phase block chaining; meth_phaser_parallel
    computes block relationships and meth_phaser_post_processing emits the phased
    VCF and re-haplotagged BAM
  repo: https://github.com/treangenlab/methphaser
- name: minimap2
  role: Aligns methylation-aware ONT reads to the GRCh38 no-alt reference while retaining
    MM and ML base-modification tags (v2.24)
- name: Clair3
  role: Calls SNVs (v0.1-r12) that are filtered to AF > 0.25 within GIAB high-confidence
    regions before phasing
- name: HapCUT2
  role: Produces the initial SNV-only phasing (extractHAIRS --ont 1; hapcut2 --outvcf
    1) whose phase blocks MethPhaser chains
- name: WhatsHap
  role: haplotag module tags reads from HapCUT2 results, stats module generates the
    phase-block GTF, and compare benchmarks against the GIAB trio-phased truth VCF
- name: SAMtools
  role: Filters the input BAM to primary alignments only (samtools view -bF 2304)
    and counts HP-tagged reads for validation
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:2d59a8e3eea21969ffbe00444d93097584d65b5ba27b4128beca4d96c2aace17
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
  - WhatsHap compare
  - meth_phaser_parallel
  - minimap2
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# long-read-haplotype-phasing

## Summary

Chain SNV-based phase blocks into chromosome-scale blocks using allele-specific CpG methylation from ONT MM/ML tags (MethPhaser), applied after HapCUT2/WhatsHap SNV phasing. Extends phase-block N50 by 78%-151% on ONT R9/R10 data with only a minimal switch-error cost.

## When to use

Use when ONT long reads carry methylation tags (MM/ML) and existing SNV-only phase blocks from HapCUT2 or WhatsHap need to be joined into longer autosomal phase blocks.

## When NOT to use

- Phasing sex chromosomes (X/Y) — random X inactivation in females produces inconsistent methylation haplotype patterns; restrict to autosomes
- Coverage below ~30X ONT, since SNV phasing methods (HapCUT2, WhatsHap) only work reasonably above that coverage
- A BAM containing secondary or supplementary alignments, which triggers a pysam MM tag reading error — filter with `samtools view -bF 2304` first

## Inputs

- Methylation-annotated ONT BAM (primary alignments only, retaining MM/ML tags)
- GRCh38 no-alt reference FASTA (GCA_000001405.15_GRCh38_no_alt_analysis_set)
- WhatsHap stats phase-block GTF
- HapCUT2-phased SNV VCF (Clair3 calls filtered to AF > 0.25 in GIAB high-confidence regions)
- Optional GIAB trio-phased truth VCF (-vt) for benchmarking

## Outputs

- Methylation-phased VCF (output.vcf) with chained phase blocks
- Re-haplotagged BAM with updated HP tags from meth_phaser_post_processing
- Block-relationship work directory from meth_phaser_parallel
- WhatsHap compare metrics: corrected phase-block N50, switch/flip error rates, hamming distance

## How to apply

Basecall raw ONT reads with a methylation-aware basecaller (Dorado/Bonito/Guppy), align with minimap2 v2.24 keeping MM and ML tags, and keep only primary alignments via `samtools view -bF 2304`. Call variants with Clair3 v0.1-r12, filter to AF > 0.25 within GIAB high-confidence regions, phase with HapCUT2 (extractHAIRS --ont 1; hapcut2 --outvcf 1), haplotag reads with WhatsHap haplotag, and generate a WhatsHap stats GTF of phase blocks. Run meth_phaser_parallel with the haplotagged BAM (-b), reference (-r), GTF (-g), and HapCUT2 VCF (-vc), using -ml -1 to skip the largest (centromeric) homozygous gap and best-accuracy defaults: voting confidence > 0.5, minimum votes = 3, minimum haplotype coverage = 3, minimum read number = read coverage/(genome-wide block number/1000), and Wilcoxon rank-sum p < 0.05 for allele-specific CpGs. Then run meth_phaser_post_processing to emit the methylation-phased VCF (output.vcf) and the re-haplotagged BAM.

## Related tools

- **MethPhaser** (Performs methylation-based read phasing and SNV-phase block chaining; meth_phaser_parallel computes block relationships and meth_phaser_post_processing emits the phased VCF and re-haplotagged BAM) — https://github.com/treangenlab/methphaser
- **minimap2** (Aligns methylation-aware ONT reads to the GRCh38 no-alt reference while retaining MM and ML base-modification tags (v2.24))
- **Clair3** (Calls SNVs (v0.1-r12) that are filtered to AF > 0.25 within GIAB high-confidence regions before phasing)
- **HapCUT2** (Produces the initial SNV-only phasing (extractHAIRS --ont 1; hapcut2 --outvcf 1) whose phase blocks MethPhaser chains)
- **WhatsHap** (haplotag module tags reads from HapCUT2 results, stats module generates the phase-block GTF, and compare benchmarks against the GIAB trio-phased truth VCF)
- **SAMtools** (Filters the input BAM to primary alignments only (samtools view -bF 2304) and counts HP-tagged reads for validation)

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -g test_data/LSK.filtered.gtf -vc test_data/HLA.R10.phased.vcf.gz -o test_data/work
```

## Evaluation signals

- Corrected phase-block N50 after removing incorrectly connected blocks exceeds SNV-only phasing by 78%-151% (e.g., HG002 R10 30X: 3,152,506 vs 3,034,229 for 60X SNV-only)
- Switch-error increase is small: ~0.0532% on HG002 R10 and 0.02% flip-rate increase on HG002 R9 60X versus GIAB V4.2.1 trio truth
- WhatsHap compare against the GIAB trio-phased VCF shows correctly chained blocks; on HPRC chr1 evaluation hamming distance rises only ~5% with about one switched block
- Phase-block and medical-gene block counts drop (e.g., HG02080 1087->743 blocks, medical-gene blocks 123->113); HP-tagged read counts increase via SAMtools
- The input BAM contains only primary alignments and the autosomes-only restriction was honored

## Limitations

- Cannot improve phasing of sex chromosomes due to random X inactivation in females (use -ml -2 only if no centromere-like gap exists; default -ml -1 skips the largest homozygous region)
- Depends on initial SNV phasing accuracy — very high SNV phasing errors (e.g., very high hamming distance in R9 80x results) negatively impact MethPhaser accuracy
- Results depend on methylation detection accuracy, overall read length, and a minimum coverage; tissue-specific methylation (e.g., blood samples with shorter 12-15 kbp reads) limits gains

## Evidence

- [methods] MethPhaser input/output contract: "MethPhaser takes WhatsHap haplotagged reads, HapCUT2 phased SNVs as input and performs methylation-based read phasing and SNV-phase block chaining."
- [methods] Variant filtering threshold: "We also filtered the allele frequency (AF > 0.25)"
- [methods] Allele-specific methylation test: "We use the Wilcoxon rank sum test44,59 to determine whether the two allele’s base modification probabilities are statistically different (p-value < 0.05)."
- [readme] Autosome-only restriction: "MethPhaser won't work on sex chromosomes because of random X inactivation! Please only consider autosomes."
- [readme] Primary-alignment requirement: "The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error."
- [abstract] Expected N50 improvement: "For ONT R9 and R10 cell line data, we increase the phase length N50 by 78%-151% at a phasing accuracy of 83.4-98.7%"
- [discussion] Error-cost expectation: "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
- [discussion] Dependency on SNV phasing accuracy: "SNV phasing methods produce high phasing errors (e.g., very high hamming distance in R9 80x results) it will negatively impact the accuracy of MethPhaser"
