---
name: multi-caller-accuracy-comparison-across-coverage-levels
description: Use when you must quantify how long-read SV caller accuracy (F1 versus
  a GIAB truth set) degrades at reduced HiFi depth, e.g. testing whether a caller
  at 10x or 15x outperforms competitors.
when_to_use_negative:
- The goal is single-caller optimization or genotyping quality (e.g. pedigree concordance
  on CEPH-1463) rather than cross-caller accuracy comparison against a truth set.
- Data are short-read, ONT, or non-HiFi inputs — the protocol is built on mapped HiFi
  reads called with pbmm2 --preset CCS.
- No validated benchmark VCF/BED exists for the sample (e.g. not HG002 against the
  GIAB T2T SV set), so F1 cannot be computed with truvari bench.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: sawfish
  role: primary SV/CNV caller under test; run as sawfish discover then sawfish joint-call
    per subsample
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbsv
  role: competing long-read SV caller for the accuracy comparison
  repo: https://github.com/PacificBiosciences/pbsv
- name: pbmm2
  role: maps subsampled HiFi reads to GRCh38 with --preset CCS --sort
- name: truvari
  role: bench/refine/ga4gh assessment of caller VCFs against the T2T benchmark, with
    MAFFT refinement
  repo: library://ctsa/truvari/truvari:4.2.2
- name: bcftools
  role: filters benchmark VCF to structural variant records (INFO/SVTYPE!=".")
- name: hifiasm
  role: global assembly of subsampled reads for PAV-based SV calls (optional arm)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:74a3b6c10977003cfeddc00845807e94d7f97aca957842e44acf65106ce1e181
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
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
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# multi-caller-accuracy-comparison-across-coverage-levels

## Summary

A benchmarking protocol that subsamples HG002 Revio HiFi reads to 10-, 15-, and 30-fold coverage, calls SVs with sawfish, Sniffles2, and pbsv (plus hifiasm+PAV from assemblies), and scores each caller against the GIAB T2T SV benchmark with truvari 4.2.2 to compare F1 across depths.

## When to use

Use when you must quantify how long-read SV caller accuracy (F1 versus a GIAB truth set) degrades at reduced HiFi depth, e.g. testing whether a caller at 10x or 15x outperforms competitors.

## When NOT to use

- The goal is single-caller optimization or genotyping quality (e.g. pedigree concordance on CEPH-1463) rather than cross-caller accuracy comparison against a truth set.
- Data are short-read, ONT, or non-HiFi inputs — the protocol is built on mapped HiFi reads called with pbmm2 --preset CCS.
- No validated benchmark VCF/BED exists for the sample (e.g. not HG002 against the GIAB T2T SV set), so F1 cannot be computed with truvari bench.

## Inputs

- HG002 Revio HiFi BAM (m84011_220902_175841_s1.hifi_reads.bam)
- human_GRCh38_no_alt_analysis_set.fasta reference
- GIAB draft T2T SV benchmark V0.019-20241113 (GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz with benchmark BED)

## Outputs

- Caller VCFs per tool per coverage level (10x, 15x, 30x)
- truvari bench/refine/ga4gh comparison outputs with MAFFT refinement
- F1-score table per caller per coverage level (cf. Supplementary Table S3 / Fig. S2)

## How to apply

Subsample the HG002 Revio HiFi BAM (m84011_220902_175841_s1.hifi_reads.bam) to 10-, 15-, and 30-fold mean coverage, then map each subsample to human_GRCh38_no_alt_analysis_set.fasta with pbmm2 1.13.1 using --preset CCS --sort. Call SVs per coverage level with sawfish (sawfish discover followed by sawfish joint-call), Sniffles2, and pbsv, and optionally assemble each subsample globally with hifiasm and call SVs with PAV. Prepare the truth set by filtering GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz with bcftools (INFO/SVTYPE!=".") and restricting the benchmark BED to autosomes via grep -v chr[XY]. Assess each caller VCF with truvari 4.2.2 bench (--passonly --pick ac --dup-to-ins), refine with MAFFT (--recount --use-region-coords --use-original-vcfs --align mafft), consolidate via truvari ga4gh, and tabulate F1 per caller per coverage level against Supplementary Table S3 / Supplementary Fig. S2.

## Related tools

- **sawfish** (primary SV/CNV caller under test; run as sawfish discover then sawfish joint-call per subsample) — https://github.com/PacificBiosciences/sawfish
- **pbsv** (competing long-read SV caller for the accuracy comparison) — https://github.com/PacificBiosciences/pbsv
- **pbmm2** (maps subsampled HiFi reads to GRCh38 with --preset CCS --sort)
- **truvari** (bench/refine/ga4gh assessment of caller VCFs against the T2T benchmark, with MAFFT refinement) — library://ctsa/truvari/truvari:4.2.2
- **bcftools** (filters benchmark VCF to structural variant records (INFO/SVTYPE!="."))
- **hifiasm** (global assembly of subsampled reads for PAV-based SV calls (optional arm))

## Examples

```
bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz
```

## Evaluation signals

- Sawfish has the highest F1 at 10x coverage, exceeding Sniffles2 by >2.1% and pbsv by >7.9% on the GIAB T2T SV benchmark.
- Other callers need at least 30x coverage to match sawfish's 15x F1 (pattern of Supplementary Table S3 / Fig. S2).
- Benchmark VCF contains only SVTYPE-recorded variants and the benchmark BED is autosomes-only (no chrX/chrY).
- All truvari assessments use version 4.2.2 with MAFFT refinement and are consolidated via truvari ga4gh before summarization.
- Downsampled BAMs have nominal mean depths of 10-, 15-, and 30-fold with pbmm2 --preset CCS alignments.

## Limitations

- Coverage levels tested span 10- to 32-fold; behavior outside this range is not established.
- F1 comparisons depend on the specific GIAB draft T2T SV benchmark (V0.019-20241113) and autosomal restriction; other truth sets may rank callers differently.
- VACmap is still unsupported for problematic alignments, per the article's discussion.

## Evidence

- [abstract] Depth comparison shows sawfish leads at all coverages: "sawfish maintains the highest accuracy at every tested depth level from 10- to 32-fold coverage, such that other callers required at least 30-fold coverage to match sawfish accuracy at 15-fold"
- [methods] truvari version and refinement method: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] Benchmark VCF filtering step: "The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."'"
- [methods] Trusted confident-region restriction to autosomes: "The confident regions are restricted to autosomes as follows: grep -v chr[XY]"
- [methods] Read mapping setup for each subsample: "pbmm2 --log-level INFO align -j 16 --preset CCS --sample HG002 --sort --unmapped"
