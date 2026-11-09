---
name: alternative-allele-ratio-thresholding
description: Use when generating candidate SNV loci from ONT read pileups and you
  must decide which position/allele combinations pass to genotyping based on alternative
  allele support ratio.
when_to_use_negative:
- Input is already a called VCF of genotyped variants — this skill operates on raw
  read pileups during candidate discovery, not post-genotyping.
- Alignments lack base quality tags or mapping quality, since the pileup counting
  depends on baseQ >= 13 and MAPQ >= 20 filters.
- Calling indels/SVs — this thresholding applies to single nucleotide variant alleles
  only.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0199
tools:
- name: miniSNV
  role: Performs the read-pileup stage and applies the R_alt threshold (default 0.2)
    with read_Ratio 0.98 and baseQ 13 filters to emit candidate loci
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Oxford Nanopore Technologies (ONT)
  role: Source of the long-read alignments whose per-position allele counts feed the
    ratio test
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:61c78c5f360404f1f85f6157eed25ced534ecafc585bcae2e9b79ed21d9ae827
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
  - Oxford Nanopore Technologies (ONT)
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# alternative-allele-ratio-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A candidate-SNV filtering step from miniSNV that marks a genome position with a specific alternative allele as a candidate locus only when the supporting read ratio (R_alt) exceeds a user-defined threshold (default 0.2), evaluated per allele from ONT read pileups.

## When to use

Use when generating candidate SNV loci from ONT read pileups and you must decide which position/allele combinations pass to genotyping based on alternative allele support ratio.

## When NOT to use

- Input is already a called VCF of genotyped variants — this skill operates on raw read pileups during candidate discovery, not post-genotyping.
- Alignments lack base quality tags or mapping quality, since the pileup counting depends on baseQ >= 13 and MAPQ >= 20 filters.
- Calling indels/SVs — this thresholding applies to single nucleotide variant alleles only.

## Inputs

- Sorted, samtools-indexed ONT BAM alignment (e.g. HG002 k12-data)
- Samtools-indexed FASTA reference (e.g. GRCh38)
- miniSNV index built from the 1000 Genomes population VCF (optional, with --useindex)

## Outputs

- candidate_sites.tsv: tab-separated table with chrom, start, ref, alt, and supporting-read count columns, restricted to loci with R_alt above the 0.2 threshold

## How to apply

Run the miniSNV read-pileup stage over chunked reference windows (default chunkWidth 10,000,000 bp), tallying cumulative counts of alleles (A, C, G, T) at each position from MAPQ-filtered alignments (MAPQ < 20, supplementary, secondary, and unmapped records discarded). At each position, evaluate each alternative allele separately and compare its supporting-read ratio (R_alt) against the user-defined threshold (default 0.2); only exceedances become candidate loci. Apply the accompanying alignment-consistency threshold (read_Ratio 0.98) and minimum base quality (baseQ 13) during pileup counting. Downstream, the same R_alt values stratify candidates: known common variants (1000 Genomes background) with R_alt > 0.3, or unknown loci with R_alt > 0.7 and no other candidate within 10 bp, are classified high-quality; others, including low-complexity-region loci, go to the low-quality consensus/phasing pipeline. Write retained loci to candidate_sites.tsv with chrom, start, ref, alt, and supporting-read count columns.

## Related tools

- **miniSNV** (Performs the read-pileup stage and applies the R_alt threshold (default 0.2) with read_Ratio 0.98 and baseQ 13 filters to emit candidate loci) — https://github.com/CuiMiao-HIT/miniSNV
- **Oxford Nanopore Technologies (ONT)** (Source of the long-read alignments whose per-position allele counts feed the ratio test)

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG002.bam --workDir work/ --useindex -i miniSNV_index --read_Ratio 0.98 --baseQ 13
```

## Evaluation signals

- Every retained site in candidate_sites.tsv has an alternative allele ratio above the 0.2 threshold consistent with the pileup criteria.
- candidate_sites.tsv exists and is a tab-separated table containing the required chrom, start, ref, alt, and supporting-read count fields.
- No candidate arises from alignments that were filtered (MAPQ < 20, supplementary, secondary, unmapped) or bases below quality 13.
- High/low-quality classification agrees with the R_alt cutoffs: known variants > 0.3 high-quality; unknown loci need > 0.7 and no other candidate within 10 bp.
- Per-chunk outputs merge cleanly into a single candidate_sites.tsv when chunkWidth (default 10,000,000) windows are processed in parallel.

## Limitations

- The R_alt operating thresholds (0.3 and 0.7 for high/low-quality stratification) were selected from in-house experiments and may not transfer to other error profiles without re-tuning.
- Any candidate locus falling in a low-complexity region (GIAB v3.3 stratifications) is forced into the low-quality pipeline regardless of its R_alt.
- The 0.2 candidate threshold is a recall-oriented pre-filter; it does not by itself control false positives, which are handled later by haplotype-aware genotyping and filtering.

## Evidence

- [methods] compare the ratio and a user-defined threshold (default 0.2) at a genome position pos: [evidence span withheld — anchor status: not_found]
- [methods] R_alt threshold selection: [evidence span withheld — anchor status: not_found]
- [methods] high-quality classification: [evidence span withheld — anchor status: not_found]
- [methods] pileup candidate recognition: "miniSNV applies read pileup to recognize the candidate loci with divergences between reads and reference for variant calling."
- [readme] readme parameter: "--baseQ(INT)			Minimum base quality for candidate sites.(default :13)"
