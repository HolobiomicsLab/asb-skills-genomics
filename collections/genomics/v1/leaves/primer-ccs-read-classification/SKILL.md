---
name: primer-ccs-read-classification
description: Use when starting from a PacBio CCS BAM produced by HIT-scISOseq or 10x-coupled
  PacBio scIsoSeq libraries and you need to identify FLNC reads and extract CellBC/UMI/polyA-trimmed
  transcript sequences via primer orientation.
when_to_use_negative:
- Input is raw subreads BAM rather than CCS reads — run the ccs step first (SMRTlink
  8.0+)
- Libraries whose primer sequences differ from the PacBio scIsoSeq / HIT-scISOseq
  10x primer set, since primer_F/primer_S sequences and orientation semantics are
  hard-coded to that convention
- Reads already classified as FLNC by PacBio official isoseq3 software — the cDNA_Cupcake
  README notes many functions now have official isoseq equivalents
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: scISA-Tools
  role: Provides classify_by_primer.pl, which classifies CCS reads by primer orientation
    and outputs FLNC BarcodeUMI, Transcript FASTQs, and PrimerStat CSV
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: samtools
  role: Converts the CCS BAM to ccs.fa and ccs.fq via samtools view piped through
    awk before BLAST and classification
- name: ncbi-blast+
  role: makeblastdb builds the nucleotide primer database and blastn (-outfmt 7 -word_size
    5) maps CCS reads to primers to produce mapped.m7
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:a39054e8f44a5fc77ec07692145d34cb3cbf29666a6db90f2e2d65ee25edfa9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - classify_by_primer
  - ncbi-blast+
  - samtools
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# primer-ccs-read-classification

## Summary

Classifies PacBio CCS reads as full-length non-chimeric (FLNC) by BLAST-mapping them against 5' and 3' primer sequences and running scISA-Tools' classify_by_primer, which extracts cell barcodes, UMIs, and polyA-trimmed transcripts. It is the FLNC-detection step of the HIT-scISOseq/scISA-Tools pipeline.

## When to use

Use when starting from a PacBio CCS BAM produced by HIT-scISOseq or 10x-coupled PacBio scIsoSeq libraries and you need to identify FLNC reads and extract CellBC/UMI/polyA-trimmed transcript sequences via primer orientation.

## When NOT to use

- Input is raw subreads BAM rather than CCS reads — run the ccs step first (SMRTlink 8.0+)
- Libraries whose primer sequences differ from the PacBio scIsoSeq / HIT-scISOseq 10x primer set, since primer_F/primer_S sequences and orientation semantics are hard-coded to that convention
- Reads already classified as FLNC by PacBio official isoseq3 software — the cDNA_Cupcake README notes many functions now have official isoseq equivalents

## Inputs

- PacBio CCS BAM file (e.g. ccs.bam generated with --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75)
- Primer FASTA (primer.fa) with entries primer_F (5' primer) and primer_S (3' primer)

## Outputs

- isoseq_flnc.BarcodeUMI.fastq (cell barcode and UMI sequences per FLNC read)
- isoseq_flnc.Transcript.fastq (polyA-trimmed FLNC transcript sequences)
- isoseq.PrimerStat.csv (per-read primer classification statistics)
- mapped.m7 (blastn -outfmt 7 primer alignment intermediate)

## How to apply

Convert the CCS BAM to FASTA and FASTQ with samtools view piped through awk to produce ccs.fa and ccs.fq. Build a nucleotide BLAST database from primer.fa, which must contain exactly two entries named primer_F (5' primer, AAGCAGTGGTATCAACGCAGAGTACATGGG) and primer_S (3' primer, CTACACGACGCTCTTCCGATCT), using makeblastdb -dbtype nucl. Align CCS reads against the primer database with blastn -outfmt 7 -word_size 5 to obtain mapped.m7, then run classify_by_primer.pl with -min_primerlen 16 (minimum primer alignment length) and -min_isolen 50 (minimum transcript length excluding the polyA tail). The classifier treats only reads with 5'/3' primer hits at the two terminals in opposite orientations (5p + 3p- or 3p + 5p-) as FLNC; the naming of the primer FASTA entries in this exact format is mandatory for the utility to interpret orientation. Success is judged by production of the three output files with FLNC transcripts meeting both thresholds.

## Related tools

- **scISA-Tools** (Provides classify_by_primer.pl, which classifies CCS reads by primer orientation and outputs FLNC BarcodeUMI, Transcript FASTQs, and PrimerStat CSV) — https://github.com/shizhuoxing/scISA-Tools
- **samtools** (Converts the CCS BAM to ccs.fa and ccs.fq via samtools view piped through awk before BLAST and classification)
- **ncbi-blast+** (makeblastdb builds the nucleotide primer database and blastn (-outfmt 7 -word_size 5) maps CCS reads to primers to produce mapped.m7)

## Examples

```
makeblastdb -in primer.fa -dbtype nucl; blastn -query ccs.fa -db primer.fa -outfmt 7 -word_size 5 > mapped.m7; perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./
```

## Evaluation signals

- All three output files (isoseq_flnc.BarcodeUMI.fastq, isoseq_flnc.Transcript.fastq, isoseq.PrimerStat.csv) exist in the -outdir after the run
- Every FLNC read in isoseq_flnc.Transcript.fastq has primer alignments of at least 16 bp (-min_primerlen 16)
- Every retained transcript is at least 50 bp excluding the polyA tail (-min_isolen 50)
- Each classified FLNC read shows 5' and 3' primer hits at the two terminals in opposite orientations (5p + 3p- or 3p + 5p-)
- The classify_by_primer.pl -h help text runs and confirms the required -blastm7, -ccsfq, -min_primerlen, -min_isolen, -outdir arguments were supplied

## Limitations

- Primer FASTA headers must be exactly primer_F (5' primer) and primer_S (3' primer); other naming breaks classify_by_primer orientation logic
- The README example covers the PacBio original scIsoSeq protocol with 10x Genomics and the BGI patented HIT-scISOseq protocol; other library designs with different primers are not addressed
- Parameter naming differs across the pipeline: the article's discussion cites '-min_primerlen 16 -min_seqlen 50' while the README invocation uses '-min_primerlen 16 -min_isolen 50'

## Evidence

- [discussion] FLNC read filters: min_primerlen 16, min_seqlen 50: [evidence span withheld — anchor status: not_found]
- [methods] classify_by_primer invocation with thresholds: "perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./"
- [readme] Primer naming requirement from README: "here primer_F means 5’ primer, primer_S means 3’ primer, you must set the name in this format for `classify_by_primer` utility."
- [readme] Primer sequences used for the BLAST database: ">primer_F
AAGCAGTGGTATCAACGCAGAGTACATGGG
>primer_S
CTACACGACGCTCTTCCGATCT"
- [readme] Three expected output files: "After running finished, will find three file in your specified output path: `isoseq_flnc.BarcodeUMI.fastq`, `isoseq_flnc.Transcript.fastq`, `isoseq.PrimerStat.csv`"
