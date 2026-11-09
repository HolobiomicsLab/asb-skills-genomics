---
name: sequencing-summary-table-reproduction
description: Use when a paper reports per-sample sequencing metrics (read counts,
  lengths, FLNC/artifact fractions) and the deposited subreads BAM files plus stated
  processing parameters are available to independently reproduce the table.
when_to_use_negative:
- The paper does not deposit raw subreads BAM files, so CCS generation cannot be re-run
  and only summary statistics are available
- The downstream goal is isoform characterization or quantification (use collapse_isoforms_by_sam.py
  / SQANTI3 workflows instead of table reproduction)
- The sequencing platform is not PacBio CCS-based (e.g. Nanopore PromethION reads,
  where barcode demultiplex efficiency is limited by error rate and the ccs pipeline
  does not apply)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: PacBio ccs (SMRTlink)
  role: Generates circular consensus reads from subreads.bam with the paper's modified
    parameters --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: scISA-Tools (classify_by_primer.pl)
  role: Extracts FLNC reads via primer blast alignment and separates BarcodeUMI vs
    Transcript fastq files using -min_primerlen 16 -min_isolen 50
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: ncbi-blast
  role: makeblastdb on primer.fa and blastn (-word_size 5, -outfmt 7) maps primers
    against CCS reads for FLNC classification
- name: samtools
  role: Converts ccs.bam to ccs.fa and ccs.fq via samtools view piped through awk
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:35037c31024afcafd91d3be4fcc8ec15ac31e07fbeba3281012a25e903ed9eb7
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
  - scISA-Tools (classify_by_primer.pl)
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# sequencing-summary-table-reproduction

## Summary

A benchmarking skill for recomputing the per-sample read-processing metrics reported in a sequencing paper's summary table (e.g. Table 1: CCS read counts, mean CCS read lengths, FLNC counts, artifact cDNA percentages) directly from the deposited raw data using the authors' stated parameters.

## When to use

Use when a paper reports per-sample sequencing metrics (read counts, lengths, FLNC/artifact fractions) and the deposited subreads BAM files plus stated processing parameters are available to independently reproduce the table.

## When NOT to use

- The paper does not deposit raw subreads BAM files, so CCS generation cannot be re-run and only summary statistics are available
- The downstream goal is isoform characterization or quantification (use collapse_isoforms_by_sam.py / SQANTI3 workflows instead of table reproduction)
- The sequencing platform is not PacBio CCS-based (e.g. Nanopore PromethION reads, where barcode demultiplex efficiency is limited by error rate and the ccs pipeline does not apply)

## Inputs

- Deposited PacBio subreads BAM files per sample (HIT-scISOseq and ScISOr-Seq/Linked-scISOseq baseline libraries)
- primer.fa containing primer_F (5' primer, AAGCAGTGGTATCAACGCAGAGTACATGGG) and primer_S (3' primer, CTACACGACGCTCTTCCGATCT)
- Published summary table (Table 1) with per-sample CCS counts/lengths, FLNC counts, and artifact cDNA percentages
- Stated CCS and FLNC processing parameters from the paper's methods

## Outputs

- Recomputed per-sample table: total CCS read counts, mean CCS read length, FLNC read counts, and artifact cDNA percentages
- Per-sample ccs.bam, ccs.fa, ccs.fq, mapped.m7 blast output, and isoseq_flnc.BarcodeUMI.fastq / isoseq_flnc.Transcript.fastq / isoseq.PrimerStat.csv intermediates
- Comparison against reported Table 1 values with pass/fail assessment per row

## How to apply

Retrieve the deposited subreads BAM files for each sample and run PacBio ccs with the paper's exact parameters (--min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75) to generate ccs.bam per sample. Convert ccs.bam to ccs.fa/ccs.fq via samtools view and awk, build a primer BLAST database from primer.fa (primer_F as 5' primer, primer_S as 3' primer) with makeblastdb, and align CCS reads with blastn -word_size 5 -outfmt 7. Run classify_by_primer.pl with -min_primerlen 16 and -min_isolen 50 to extract FLNC reads and separate BarcodeUMI vs Transcript fastq files, then count total CCS reads, compute mean CCS length, count FLNC reads, and derive the artifact cDNA percentage (reads lacking valid cell barcode/primer structure) per sample. Assemble these per-sample values into a table and compare against the published rows; accept reproduction when values fall within the reported ranges (e.g. ~51-53% artifact cDNA for the ScISOr-Seq/Linked-scISOseq baselines vs ~7.58-8.20% for HIT-scISOseq, confirming the biotin-capture TSO-artifact removal).

## Related tools

- **PacBio ccs (SMRTlink)** (Generates circular consensus reads from subreads.bam with the paper's modified parameters --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75) — https://github.com/shizhuoxing/scISA-Tools
- **scISA-Tools (classify_by_primer.pl)** (Extracts FLNC reads via primer blast alignment and separates BarcodeUMI vs Transcript fastq files using -min_primerlen 16 -min_isolen 50) — https://github.com/shizhuoxing/scISA-Tools
- **ncbi-blast** (makeblastdb on primer.fa and blastn (-word_size 5, -outfmt 7) maps primers against CCS reads for FLNC classification)
- **samtools** (Converts ccs.bam to ccs.fa and ccs.fq via samtools view piped through awk)

## Examples

```
ccs *.subreads.bam ccs.bam --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75 -j 20 && samtools view ccs.bam | awk '{print ">"$1"\n"$10}' > ccs.fa && blastn -query ccs.fa -db primer.fa -outfmt 7 -word_size 5 > mapped.m7 && perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./
```

## Evaluation signals

- Recomputed artifact cDNA percentages fall within reported ranges: ~51.31-53.44% for ScISOr-Seq (s1/s2) and 53.42% for Linked-scISOseq vs 7.58% (s1) and 8.20% (s2) for HIT-scISOseq
- Per-sample total CCS read counts and mean CCS read lengths match the corresponding Table 1 rows
- Per-sample FLNC read counts match the Table 1 rows after classify_by_primer with -min_primerlen 16 -min_isolen 50
- isoseq.PrimerStat.csv primer statistics are consistent with expected primer fractions for each library type
- Example sanity check: example.ccs.bam in the repo contains 5000 CCS reads and should pass through the pipeline without classification errors

## Limitations

- Artifact cDNA percentage is derived from reads lacking valid cell barcode/primer structure; slight software-version differences in ccs or blast may shift counts a few percent
- The wet-lab protocol is hosted on an access-restricted private protocols.io URL, so library-construction provenance cannot always be independently verified
- Comparison is limited to the two reported samples (s1/s2) and two baselines; the article notes only two animals (one sample each), no randomization or blinding

## Evidence

- [methods] CCS parameters from methods: [evidence span withheld — anchor status: not_found]
- [methods] classify_by_primer parameters: "perl classify_by_primer.pl -blastm7 mapped.m7 -ccsfq ccs.fq -min_primerlen 16 -min_isolen 50 -outdir ./"
- [intro] TSO artifact proportion: "the 10× Genomics single-cell preparation pipeline of ScISOr-Seq introduces a high proportion (~50%) of undesirable cell-barcode-free reads, mostly template-switching oligonucleotide (TSO) artifacts"
- [readme] README CCS command: "ccs *.subreads.bam ccs.bam --min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75 -j 20"
- [readme] README primer blast step: "blastn -query ccs.fa -db primer.fa -outfmt 7 -word_size 5 > mapped.m7"
