---
name: sv-callset-f1-comparison-across-callers
description: Use when comparing germline SV calling accuracy of multiple long-read
  callers on HG002 HiFi WGS data against the GIAB draft T2T SV benchmark (GRCh38_HG2-T2TQ100-V1.1_stvar)
  or the GIAB CMRG SV benchmark, to obtain size-stratified F1-scores and caller rankings.
when_to_use_negative:
- Assessing short-read SV callers or somatic/low-VAF SV calls — this workflow targets
  germline SVs from mapped HiFi reads, as sawfish is a germline SV/CNV caller for
  HiFi data
- When a bench-only comparison is sufficient for the CMRG benchmark — the bench/refine/ga4gh
  consolidation with MAFFT is specifically required for the T2T SV assessment, not
  the CMRG assessment which 'uses the truvari bench command alone'
- Benchmarking against older/GRCh37 truth sets without breakpoint-refinement support,
  where MAFFT refinement of candidate.refine.bed regions cannot improve match accounting
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: truvari 4.2.2
  role: Benchmark caller VCFs against the filtered T2T truth set via bench/refine/ga4gh
    to compute F1-scores
  repo: library://ctsa/truvari/truvari:4.2.2
- name: sawfish 0.12.9
  role: Query SV caller (sawfish discover then sawfish joint-call) whose size-stratified
    F1 is compared against Sniffles2 and pbsv
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2 1.13.1
  role: Aligns HG002 HiFi reads to the GRCh38 no-alt reference with --preset CCS --sort
    to produce caller input BAMs
- name: MAFFT
  role: Alignment engine used by truvari refine (--align mafft) to refine bench candidates
    before ga4gh consolidation
- name: bcftools
  role: Filters the T2T benchmark VCF to SV records only via bcftools view -i 'INFO/SVTYPE!="."'
- name: pbsv
  role: Competing long-read SV caller included in the three-caller F1 comparison
  repo: https://github.com/PacificBiosciences/pbsv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:8a21293d8e3547b8a603f1c498be63e2979d952208164c984ee81f426028fcf3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAFFT
  - bcftools
  - pbmm2
  - pbmm2 1.13.1
  - pbsv
  - sawfish
  - sawfish 0.12.9
  - truvari 4.2.2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sv-callset-f1-comparison-across-callers

## Summary

Benchmark structural variant callsets (e.g. sawfish, Sniffles2, pbsv) against a GIAB SV truth benchmark using truvari bench/refine with MAFFT refinement, consolidated via truvari ga4gh, to compute and compare per-caller F1-scores. This replicates the head-to-head accuracy comparison from the sawfish App Note on HG002 HiFi data at 32-fold coverage.

## When to use

Use when comparing germline SV calling accuracy of multiple long-read callers on HG002 HiFi WGS data against the GIAB draft T2T SV benchmark (GRCh38_HG2-T2TQ100-V1.1_stvar) or the GIAB CMRG SV benchmark, to obtain size-stratified F1-scores and caller rankings.

## When NOT to use

- Assessing short-read SV callers or somatic/low-VAF SV calls — this workflow targets germline SVs from mapped HiFi reads, as sawfish is a germline SV/CNV caller for HiFi data
- When a bench-only comparison is sufficient for the CMRG benchmark — the bench/refine/ga4gh consolidation with MAFFT is specifically required for the T2T SV assessment, not the CMRG assessment which 'uses the truvari bench command alone'
- Benchmarking against older/GRCh37 truth sets without breakpoint-refinement support, where MAFFT refinement of candidate.refine.bed regions cannot improve match accounting

## Inputs

- HG002 Revio HiFi reads BAM (e.g. m84011_220902_175841_s1.hifi_reads.bam)
- human_GRCh38_no_alt_analysis_set.fasta reference
- GIAB draft T2T SV benchmark V0.019-20241113 (GRCh38_HG2-T2TQ100-V1.1_stvar) VCF and confident-region BED
- GIAB CMRG SV benchmark HG002_GRCh38_CMRG_SV_v1.00 (alternative assessment)
- Query VCFs from each caller: sawfish 0.12.9, Sniffles2, pbsv at 32-fold coverage
- truvari 4.2.2 singularity image (library://ctsa/truvari/truvari:4.2.2) and MAFFT

## Outputs

- truvari bench output directories (candidate.refine.bed regions) per caller
- truvari refine-recounted per-region results and truvari ga4gh-consolidated comparison VCFs
- Per-caller F1-scores (precision/recall) summarized via process_truvari_ga4gh_vcfs.py, including size-stratified accuracy stats

## How to apply

Map HG002 Revio HiFi reads (e.g. m84011_220902_175841_s1.hifi_reads.bam) to human_GRCh38_no_alt_analysis_set with pbmm2 1.13.1 using --preset CCS and --sort, then call SVs with sawfish 0.12.9 (sawfish discover --threads 16 followed by sawfish joint-call --threads 16) plus the competing callers (Sniffles2, pbsv). Preprocess the T2T benchmark by retaining SV records only with bcftools view -i 'INFO/SVTYPE!="."' and restricting the confident regions BED to autosomes with grep -v chr[XY]. Run truvari bench (4.2.2, singularity image library://ctsa/truvari/truvari:4.2.2) per caller with --reference, --includebed (autosomal benchmark BED), --base (filtered T2T benchmark VCF), --comp (caller VCF), --passonly, --pick ac, and --dup-to-ins; for the T2T assessment it is important to use bench plus refine with MAFFT, not bench alone. Run truvari refine with --recount --use-region-coords --use-original-vcfs --align mafft --threads 16 on candidate.refine.bed regions (using a regular-filesystem tmp dir for MAFFT), consolidate with truvari ga4gh --with-refine, and summarize with process_truvari_ga4gh_vcfs.py. For the CMRG benchmark, truvari bench alone suffices. Validate by checking that sawfish's size-stratified accuracy reproduces an F1 of ~0.976, exceeding Sniffles2 by ~1.2% and pbsv by ~1.4%.

## Related tools

- **truvari 4.2.2** (Benchmark caller VCFs against the filtered T2T truth set via bench/refine/ga4gh to compute F1-scores) — library://ctsa/truvari/truvari:4.2.2
- **sawfish 0.12.9** (Query SV caller (sawfish discover then sawfish joint-call) whose size-stratified F1 is compared against Sniffles2 and pbsv) — https://github.com/PacificBiosciences/sawfish
- **pbmm2 1.13.1** (Aligns HG002 HiFi reads to the GRCh38 no-alt reference with --preset CCS --sort to produce caller input BAMs)
- **MAFFT** (Alignment engine used by truvari refine (--align mafft) to refine bench candidates before ga4gh consolidation)
- **bcftools** (Filters the T2T benchmark VCF to SV records only via bcftools view -i 'INFO/SVTYPE!="."')
- **pbsv** (Competing long-read SV caller included in the three-caller F1 comparison) — https://github.com/PacificBiosciences/pbsv

## Examples

```
truvari bench --reference human_GRCh38_no_alt_analysis_set.fasta --includebed t2t_conf_regions.auto.bed --base GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz --comp sawfish.vcf.gz --passonly --pick ac --dup-to-ins -o bench.sawfish
```

## Evaluation signals

- sawfish size-stratified accuracy against the T2T benchmark reproduces an F1-score of approximately 0.976 on 32-fold HG002 HiFi data
- sawfish F1 exceeds Sniffles2 by ~1.2% and pbsv by ~1.4%, matching Supplementary Table S1
- The benchmark VCF contains only SV records (no SVTYPE='.') after bcftools filtering, and the confident-regions BED contains no chrX/chrY entries
- truvari refine completed over candidate.refine.bed with --recount --use-region-coords --use-original-vcfs --align mafft before ga4gh consolidation (bench+refine, not bench alone, for the T2T assessment)
- sawfish shows the highest accuracy among compared callers across every tested SV size group

## Limitations

- The T2T assessment requires the bench/refine/ga4gh workflow with MAFFT; using bench alone would not reproduce the reported F1-scores, and MAFFT needs tmp on a regular (non-shared) filesystem
- The benchmark files must be restricted to SV records and autosomes; unfiltered benchmark/confident regions change the comparison scope
- Results depend on the specific draft benchmark version (V0.019-20241113, GRCh38_HG2-T2TQ100-V1.1_stvar) and coverage level (32-fold); other depth levels were assessed separately from 10- to 32-fold

## Evidence

- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] it is important to use the bench and refine command, with the bench/refine results consolidated as described further below.: "it is important to use the bench and refine command, with the bench/refine results consolidated as described further below."
- [abstract] Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy: "Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy"
- [methods] The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."': "The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."'"
- [methods] The confident regions are restricted to autosomes as follows: grep -v chr[XY]: "The confident regions are restricted to autosomes as follows: grep -v chr[XY]"
