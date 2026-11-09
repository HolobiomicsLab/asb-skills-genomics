---
name: snp-quality-and-boundary-proximity-filtering
description: Use when you have called SNPs (e.g. BCFTools on reads mapped to predicted
  exons) and need to derive genotypable regions before comparing against a validated
  SNP reference set.
when_to_use_negative:
- The input is raw reads or an unfiltered splice graph — quality/proximity filtering
  applies only downstream of read mapping (Bowtie2) and variant calling (BCFTools).
- The goal is exon/splice graph prediction or read filtering (biobloomcategorizer,
  cascading Bloom filter) — those are upstream EXFI steps, not SNP post-filtering.
- There are no predicted exon boundaries available, so the 35 bp proximity-to-boundary
  criterion cannot be applied meaningfully.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BCFTools
  role: Performs SNP calling on reads mapped to predicted exons and applies the quality-value
    filter (< 20 discarded)
- name: Bowtie2
  role: Maps raw pooled genomic reads to the predicted exon sequences before variant
    calling
- name: EXFI
  role: Provides the predicted exon sequences and boundaries that define the 35 bp
    proximity exclusion zones
  repo: https://github.com/jlanga/exfi
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:b341eb6ddf68c16d8ccc2f72a4d616d03282dfd939da7ad7090c18a14251b630
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BCFTools
  - Bowtie2
  - EXFI
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# snp-quality-and-boundary-proximity-filtering

## Summary

Post-SNP-calling filtering strategy from the EXFI tench retrospective analysis: retain only variants with quality ≥ 20 and exclude variants within 35 bp of another variant or of a predicted exon boundary, to derive genotypable exonic SNP regions suitable for downstream genotyping.

## When to use

Use when you have called SNPs (e.g. BCFTools on reads mapped to predicted exons) and need to derive genotypable regions before comparing against a validated SNP reference set.

## When NOT to use

- The input is raw reads or an unfiltered splice graph — quality/proximity filtering applies only downstream of read mapping (Bowtie2) and variant calling (BCFTools).
- The goal is exon/splice graph prediction or read filtering (biobloomcategorizer, cascading Bloom filter) — those are upstream EXFI steps, not SNP post-filtering.
- There are no predicted exon boundaries available, so the 35 bp proximity-to-boundary criterion cannot be applied meaningfully.

## Inputs

- VCF/BCF variant calls from BCFTools on reads mapped to predicted exons
- Predicted exon boundary coordinates (e.g. from the EXFI splice graph in GFA1 format / gfa1_to_fasta FASTA output)
- Reference set of previously validated transcriptomic SNP positions (for retrospective recall/precision evaluation)

## Outputs

- Filtered set of SNPs in genotypable regions (QUAL ≥ 20, ≥ 35 bp from other variants and exon boundaries)
- Counts of recovered vs. missed SNPs relative to a validated SNP reference set (e.g. 92/96)

## How to apply

After mapping pooled genomic reads to predicted exon sequences (e.g. with Bowtie2) and calling variants with BCFTools, first discard all variants with a quality value below 20. Then exclude any variant located within 35 bp of another variant or of a predicted exon boundary; the surviving positions define the genotypable regions. This two-step filter mimics practical genotyping constraints, where SNPs too close to indels, other SNPs, or exon edges cannot be reliably assayed. Rationale: in the tench retrospective evaluation (Kumar et al., 2019), applying these filters recovered 92 of the 96 validated transcriptomic SNPs (100% precision, ~99% recall), so the same thresholds (QUAL ≥ 20, 35 bp proximity window) are the benchmark parameters for reuse.

## Related tools

- **BCFTools** (Performs SNP calling on reads mapped to predicted exons and applies the quality-value filter (< 20 discarded))
- **Bowtie2** (Maps raw pooled genomic reads to the predicted exon sequences before variant calling)
- **EXFI** (Provides the predicted exon sequences and boundaries that define the 35 bp proximity exclusion zones) — https://github.com/jlanga/exfi

## Evaluation signals

- No retained variant has a quality value below 20.
- No retained variant lies within 35 bp of another variant or of a predicted exon boundary.
- Against the Kumar et al. (2019) tench reference, ~92/96 SNPs (≈ 99% recall) with 100% precision should be recovered on this dataset.
- Retained positions fall inside predicted exons and away from IEBs, i.e. within the derived genotypable regions.

## Limitations

- The 35 bp window removes true variants near variant clusters or exon boundaries, so recall is not 100% (4 of 96 tench SNPs were not genotyped).
- Filtering performance depends on upstream exon prediction quality; mappability covers only interior exon sections and can be optimistic relative to full exon precision.
- Results degrade when the input transcriptome is de novo-assembled or from a single tissue, as fragmented/missing transcripts affect boundary prediction.

## Evidence

- [methods] Quality filter threshold: "variants with a quality value below 20 were filtered out"
- [methods] Proximity filter definition: "and then those that were within 35 bp to another variant or a predicted exon boundary"
- [results] Genotypable variant yield after filtering: "after quality and distance filtering, 228,931 SNPs and 26,169 indels were predicted suitable for genotyping"
- [discussion] Read filtering motivates SNP-level filtering success: "These positive results for EXFI are due to the read filtering step and the exon prediction rules used."
