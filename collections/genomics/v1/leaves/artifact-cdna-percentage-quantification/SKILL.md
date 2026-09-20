---
name: artifact-cdna-percentage-quantification
description: Use when reproducing Table 1 read-processing metrics (CCS counts/lengths,
  FLNC counts, artifact cDNA percentages) from deposited PacBio subreads BAM files
  of 10x Genomics-based single-cell isoform libraries.
when_to_use_negative:
- Input data is already demultiplexed single-cell cDNA fastq with cell barcodes extracted
  (artifact fraction can no longer be derived from raw subreads)
- Libraries were generated with a different wet-lab protocol whose primer sequences
  or library structure differ from the 10x Genomics ScISOr-Seq / HIT-scISOseq constructs
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: PacBio ccs (SMRTlink)
  role: Generates circular consensus (CCS) reads from subreads BAMs with modified
    parameters --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: samtools
  role: Converts ccs.bam to ccs.fa and ccs.fq via samtools view and awk for primer
    classification
- name: ncbi-blast
  role: makeblastdb builds the primer database; blastn (-word_size 5, -outfmt 7) maps
    primers against CCS reads for FLNC detection
- name: scISA-Tools classify_by_primer.pl
  role: Extracts FLNC reads, cell barcodes and UMIs with -min_primerlen 16 -min_isolen
    50; reads lacking valid structure are counted as artifact cDNA
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: PacBio Sequel II
  role: Instrument platform whose SMRT Cell 8M output (subreads BAMs) is the source
    data for recomputation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:d51057b13719752893ddbfa35dd34aac960a17edf215753b0923279039de11f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - PacBio Sequel II
  - PacBio ccs (SMRTlink)
  - ncbi-blast
  - samtools
  - scISA-Tools
  - scISA-Tools classify_by_primer.pl
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# artifact-cdna-percentage-quantification

## Summary

Quantify the percentage of artifact cDNA reads (cell-barcode-free reads, mostly TSO artifacts) per sample by recomputing CCS and FLNC counts from PacBio subreads BAMs using the paper's stated CCS parameters, to compare ScISOr-Seq (~51-53% artifact) against HIT-scISOseq (~7.6-8.2% artifact).

## When to use

Use when reproducing Table 1 read-processing metrics (CCS counts/lengths, FLNC counts, artifact cDNA percentages) from deposited PacBio subreads BAM files of 10x Genomics-based single-cell isoform libraries.

## When NOT to use

- Input data is already demultiplexed single-cell cDNA fastq with cell barcodes extracted (artifact fraction can no longer be derived from raw subreads)
- Libraries were generated with a different wet-lab protocol whose primer sequences or library structure differ from the 10x Genomics ScISOr-Seq / HIT-scISOseq constructs

## Inputs

- Deposited PacBio subreads BAM files per sample (HIT-scISOseq s1/s2 and ScISOr-Seq s1/s2 baseline libraries)
- primer.fa containing primer_F (AAGCAGTGGTATCAACGCAGAGTACATGGG, 5' primer) and primer_S (CTACACGACGCTCTTCCGATCT, 3' primer)

## Outputs

- Per-sample CCS read counts and mean CCS read lengths
- Per-sample FLNC read counts
- Per-sample artifact cDNA percentages (expected ~51.31-53.44% for ScISOr-Seq/Linked-scISOseq, 7.58-8.20% for HIT-scISOseq)
- Table 1 replica comparing ScISOr-Seq, Linked-scISOseq, and HIT-scISOseq metrics

## How to apply

Retrieve the deposited subreads BAM files for each sample (HIT-scISOseq and ScISOr-Seq baseline libraries) and run PacBio ccs with the reported parameters '--min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75' to generate ccs.bam. Convert ccs.bam to ccs.fa and ccs.fq with samtools view and awk, build a primer blast database from primer.fa (primer_F as 5' primer, primer_S as 3' primer) with makeblastdb, and align CCS reads using blastn with -word_size 5 and -outfmt 7. Run classify_by_primer.pl with -min_primerlen 16 and -min_isolen 50 to extract full-length non-chimeric (FLNC) reads and separate BarcodeUMI vs Transcript fastq files. Count total CCS reads, compute mean CCS read length, count FLNC reads, and derive the artifact cDNA percentage (reads lacking a valid cell barcode/primer structure) per sample; assemble these into a table mirroring Table 1. The biotin-capture step of HIT-scISOseq is validated when its artifact fraction is far below the original 10x ScISOr-Seq protocol.

## Related tools

- **PacBio ccs (SMRTlink)** (Generates circular consensus (CCS) reads from subreads BAMs with modified parameters --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75) — https://github.com/shizhuoxing/scISA-Tools
- **samtools** (Converts ccs.bam to ccs.fa and ccs.fq via samtools view and awk for primer classification)
- **ncbi-blast** (makeblastdb builds the primer database; blastn (-word_size 5, -outfmt 7) maps primers against CCS reads for FLNC detection)
- **scISA-Tools classify_by_primer.pl** (Extracts FLNC reads, cell barcodes and UMIs with -min_primerlen 16 -min_isolen 50; reads lacking valid structure are counted as artifact cDNA) — https://github.com/shizhuoxing/scISA-Tools
- **PacBio Sequel II** (Instrument platform whose SMRT Cell 8M output (subreads BAMs) is the source data for recomputation)

## Examples

```
ccs *.subreads.bam ccs.bam --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75 -j 20
```

## Evaluation signals

- Recomputed artifact cDNA percentages fall within reported ranges: ~51-53% for ScISOr-Seq (51.31%/53.44% s1/s2) and 53.42% for Linked-scISOseq vs ~7.6-8.2% for HIT-scISOseq (7.58% s1, 8.20% s2)
- Per-sample CCS read counts and mean CCS read lengths match the corresponding Table 1 rows
- Per-sample FLNC read counts match the corresponding Table 1 rows
- isoseq.PrimerStat.csv from classify_by_primer confirms primer classification statistics consistent with FLNC counts
- HIT-scISOseq artifact percentages are roughly 7x lower than the baseline protocols, confirming the biotin-capture step removes most TSO artifacts

## Limitations

- The wet-lab protocol is hosted on a private (access-restricted) protocols.io URL (https://www.protocols.io/private/7472E845C45C11EC97780A58A9FEAC02), so re-deriving artifact structure requires trusting the reported parameters
- Artifact cDNA percentage is operationally defined as reads lacking a valid cell barcode/primer structure; alternative definitions (e.g. short-insert or TSO-dimer detection) may yield different percentages
- Only two samples per condition (s1, s2) are available for the comparison, limiting generalizability of the reproduced metrics

## Evidence

- [discussion] CCS read generation with modified SMRT-Link parameters: [evidence span withheld — anchor status: not_found]
- [discussion] FLNC generation parameters: [evidence span withheld — anchor status: not_found]
- [methods] classify_by_primer invocation: "perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./"
- [intro] 10x ScISOr-Seq artifact proportion: "the 10× Genomics single-cell preparation pipeline of ScISOr-Seq introduces a high proportion (~50%) of undesirable cell-barcode-free reads, mostly template-switching oligonucleotide (TSO) artifacts"
- [readme] classify_by_primer usage in README: "perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./"
- [readme] CCS command in README: "ccs *.subreads.bam ccs.bam --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75 -j 20"
