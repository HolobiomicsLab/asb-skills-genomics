---
name: assembler-benchmark-comparison
description: Use when comparing assembler quality on downsampled (~5x) HiFi read sets
  and you need misassembled contig length, NG50/NGA50, BUSCO/Merqury deltas plus wall-time
  and max-RSS ratios between pipelines.
when_to_use_negative:
- Reads are high coverage (e.g. 25-40x) where hifiasm alone already produces high-quality
  assemblies — the skill targets ~5x low-coverage HiFi data
- Data are noisy long reads (ONT or PacBio CLR) rather than HiFi — HiFiCCL is designed
  specifically for low-coverage high-fidelity reads
- No high-quality reference genome is available — the HiFiCCL pipeline requires a
  reference (e.g. CHM13v2.0) for reference-guided chromosome-by-chromosome binning
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: HiFiCCL
  role: 'Test pipeline: reference-guided chromosome-by-chromosome binning of ~5x HiFi
    reads followed by per-chromosome de novo assembly, invoked as hificcl.py'
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: Hifiasm
  role: Base/baseline assembler run with --primary (0.19.5-r592, -t20) both standalone
    and invoked by HiFiCCL
  repo: https://github.com/chhylp123/hifiasm
- name: QUAST
  role: Contig evaluation against published assemblies for MCL, NG50 and NGA50
- name: BUSCO
  role: Gene completeness scoring with the vertebrate conserved single-copy ortholog
    database (v5.4.7)
- name: Merqury
  role: k-mer-based completeness and QV estimation of the compared assemblies
- name: seqtk
  role: Downsampling HiFi read sets to ~5x coverage for the benchmark (1.3-r106)
  repo: https://github.com/lh3/seqtk
- name: minimap2
  role: Sequence alignment underlying HiFiCCL's inter-sequence comparison and read
    partitioning
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:391081c1bb9a0fd98ade576db1f68f960d98c26d9870e7244845dce7ff2ab0b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zjjbuqi/HiFiCCL
  tools:
  - BUSCO
  - HiFiCCL
  - Hifiasm
  - Merqury
  - QUAST
  - minimap2
  - seqtk
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# assembler-benchmark-comparison

## Summary

Benchmark contig-level quality of HiFiCCL-Hifiasm against hifiasm-alone on low-coverage (~5x) PacBio HiFi human read sets using QUAST, BUSCO 5.4.7 and Merqury, quantifying misassembled contig length (MCL) reduction, NG50/NGA50, and runtime/memory cost.

## When to use

Use when comparing assembler quality on downsampled (~5x) HiFi read sets and you need misassembled contig length, NG50/NGA50, BUSCO/Merqury deltas plus wall-time and max-RSS ratios between pipelines.

## When NOT to use

- Reads are high coverage (e.g. 25-40x) where hifiasm alone already produces high-quality assemblies — the skill targets ~5x low-coverage HiFi data
- Data are noisy long reads (ONT or PacBio CLR) rather than HiFi — HiFiCCL is designed specifically for low-coverage high-fidelity reads
- No high-quality reference genome is available — the HiFiCCL pipeline requires a reference (e.g. CHM13v2.0) for reference-guided chromosome-by-chromosome binning

## Inputs

- HiFi read set (FASTQ/FASTA, e.g. HG002 SRR10382244, NA19240 SRR14611231) downsampled to ~5x coverage with seqtk 1.3-r106
- High-quality reference genome FASTA (CHM13 v2.0 or GRCh38) used by HiFiCCL for chromosome-by-chromosome binning and by QUAST as evaluation reference
- Published high-quality assemblies for QUAST comparison

## Outputs

- Primary contig FASTA assemblies from HiFiCCL-Hifiasm and hifiasm baseline
- QUAST v5.2.0 metrics: contig size, number of contigs, misassembled contig length, NG50, NGA50
- BUSCO 5.4.7 completeness scores (vertebrate conserved single-copy ortholog database) and Merqury completeness/QV
- assembly_evaluation_metrics.csv containing MCL reduction percentages, BUSCO/Merqury/NG50 deltas, and runtime ratio with max resident set size

## How to apply

Downsample each HiFi read set to ~5x coverage with seqtk (1.3-r106), then run the test assembler, HiFiCCL in default mode (hificcl.py -f <reads> -r CHM13v2.0.fasta -t 20), which invokes Hifiasm with --primary as the base assembler, and the baseline, Hifiasm 0.19.5-r592 (-t20 --primary), on identical inputs. Evaluate both contig sets with QUAST v5.2.0 against high-quality published assemblies (CHM13 v2.0, GRCh38) to obtain contig size, contig number, misassembled contig length, NG50 and NGA50. Assess gene completeness with BUSCO 5.4.7 using the vertebrate conserved single-copy ortholog database, and completeness/QV with Merqury. Record wall time and maximum resident set size for both pipelines and compute MCL reduction percentages and the runtime ratio. Expect HiFiCCL-Hifiasm to reduce MCL relative to Hifiasm by an average of 21.19% and up to 38.58% across 45 ~5x human datasets (e.g. HG002: 17.38%, 331.84 to 274.16 Mb; NA19240: 27.16%, 245.96 to 179.15 Mb), with improved BUSCO, Merqury and NG50 values, writing all metrics to assembly_evaluation_metrics.csv.

## Related tools

- **HiFiCCL** (Test pipeline: reference-guided chromosome-by-chromosome binning of ~5x HiFi reads followed by per-chromosome de novo assembly, invoked as hificcl.py) — https://github.com/zjjbuqi/HiFiCCL
- **Hifiasm** (Base/baseline assembler run with --primary (0.19.5-r592, -t20) both standalone and invoked by HiFiCCL) — https://github.com/chhylp123/hifiasm
- **QUAST** (Contig evaluation against published assemblies for MCL, NG50 and NGA50)
- **BUSCO** (Gene completeness scoring with the vertebrate conserved single-copy ortholog database (v5.4.7))
- **Merqury** (k-mer-based completeness and QV estimation of the compared assemblies)
- **seqtk** (Downsampling HiFi read sets to ~5x coverage for the benchmark (1.3-r106)) — https://github.com/lh3/seqtk
- **minimap2** (Sequence alignment underlying HiFiCCL's inter-sequence comparison and read partitioning) — https://github.com/lh3/minimap2

## Examples

```
python hificcl.py -f <reads> -r CHM13v2.0.fasta -t 20
```

## Evaluation signals

- MCL reduction percentage is positive and matches expected magnitudes: average 21.19%, maximum 38.58% across the 45 ~5x human datasets
- Dataset-level checks reproduce HG002 at 17.38% (331.84 to 274.16 Mb) and NA19240 at 27.16% (245.96 to 179.15 Mb) MCL reduction
- HiFiCCL-Hifiasm assembly shows non-inferior or improved BUSCO completeness, Merqury QV/completeness and NG50 relative to hifiasm alone
- Both pipelines use identical downsampled inputs (~5x) and thread settings (-t20) so the runtime ratio and max resident set size comparison is fair
- assembly_evaluation_metrics.csv is fully populated with MCL, NG50/NGA50, BUSCO, Merqury and runtime columns for every dataset

## Limitations

- Due to low coverage (~5x) and other limitations, many genomic regions remain missing; completeness gaps are unresolved
- Only two plant samples were tested; broader plant evaluation with higher genetic diversity has not been performed
- Integration of additional data types such as Hi-C or nanopore sequencing is not yet implemented
- No version numbers are stated for HiFiCCL itself (hifiasm version given only in the benchmark protocol as 0.19.5-r592)

## Evidence

- [abstract] reduces the length of misassembled contigs relative to hifiasm by an average of 21.19% and up to 38.58%: "reduces the length of misassembled contigs relative to hifiasm by an average of 21.19% and up to 38.58%"
- [methods] Assemble each dataset with HiFiCCL in default mode (hificcl.py -f <reads> -r CHM13v2.0.fasta -t 20), which invokes Hifiasm with --primary as the base assembler.: [evidence span withheld — anchor status: not_found]
- [methods] Evaluate contigs with QUAST v5.2.0 against high-quality published assemblies to obtain contig size, number of contigs, misassembled contig length, NG50 and NGA50.: [evidence span withheld — anchor status: not_found]
- [methods] Assess gene completeness with BUSCO 5.4.7 using the vertebrate conserved single-copy ortholog database, and completeness/QV with Merqury.: [evidence span withheld — anchor status: not_found]
- [discussion] due to low coverage and other limitations, many genomic regions remain missing: "due to low coverage and other limitations, many genomic regions remain missing"
