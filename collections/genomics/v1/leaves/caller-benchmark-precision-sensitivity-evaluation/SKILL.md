---
name: caller-benchmark-precision-sensitivity-evaluation
description: Use when you have SV calls (VCF) from one or more long-read callers (e.g.
  NanoSV, Lumpy, Sniffles) on nanopore alignments and public truth SV call sets (1KG,
  PacBio) for the same sample/region, and you need caller-level sensitivity/precision
  numbers.
when_to_use_negative:
- The data are short-read Illumina alignments rather than long nanopore/PacBio reads
  — the NanoSV algorithm depends on split- and gapped-aligned long reads and the truth
  sets and numbers here are long-read-specific
- No public truth SV call set exists for the sample or region, so sensitivity/precision
  cannot be computed (de novo discovery would be the right task instead)
- You need a full genome assembly or SNV/indel benchmarking — this skill evaluates
  only SV breakpoint-junction calls against SV truth sets
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoSV
  role: Long-read SV caller being benchmarked; produces the chromosome 1 VCF call
    set and the filtering protocol applied to all callers
  repo: https://github.com/mroosmalen/nanosv
- name: LAST
  role: Long-read mapper producing the split/gapped alignments most accurate for NanoSV
    SV calling
- name: Lumpy
  role: Comparison caller run on the same aligned BAM with caller-appropriate parameters
- name: Sniffles
  role: Comparison caller run on the same aligned BAM with caller-appropriate parameters
- name: bedtools
  role: Intersects filtered caller VCFs with 1KG and PacBio truth sets, matching events
    by SV type and breakpoint proximity
- name: sambamba
  role: Converts LAST MAF→SAM→BAM output and sorts/indexes the BAM for SV calling
  repo: https://github.com/biod/sambamba
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:e68c47e47436d779af5166d4d984183db9a2579a653e3eb4931494bc3d644c90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - LAST
  - Lumpy
  - NanoSV
  - Sniffles
  - bedtools
  - sambamba
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# caller-benchmark-precision-sensitivity-evaluation

## Summary

Benchmark structural-variant (SV) callers on long-read nanopore data by intersecting each caller's filtered VCF with public truth sets (1KG Illumina, PacBio) and computing per-caller sensitivity and precision. In the source study this reproduced NanoSV's 78%/88% sensitivity and 54% precision on NA12878 chromosome 1 versus Lumpy (15%/32%, 2% precision) and Sniffles (72%/77%, 50% precision).

## When to use

Use when you have SV calls (VCF) from one or more long-read callers (e.g. NanoSV, Lumpy, Sniffles) on nanopore alignments and public truth SV call sets (1KG, PacBio) for the same sample/region, and you need caller-level sensitivity/precision numbers.

## When NOT to use

- The data are short-read Illumina alignments rather than long nanopore/PacBio reads — the NanoSV algorithm depends on split- and gapped-aligned long reads and the truth sets and numbers here are long-read-specific
- No public truth SV call set exists for the sample or region, so sensitivity/precision cannot be computed (de novo discovery would be the right task instead)
- You need a full genome assembly or SNV/indel benchmarking — this skill evaluates only SV breakpoint-junction calls against SV truth sets

## Inputs

- Sorted BAM of aligned long nanopore reads (e.g. NA12878 MinION R9.4 chromosome 1 reads aligned with LAST)
- SV call sets in VCF format from each caller being compared (NanoSV, Lumpy, Sniffles)
- Public truth SV call sets: 1000 Genomes (1KG) Illumina-based SV calls and PacBio (PB) SV calls for the same sample/region
- Caller-appropriate parameter sets and the NanoSV post-calling filtering criteria

## Outputs

- Per-caller sensitivity/precision summary table (e.g. sv_benchmark_na12878_chr1.tsv) with NanoSV, Lumpy, and Sniffles values and a combined-call-set row
- Intersection results: truth-set SVs recovered per caller and caller calls matching truth, stratified by truth set (1KG vs PB)

## How to apply

Align the NA12878 MinION R9.4 chromosome 1 reads to the human reference with LAST (the NanoSV authors report LAST alignments give the most accurate results for SV calling, though it is compute-heavy), converting MAF to SAM to BAM via maf-convert and sambamba. Run NanoSV, Lumpy, and Sniffles on the same sorted BAM with caller-appropriate parameters to produce chromosome 1 VCF call sets. Apply the same post-calling filtering criteria described in the NanoSV protocol to all three call sets so the comparison is fair. Intersect each filtered call set with the 1KG Illumina and PacBio (PB) truth sets using bedtools, matching events by SV type and breakpoint proximity rather than exact coordinate identity. Compute sensitivity as the fraction of truth SVs recovered (e.g. NanoSV 131/168 = 78% vs 1KG, 292/332 = 88% vs PB) and precision as the fraction of caller calls matching truth (NanoSV 54%, Sniffles 50%, Lumpy 2%), and write a per-caller summary table.

## Related tools

- **NanoSV** (Long-read SV caller being benchmarked; produces the chromosome 1 VCF call set and the filtering protocol applied to all callers) — https://github.com/mroosmalen/nanosv
- **LAST** (Long-read mapper producing the split/gapped alignments most accurate for NanoSV SV calling)
- **Lumpy** (Comparison caller run on the same aligned BAM with caller-appropriate parameters)
- **Sniffles** (Comparison caller run on the same aligned BAM with caller-appropriate parameters)
- **bedtools** (Intersects filtered caller VCFs with 1KG and PacBio truth sets, matching events by SV type and breakpoint proximity)
- **sambamba** (Converts LAST MAF→SAM→BAM output and sorts/indexes the BAM for SV calling) — https://github.com/biod/sambamba

## Examples

```
bedtools intersect -u -a nanosv.filtered.chr1.vcf -b truth_1KG_pb.chr1.vcf > nanosv_truth_overlap.chr1.vcf
```

## Evaluation signals

- NanoSV on NA12878 chromosome 1 should recover 131 of 168 1KG SVs (78% sensitivity) and 292 of 332 PacBio SVs (88% sensitivity)
- Reported precision checks: NanoSV 54%, Sniffles 50%, Lumpy 2% on chromosome 1 nanopore data
- Comparison calls: Lumpy 15%/32% and Sniffles 72%/77% sensitivity against 1KG/PB truth sets respectively
- Output table sv_benchmark_na12878_chr1.tsv must contain per-caller sensitivity and precision rows plus a combined call-set row
- Intersections must match events by SV type and breakpoint proximity — verify that same-type, nearby-breakpoint events are counted as matches rather than requiring identical coordinates

## Limitations

- Sensitivity is limited by medium coverage (~11–16x in the source study) and intrinsic nanopore sequencing biases
- Extremely stringent post-calling filtering (used in the article's own patient-genome pipeline) improves precision (>95%) but lowers estimated sensitivity to ~75%, so filtering choice directly shifts benchmark numbers
- Lumpy performed very poorly on this nanopore data (2% precision against truth), showing short-read-oriented callers may not transfer to long-read SV benchmarking
- Matching by SV type and breakpoint proximity is approximate; borderline breakpoint-distance choices can change counted TP/FP events

## Evidence

- [readme] We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources.: "We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources."
- [readme] NanoSV algorithm definition: "The core algorithm of NanoSV identifies split- and gapped-aligned reads and clusters the reads according to the orientations and genomic positions of the read segments to define breakpoint-junctions"
- [discussion] Filtering/sensitivity tradeoff: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] Precision benchmark context: "SV calls matching any state of the art precision benchmark (>95% precision)"
- [intro] Long reads recover SVs missed by short reads: "we identify a significant proportion of SVs that are not detected in short-read Illumina sequencing data of the same patient genomes."
