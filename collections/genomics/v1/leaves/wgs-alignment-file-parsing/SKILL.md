---
name: wgs-alignment-file-parsing
description: Use when a task requires parsing whole-genome BAM alignment files for
  CNV read-depth analysis, or when benchmarking CNVpytor against CNVnator parse speed
  and intermediate file size is needed.
when_to_use_negative:
- The task requires downstream CNV calling, BAF/SNP analysis, or call filtering rather
  than the read-depth parsing step itself
- Input is already a parsed .pytor file or CNVnator ROOT histogram, so no alignment-file
  parsing is needed
- The alignment data are not whole-genome BAMs compatible with CNVnator/CNVpytor read-depth
  parsing (e.g., small targeted panels)
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: CNVpytor
  role: Parses BAM alignment files into a compressed HDF5 .pytor intermediate using
    its pysam-based parallelized read-depth engine
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Baseline read-depth parser whose parse wall-time and intermediate output file
    size are benchmarked against CNVpytor
- name: Python
  role: Implementation language of CNVpytor and used for assembling the per-sample
    benchmark metrics table
  repo: https://github.com/abyzovlab/CNVpytor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:a604bf7bafb2d4387831e177602d88f4175541ece73cc07bf5b9a52b02290010
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Python
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# wgs-alignment-file-parsing

## Summary

Benchmark and execute read-depth parsing of whole-genome alignment (BAM) files with CNVpytor's pysam-based parallelized engine versus CNVnator, quantifying parse wall-time and intermediate (.pytor HDF5 vs ROOT) file-size footprint on the CEPH trio example dataset.

## When to use

Use when a task requires parsing whole-genome BAM alignment files for CNV read-depth analysis, or when benchmarking CNVpytor against CNVnator parse speed and intermediate file size is needed.

## When NOT to use

- The task requires downstream CNV calling, BAF/SNP analysis, or call filtering rather than the read-depth parsing step itself
- Input is already a parsed .pytor file or CNVnator ROOT histogram, so no alignment-file parsing is needed
- The alignment data are not whole-genome BAMs compatible with CNVnator/CNVpytor read-depth parsing (e.g., small targeted panels)

## Inputs

- CEPH (Platinum Genomes) trio example BAM whole-genome alignment files (3 samples)
- CNVpytor installation (pip install cnvpytor; cnvpytor -download)
- CNVnator installation (ROOT-based read-depth parser)

## Outputs

- Per-sample CNVpytor .pytor intermediate files (HDF5 with compressed 100-bp RD bins)
- Per-sample CNVnator intermediate ROOT output files
- Parsing benchmark table (parsing_benchmark.tsv) with per-sample wall-times, intermediate file sizes, speedup ratio, and size ratio

## How to apply

Obtain the CEPH (Platinum Genomes) trio example BAM files (available via the Platinum Genomes public data bucket referenced in the paper). For each sample, parse with CNVpytor using 'cnvpytor -root file.pytor -rd file.bam', which uses a pysam-based parallelized read-depth engine and stores 100-bp RD bins compressed in an HDF5-format .pytor file; record parse wall-time. Parse the same BAM with CNVnator's read-depth parser and record its wall-time and intermediate file size. Compute the per-sample CNVpytor-over-CNVnator speedup ratio and the CNVnator-over-CNVpytor intermediate-size ratio, and assemble them into a metrics table (parsing_benchmark.tsv). The paper reports CNVpytor parses alignment files 2-20 times faster and produces intermediate files 20-50 times smaller than CNVnator, which serves as the expected operating range.

## Related tools

- **CNVpytor** (Parses BAM alignment files into a compressed HDF5 .pytor intermediate using its pysam-based parallelized read-depth engine) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Baseline read-depth parser whose parse wall-time and intermediate output file size are benchmarked against CNVpytor)
- **Python** (Implementation language of CNVpytor and used for assembling the per-sample benchmark metrics table) — https://github.com/abyzovlab/CNVpytor

## Examples

```
cnvpytor -root file.pytor -rd file.bam
```

## Evaluation signals

- Computed per-sample CNVpytor-over-CNVnator parse speedup falls within the paper-reported 2-20x range for BAM parsing
- Computed CNVnator-over-CNVpytor intermediate-size ratio falls within the paper-reported 20-50x range
- Each CNVpytor run produces a valid .pytor (HDF5) file containing 100-bp RD bins
- parsing_benchmark.tsv contains wall-times, intermediate file sizes, and both ratios for all three CEPH trio samples
- All three trio samples parse without error using the documented cnvpytor -rd command

## Limitations

- Reported speedup (2-20x) and size reduction (20-50x) ranges are hardware- and dataset-dependent; CNVpytor v1.3.1 further reduces .pytor size via BAF likelihood matrix compression and the -nolh option
- Benchmark validates the parsing step only; it does not assess CNV call concordance (though the paper reports >99% overlap between CNVnator and CNVpytor called regions)
- CNVnator depends on ROOT, adding installation overhead relative to the pip-installable CNVpytor

## Evidence

- [abstract] CNVpytor is significantly faster than CNVnator—particularly for parsing alignment files (2–20 times faster): "CNVpytor is significantly faster than CNVnator—particularly for parsing alignment files (2–20 times faster)"
- [abstract] smaller intermediate files: "and has (20–50 times) smaller intermediate files"
- [methods] call CNVs using read depth: "> cnvpytor -root file.pytor -rd file.bam"
- [discussion] HDF5 100-bp bin compression: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [readme] v1.3.1 pytor size reduction options: "Reduced Pytor file size by compressing the BAF likelihood matrix"
