---
name: polya-tail-trimming
description: Use when PacBio CCS reads (BAM/FASTQ) from HIT-scISOseq or 10x-compatible
  scIso-Seq libraries must be classified as full-length by primer orientation and
  have polyA tails, barcodes, and UMIs trimmed before genome mapping.
when_to_use_negative:
- Input reads are already FLNC transcript FASTQs that have been primer-, barcode-,
  UMI-, and polyA-trimmed (proceed directly to minimap2 splice alignment).
- The library was not built with the PacBio original scIso-Seq or BGI HIT-scISOseq
  10x-compatible primer scheme, so primer_F/primer_S sequences would not be present
  at read termini.
- Reads are raw subreads rather than CCS consensus reads — FLNC classification relies
  on high-accuracy CCS sequence.
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: scISA-Tools
  role: Provides classify_by_primer.pl, which classifies CCS reads by primer orientation,
    extracts cell barcode/UMI, and trims the polyA tail to yield FLNC reads
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: samtools
  role: Converts the CCS BAM into ccs.fa and ccs.fq via samtools view piped through
    awk prior to BLAST and classification
- name: ncbi-blast+
  role: makeblastdb builds the nucleotide primer database and blastn (-outfmt 7, -word_size
    5) maps primers to CCS reads to produce mapped.m7
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:68a6eb4768fed6a27c6eb8f0b1fbd1eb65e17abea367b3a59ca4357bf771bc38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - ncbi-blast+
  - samtools
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# polya-tail-trimming

## Summary

Detect and extract full-length non-concatemer (FLNC) reads from HIT-scISOseq CCS reads by BLAST-mapping 5'/3' primer sequences and trimming poly(A) tails, cell barcodes, and UMIs with scISA-Tools' classify_by_primer.

## When to use

Use when PacBio CCS reads (BAM/FASTQ) from HIT-scISOseq or 10x-compatible scIso-Seq libraries must be classified as full-length by primer orientation and have polyA tails, barcodes, and UMIs trimmed before genome mapping.

## When NOT to use

- Input reads are already FLNC transcript FASTQs that have been primer-, barcode-, UMI-, and polyA-trimmed (proceed directly to minimap2 splice alignment).
- The library was not built with the PacBio original scIso-Seq or BGI HIT-scISOseq 10x-compatible primer scheme, so primer_F/primer_S sequences would not be present at read termini.
- Reads are raw subreads rather than CCS consensus reads — FLNC classification relies on high-accuracy CCS sequence.

## Inputs

- PacBio CCS reads in BAM format (e.g. ccs.bam from SMRT-Link with --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75)
- CCS FASTA (ccs.fa) and FASTQ (ccs.fq) derived from the BAM via samtools view + awk
- Primer FASTA (primer.fa) with entries named primer_F (5' primer) and primer_S (3' primer)
- BLAST tabular output (mapped.m7, -outfmt 7) from blastn of ccs.fa against the primer database

## Outputs

- isoseq_flnc.BarcodeUMI.fastq (extracted cell barcodes and UMIs per FLNC read)
- isoseq_flnc.Transcript.fastq (polyA-trimmed FLNC transcript sequences)
- isoseq.PrimerStat.csv (per-read primer classification statistics)

## How to apply

Convert the CCS BAM to ccs.fa and ccs.fq using samtools view piped through awk, then build a nucleotide BLAST database from primer.fa containing primer_F (AAGCAGTGGTATCAACGCAGAGTACATGGG, the 5' primer) and primer_S (CTACACGACGCTCTTCCGATCT, the 3' primer); names must be exactly primer_F/primer_S for classify_by_primer to recognize orientation. Align CCS reads to the primer database with blastn -word_size 5 in -outfmt 7 format, then run classify_by_primer.pl with -min_primerlen 16 (at least 16 bp of primer alignment required) and -min_isolen 50 (minimum transcript length excluding the polyA tail). A read is called FLNC only when 5' and 3' primer sequences are present at the two terminals in opposite orientations (5p + 3p- or 3p + 5p-). Successful runs yield three outputs: isoseq_flnc.BarcodeUMI.fastq, isoseq_flnc.Transcript.fastq, and isoseq.PrimerStat.csv; the transcript FASTQ is then ready for minimap2 splice alignment.

## Related tools

- **scISA-Tools** (Provides classify_by_primer.pl, which classifies CCS reads by primer orientation, extracts cell barcode/UMI, and trims the polyA tail to yield FLNC reads) — https://github.com/shizhuoxing/scISA-Tools
- **samtools** (Converts the CCS BAM into ccs.fa and ccs.fq via samtools view piped through awk prior to BLAST and classification)
- **ncbi-blast+** (makeblastdb builds the nucleotide primer database and blastn (-outfmt 7, -word_size 5) maps primers to CCS reads to produce mapped.m7)

## Examples

```
perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./
```

## Evaluation signals

- All three output files (isoseq_flnc.BarcodeUMI.fastq, isoseq_flnc.Transcript.fastq, isoseq.PrimerStat.csv) exist in the specified -outdir.
- Every FLNC read in isoseq_flnc.Transcript.fastq has at least 16 bp of primer alignment (-min_primerlen 16) at both termini in opposite orientations (5p + 3p- or 3p + 5p-).
- Retained transcripts are at least 50 bp after polyA tail removal (-min_isolen 50).
- Terminal primer names in primer.fa are exactly primer_F and primer_S; misnamed entries cause classify_by_primer to misclassify orientation.
- mapped.m7 is valid BLAST -outfmt 7 tabular output and non-empty for libraries expected to contain full-length molecules.

## Limitations

- Only cDNA fragments with 5' and 3' primer sequences at both terminals in opposite orientations are regarded as FLNC; reads lacking one primer are discarded.
- Primer names must follow the exact primer_F/primer_S format required by classify_by_primer, restricting use to the specified PacBio scIso-Seq / HIT-scISOseq primer scheme.
- The example.ccs.bam test set contains only 5000 CCS reads, so validation on full-scale data (e.g. >10 million reads per SMRT Cell 8M) should confirm scaling behavior separately.

## Evidence

- [methods] The FLNC detection filter regards only cDNA fragments with 5′ and 3′ primer sequences at the two terminals in opposite orientations (5p + 3p- and 3p + 5p-) as FLNC reads.: [evidence span withheld — anchor status: not_found]
- [methods] perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./: "perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./"
- [readme] Here primer_F means 5' primer, primer_S means 3' primer, you must set the name in this format for classify_by_primer utility.: "here primer_F means 5’ primer, primer_S means 3’ primer, you must set the name in this format for `classify_by_primer` utility"
- [readme] After running finished, will find three file in your specified output path: isoseq_flnc.BarcodeUMI.fastq, isoseq_flnc.Transcript.fastq, isoseq.PrimerStat.csv: "will find three file in your specified output path: `isoseq_flnc.BarcodeUMI.fastq`, `isoseq_flnc.Transcript.fastq`, `isoseq.PrimerStat.csv`"
- [discussion] the classify_by_primer utility was employed to extract cell barcodes and UMIs: "the classify_by_primer utility was employed to extract cell barcodes and UMIs"
