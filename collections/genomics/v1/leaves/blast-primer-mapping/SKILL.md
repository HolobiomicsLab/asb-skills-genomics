---
name: blast-primer-mapping
description: Use when input is a PacBio CCS BAM from HIT-scISOseq or 10x-based scIso-Seq
  libraries and you must identify FLNC reads, demultiplex cell barcodes/UMIs, and
  trim poly(A) tails before genome mapping.
when_to_use_negative:
- Input is raw subreads BAM rather than CCS reads — generate CCS first (ccs with min-rq
  0.75) before primer classification
- Library construction does not use the PacBio original scIso-Seq or BGI HIT-scISOseq
  10x-compatible primers, so primer_F/primer_S sequences would not match the reads
- You only need gene-level expression matrices and already have FLNC FASTQs — skip
  to downstream cellBC/UMI correction and scGene_matrix steps
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: scISA-Tools
  role: Provides the classify_by_primer.pl utility that detects FLNC reads from primer
    blast hits and extracts cell barcode, UMI, and polyA-trimmed transcript
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: samtools
  role: Converts the CCS BAM to ccs.fa and ccs.fq via samtools view piped through
    awk before primer blasting
- name: ncbi-blast+
  role: makeblastdb builds the nucleotide primer database and blastn (-outfmt 7 -word_size
    5) maps CCS reads to primers, producing mapped.m7
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:b53b3495e73c08303c371d04b28fa01656a79a8186dbde423632e6acfda42078
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - classify_by_primer
  - ncbi-blast+
  - samtools
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# blast-primer-mapping

## Summary

Classifies PacBio CCS reads from HIT-scISOseq/10x scIso-Seq libraries as full-length non-concatemer (FLNC) reads by mapping 5' and 3' primer sequences with blastn and running scISA-Tools' classify_by_primer, extracting cell barcodes, UMIs, and trimmed transcripts.

## When to use

Use when input is a PacBio CCS BAM from HIT-scISOseq or 10x-based scIso-Seq libraries and you must identify FLNC reads, demultiplex cell barcodes/UMIs, and trim poly(A) tails before genome mapping.

## When NOT to use

- Input is raw subreads BAM rather than CCS reads — generate CCS first (ccs with min-rq 0.75) before primer classification
- Library construction does not use the PacBio original scIso-Seq or BGI HIT-scISOseq 10x-compatible primers, so primer_F/primer_S sequences would not match the reads
- You only need gene-level expression matrices and already have FLNC FASTQs — skip to downstream cellBC/UMI correction and scGene_matrix steps

## Inputs

- PacBio CCS BAM (e.g. ccs.bam generated with --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75)
- Primer FASTA with records named primer_F (5') and primer_S (3')

## Outputs

- isoseq_flnc.BarcodeUMI.fastq (cell barcode and UMI per FLNC read)
- isoseq_flnc.Transcript.fastq (primer- and polyA-trimmed FLNC transcript sequences)
- isoseq.PrimerStat.csv (per-read primer classification statistics)
- mapped.m7 (blastn -outfmt 7 primer alignment table)

## How to apply

Convert the CCS BAM to FASTA and FASTQ with samtools view piped through awk, producing ccs.fa and ccs.fq. Build a nucleotide BLAST database from a primer FASTA containing primer_F (AAGCAGTGGTATCAACGCAGAGTACATGGG, the 5' primer) and primer_S (CTACACGACGCTCTTCCGATCT, the 3' primer), then align CCS reads to it with blastn -outfmt 7 -word_size 5 to obtain mapped.m7. The 5'/3' primer naming convention is mandatory for the classify_by_primer utility to interpret orientation. Run classify_by_primer.pl with -min_primerlen 16 (minimum primer alignment length in ccs.fa) and -min_isolen 50 (minimum transcript length excluding the polyA tail); the FLNC filter keeps only reads with 5' and 3' primer hits at both terminals in opposite orientations (5p + 3p- or 3p + 5p-). Collect the three outputs in the specified -outdir.

## Related tools

- **scISA-Tools** (Provides the classify_by_primer.pl utility that detects FLNC reads from primer blast hits and extracts cell barcode, UMI, and polyA-trimmed transcript) — https://github.com/shizhuoxing/scISA-Tools
- **samtools** (Converts the CCS BAM to ccs.fa and ccs.fq via samtools view piped through awk before primer blasting)
- **ncbi-blast+** (makeblastdb builds the nucleotide primer database and blastn (-outfmt 7 -word_size 5) maps CCS reads to primers, producing mapped.m7)

## Examples

```
perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./
```

## Evaluation signals

- All three expected output files (isoseq_flnc.BarcodeUMI.fastq, isoseq_flnc.Transcript.fastq, isoseq.PrimerStat.csv) are present in the specified -outdir
- Every retained FLNC read in isoseq_flnc.Transcript.fastq has ≥16 bp of primer alignment (min_primerlen 16) at both ends
- Every FLNC transcript is at least 50 bp excluding the poly(A) tail (min_isolen 50)
- Primer names in the BLAST database follow the mandatory primer_F (5') / primer_S (3') naming format, otherwise classify_by_primer misclassifies orientation
- mapped.m7 is in blastn -outfmt 7 tabular format generated with -word_size 5

## Limitations

- Primer sequences are specific to the PacBio original scIso-Seq protocol with 10x Genomics and the BGI patented HIT-scISOseq protocol; other library schemes need different primers and names
- classify_by_primer requires the exact primer_F/primer_S record naming convention; alternative names break classification
- Min primer alignment of 16 bp and min isoform length of 50 bp may exclude short or partially primed cDNA fragments by design

## Evidence

- [discussion] FLNC read generation via primer mapping: "the classify_by_primer utility was employed to extract cell barcodes and UMIs"
- [discussion] FLNC filter thresholds: [evidence span withheld — anchor status: not_found]
- [methods] classify_by_primer command: "perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./"
- [readme] primer naming convention: "here primer_F means 5’ primer, primer_S means 3’ primer, you must set the name in this format for `classify_by_primer` utility."
- [readme] blastn settings: "blastn -query ccs.fa -db primer.fa -outfmt 7 -word_size 5 > mapped.m7"
