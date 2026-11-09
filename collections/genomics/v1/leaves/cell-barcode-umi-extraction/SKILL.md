---
name: cell-barcode-umi-extraction
description: Use when processing HIT-scISOseq CCS reads where primer-based FLNC classification
  must also yield per-read cellBC and UMI (isoseq_flnc.BarcodeUMI.fastq), or when
  corrected barcodes/UMIs are needed for isoform count matrices.
when_to_use_negative:
- Input reads are 10x ScISOr-Seq cell-barcode-free reads or TSO artifacts without
  both 5' and 3' primer hits in opposite orientation — classify_by_primer would reject
  them as non-FLNC.
- The library was not built with the 10x Genomics-compatible or HIT-scISOseq primer
  design; primer_F/primer_S names and sequences would not match the read structure.
- Data already consist of corrected, demultiplexed barcode/UMI tables (e.g. an existing
  cellBC-UMI matrix) rather than raw CCS reads.
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: scISA-Tools (classify_by_primer)
  role: Classifies CCS reads by 5'/3' primer orientation and extracts cellBC, UMI,
    and FLNC transcript sequences with min_primerlen 16 and min_isolen 50 thresholds
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: ncbi-blast+
  role: makeblastdb builds the primer nucleotide database and blastn (-outfmt 7 -word_size
    5) maps CCS reads to primers for the classifier
- name: samtools
  role: Converts the CCS BAM into ccs.fa and ccs.fq consumed by blastn and classify_by_primer
- name: cellBC_UMI_corrector (scISA-Tools / CellRanger)
  role: Corrects extracted cell barcodes against the whitelist (>0.975 posterior)
    and filters UMIs (no homopolymers, no N, base quality >=10)
  repo: https://github.com/shizhuoxing/scISA-Tools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:096852b07ad06f03ddc7a47afe380f391e4413274f9ef84263b1297bc79bd8d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - cellBC_UMI_corrector (scISA-Tools / CellRanger)
  - ncbi-blast+
  - samtools
  - scISA-Tools
  - scISA-Tools (classify_by_primer)
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# cell-barcode-umi-extraction

## Summary

Extracts 10x Genomics cell barcodes and UMIs embedded in HIT-scISOseq full-length non-concatemer (FLNC) reads during classify_by_primer, then corrects them with a CellRanger-like whitelist strategy so single-cell isoform matrices can be built from PacBio CCS data.

## When to use

Use when processing HIT-scISOseq CCS reads where primer-based FLNC classification must also yield per-read cellBC and UMI (isoseq_flnc.BarcodeUMI.fastq), or when corrected barcodes/UMIs are needed for isoform count matrices.

## When NOT to use

- Input reads are 10x ScISOr-Seq cell-barcode-free reads or TSO artifacts without both 5' and 3' primer hits in opposite orientation — classify_by_primer would reject them as non-FLNC.
- The library was not built with the 10x Genomics-compatible or HIT-scISOseq primer design; primer_F/primer_S names and sequences would not match the read structure.
- Data already consist of corrected, demultiplexed barcode/UMI tables (e.g. an existing cellBC-UMI matrix) rather than raw CCS reads.

## Inputs

- PacBio CCS BAM (e.g. ccs.bam, generated with --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75)
- ccs.fa / ccs.fq converted from the CCS BAM via samtools view + awk
- primer.fa BLAST database with 10x Genomics-compatible primer_F and primer_S sequences
- mapped.m7 blastn -outfmt 7 primer-alignment file
- 10x Genomics cell barcode whitelist (3M-february-2018.txt)

## Outputs

- isoseq_flnc.BarcodeUMI.fastq (FLNC reads carrying extracted cellBC and UMI)
- isoseq_flnc.Transcript.fastq (FLNC transcript sequences after primer and polyA trimming)
- isoseq.PrimerStat.csv (primer classification statistics)
- corrected cellBC/UMI assignments from cellBC_UMI_corrector for matrix generation

## How to apply

Convert the CCS BAM to FASTA/FASTQ with samtools view piped through awk, build a nucleotide BLAST database from primer_F (AAGCAGTGGTATCAACGCAGAGTACATGGG, 5' primer) and primer_S (CTACACGACGCTCTTCCGATCT, 3' primer), and blastn the CCS reads with -word_size 5. Run classify_by_primer.pl with -min_primerlen 16 (minimum primer alignment length) and -min_isolen 50 (minimum transcript length excluding the polyA tail); it classifies reads by primer orientation (FLNC requires 5p + 3p- or 3p + 5p- at the terminals), trims primer and polyA, and emits isoseq_flnc.BarcodeUMI.fastq alongside isoseq_flnc.Transcript.fastq and isoseq.PrimerStat.csv. The BarcodeUMI fastq preserves the raw cell barcode and UMI, which the downstream cellBC_UMI_corrector module (wrapping CellRanger logic) corrects: UMIs must not be homopolymers, contain N, or contain bases with quality <10, and an observed barcode is replaced by the whitelist barcode with the highest posterior probability only if that probability exceeds 0.975.

## Related tools

- **scISA-Tools (classify_by_primer)** (Classifies CCS reads by 5'/3' primer orientation and extracts cellBC, UMI, and FLNC transcript sequences with min_primerlen 16 and min_isolen 50 thresholds) — https://github.com/shizhuoxing/scISA-Tools
- **ncbi-blast+** (makeblastdb builds the primer nucleotide database and blastn (-outfmt 7 -word_size 5) maps CCS reads to primers for the classifier)
- **samtools** (Converts the CCS BAM into ccs.fa and ccs.fq consumed by blastn and classify_by_primer)
- **cellBC_UMI_corrector (scISA-Tools / CellRanger)** (Corrects extracted cell barcodes against the whitelist (>0.975 posterior) and filters UMIs (no homopolymers, no N, base quality >=10)) — https://github.com/shizhuoxing/scISA-Tools

## Examples

```
perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./
```

## Evaluation signals

- All three outputs (isoseq_flnc.BarcodeUMI.fastq, isoseq_flnc.Transcript.fastq, isoseq.PrimerStat.csv) are produced in the specified output directory.
- FLNC reads in isoseq_flnc.Transcript.fastq satisfy -min_primerlen 16 and -min_isolen 50 thresholds, with primer and polyA tail trimmed.
- Retained reads carry 5' and 3' primer hits in opposite orientations (5p + 3p- or 3p + 5p-) consistent with FLNC definition.
- Corrected cell barcodes map to the 10x whitelist with posterior probability >0.975; rejected UMIs include any homopolymer, N-containing, or sub-quality-10 sequences.
- Downstream scIsoform_matrix/scGene_matrix generation succeeds only if barcode/UMI extraction was correct, e.g. isoform matrix after -minUMIcount 3 and detection in >=5 cells.

## Limitations

- classify_by_primer requires primers named exactly primer_F (5') and primer_S (3') in the BLAST database; other naming or primer designs will fail or misclassify.
- cellBC_UMI_corrector discards barcodes whose best whitelist posterior probability does not exceed 0.975, and UMIs failing quality rules are dropped, so barcode-rich but low-quality reads are lost.
- Only libraries compatible with the PacBio original scIsoSeq protocol with 10x Genomics or the BGI HIT-scISOseq protocol are supported by this classification flow.

## Evidence

- [readme] classify_by_primer invocation with thresholds: "perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./"
- [discussion] classify_by_primer extracts barcodes and UMIs: "the classify_by_primer utility was employed to extract cell barcodes and UMIs"
- [discussion] UMI quality filtering rules: "Must not be a homopolymer, e.g. AAAAAAAAAA;Must not contain N;Must not contain bases with base quality <10"
- [discussion] Cell barcode whitelist correction threshold: [evidence span withheld — anchor status: not_found]
- [methods] FLNC definition by primer orientation: "classify_by_primer wraps a tool to detect full-length transcript from CCS base on PacBio original scIsoSeq library construction protocol"
