---
name: t2t-sv-benchmark-preprocessing
description: Use when benchmarking long-read SV callers (e.g., sawfish, Sniffles2,
  pbsv) against the GIAB T2T-HG002-Q100 SV benchmark and the raw VCF/BED truth files
  must be filtered to SV records on autosomes first.
when_to_use_negative:
- The truth set is the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), which
  in this workflow uses truvari bench alone without the SVTYPE and autosomal filtering
  steps
- You need to assess SVs on sex chromosomes, since the confident regions are deliberately
  restricted to autosomes here
- Comparing callsets with a different evaluation engine (e.g., hap-eval) that does
  not require the truvari-specific base/includebed preprocessing
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: bcftools
  role: Filters the T2T benchmark VCF to records with a defined SVTYPE (`bcftools
    view -i 'INFO/SVTYPE!="."'`)
- name: truvari 4.2.2
  role: Performs bench/refine/ga4gh comparison of caller VCFs against the preprocessed
    T2T truth set, with MAFFT-based refinement
  repo: library://ctsa/truvari/truvari:4.2.2
- name: MAFFT
  role: Alignment engine used by truvari refine (--align mafft) when consolidating
    benchmark results
- name: sawfish
  role: Query SV caller whose 32x HG002 HiFi calls are assessed against the preprocessed
    T2T benchmark (expected F1 ~0.976)
  repo: https://github.com/PacificBiosciences/sawfish
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:0bcc469fd1cde39f4b8eb6cd41e48960d56cc62b5918a59afc9a33b6b4fa4ec8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAFFT
  - bcftools
  - pbmm2
  - sawfish
  - truvari 4.2.2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# t2t-sv-benchmark-preprocessing

## Summary

Preprocesses the GIAB draft T2T SV benchmark (GRCh38_HG2-T2TQ100-V1.1_stvar) so it can be used with truvari bench: retains only structural-variant records via bcftools and restricts the confident-region BED to autosomes, yielding a clean truth set for HiFi SV caller benchmarking.

## When to use

Use when benchmarking long-read SV callers (e.g., sawfish, Sniffles2, pbsv) against the GIAB T2T-HG002-Q100 SV benchmark and the raw VCF/BED truth files must be filtered to SV records on autosomes first.

## When NOT to use

- The truth set is the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), which in this workflow uses truvari bench alone without the SVTYPE and autosomal filtering steps
- You need to assess SVs on sex chromosomes, since the confident regions are deliberately restricted to autosomes here
- Comparing callsets with a different evaluation engine (e.g., hap-eval) that does not require the truvari-specific base/includebed preprocessing

## Inputs

- GIAB draft T2T SV benchmark VCF (GRCh38_HG2-T2TQ100-V1.1_stvar, V0.019-20241113)
- GIAB T2T confident-region BED file
- Caller query VCFs (sawfish, Sniffles2, pbsv) from HG002 Revio HiFi reads aligned to human_GRCh38_no_alt_analysis_set

## Outputs

- SV-only benchmark VCF (bgzipped, e.g. GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz) with all records having an SVTYPE
- Autosome-only confident-region BED (chr[XY] lines removed)
- truvari bench/refine/ga4gh comparison results for each caller against the preprocessed truth set

## How to apply

Download the GIAB draft T2T SV benchmark VCF (V0.019-20241113, GRCh38_HG2-T2TQ100-V1.1_stvar) and its confident-region BED. Apply the SV-only filter with `bcftools view -i 'INFO/SVTYPE!="."' -Oz` to drop placeholder/non-SV records and index the output. Restrict the confident regions to autosomes with `grep -v chr[XY]` so sex chromosomes are excluded from the assessed regions. Use the filtered VCF as the truvari bench --base and the autosomal BED as --includebed, running truvari 4.2.2 with MAFFT refinement (--passonly --pick ac --dup-to-ins) consolidated via bench/refine/ga4gh.

## Related tools

- **bcftools** (Filters the T2T benchmark VCF to records with a defined SVTYPE (`bcftools view -i 'INFO/SVTYPE!="."'`))
- **truvari 4.2.2** (Performs bench/refine/ga4gh comparison of caller VCFs against the preprocessed T2T truth set, with MAFFT-based refinement) — library://ctsa/truvari/truvari:4.2.2
- **MAFFT** (Alignment engine used by truvari refine (--align mafft) when consolidating benchmark results)
- **sawfish** (Query SV caller whose 32x HG002 HiFi calls are assessed against the preprocessed T2T benchmark (expected F1 ~0.976)) — https://github.com/PacificBiosciences/sawfish

## Examples

```
bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz; grep -v chr[XY] confident_regions.bed > confident_regions.autosomal.bed
```

## Evaluation signals

- Filtered benchmark VCF contains no records with SVTYPE '.'; every retained record has an assigned SVTYPE
- Autosomal confident-region BED contains no lines for chrX or chrY
- Reprocessing sawfish 0.12.9 calls at 32x HG002 HiFi coverage through bench/refine/ga4gh yields an F1 of approximately 0.976 (Supplementary Table S1)
- Reported F1 for sawfish exceeds Sniffles2 by ~1.2% and pbsv by ~1.4% using --passonly --pick ac --dup-to-ins settings
- truvari ga4gh consolidation of bench/refine results (with --recount --use-region-coords --use-original-vcfs --align mafft) completes without MAFFT temp-path errors

## Limitations

- The draft T2T benchmark (V0.019-20241113) is a draft GIAB release, so absolute accuracy values may shift with benchmark updates
- Sex chromosomes are excluded by design (grep -v chr[XY]), so accuracy on chrX/chrY SVs cannot be measured with this setup
- MAFFT during truvari refine can fail on tmp filesystems; the workflow specifies running with tmp on the regular filesystem

## Evidence

- [methods] bcftools filtering of T2T benchmark VCF to SVTYPE records: "bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz"
- [methods] Autosome restriction of confident regions: "The confident regions are restricted to autosomes as follows: grep -v chr[XY]"
- [methods] Truvari assessment configuration: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] T2T benchmark consolidation requirement: "it is important to use the bench and refine command, with the bench/refine results consolidated as described further below."
- [abstract] Dataset provenance for the T2T truth set: "the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly"
