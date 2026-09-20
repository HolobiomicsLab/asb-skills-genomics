---
name: genome-phasing-benchmark-evaluation
description: Use when evaluating whether methylation-based read phasing extends SNV-only
  phasing on ONT R9/R10 autosomal data — e.g., reproducing HG002 R9 coverage-sweep
  N50 fold improvements at 30X/60X/80X with switch-error accounting.
when_to_use_negative:
- Phasing or benchmarking on sex chromosomes — random X inactivation in females yields
  inconsistent haplotype methylation patterns, so X/Y are excluded
- Data with high SNV-phasing error (e.g., very high hamming distance in R9 80x) or
  < 30X coverage — SNV phasing methods (HapCUT2, WhatsHap) only work reasonably above
  ~30X, and MethPhaser depends on the accuracy of the initial SNV phasing
- Very short reads (e.g., 12–15 kbp N50 patient blood samples) where initially small
  phase blocks limit improvement
tools:
- name: MethPhaser
  role: Methylation-based read phasing and SNV-phase block chaining run on WhatsHap
    haplotagged BAM and HapCUT2-phased SNVs
  repo: https://github.com/treangenlab/methphaser
- name: HapCUT2
  role: SNV-only phasing baseline (extractHAIRS --ont 1; hapcut2 --outvcf 1) supplying
    phase blocks that MethPhaser extends
- name: WhatsHap
  role: haplotag module tags reads from HapCUT2 phasing; compare module computes switch
    and flip errors against truth
- name: minimap2
  role: Aligns ONT reads to GRCh38 v2.24 preserving MM and ML methylation tags
- name: Clair3
  role: Variant calling (v0.1-r12) with allele frequency > 0.25 filtering on GIAB
    high-confidence regions
- name: SAMtools
  role: Filters BAM to primary alignments (view -bF 2304) before methylation tag processing
- name: meth_phaser_post_processing
  role: Post-processing script producing the modified reads and output VCF with chained
    phase blocks
  repo: https://github.com/treangenlab/methphaser
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:e22c4f4e14684dd4a6a5e0d9e4ae2168650f9d41a63ffcc740e0dbaec047cf63
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# genome-phasing-benchmark-evaluation

## Summary

Benchmark and evaluate methylation-assisted long-read haplotype phasing (e.g., MethPhaser) against an SNV-only baseline on ONT data, quantifying phase block N50 fold improvements and switch-error increases at multiple coverage levels.

## When to use

Use when evaluating whether methylation-based read phasing extends SNV-only phasing on ONT R9/R10 autosomal data — e.g., reproducing HG002 R9 coverage-sweep N50 fold improvements at 30X/60X/80X with switch-error accounting.

## When NOT to use

- Phasing or benchmarking on sex chromosomes — random X inactivation in females yields inconsistent haplotype methylation patterns, so X/Y are excluded
- Data with high SNV-phasing error (e.g., very high hamming distance in R9 80x) or < 30X coverage — SNV phasing methods (HapCUT2, WhatsHap) only work reasonably above ~30X, and MethPhaser depends on the accuracy of the initial SNV phasing
- Very short reads (e.g., 12–15 kbp N50 patient blood samples) where initially small phase blocks limit improvement

## Inputs

- Subsampled ONT R9/R10 reads (e.g., HG002 at 30X/60X/80X) with methylation MM/ML tags
- GRCh38 no_alt_analysis_set reference FASTQ/FASTA (GCA_000001405.15_GRCh38_no_alt_analysis_set)
- HapCUT2-phased SNV VCF and WhatsHap haplotagged BAM (primary alignments only)
- GIAB trio-phased truth VCF and GIAB high-confidence regions BED for HG002
- GTF block file from WhatsHap visualization (input to meth_phaser_parallel -g)

## Outputs

- MethPhaser output VCF with methylation-extended phase blocks (output.vcf)
- Phase block N50 per coverage level for SNV-only baseline vs. MethPhaser (e.g., fold improvements 1.78x/2.34x/2.53x at 30X/60X/80X)
- Switch-error and flip-error metrics per method (WhatsHap compare) and per-connection switch rate (e.g., 0.0010% to 0.0152%)
- Benchmark metrics CSV (e.g., methphaser_hg002_r9_benchmark.csv)

## How to apply

Subsample ONT reads to target coverages (30X, 60X, 80X), align with minimap2 v2.24 while preserving MM and ML methylation tags, and filter the BAM to primary alignments with `samtools view -bF 2304`. Call variants with Clair3 v0.1-r12, restrict to AF > 0.25 within GIAB high-confidence autosomal regions, then phase SNVs with HapCUT2 (extractHAIRS --ont 1) and haplotag reads with WhatsHap to build the SNV-only baseline. Run MethPhaser (meth_phaser_parallel followed by meth_phaser_post_processing) with its best-accuracy parameters — minimum read number = read coverage/(genome-wide block number/1000), voting confidence > 0.5, minimum haplotype coverage of 3, and minimum 3 votes per haplotype assignment. Compute phase block N50 after removing larger blocks incorrectly connected relative to the GIAB trio-phased truth VCF, and assess accuracy with WhatsHap compare, dividing new switches by the number of block connections MethPhaser established; report per-connection switch error in Supplementary Data2-style tables. The rationale: joining phase blocks via methylation-driven majority votes should lengthen N50 substantially while introducing only minimal per-connection switch error.

## Related tools

- **MethPhaser** (Methylation-based read phasing and SNV-phase block chaining run on WhatsHap haplotagged BAM and HapCUT2-phased SNVs) — https://github.com/treangenlab/methphaser
- **HapCUT2** (SNV-only phasing baseline (extractHAIRS --ont 1; hapcut2 --outvcf 1) supplying phase blocks that MethPhaser extends)
- **WhatsHap** (haplotag module tags reads from HapCUT2 phasing; compare module computes switch and flip errors against truth)
- **minimap2** (Aligns ONT reads to GRCh38 v2.24 preserving MM and ML methylation tags)
- **Clair3** (Variant calling (v0.1-r12) with allele frequency > 0.25 filtering on GIAB high-confidence regions)
- **SAMtools** (Filters BAM to primary alignments (view -bF 2304) before methylation tag processing)
- **meth_phaser_post_processing** (Post-processing script producing the modified reads and output VCF with chained phase blocks) — https://github.com/treangenlab/methphaser

## Examples

```
./meth_phaser_parallel -b test_data/HLA.R10.haplotagged.bam -r test_data/GCA_000001405.15_GRCh38_no_alt_analysis_set.chr6.fna -g test_data/LSK.filtered.gtf -vc test_data/HLA.R10.phased.vcf.gz -o test_data/work
```

## Evaluation signals

- Phase block N50 fold improvements over SNV-only phasing fall within expected ranges (e.g., 1.78x, 2.34x, 2.53x at 30X/60X/80X on HG002 R9)
- Per-connection switch-error increase stays within the reported range (0.0010% to 0.0152%) in Supplementary Data2
- All benchmarked regions restricted to GIAB high-confidence autosomal intervals; AF > 0.25 variant filter applied
- N50 computed only after removal of larger blocks incorrectly connected relative to the GIAB trio-phased truth VCF
- Input BAM contains only primary alignments (filtered with samtools view -bF 2304) to avoid pysam MM tag reading errors

## Limitations

- Cannot improve phasing for human sex chromosomes — random X inactivation in females leads to inconsistent haplotype methylation patterns; the README warns MethPhaser won't work on sex chromosomes and only autosomes should be considered
- Depends on initial SNV phasing accuracy — high SNV phasing errors (e.g., very high hamming distance in R9 80x results) negatively impact MethPhaser accuracy
- Results depend on methylation detection accuracy, overall read length, and a certain minimum data coverage; tissue-specific methylation signals make performance across tissues hard to predict
- The default `-ml -1` ignores the largest homozygous/centromeric region; use `-ml -2` when the input has no large poorly mapped region
- Concrete thresholds MethPhaser uses to uphold phasing accuracy are not fully detailed; robustness to high SNV phasing errors is deferred to future versions

## Evidence

- [abstract] MethPhaser designed abstract: "we designed MethPhaser, a method that utilizes methylation signals from Oxford Nanopore Technologies to extend Single Nucleotide Variation (SNV)-based phasing"
- [methods] N50 block removal vs truth: "we removed the larger phase blocks that we incorrectly connected based on the comparison of the SNV-based phasing method and the trio-phased VCF provided by GIAB."
- [methods] GIAB high-confidence filter: "limited our interested region to “high confidence” regions provided by GIAB to remove potential Clair3’s false positive calls."
- [discussion] minimal switch error rationale: "we could demonstrate that this joining of phase blocks leads to only a minimal increase in phasing error (i.e., switch error)"
- [readme] primary alignment BAM filter: "The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error."
