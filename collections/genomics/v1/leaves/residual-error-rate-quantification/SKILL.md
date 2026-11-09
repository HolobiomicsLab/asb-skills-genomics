---
name: residual-error-rate-quantification
description: Use when comparing error-correction tools on long reads (PacBio CLR or
  ONT) where the ground-truth reference haplotypes are known and you need a per-tool
  residual error rate / QV to fill a benchmark table.
when_to_use_negative:
- No ground-truth reference haplotypes exist (e.g. purely real metagenome samples
  without known composition) — residual error cannot be computed against simulated
  truth
- You need switch-error or assembly-level haplotype-awareness metrics rather than
  residual base errors on reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Aligns each tool's corrected reads back against the simulated reference haplotypes
    (using map-pb or map-ont presets) so residual error rates can be computed
  repo: https://github.com/lh3/minimap2
- name: VeChat
  role: Produces the corrected-read output whose residual error rate is quantified
    and compared against baseline tools
  repo: https://github.com/HaploKit/vechat
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:fd7d63c59d436480f1a463ca767ddff53b93db5feb222b47f8fb06a8a80a88a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - CONSENT
  - Canu
  - Daccord
  - Minimap2
  - PBSIM2
  - Racon
  - VeChat
  - fpa
  - minimap2
  - yacrd
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# residual-error-rate-quantification

## Summary

Quantify how many sequencing errors remain after long-read error correction by aligning corrected reads back to known reference haplotypes and computing residual error rates (QV) per tool, platform, and ploidy/metagenome condition — the core metric used to benchmark VeChat against Racon, Canu, Daccord and CONSENT.

## When to use

Use when comparing error-correction tools on long reads (PacBio CLR or ONT) where the ground-truth reference haplotypes are known and you need a per-tool residual error rate / QV to fill a benchmark table.

## When NOT to use

- No ground-truth reference haplotypes exist (e.g. purely real metagenome samples without known composition) — residual error cannot be computed against simulated truth
- You need switch-error or assembly-level haplotype-awareness metrics rather than residual base errors on reads

## Inputs

- Corrected long reads (FASTA, e.g. reads.corrected.fa) from each correction tool being benchmarked
- Raw long reads (FASTQ/FASTA) from the same run for baseline comparison
- Reference haplotype sequences (e.g. E. coli strain references used to simulate diploid/triploid/tetraploid read sets)

## Outputs

- Per-read alignment file (SAM/PAF) of corrected reads against reference haplotypes
- Residual sequencing error rate / QV per tool, platform, and ploidy or metagenome condition
- Consolidated benchmark table (error_rate_table.tsv) with one row per tool x condition

## How to apply

After each correction tool (e.g. VeChat, Racon, the Canu correction module, Daccord, CONSENT) has produced corrected reads, align each tool's corrected reads back against the simulated reference haplotypes with minimap2 using the platform-appropriate preset (map-pb for PacBio CLR, map-ont for ONT). From these alignments compute residual sequencing error rates, expressed as QV, per tool, sequencing platform, and ploidy/metagenome condition (diploid, triploid, tetraploid, metagenomes). Assemble all error-rate rows into a single table (e.g. error_rate_table.tsv) so per-tool columns can be compared directly. Judge correctness by checking that the expected improvement ratios hold — VeChat approximately 14–30x, 9–26x and 4–11x lower error rates on diploid, triploid and tetraploid simulated E. coli genomes respectively — and treat deviations beyond expected stochastic simulation variance as a failed reproduction.

## Related tools

- **minimap2** (Aligns each tool's corrected reads back against the simulated reference haplotypes (using map-pb or map-ont presets) so residual error rates can be computed) — https://github.com/lh3/minimap2
- **VeChat** (Produces the corrected-read output whose residual error rate is quantified and compared against baseline tools) — https://github.com/HaploKit/vechat

## Examples

```
minimap2 -ax map-pb ecoli_refs.fa reads.corrected.fa > corrected_vs_ref.sam; # then compute residual error rate (QV) per tool from alignment mismatches
```

## Evaluation signals

- The reproduced error_rate_table.tsv shows VeChat with the lowest error rate on the simulated E. coli conditions (e.g. VeChat ≈ 0.014% vs Racon 0.276% on PacBio diploid)
- Per-condition improvement ratios fall within the reported ranges: ~14–30x lower (diploid), ~9–26x (triploid), ~4–11x (tetraploid) relative to baselines
- Coverage scaling check: on PacBio CLR triploid reads, VeChat's corrected-read error rate drops from ≈0.311% at 10x to ≈0.017% at 50x coverage per haplotype
- QV values are reported per tool, per platform (PacBio CLR vs ONT), and per ploidy/metagenome condition without gaps
- Deviations beyond expected stochastic simulation variance relative to the paper's benchmark flag the reproduction as failed

## Limitations

- Results on simulated data carry stochastic simulation variance; exact QV values may deviate, and only deviations beyond expected variance should flag failure
- Residual error rate on reads does not by itself capture downstream effects — VeChat's benefit on assembly haplotype-awareness must be evaluated separately
- Real metagenome datasets (e.g. yeast pseudo-diploid mock community, NWC metagenome, Microbial 10-plex) may not yield a VeChat win — on the human gut microbiome dataset Daccord achieved the lowest error rate (QV) while VeChat obtained comparable read accuracy

## Evidence

- [abstract] long reads corrected by VeChat contain 4 to 15 (Pacific Biosciences) and 1 to 10 times (Oxford Nanopore Technologies) less errors than when being corrected by state of the art approaches: "long reads corrected by VeChat contain 4 to 15 (Pacific Biosciences) and 1 to 10 times (Oxford Nanopore Technologies) less errors than when being corrected by state of the art approaches"
- [results] VeChat achieves approximately 1.5 times lower error rate (QV) and 1.2 times lower switch error on HG002 compared with CONSENT: "VeChat achieves approximately 1.5 times lower error rate (QV) and 1.2 times lower switch error on HG002 compared with CONSENT"
- [readme] ./minimap2 -ax map-pb ref.fa pacbio.fq.gz > aln.sam       # PacBio CLR genomic reads: [evidence span withheld — anchor status: external_doc]
- [readme] minimap2 -a test/MT-human.fa test/MT-orang.fa > test.sam: "Minimap2 seamlessly works with gzip'd FASTA and FASTQ formats as input. You
don't need to convert between FASTA and FASTQ or decompress gzip'd files first."
