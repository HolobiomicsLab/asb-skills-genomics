---
name: variant-callset-vcf-processing
description: 'Use when a Novel-X (or similar linked-read insertion caller) VCF has
  been produced and you must characterize callset biology: how many insertions sit
  in reference repeats, in non-uniquely mappable regions, or overlap annotated genes.'
when_to_use_negative:
- You need to CALL insertions rather than characterize an existing callset — run the
  Novel-X pipeline itself (novel-x.py) instead; this skill starts from a VCF.
- The callset contains variants shorter than 300 bp — Novel-X explicitly filters these
  out, and the annotation workflow was validated only on insertions >300 bp.
- You need per-base haplotype-resolved validation rather than positional annotation
  — use a T2T/CHM13 assembly comparison as in the paper's validation strategy instead
  of track intersections.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Novel-X
  role: Source of the insertion VCF that this characterization workflow annotates;
    calls NRS insertions >300 bp from 10x linked-reads
  repo: github.com/1dayac/Novel-X
- name: RepeatMasker
  role: Screens 300 bp sequences flanking each insertion breakpoint for repetitive
    reference sequence
- name: Umap (UCSC Multitrack)
  role: k100 and k50 mappability tracks used to classify breakpoint positions as uniquely
    vs non-uniquely mappable
- name: GENCODE v.24
  role: Annotation intersected with breakpoint positions to count gene-overlapping
    and exonic protein-coding insertions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:496f37ccfff4a58b464fff0f7921e797d7c49aad7c096a69cdb88e14a2c1e7dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GENCODE v.24
  - LongRanger
  - Novel-X
  - RepeatMasker
  - Umap (UCSC Multitrack)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# variant-callset-vcf-processing

## Summary

Characterize a structural-variant (NRS insertion) VCF callset — here the 226 Novel-X insertions on CHM1 10x data — by annotating breakpoints against RepeatMasker repeats, UCSC Umap mappability tracks (k100/k50), and GENCODE v24 gene models, yielding per-category counts that can be checked against published values.

## When to use

Use when a Novel-X (or similar linked-read insertion caller) VCF has been produced and you must characterize callset biology: how many insertions sit in reference repeats, in non-uniquely mappable regions, or overlap annotated genes.

## When NOT to use

- You need to CALL insertions rather than characterize an existing callset — run the Novel-X pipeline itself (novel-x.py) instead; this skill starts from a VCF.
- The callset contains variants shorter than 300 bp — Novel-X explicitly filters these out, and the annotation workflow was validated only on insertions >300 bp.
- You need per-base haplotype-resolved validation rather than positional annotation — use a T2T/CHM13 assembly comparison as in the paper's validation strategy instead of track intersections.

## Inputs

- Novel-X output VCF of non-reference sequence (NRS) insertions, e.g. the CHM1 10x Genomics callset with 226 insertion records
- GRCh38 reference genome FASTA for extracting 300 bp breakpoint flanks
- GENCODE annotation v.24 (GTF/GFF)
- UCSC Umap Multitrack mappability tracks for k100 and k50
- RepeatMasker (with the built-in Dfam/Repbase repeat library) for flanking-sequence screening

## Outputs

- Per-insertion repeat annotations of 300 bp upstream/downstream breakpoint flanks (RepeatMasker hits)
- Mappability classifications of breakpoint positions against Umap k100 and k50 tracks
- Gene-overlap counts: insertions inside GENCODE v.24 genes and subset in exons of known protein-coding genes
- Summary counts (40 RepeatMasker, 24 k100, 93 k50, 111 gene-overlapping insertions) suitable for comparison with published figures

## How to apply

Start from the Novel-X VCF, which by design contains only insertions longer than 300 bp whose flanking anchor lengths sum to >300 bp. For each reported insertion breakpoint, extract 300 bp of sequence upstream and downstream of the breakpoint in the GRCh38 reference and screen these flanks with RepeatMasker to count insertions located in repetitive reference regions (40/226 on CHM1). Cross-check placement against UCSC Umap Multitrack mappability for k=100 and k=50, counting insertions in non-uniquely mappable regions (24 and 93 respectively). Intersect breakpoint positions with GENCODE annotation v.24 to count insertions inside known gene sequences (111/226) and, separately, those in exonic regions of known protein-coding genes (2/226). These annotation counts are the primary correctness criteria: they should reproduce the paper's values when applied to the released CHM1 VCF.

## Related tools

- **Novel-X** (Source of the insertion VCF that this characterization workflow annotates; calls NRS insertions >300 bp from 10x linked-reads) — github.com/1dayac/Novel-X
- **RepeatMasker** (Screens 300 bp sequences flanking each insertion breakpoint for repetitive reference sequence)
- **Umap (UCSC Multitrack)** (k100 and k50 mappability tracks used to classify breakpoint positions as uniquely vs non-uniquely mappable)
- **GENCODE v.24** (Annotation intersected with breakpoint positions to count gene-overlapping and exonic protein-coding insertions)

## Evaluation signals

- VCF record count matches expectation (226 insertions for the CHM1 10x benchmark callset).
- RepeatMasker flanking-screen count of insertions in repetitive reference regions equals 40.
- Umap cross-check yields 24 insertions in non-uniquely mappable k100 regions and 93 in k50 regions.
- GENCODE v.24 intersection gives 111 of 226 insertions inside known genes, with only 2 in exonic regions of known protein-coding genes.
- Every reported insertion is >300 bp with the sum of flanking anchor lengths exceeding 300 bp, per the Novel-X output filter.

## Limitations

- Novel-X needs a substantial number of distinct barcodes per insertion; small insertions usually have too few associated barcodes, so small events are systematically under-represented in the VCF being characterized.
- The published count set (40/24/93/111) is specific to the CHM1 10x Genomics dataset at 40x (0.8-downsampled) with GRCh38; other samples/platforms (TELL-Seq, stLFR) will not reproduce these exact values.
- Annotation categories are not mutually exclusive — the same insertion can be both repeat-overlapping and gene-overlapping — so counts cannot be summed to the callset size.

## Evidence

- [methods] Only >300 bp insertions with sufficient anchors enter the VCF: [evidence span withheld — anchor status: not_found]
- [readme] Supplementary VCFs for all benchmark samples including CHM1 are released: [evidence span withheld — anchor status: external_doc]
- [abstract] Callset-level totals that downstream annotation counts should be consistent with: "Novel-X discovered 16 691 NRS insertions of size > 300 bp (total length 18.2 Mb)."
- [methods] CHM1/CHM13 haploid datasets are the benchmark context for this callset: "they do not work with our CHM1 or CHM13 haploid datasets"
