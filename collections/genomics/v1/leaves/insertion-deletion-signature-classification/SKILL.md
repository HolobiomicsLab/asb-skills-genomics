---
name: insertion-deletion-signature-classification
description: Use when a contig-to-reference BAM (primary + supplementary split alignments)
  from a diploid assembly must be converted into large INS/DEL SV signatures, e.g.
  after running VolcanoSV-asm on HiFi/CLR/ONT reads.
when_to_use_negative:
- 'Input is raw read-to-reference alignments without an assembly: the read-based signature
  filtering (e.g. supporting-signature-depth bounds lb=0.2/rb=2.6 for HiFi) is a separate
  downstream step, not this block-geometry classification'
- 'Target is sex chromosomes: VolcanoSV is designed only for autosome chromosomes
  since the phasing procedure for sex chromosomes is very different'
- Target is small indels or complex SVs (TRA requires WGS BAM support and is handled
  by separate VolcanoSV-vc modules)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VolcanoSV
  role: Provides the VolcanoSV-vc Large_INDEL module (volcanosv-vc-large-indel.py)
    that consumes assembly-output BAM alignments and emits classified large INS/DEL
    signatures toward a phased VCF
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: pysam
  role: Python library used to parse the contig-to-reference BAM and extract primary/supplementary
    alignment blocks per contig for Diff_dis/Diff_olp computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:86de0d8d4234a3d83ded206b08024a1832ddb3b4cd1e86acb75ca6737ae8fa69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  - pysam
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# insertion-deletion-signature-classification

## Summary

Classifies large insertion and deletion SV signatures (50–250 bp and larger) by analyzing consecutive split alignment blocks of contigs aligned to a reference, using the Diff_dis reference-distance and Diff_olp overlap metrics with thresholds TH_dis=30 bp and TH_olp=3000 bp. This is the large-indel detection module of VolcanoSV-vc, which ultimately contributes to a phased VCF output.

## When to use

Use when a contig-to-reference BAM (primary + supplementary split alignments) from a diploid assembly must be converted into large INS/DEL SV signatures, e.g. after running VolcanoSV-asm on HiFi/CLR/ONT reads.

## When NOT to use

- Input is raw read-to-reference alignments without an assembly: the read-based signature filtering (e.g. supporting-signature-depth bounds lb=0.2/rb=2.6 for HiFi) is a separate downstream step, not this block-geometry classification
- Target is sex chromosomes: VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- Target is small indels or complex SVs (TRA requires WGS BAM support and is handled by separate VolcanoSV-vc modules)

## Inputs

- Contig-to-reference BAM (e.g. minimap2 alignments of VolcanoSV-asm contigs to hg19, Hifi_L2_hg19_minimap2_chr10.bam) with primary and supplementary records
- Reference FASTA and index (e.g. genome.fa / genome.fa.fai from Zenodo test data)
- VolcanoSV-asm diploid assembly contigs per chromosome

## Outputs

- signatures.tsv with columns chromosome, start, svlen (svlen<0 for DEL, svlen>0 for INS)
- Phased VCF file of large indel SVs produced by the VolcanoSV-vc large indel module (bin/VolcanoSV-vc/Large_INDEL/volcanosv-vc-large-indel.py)

## How to apply

Parse the contig-to-reference BAM (e.g. Hifi_L2_hg19_minimap2_chr10.bam from the Zenodo test data) with pysam, extracting all primary and supplementary alignment blocks per contig. For each pair of consecutive blocks on the same contig compute Diff_dis (reference distance) and Diff_olp (contig overlap). Classify as large INS when Diff_dis exceeds the deletion-distance threshold (TH_dis=30 bp) with non-overlapping blocks, and as large DEL when Diff_olp exceeds the overlap threshold (TH_olp=3000 bp); record the reference chromosome and start, and derive svlen from block geometry (svlen<0 for DEL, svlen>0 for INS). Rationale: assembly contigs span breakpoints fully, so alignment-block geometry between consecutive blocks directly encodes the SV type and length; the module feeds the downstream signature-depth and clustering filters that remove false calls before emission of the phased VCF.

## Related tools

- **VolcanoSV** (Provides the VolcanoSV-vc Large_INDEL module (volcanosv-vc-large-indel.py) that consumes assembly-output BAM alignments and emits classified large INS/DEL signatures toward a phased VCF) — https://github.com/maiziezhoulab/VolcanoSV
- **pysam** (Python library used to parse the contig-to-reference BAM and extract primary/supplementary alignment blocks per contig for Diff_dis/Diff_olp computation)

## Examples

```
import pysam; bam = pysam.AlignmentFile('Hifi_L2_hg19_minimap2_chr10.bam','rb'); [a for a in bam.fetch() if a.is_supplementary or not a.is_secondary]
```

## Evaluation signals

- signatures.tsv exists and every row contains the chromosome, start, and svlen fields with a non-zero row count
- Every svlen sign is consistent with its INS/DEL classification under the TH_dis=30 bp / TH_olp=3000 bp thresholds (svlen>0 for INS, svlen<0 for DEL)
- Diff_dis values for INS signatures exceed 30 bp and Diff_olp values for DEL signatures exceed 3000 bp when re-derived from block coordinates
- Running the provided Zenodo test data (Hifi_L2_hg19_minimap2_chr10.bam) reproduces the expected result before applying to user data, as the README recommends
- Downstream, calls survive the supporting-signature-depth filter (HiFi bounds lb=0.2, rb=2.6) and appear in the final phased VCF

## Limitations

- The GIAB gold standard SV callset used for evaluation is not a complete set and could also contain false positives
- Assembly algorithms may miss complex SVs because many build contigs by following the simplest paths through overlapping reads; this module handles only large INS/DEL signatures, not complex events
- VolcanoSV is designed only for autosome chromosomes; sex-chromosome phasing is not supported
- Detecting small indels based on assembly proved ineffective for ONT data (low precision for all three tools)

## Evidence

- [other] thresholds from workflow: [evidence span withheld — anchor status: not_found]
- [other] module and output: [evidence span withheld — anchor status: not_found]
- [discussion] downstream false-call filtering: "A raw SV call needs at least one supporting signature, otherwise it will be filtered out as a false call"
- [readme] test data validation: "we highly recommend trying the tool on the provided [test data](https://zenodo.org/records/10520476) and checking whether you have the correct result before trying it on your data"
- [readme] autosome restriction: "VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!"
