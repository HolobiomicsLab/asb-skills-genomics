---
name: assignment-efficiency-quantification
description: Use when evaluating Nanopore/ScNaUmi-seq runs for how many reads pass
  stranding and barcode/UMI assignment, and when Nanopore sequencing errors may cause
  UMI misassignment that must be quantified before downstream analysis.
when_to_use_negative:
- Input is already a cellBC/UMI-annotated BAM or gene/isoform count matrix — assignment
  and efficiency estimation have already been performed
- Data are short-read Illumina-only without Nanopore long reads; barcode assignment
  is handled directly by Cellranger rather than by this Nanopore-guided workflow
- Reads lack polyA tails and adapter structure (e.g. RACE PCR with 5' barcoding) and
  --noPolyARequired applies, changing the stranding/efficiency computation
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: NanoporeBC_UMI_finder.jar (sicelore)
  role: Performs scanfastq (chimeric read splitting, polyA/adapter stranding, cell
    barcode assignment at --bcEditDistance 1) and assignumis (UMI clustering within
    500 nt windows at edit distance <= 2); randomBarcode/randomUMI options estimate
    false-assignment rates
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: minimap2
  role: Maps the 'pass'-folder Nanopore reads against the reference genome (-ax splice
    -uf --sam-hit-only) as the prerequisite for UMI assignment
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2 BAM before UMI assignment
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:c213fe670c7fe099010da0d8195506d718fab8fef04a057ee0db240d16e2ea2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - NanoporeBC_UMI_finder.jar (sicelore)
  - minimap2
  - samtools
  - sicelore-2.1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# assignment-efficiency-quantification

## Summary

Quantifies how efficiently and accurately Illumina-guided cell barcodes (cellBC) and UMIs are assigned to Nanopore single-cell reads in the ScNaUmi-seq workflow, combining efficiency fractions over the NanoporeBC_UMI_finder pipeline with random-sequence replacement simulation to estimate false-assignment rates.

## When to use

Use when evaluating Nanopore/ScNaUmi-seq runs for how many reads pass stranding and barcode/UMI assignment, and when Nanopore sequencing errors may cause UMI misassignment that must be quantified before downstream analysis.

## When NOT to use

- Input is already a cellBC/UMI-annotated BAM or gene/isoform count matrix — assignment and efficiency estimation have already been performed
- Data are short-read Illumina-only without Nanopore long reads; barcode assignment is handled directly by Cellranger rather than by this Nanopore-guided workflow
- Reads lack polyA tails and adapter structure (e.g. RACE PCR with 5' barcoding) and --noPolyARequired applies, changing the stranding/efficiency computation

## Inputs

- Nanopore fastq files from the sequencer fastq_pass folder (individually gzipped, unmerged)
- Illumina-derived Cellranger list of used cell barcodes (--cellRangerBCs) from the same sample
- 10x Genomics barcode whitelist (e.g. 3M-february-2018.txt.gz or 737K-august-2016.txt)
- Reference genome and annotation for minimap2 mapping
- Independent Illumina single-cell dataset (e.g. GSM4224249) for combined-precision cross-check

## Outputs

- Pass/failed read folders (stranded forward reads in 'pass') and an HTML stats file from scanfastq
- Sorted, indexed BAM with cell barcodes (BC tag) and UMIs (U8 tag) from assignumis
- Assignment efficiency metrics (fraction of reads with polyA+adapter found, cellBC assigned, UMI assigned)
- False-assignment rate estimates from randomBarcode (-e) and randomUMI (-f) simulations
- Combined cellBC+UMI assignment precision versus the independent Illumina dataset

## How to apply

Run NanoporeBC_UMI_finder.jar scanfastq on the sequencer fastq files (unmerged, individually gzipped) with --bcEditDistance 1 and Illumina-guided barcodes supplied via --cellRangerBCs from the same-sample Cellranger run; this splits chimeric reads, strands reads by poly(A)/adapter detection (>= 15 nt polyA with >= 75% As), and writes pass/failed folders plus a stats HTML. Map the 'pass' reads against the reference genome with minimap2 (-ax splice -uf --sam-hit-only) and sort/index with samtools, then run assignumis to cluster UMIs per cell within 500 nt genomic windows at edit distance <= 2, producing a BAM with BC and U8 tags. Compute assignment efficiencies as the fraction of reads with poly(A)+adapter found, with a cell barcode assigned, and with a UMI assigned. Estimate false-assignment rates by re-running scanfastq and assignumis with randomBarcode (-e) and randomUMI (-f) simulations, and cross-check combined cellBC+UMI precision against an independent Illumina dataset (e.g. GSM4224249). The expected reference outcome is 99.8% cellBC and 97.4% UMI accuracy by random-sequence replacement and 99.3% combined precision.

## Related tools

- **NanoporeBC_UMI_finder.jar (sicelore)** (Performs scanfastq (chimeric read splitting, polyA/adapter stranding, cell barcode assignment at --bcEditDistance 1) and assignumis (UMI clustering within 500 nt windows at edit distance <= 2); randomBarcode/randomUMI options estimate false-assignment rates) — https://github.com/ucagenomix/sicelore-2.1
- **minimap2** (Maps the 'pass'-folder Nanopore reads against the reference genome (-ax splice -uf --sam-hit-only) as the prerequisite for UMI assignment) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2 BAM before UMI assignment) — https://github.com/samtools/samtools

## Examples

```
java -jar -Xmx40g <path>/NanoporeBC_UMI_finder.jar scanfastq -d <fastq_pass directory> -o outPutDirectory --bcEditDistance 1 -g <cellRangerBCs> -e -f
```

## Evaluation signals

- Random-sequence replacement simulation yields cellBC accuracy ~99.8% and UMI accuracy ~97.4%
- Combined cellBC+UMI assignment precision versus the independent Illumina dataset (GSM4224249) reaches ~99.3%
- Assignment efficiencies (polyA+adapter found, cellBC assigned, UMI assigned) fall near the paper-reported values (~57/68/76%) with matching sample-to-sample variation (~11/4/3%)
- Barcode assignment accuracy at --bcEditDistance 1 is > 99.9%, consistent with the README expectation for edit distance 1
- A stats HTML file is produced and 'pass' reads are stranded (forward) while failed reads remain unstranded

## Limitations

- Edit distance 2 for barcodes yields about 5-10% more barcode-assigned reads but lowers assignment accuracy; edit distance 1 is largely sufficient with current read qualities
- At least one-third of UMIs were mis-assigned in a previous strategy due to Nanopore sequencing errors — randomUMI simulation is required to quantify residual misassignment
- Scanning 100M reads takes ~80 min on the 96-core Promethion and requires generous RAM (-Xmx40g to -Xmx64G); fastq files must not be merged into a single archive

## Evidence

- [other] The accuracy of the Illumina-guided cellBC and UMI assignment to Nanopore reads, evaluated by random-sequence replacement, was 99.8% for cellBC and 97.4% for UMI.: [evidence span withheld — anchor status: not_found]
- [readme] Barcode assignment accuracy is typically > 99.9% for edit distance 1, edit distance 2 yields about 5 - 10% more barcode assigned reads but the tradeoff is a lower assignment accuracy.: "Barcode assignment accuracy is typically > 99.9% for edit distance 1, edit distance 2 yields about 5 - 10% more barcode assigned reads but the tradeoff is a lower assignment accuracy."
- [intro] The authors suggest that at least one-third of the UMIs were mis-assigned with their strategy due to Nanopore sequencing errors.: "The authors suggest that at least one-third of the UMIs were mis-assigned with their strategy due to Nanopore sequencing errors."
- [readme] Scans the Nanopore fastq reads for poly(A/T) and adapter sequence and generates stranded (forward) reads for reads with found polyA and adapter.: "Scans the Nanopore fastq reads for poly(A/T) and adapter sequence and generates stranded (forward) reads for reads with found polyA and adapter."
