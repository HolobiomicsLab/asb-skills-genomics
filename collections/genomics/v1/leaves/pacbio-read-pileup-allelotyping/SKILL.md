---
name: pacbio-read-pileup-allelotyping
description: Use when calling SNVs from error-prone PacBio SMRT (or ONT) whole-genome
  BAM alignments, where candidate variants must be detected from a read pileup and
  read-level allele observations must be extracted for haplotype-informed genotyping.
when_to_use_negative:
- The input is short-read data (e.g. Illumina aligned with BWA-MEM) — short-read callers
  such as GATK HaplotypeCaller or FreeBayes are the intended tools for that regime
- The goal is de novo calling of short indels or structural variants — Longshot only
  calls SNVs (it can genotype indels only when provided in an input VCF, and SVs >50
  bp are unsupported)
- The BAM lacks accurate alignments in highly similar segmental duplications, where
  correctly mapping PacBio reads remains a challenge and pileup-based allelotyping
  may be unreliable
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: Performs pileup-based candidate SNV identification, pair-HMM local realignment/allelotyping,
    and haplotype-informed phased genotyping on PacBio BAM alignments
  repo: https://github.com/pjedge/longshot
- name: HapCUT2
  role: Called internally by Longshot to estimate the most likely pair of haplotypes
    during iterative likelihood optimization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:1b8c49fa806b7cc927bd9772d9da606c0eb11ab7f24000a34d7ab824a50cbbbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - HapCUT2
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pacbio-read-pileup-allelotyping

## Summary

Identifies candidate SNVs from a pileup of aligned PacBio long reads by computing genotype likelihoods (as in Samtools) and then performs sensitive allelotyping of reads at those sites, feeding haplotype-informed phased genotyping as implemented in Longshot.

## When to use

Use when calling SNVs from error-prone PacBio SMRT (or ONT) whole-genome BAM alignments, where candidate variants must be detected from a read pileup and read-level allele observations must be extracted for haplotype-informed genotyping.

## When NOT to use

- The input is short-read data (e.g. Illumina aligned with BWA-MEM) — short-read callers such as GATK HaplotypeCaller or FreeBayes are the intended tools for that regime
- The goal is de novo calling of short indels or structural variants — Longshot only calls SNVs (it can genotype indels only when provided in an input VCF, and SVs >50 bp are unsupported)
- The BAM lacks accurate alignments in highly similar segmental duplications, where correctly mapping PacBio reads remains a challenge and pileup-based allelotyping may be unreliable

## Inputs

- Sorted, indexed BAM/CRAM of error-prone long SMS reads (e.g. PacBio SMRT reads aligned with BLASR to hs37d5 or hg38)
- Indexed FASTA reference genome matching the BAM alignment
- Optional: gzipped, tabix-indexed VCF of potential variants for genotyping/phasing mode (-v)

## Outputs

- Phased VCF of called SNVs with genotype qualities (GQ) and haplotype/phase information
- Optional haplotype-tagged BAM (HP:i:1 / HP:i:2 tags) from --out_bam
- Per-sample SNV precision and recall when compared against GIAB high-confidence truth sets (e.g. NA24385 recall 0.9608 at 28x to 0.9798 at 64x)

## How to apply

Align long SMS reads (e.g. GIAB PacBio data aligned with BLASR to hs37d5 or hg38) into a sorted, indexed BAM and run Longshot with --bam/--ref/--out. Candidate SNVs are identified from the pileup via a Samtools-like genotype likelihood calculation, then reads are locally realigned to ref/alt sequences using a pair-HMM forward algorithm for accurate allelotyping despite the high SMS error rate. Apply the default candidate filters: min_mapq 20, min_cov 6, min_alt_count 3, min_alt_frac 0.125, and discard allele observations with phred quality below 7.0; set maximum coverage via -A (auto_max_cov = mean coverage + 5*sqrt(mean coverage)) to guard against copy-number-amplification false SNVs. Variants flagged by the strand-bias filter (two-tailed Fisher's exact test p < 0.01) are excluded from haplotype-informed genotyping, and SNVs in dense windows (>10 SNVs per 500 bp) are noted as density-filter ('dn') candidates. Processing chromosomes separately with --region improves scalability on whole-genome BAMs.

## Related tools

- **Longshot** (Performs pileup-based candidate SNV identification, pair-HMM local realignment/allelotyping, and haplotype-informed phased genotyping on PacBio BAM alignments) — https://github.com/pjedge/longshot
- **HapCUT2** (Called internally by Longshot to estimate the most likely pair of haplotypes during iterative likelihood optimization)

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out output.vcf -A --region chr1
```

## Evaluation signals

- Reproduced NA12878 30x metrics match the reported P=0.9942 / R=0.9592 against the GIAB truth set (45x R=0.9734)
- NA24385 recall rises monotonically with coverage, from 0.9608 at 28x to 0.9798 at 64x, with precision changing only moderately
- Candidate sites excluded: strand-bias Fisher's exact test p < 0.01 variants absent from haplotype-informed genotyping; density-filter 'dn' flags (>10 SNVs per 500 bp) recorded
- Auto max-coverage flag set to mean coverage + 5*sqrt(mean coverage) for each downsampled BAM; min_mapq 20 / min_cov 6 defaults applied
- Output VCF is phased and GQ-filtered with a variable GQ cutoff matched to the median read coverage for comparison against GIAB calls

## Limitations

- Allele observations with phred-scaled quality below 7.0 are discarded, which reduces the effective read depth for SMS reads
- Some Longshot false positives may correspond to false negatives in the GIAB high-confidence call sets, since those truth sets were generated with short reads and are biased in favor of short-read technologies
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, affecting pileup allelotyping in those regions
- Accurate calling of short indels from SMS reads is challenging due to the high insertion/deletion error rate; Longshot calls only SNVs de novo

## Evidence

- [discussion] Candidate SNV identification from pileup: "Potential SNVs are identified from the pileup of aligned reads by performing a genotype likelihood calculation similar to Samtools"
- [discussion] Pair-HMM allelotyping: "This local realignment is performed using a pair-HMM29."
- [discussion] Strand bias filter: [evidence span withheld — anchor status: not_found]
- [readme] Auto max coverage: "-A, --auto_max_cov        Automatically calculate mean coverage for region and set max coverage to mean_coverage +
                              5*sqrt(mean_coverage). (SLOWER)"
- [methods] Default filters: "Minimum coverage (of reads passing filters) to consider position as a potential SNV. [default: 6]"
- [discussion] Effective depth caveat: [evidence span withheld — anchor status: not_found]
- [readme] Example invocation: "longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf"
