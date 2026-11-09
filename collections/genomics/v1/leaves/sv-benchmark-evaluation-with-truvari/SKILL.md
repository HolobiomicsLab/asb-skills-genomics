---
name: sv-benchmark-evaluation-with-truvari
description: Use when query VCFs from SV callers must be assessed against GIAB benchmark
  SV sets (T2T-HG002-Q100 stvar or CMRG v1.00) to produce per-caller TP/FP/FN, precision,
  recall, and F1 metrics.
when_to_use_negative:
- Benchmarking small variants or SNVs — truvari bench targets structural variants
  and will not score SNV/small-indel accuracy
- Comparing two caller VCFs directly without a GIAB truth set and confident-region
  BED — bench requires a benchmark VCF and --includebed to restrict assessment
- Assessing calls that include non-PASS records when using --passonly, since these
  are excluded from scoring by design
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: truvari
  role: SV benchmark comparison engine; bench for CMRG, bench+refine+ga4gh consolidation
    for the T2T benchmark (v4.2.2 with MAFFT refinement)
  repo: library://ctsa/truvari/truvari:4.2.2
- name: MAFFT
  role: Alignment engine used by truvari refine to re-score GT matched pairs before
    recounting
- name: bcftools
  role: Filters the T2T benchmark VCF to records with INFO/SVTYPE != "." prior to
    benchmarking
- name: sawfish
  role: Produces the query SV VCF (discover + joint-call) being benchmarked against
    Sniffles2 and pbsv
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbsv
  role: Competing SV caller whose VCF is benchmarked for comparison
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:cc6d61a7d1f248a7aed79497ec6cc19c3fda34db17ee2e37c98a11cf3a06f5a9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAFFT
  - PAV
  - Sniffles2
  - bcftools
  - hifiasm
  - pbmm2
  - pbsv
  - sawfish
  - singularity
  - truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# sv-benchmark-evaluation-with-truvari

## Summary

Benchmark long-read SV caller output (sawfish, Sniffles2, pbsv) against GIAB truth sets using truvari 4.2.2 bench (and bench+refine with MAFFT for the T2T set), computing precision, recall, and F1 for caller comparison.

## When to use

Use when query VCFs from SV callers must be assessed against GIAB benchmark SV sets (T2T-HG002-Q100 stvar or CMRG v1.00) to produce per-caller TP/FP/FN, precision, recall, and F1 metrics.

## When NOT to use

- Benchmarking small variants or SNVs — truvari bench targets structural variants and will not score SNV/small-indel accuracy
- Comparing two caller VCFs directly without a GIAB truth set and confident-region BED — bench requires a benchmark VCF and --includebed to restrict assessment
- Assessing calls that include non-PASS records when using --passonly, since these are excluded from scoring by design

## Inputs

- Query SV VCFs from sawfish joint-call (genotyped.sv.vcf.gz), Sniffles2, and pbsv
- GIAB T2T SV benchmark GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz with benchmark BED
- GIAB CMRG SV benchmark HG002_GRCh38_CMRG_SV_v1.00.vcf.gz, .tbi, and .bed
- Reference genome human_GRCh38_no_alt_analysis_set.fasta

## Outputs

- truvari bench summary. per caller (TP, FP, FN, precision, recall, F1)
- Consolidated GA4GH-interoperable VCFs from truvari ga4gh --with-refine for the T2T assessment
- Compiled per-caller comparison tables (e.g. cmrg_sv_f1_comparison.csv)

## How to apply

Prepare the benchmark first: filter the T2T VCF to structural variants only with 'bcftools view -i INFO/SVTYPE!="."' and restrict the confident-regions BED to autosomes with 'grep -v chr[XY]'. For the CMRG benchmark (HG002_GRCh38_CMRG_SV_v1.00, 217 SVs), run truvari bench alone per caller VCF with --reference, --includebed, --passonly, --pick ac, and --dup-to-ins, using the library://ctsa/truvari/truvari:4.2.2 singularity image. For the GIAB T2T benchmark, use both bench and refine (refine with --recount --use-region-coords --use-original-vcfs --align mafft for MAFFT-based refinement), then consolidate with 'truvari ga4gh --with-refine' and summarize via process_truvari_ga4gh_vcfs.py. Parse each summary for TP, FP, FN, precision, recall, and F1 (e.g. compile into a per-caller CSV) and compare against the published Supplementary Tables S3/S4.

## Related tools

- **truvari** (SV benchmark comparison engine; bench for CMRG, bench+refine+ga4gh consolidation for the T2T benchmark (v4.2.2 with MAFFT refinement)) — library://ctsa/truvari/truvari:4.2.2
- **MAFFT** (Alignment engine used by truvari refine to re-score GT matched pairs before recounting)
- **bcftools** (Filters the T2T benchmark VCF to records with INFO/SVTYPE != "." prior to benchmarking)
- **sawfish** (Produces the query SV VCF (discover + joint-call) being benchmarked against Sniffles2 and pbsv) — https://github.com/PacificBiosciences/sawfish
- **pbsv** (Competing SV caller whose VCF is benchmarked for comparison)

## Examples

```
singularity exec library://ctsa/truvari/truvari:4.2.2 truvari bench --reference human_GRCh38_no_alt_analysis_set.fasta --includebed HG002_GRCh38_CMRG_SV_v1.00.bed --passonly --pick ac --dup-to-ins -b HG002_GRCh38_CMRG_SV_v1.00.vcf.gz -c genotyped.sv.vcf.gz -O truvari_cmrg_result_sawfish
```

## Evaluation signals

- CMRG F1 scores match Supplementary Table S4: sawfish 0.993 (two FN, one FP vs the 217-SV benchmark), Sniffles2 0.964, pbsv 0.971
- Subsampled-coverage F1 table reproduces the S3 pattern: sawfish F1 at 10x exceeds Sniffles2 by >2.1% and pbsv by >7.9%, and other callers need ~30x coverage to match sawfish at 15x
- The T2T benchmark VCF retains only records with non-missing SVTYPE after bcftools filtering, and the confident-regions BED contains no chrX/chrY entries after grep -v chr[XY]
- truvari ga4gh consolidation with --with-refine completes without error and refined GT counts differ from unrefined bench counts, confirming MAFFT refinement was applied
- Per-caller summaries parse to consistent TP+FN totals approximately equal to the benchmark size (217 SVs for CMRG)

## Limitations

- The T2T assessment must use bench AND refine with ga4gh consolidation; using bench alone (as is valid for CMRG) is not sufficient for the T2T benchmark comparison
- truvari bench with --passonly discards non-PASS query records, so caller-specific quality filtering affects comparability
- VACmap is still unsupported in the released tooling; error messages for problematic alignments are not yet available

## Evidence

- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] bcftools T2T SVTYPE filtering: "bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz"
- [methods] bench/refine consolidation requirement: "it is important to use the bench and refine command, with the bench/refine results consolidated as described further below."
- [methods] CMRG benchmark download location: "The CMRG SV benchmark was obtained from the following URL: https://ftp-trace.ncbi.nlm.nih.gov/ReferenceSamples/giab/release/AshkenazimTrio/HG002_NA24385_son/CMRG_v1.00"
- [methods] truvari singularity image used for benchmarking: "The singularity image file used in the App Note is available from `https://cloud.sylabs.io/library/ctsa/truvari/truvari`"
