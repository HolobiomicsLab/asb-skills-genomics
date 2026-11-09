---
name: genomic-coordinate-liftover
description: Use when comparing SV call sets or truth sets built on different reference
  genome assemblies (e.g. a VCF on hg19 vs calls on GRCh38) and breakpoints must be
  mapped to one build before intersecting.
when_to_use_negative:
- All call sets and truth sets already share the same reference build — unnecessary
  conversion risks breakpoint drift
- Truth-set entries fail liftOver mapping (e.g. in regions not represented in the
  chain file); these should be flagged, not silently assumed converted
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: liftOver
  role: Converts the Valle-Inclan COLO829 somatic SV truth-set VCF from its original
    build to GRCh38 coordinates before benchmarking against nanomonsv calls
- name: nanomonsv
  role: Produces the somatic SV call sets (nanomonsv.result.txt / .vcf) on GRCh38
    that are intersected with the lifted-over truth sets
  repo: github.com/friend1ws/nanomonsv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:b9bb1c23997a3c7218959e5ddffb71ca07b0136e77961996e45bf5e0e6e26fe6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PacBio
  - liftOver
  - nanomonsv
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# genomic-coordinate-liftover

## Summary

Converts SV breakpoint coordinates between reference genome builds (e.g. hg19/GRCh37 to GRCh38) with liftOver so that somatic SV truth sets and caller outputs can be intersected on a common coordinate system during benchmarking.

## When to use

Use when comparing SV call sets or truth sets built on different reference genome assemblies (e.g. a VCF on hg19 vs calls on GRCh38) and breakpoints must be mapped to one build before intersecting.

## When NOT to use

- All call sets and truth sets already share the same reference build — unnecessary conversion risks breakpoint drift
- Truth-set entries fail liftOver mapping (e.g. in regions not represented in the chain file); these should be flagged, not silently assumed converted

## Inputs

- Valle-Inclan truthset_somaticSVs_COLO829.vcf (Zenodo record 3988185, hg19 coordinates)
- Arora high-confidence somatic SV bedpe (NY Genome high-coverage NovaSeq resource)
- Appropriate UCSC liftOver chain file (e.g. hg19ToHg38.over.chain)
- nanomonsv somatic SV call sets (nanomonsv.result.txt / .vcf) aligned to GRCh38

## Outputs

- Lifted-over truth-set VCF in GRCh38 coordinates
- Filtered truth sets with insertions/deletions ≤100 bp removed
- benchmark_precision_recall.csv with per-benchmark precision and recall

## How to apply

Identify the reference build of each SV resource: in the COLO829 benchmark, the Valle-Inclan truthset_somaticSVs_COLO829.vcf from Zenodo record 3988185 requires conversion to GRCh38 coordinates, while the Arora high-confidence somatic SV bedpe from the NY Genome high-coverage NovaSeq resource is used as-is. Run UCSC liftOver on the truth-set file with the appropriate hg19-to-hg38 chain file, dropping records that fail the mapping. Before or after liftOver, remove insertions and deletions of ≤100 bp from both truth sets so only comparable SV classes remain. Only after coordinates are harmonized should nanomonsv calls (nanomonsv.result.txt / .vcf, from the ONT and PacBio tumor/control pairs aligned to GRCh38 with minimap2) be intersected with the truth sets by breakpoint overlap to compute precision and recall.

## Related tools

- **liftOver** (Converts the Valle-Inclan COLO829 somatic SV truth-set VCF from its original build to GRCh38 coordinates before benchmarking against nanomonsv calls)
- **nanomonsv** (Produces the somatic SV call sets (nanomonsv.result.txt / .vcf) on GRCh38 that are intersected with the lifted-over truth sets) — github.com/friend1ws/nanomonsv

## Examples

```
liftOver truthset_somaticSVs_COLO829.vcf hg19ToHg38.over.chain truthset_somaticSVs_COLO829_GRCh38.vcf liftOver_unmapped.bed
```

## Evaluation signals

- Reproduced precision values match 87.8% (43/49) against the Arora benchmark and 91.8% (45/49) against the Valle-Inclan benchmark
- Reproduced recall values match 57.3% (43/75) against the Arora benchmark (75 SVs) and 77.6% (46/58) against the Valle-Inclan benchmark (58 SVs)
- Novel nanomonsv SVs treated as false positives equal 6 for Arora and 4 for Valle-Inclan
- Both truth sets contain no remaining insertions or deletions of ≤100 bp after filtering

## Limitations

- liftOver may drop or mis-map SVs in regions poorly represented in the chain file, and coordinates in highly repetitive regions are unreliable (the article notes reference genomes are 'not reliable' at locations susceptible to long repeat expansion events)
- Benchmark results are specific to COLO829 ONT/PacBio data (ENA PRJEB27698, runs ERR2752451/ERR2752452 and ERR2808247/ERR2808248) and the two chosen truth sets; worst-case precision was reported, so results depend on how novel calls are counted

## Evidence

- [methods] converting the latter to GRCh38 coordinates with liftOver, and in both truth sets remove insertions and deletions of ≤100 bp: [evidence span withheld — anchor status: not_found]
- [results] worst-case precision of 87.8% (43/49) and 91.8% (45/49), and recall of 57.3% (43/75) and 77.6% (46/58): [evidence span withheld — anchor status: not_found]
- [abstract] nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution: "We present our novel software, nanomonsv, for detecting somatic structural variations (SVs) using tumor and matched control long-read sequencing data with a single-base resolution."
- [discussion] the reference genome is not reliable at the location susceptible to these: "the reference genome is not reliable at the location susceptible to these"
