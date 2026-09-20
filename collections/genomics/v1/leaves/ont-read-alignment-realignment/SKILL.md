---
name: ont-read-alignment-realignment
description: Use when ONT (or PacBio) reads are provided as alignments to a different
  reference (e.g. GRCh38) than the one required by the downstream variant-calling
  pipeline, or when reproducing the Clairvoyante HG001 chr1 benchmark.
when_to_use_negative:
- 'Illumina reads: they are left on their original alignments and only downsampled
  to 50x coverage; NGMLR re-alignment is intended for long reads (PacBio/ONT)'
- When the downstream caller accepts alignments in the reference frame the reads already
  have — re-alignment is only needed because the ONT alignments were not to GRCh37
  decoy v5
- 'If mapping-quality-based read filtering is expected: the authors state ''We do
  not suggest removing any alignment by their mapping quality'', so this skill''s
  output should not be filtered that way'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NGMLR
  role: Long-read aligner used to re-align ONT reads to GRCh37 decoy version 5
- name: Oxford Nanopore
  role: 'Source sequencing technology: MinION 1D ligation, R9.4 chemistry rel3 reads
    for HG001'
  repo: https://github.com/nanopore-wgs-consortium/NA12878
- name: Clairvoyante
  role: Downstream variant caller consuming the re-aligned BAM (callVarBam.py, PyPy
    recommended)
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Nanopolish
  role: Benchmarking variant caller run on the same re-aligned ONT alignments in whole-genome
    calling mode
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:8eecb8bae7b04abfb62c5daf60bed6b39e0bf257dce08e338f5393c2c5d9e738
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanopore-wgs-consortium/NA12878
  tools:
  - Clairvoyante
  - NGMLR
  - Nanopolish
  - Oxford Nanopore
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ont-read-alignment-realignment

## Summary

Re-align Oxford Nanopore whole-genome reads against the GRCh37 decoy version 5 reference using NGMLR v0.2.3 so that the resulting BAMs can be consumed by variant callers such as Clairvoyante and Nanopolish. This step is necessary because the public rel3 ONT alignments are not in the reference frame the deep-learning tensor extraction expects.

## When to use

Use when ONT (or PacBio) reads are provided as alignments to a different reference (e.g. GRCh38) than the one required by the downstream variant-calling pipeline, or when reproducing the Clairvoyante HG001 chr1 benchmark.

## When NOT to use

- Illumina reads: they are left on their original alignments and only downsampled to 50x coverage; NGMLR re-alignment is intended for long reads (PacBio/ONT)
- When the downstream caller accepts alignments in the reference frame the reads already have — re-alignment is only needed because the ONT alignments were not to GRCh37 decoy v5
- If mapping-quality-based read filtering is expected: the authors state 'We do not suggest removing any alignment by their mapping quality', so this skill's output should not be filtered that way

## Inputs

- Oxford Nanopore whole-genome alignments or FASTQ for HG001 (nanopore-wgs-consortium rel3 release, hosted at s3://nanopore-human-wgs)
- GRCh37 decoy version 5 reference genome (FASTA)
- NGMLR v0.2.3 aligner

## Outputs

- Sorted, indexed BAM of ONT reads aligned to GRCh37 decoy v5 (one per sample/chromosome scope used for calling)
- Re-aligned read set suitable for Clairvoyante tensor extraction and Nanopolish variant calling

## How to apply

Extract the reads from the downloaded ONT BAM alignments (e.g. the nanopore-wgs-consortium HG001 'rel3' release, sequenced on a MinION with 1D ligation kits at 450 bp/s and R9.4 chemistry) and align them de novo to the GRCh37 decoy version 5 reference with NGMLR version 0.2.3, which is optimized for the error profile and long-read indels of nanopore data. Keep every alignment: the authors explicitly advise not filtering reads by mapping quality, since Clairvoyante's tensor extraction benefits from including confidently mis-mapped reads as negative signal. Produce a sorted, indexed BAM per sample/chromosome that downstream Clairvoyante callVarBam.py (run under PyPy for speed) and Nanopolish whole-genome calling can read. Decision point: Illumina reads used in the same study are kept on their original alignments and downsampled to 50x, whereas PacBio and ONT reads are always re-aligned with NGMLR.

## Related tools

- **NGMLR** (Long-read aligner used to re-align ONT reads to GRCh37 decoy version 5)
- **Oxford Nanopore** (Source sequencing technology: MinION 1D ligation, R9.4 chemistry rel3 reads for HG001) — https://github.com/nanopore-wgs-consortium/NA12878
- **Clairvoyante** (Downstream variant caller consuming the re-aligned BAM (callVarBam.py, PyPy recommended)) — https://github.com/aquaskyline/Clairvoyante
- **Nanopolish** (Benchmarking variant caller run on the same re-aligned ONT alignments in whole-genome calling mode)

## Examples

```
aws s3 cp s3://nanopore-human-wgs/rel3-nanopore-wgs-288418386-FAB39088.fastq.gz . && ngmlr -r GRCh37_decoy_v5.fasta -q rel3-nanopore-wgs-288418386-FAB39088.fastq.gz -o hg001_ont_grch37d5.sam
```

## Evaluation signals

- The output BAM reference is GRCh37 decoy version 5 (verify via @SQ header), matching the reference used for Clairvoyante model training tensors
- No alignments removed by mapping quality — total read count before/after re-alignment should be conserved (reads are re-mapped, not filtered)
- Re-aligned BAM is sorted and indexed so callVarBam.py and Nanopolish can stream it without errors
- Downstream sanity check: Clairvoyante on the re-aligned HG001 chr1 ONT BAM restricted to 1KGp3 common sites should reproduce F1 90.53%, FPR 2.96%, FNR 14.78%, precision 96.55%; Nanopolish F1 86.93%, FPR 0.04%, FNR 21.57%, precision 97.51%

## Limitations

- Only HG001 ONT data were available in this study, limiting cross-sample re-alignment and combined-sample training on ONT data
- Reads must be extracted from the public consortium alignments first — the rel3 BAMs are not directly in the GRCh37 decoy v5 coordinate frame
- Clairvoyante's downstream tensor extraction does not use base qualities, so the re-alignment quality gain comes from the NGMLR mapping, not from quality recalibration

## Evidence

- [discussion] Re-align ONT reads with NGMLR to GRCh37 decoy v5: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] rel3 sequencing chemistry context: "we used the ‘rel3’ release sequenced on the Oxford Nanopore MinION using 1D ligation kits (450 bp/s) and R9.4 chemistry"
- [discussion] Do not filter by mapping quality after alignment: "We do not suggest removing any alignment by their mapping quality"
- [readme] Download the source ONT read data from the consortium S3 bucket: [evidence span withheld — anchor status: external_doc]
