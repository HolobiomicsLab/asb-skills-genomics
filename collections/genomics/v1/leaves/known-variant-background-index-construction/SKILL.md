---
name: known-variant-background-index-construction
description: Use when calling SNVs from ONT (R10.4/R10.4.1) long-read BAMs with miniSNV
  and you want known common population variants (e.g. a 1000 Genomes VCF) as a variation
  background to accelerate and sharpen high-quality-locus classification.
when_to_use_negative:
- Input is short-read (Illumina-style) data — miniSNV is optimized for ONT long reads
  where pileup-based short-read methods miss repetitive-region SNVs
- No population VCF is available for the organism/sample, so no known-common-variant
  background can be indexed
- Only indels or structural variants are of interest — the index and pipeline target
  single nucleotide variant calling
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: miniSNV
  role: Constructs the known-common-variant index from the population VCF (L=300 bp
    overlapping windows, >5% allele frequency) and calls SNVs with the index enabled
    or disabled for the ablation
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Dorado
  role: Basecalls the kit-v14 (R10.4.1) ONT reads before alignment and SNV calling
    with the index-backed miniSNV run
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:6e8e67c02e611b3f741ec1fef41c8e22eb42e7373aecac95a57ed338b6b6a512
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Dorado
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# known-variant-background-index-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a miniSNV known-common-variant index from a population VCF (1000 Genomes by default) so that candidate SNV loci on ONT long reads can be classified as high-quality using population prior information, improving precision, recall, and F1 over running miniSNV without a variation background.

## When to use

Use when calling SNVs from ONT (R10.4/R10.4.1) long-read BAMs with miniSNV and you want known common population variants (e.g. a 1000 Genomes VCF) as a variation background to accelerate and sharpen high-quality-locus classification.

## When NOT to use

- Input is short-read (Illumina-style) data — miniSNV is optimized for ONT long reads where pileup-based short-read methods miss repetitive-region SNVs
- No population VCF is available for the organism/sample, so no known-common-variant background can be indexed
- Only indels or structural variants are of interest — the index and pipeline target single nucleotide variant calling

## Inputs

- Population VCF of known common variants (1000 Genomes project by default)
- Sorted, samtools-indexed ONT read BAM (e.g. HG002 k12 Guppy-basecalled or k14 Dorado-basecalled data aligned to GRCh38 with Minimap2)
- samtools-indexed FASTA reference (GRCh38)
- Optional homozygous and DUP BED annotation files (--homo_bed, --dup_bed)

## Outputs

- miniSNV index folder containing five files for use with --useindex/-i
- SNV callset VCF per input BAM produced with the known-variant background enabled
- Comparative metrics (precision, recall, F1) versus the no-background run, evaluated against GIAB HG002 truth with v3.3 stratifications

## How to apply

Construct the index from the 1000 Genomes population VCF using miniSNV's windowing scheme: fixed window length L=300 bp with half-length overlap between adjacent windows; multiallelic loci are split to biallelic records and alternative alleles with frequency >5% are recorded as wid_pos_ref_alt UIDs. The resulting index is a folder containing five files, supplied to miniSNV via --useindex and -i. With the index enabled, known loci are classified high-quality when R_alt > 0.3, whereas unknown loci require R_alt > 0.7 and no other candidate within 10 bp; all loci still pass the default filters of MAPQ >= 20 and R_alt > 0.2 for candidate detection. To quantify the background's benefit (as in the HG002 k12/k14 ablation), also run miniSNV with the index disabled so every candidate follows the unknown-locus rule, then compare precision, recall, and F1 against the GIAB HG002 truth set with GIAB v3.3 stratifications.

## Related tools

- **miniSNV** (Constructs the known-common-variant index from the population VCF (L=300 bp overlapping windows, >5% allele frequency) and calls SNVs with the index enabled or disabled for the ablation) — https://github.com/CuiMiao-HIT/miniSNV
- **Dorado** (Basecalls the kit-v14 (R10.4.1) ONT reads before alignment and SNV calling with the index-backed miniSNV run) — https://github.com/nanoporetech/dorado

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG002_k14.sorted.bam --workDir workdir --useindex -i miniSNV_index --human
```

## Evaluation signals

- Index folder contains exactly five files as required by --useindex/-i
- With the index enabled, known loci with R_alt > 0.3 are classified high-quality while unknown loci need R_alt > 0.7 and no other candidate within 10 bp
- HG002 k12 ablation shows precision gain of ~0.12% and F1 gain of ~0.04% with background; k14 shows precision +0.08%, recall +0.16%, F1 +0.12%
- miniSNV plus known-variant background exceeds 99% F1 on k12-data, with a 0.02% precision gain over Clair3 and 0.05%/1.83%/0.96% precision/recall/F1 gains over Longshot
- Index UIDs follow the wid_pos_ref_alt scheme with multiallelic loci split to biallelic and alt alleles >5% frequency retained

## Limitations

- The relaxed R_alt > 0.3 threshold applies only to loci present in the prebuilt known-variant index; unindexed loci are subject to the stricter 0.7 threshold and the 10 bp proximity condition
- Index content is population-specific — the default 1000 Genomes background may not represent rare or population-specific variation
- Candidate loci in low-complexity regions are classified low-quality regardless of the known-variant background

## Evidence
