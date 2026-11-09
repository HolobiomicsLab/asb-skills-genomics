---
name: sv-signature-threshold-filtering
description: Use when mining a contig-to-reference BAM (primary + supplementary split
  blocks) for large INS/DEL signatures with the 30 bp Diff_dis / 3000 bp Diff_olp
  thresholds of the VolcanoSV-vc large indel module.
when_to_use_negative:
- The input is raw reads with no diploid assembly — this skill operates on contig-to-reference
  alignments from the VolcanoSV-asm output, not directly on read alignments
- Calling complex SVs (TRA/INV/DUP) — those require the VolcanoSV-vc Complex_SV module
  in WGS mode with WGS BAM support; translocation detection 'does not make sense to
  run on single chromosome level'
- Small indels — assembly-based small indel detection is a separate VolcanoSV-vc Small_INDEL
  module and was shown ineffective for ONT data
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3168
tools:
- name: VolcanoSV
  role: Provides the VolcanoSV-vc Large_INDEL module (bin/VolcanoSV-vc/Large_INDEL/volcanosv-vc-large-indel.py)
    that consumes the threshold-filtered signatures
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: pysam
  role: Parses the contig-to-reference BAM to extract primary and supplementary alignment
    blocks per contig for Diff_dis/Diff_olp computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:0206c06a520d496341ac6f5db48f696949f32a25aae8504dea823fcb5bab6caf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  - pysam
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# sv-signature-threshold-filtering

## Summary

Extracts large insertion/deletion SV signatures from contig-to-reference alignment blocks using VolcanoSV-vc's distance (Diff_dis, TH_dis=30 bp) and overlap (Diff_olp, TH_olp=3000 bp) thresholds, writing them to a signatures.tsv that feeds the large indel module's phased VCF output.

## When to use

Use when mining a contig-to-reference BAM (primary + supplementary split blocks) for large INS/DEL signatures with the 30 bp Diff_dis / 3000 bp Diff_olp thresholds of the VolcanoSV-vc large indel module.

## When NOT to use

- The input is raw reads with no diploid assembly — this skill operates on contig-to-reference alignments from the VolcanoSV-asm output, not directly on read alignments
- Calling complex SVs (TRA/INV/DUP) — those require the VolcanoSV-vc Complex_SV module in WGS mode with WGS BAM support; translocation detection 'does not make sense to run on single chromosome level'
- Small indels — assembly-based small indel detection is a separate VolcanoSV-vc Small_INDEL module and was shown ineffective for ONT data

## Inputs

- Contig-to-reference alignment BAM (primary + supplementary blocks), e.g. Hifi_L2_hg19_minimap2_chr10.bam from Zenodo test data
- Reference FASTA and index (e.g. hg19 genome.fa / genome.fa.fai)
- Diploid assembly contigs produced by VolcanoSV-asm (hifiasm/Flye/wtdbg2/miniasm/Shasta/NextDenovo/Canu)

## Outputs

- signatures.tsv with columns chromosome, start, svlen (svlen<0 for DEL, svlen>0 for INS)
- Phased VCF of large indel SVs produced by the VolcanoSV-vc Large_INDEL module

## How to apply

Parse the contig-to-reference BAM (e.g. the Zenodo test file Hifi_L2_hg19_minimap2_chr10.bam, hg19) with pysam, keeping primary and supplementary (split) alignment blocks per contig. For each pair of consecutive blocks on the same contig, compute the reference-distance metric Diff_dis and the overlap metric Diff_olp. Classify the pair as a large INS when Diff_dis exceeds TH_dis=30 bp with non-overlapping blocks, and as a large DEL when Diff_olp exceeds TH_olp=3000 bp; record the reference chromosome and start of each signature and derive svlen from block geometry (svlen<0 for DEL, svlen>0 for INS). Write signatures.tsv with columns chromosome, start, svlen, then hand it to the VolcanoSV-vc Large_INDEL script (volcanosv-vc-large-indel.py), which merges signatures into calls that ultimately appear in the module's phased VCF. Rationale: this is the assembly-side signature layer of VolcanoSV's hybrid pipeline — downstream read-based filtering requires every raw SV call to retain at least one supporting signature, so an overly permissive threshold pass leaks false calls while an overly strict one starves true calls of support.

## Related tools

- **VolcanoSV** (Provides the VolcanoSV-vc Large_INDEL module (bin/VolcanoSV-vc/Large_INDEL/volcanosv-vc-large-indel.py) that consumes the threshold-filtered signatures) — https://github.com/maiziezhoulab/VolcanoSV
- **pysam** (Parses the contig-to-reference BAM to extract primary and supplementary alignment blocks per contig for Diff_dis/Diff_olp computation)

## Examples

```
import pysam; bam = pysam.AlignmentFile('Hifi_L2_hg19_minimap2_chr10.bam','rb'); blocks = [a for a in bam.fetch() if not a.is_secondary]
```

## Evaluation signals

- signatures.tsv exists, contains chromosome, start, and svlen fields on every row, and has a non-zero row count
- Every svlen sign is consistent with its INS/DEL classification under the 30 bp (TH_dis) / 3000 bp (TH_olp) thresholds (INS: svlen>0; DEL: svlen<0)
- Every recorded signature start maps within the reference chromosome's coordinates in the input BAM's header
- Running the module on the provided Zenodo test data (Hifi_L2_hg19_minimap2_chr10.bam) reproduces the expected result before use on new data
- After downstream read-based filtering, no raw SV call in the output lacks at least one supporting signature (calls without support are false calls and must be dropped)

## Limitations

- VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- Signatures capture only large INS/DEL; assembly algorithms may miss complex SVs because many build contigs by following the simplest paths through overlapping reads
- These signature thresholds are one layer only — downstream filtering (svlen_sim threshold 0.5 for 50–250 bp reads, and empirical supporting-signature-depth boundaries such as lb=0.2, rb=2.6 for Hifi) still gates which calls survive
- Threshold values (TH_dis=30 bp, TH_olp=3000 bp) are pipeline-specific and may need revalidation for non-human references or other data types

## Evidence

- [readme] VolcanoSV is an innovative hybrid SV detection pipeline that utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly.: "VolcanoSV is an innovative hybrid SV detection pipeline that utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly."
- [discussion] A raw SV call needs at least one supporting signature, otherwise it will be filtered out as a false call: "A raw SV call needs at least one supporting signature, otherwise it will be filtered out as a false call"
- [readme] we highly recommend trying the tool on the provided [test data](https://zenodo.org/records/10520476) and checking whether you have the correct result before trying it on your data: "we highly recommend trying the tool on the provided [test data](https://zenodo.org/records/10520476) and checking whether you have the correct result before trying it on your data"
- [results] Many genome assembly algorithms build contigs by following the simplest paths through overlapping reads, which may miss complex SVs: "Many genome assembly algorithms build contigs by following the simplest paths through overlapping reads, which may miss complex SVs"
