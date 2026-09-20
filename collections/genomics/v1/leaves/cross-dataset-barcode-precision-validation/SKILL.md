---
name: cross-dataset-barcode-precision-validation
description: Use when you need to quantify false cell barcode or UMI assignment rates
  for Nanopore reads, or to validate Nanopore-assigned cellBC/UMI pairs against an
  independent Illumina short-read dataset from the same sample.
when_to_use_negative:
- Data were produced without a 10x Genomics barcode structure (e.g. RACE PCR combined
  with 5' barcoding without polyA), so Illumina-guided cellBC/UMI pair comparison
  is not applicable
- You need biological/variant accuracy (e.g. SNV or isoform correctness) rather than
  barcode/UMI assignment accuracy
- No independent Illumina dataset or same-sample Cellranger barcode list is available,
  so the cross-dataset precision check cannot be performed
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: sicelore-2.1 (NanoporeBC_UMI_finder.jar)
  role: Performs scanfastq barcode/UMI assignment with --bcEditDistance 1 and the
    -e/--randomBarcode, -f/--randomUMI simulation modes used to estimate false-assignment
    rates
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: minimap2
  role: Maps 'pass' reads (-ax splice -uf --sam-hit-only) prior to assignumis so per-cell
    UMI clustering and assignment validation can run on sorted BAM records
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2 BAM consumed by the assignumis step
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:208eb2c50a202ea7e78e4b90e66a2635690b3f014cd9f98858771ec905037d3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - minimap2
  - samtools
  - sicelore-2.1
  - sicelore-2.1 (NanoporeBC_UMI_finder.jar)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-dataset-barcode-precision-validation

## Summary

Validates the accuracy of Illumina-guided cellBC and UMI assignment to Nanopore reads by random-sequence replacement simulations (--randomBarcode/--randomUMI) and by cross-checking Nanopore-assigned barcode/UMI pairs against an independent Illumina dataset (GSM4224249). Expected reference performance: 99.8% cellBC and 97.4% UMI accuracy, 99.3% combined precision.

## When to use

Use when you need to quantify false cell barcode or UMI assignment rates for Nanopore reads, or to validate Nanopore-assigned cellBC/UMI pairs against an independent Illumina short-read dataset from the same sample.

## When NOT to use

- Data were produced without a 10x Genomics barcode structure (e.g. RACE PCR combined with 5' barcoding without polyA), so Illumina-guided cellBC/UMI pair comparison is not applicable
- You need biological/variant accuracy (e.g. SNV or isoform correctness) rather than barcode/UMI assignment accuracy
- No independent Illumina dataset or same-sample Cellranger barcode list is available, so the cross-dataset precision check cannot be performed

## Inputs

- Nanopore FASTQ files (unmerged, per-file compressed fastq_pass folders)
- Illumina-derived 10x Genomics used-barcode list (--cellRangerBCs from a same-sample Cellranger run)
- 10x Genomics all-possible-barcode whitelist (e.g. 3M-february-2018.txt.gz or 737K-august-2016.txt)
- Sorted, indexed BAM of 'pass' reads mapped with minimap2 (-ax splice -uf --sam-hit-only) for UMI assignment
- Independent Illumina short-read dataset (e.g. GSM4224249) with barcode/UMI pairs

## Outputs

- False-assignment (accuracy) rates for cellBC and UMI from random-sequence replacement simulations
- Combined cellBC+UMI assignment precision against the Illumina dataset
- Assignment efficiency metrics: fraction of reads with poly(A)+adapter found, cell barcode assigned, and UMI assigned
- Stats HTML report from the scanfastq step

## How to apply

Run NanoporeBC_UMI_finder.jar scanfastq with --bcEditDistance 1 and an Illumina-derived barcode list (--cellRangerBCs from the same-sample Cellranger run); edit distance 1 is preferred because edit distance 2 yields ~5-10% more assigned reads but lower accuracy. Re-run the scan and UMI assignment with -e/--randomBarcode and -f/--randomUMI to replace real barcodes/UMIs with random sequences and estimate false-assignment rates (reported as 99.8% cellBC and 97.4% UMI accuracy). Then compute combined precision by comparing Nanopore-assigned cellBC+UMI pairs against the independent Illumina dataset (GSM4224249), which gave 99.3% precision. Also track assignment efficiencies: fraction of reads with poly(A)+adapter found, with a cell barcode assigned, and with a UMI assigned (paper values ~57/68/76% with ~11/4/3% variation), using the stats HTML the scan writes.

## Related tools

- **sicelore-2.1 (NanoporeBC_UMI_finder.jar)** (Performs scanfastq barcode/UMI assignment with --bcEditDistance 1 and the -e/--randomBarcode, -f/--randomUMI simulation modes used to estimate false-assignment rates) — https://github.com/ucagenomix/sicelore-2.1
- **minimap2** (Maps 'pass' reads (-ax splice -uf --sam-hit-only) prior to assignumis so per-cell UMI clustering and assignment validation can run on sorted BAM records) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2 BAM consumed by the assignumis step) — https://github.com/samtools/samtools

## Examples

```
java -jar -Xmx40g NanoporeBC_UMI_finder.jar scanfastq -d fastq_pass -o outdir --bcEditDistance 1 --cellRangerBCs cellranger_barcodes.tsv -e -f
```

## Evaluation signals

- Random-sequence replacement (-e/--randomBarcode) reproduces ~99.8% cellBC assignment accuracy
- Random-sequence replacement (-f/--randomUMI) reproduces ~97.4% UMI assignment accuracy
- Nanopore-assigned cellBC+UMI pairs match ~99.3% of the independent Illumina dataset (GSM4224249)
- Assignment efficiencies fall near ~57% (polyA+adapter found), ~68% (cell barcode assigned), ~76% (UMI assigned), with reported variations of ~11/4/3%
- Stats HTML from scanfastq reports barcode assignment at edit distance 1 with expected >99.9% typical accuracy

## Limitations

- Edit distance 2 assigns ~5-10% more reads but lowers barcode assignment accuracy; accuracy is quoted only for --bcEditDistance 1
- UMI assignment accuracy (97.4%) is lower than cellBC accuracy (99.8%), so a residual UMI false-assignment fraction persists
- The combined precision check requires an independent Illumina dataset from the same sample; it cannot be done short-read-free

## Evidence

- [results] The accuracy of the Illumina-guided cellBC and UMI assignment to Nanopore reads, evaluated by random-sequence replacement, was 99.8% for cellBC and 97.4% for UMI.: [evidence span withheld — anchor status: not_found]
- [supplementary] Combined precision against GSM4224249 is 99.3% when Nanopore-assigned barcode/UMI pairs are compared to the independent Illumina dataset.: [evidence span withheld — anchor status: not_found]
- [readme] If Illumina data are available the file with the list of used barcodes can be supplied (--cellRangerBCs) and the supplied barcode list is used instead of the first pass.: "If Illumina data are available the file with the list of used barcodes can be supplied (--cellRangerBCs < path to file >) -> the first pass will be skipped"
- [readme] Barcode assignment accuracy is typically >99.9% for edit distance 1; edit distance 2 gains ~5-10% more reads at lower accuracy.: "Barcode assignment accuracy is typically > 99.9% for edit distance 1, edit distance 2 yields about 5 - 10% more barcode assigned reads but the tradeoff is a lower assignment accuracy."
