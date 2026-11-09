---
name: tpm-abundance-normalization
description: Use when nanopore DRS FASTQ reads must be quantified against a known
  transcriptome (e.g. SIRV spike-ins or GENCODE), reads map to multiple isoforms,
  and per-transcript raw/est_count/tpm abundances are the required output.
when_to_use_negative:
- Input is short-read RNA-seq — use a short-read-aware quantifier (Salmon, kallisto)
  instead; NanoCount is developed specifically for DRS
- Reads are already quantified or you already hold a per-transcript count/TPM feature
  table — re-running EM would be redundant
- The goal is discovering novel unannotated isoforms from genome-spliced alignments
  rather than quantifying annotated transcriptome references
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NanoCount
  role: Performs DRS-specific alignment filtering and expectation-maximisation quantification
    of multi-mapping reads into raw/est_count/tpm abundances
  repo: https://github.com/a-slide/NanoCount
- name: minimap2
  role: Aligns DRS reads to the transcriptome with -ax map-ont -p 0 to retain secondary
    mappings required by the EM step
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Converts minimap2 SAM output to BAM (samtools view -bh) feeding NanoCount
- name: nanopore-rna-analysis
  role: Jared Simpson's nanopore_transcript_abundance.py, the EM-transcript-quantification
    approach that inspired NanoCount
  repo: https://github.com/jts/nanopore-rna-analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:0cd69d465aea484779fd0f5262e73ed4951faba904d8ef50a684824bf8799722
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-slide/NanoCount
  tools:
  - NanoCount
  - minimap2
  - nanopore-rna-analysis
  - samtools
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tpm-abundance-normalization

## Summary

Estimate transcript abundances from Oxford Nanopore direct-RNA sequencing (DRS) alignments using NanoCount's DRS-specific filtering and expectation-maximisation (EM) algorithm, which resolves multi-mapping reads and reports per-transcript raw abundance, est_count and tpm.

## When to use

Use when nanopore DRS FASTQ reads must be quantified against a known transcriptome (e.g. SIRV spike-ins or GENCODE), reads map to multiple isoforms, and per-transcript raw/est_count/tpm abundances are the required output.

## When NOT to use

- Input is short-read RNA-seq — use a short-read-aware quantifier (Salmon, kallisto) instead; NanoCount is developed specifically for DRS
- Reads are already quantified or you already hold a per-transcript count/TPM feature table — re-running EM would be redundant
- The goal is discovering novel unannotated isoforms from genome-spliced alignments rather than quantifying annotated transcriptome references

## Inputs

- Oxford Nanopore direct-RNA sequencing FASTQ reads (e.g. SIRV mix E0/E2 reads for ENA accession SRR6058583)
- Reference transcriptome FASTA (e.g. SIRV transcriptome or GENCODE v31)
- BAM of transcriptome alignments (minimap2 -ax map-ont -p 0, secondary alignments retained)

## Outputs

- Per-transcript abundance table (e.g. sirv_transcript_counts.tsv) with transcript_name, raw (summing to 1), est_count (raw x number of primary alignments) and tpm (raw x 1M) columns

## How to apply

Align DRS FASTQ reads to the transcriptome reference with minimap2 using -ax map-ont -p 0 -N 100 (retaining up to 100 secondary mappings, needed because EM handles multi-mappers) and pipe through samtools view -bh to get a BAM. Apply NanoCount's DRS-specific filters to improve transcript assignment: discard alignments with aligned fraction <0.5, alignment length <50 nt, negative-strand orientation, or 3' ends >50 nt from the transcript 3' end, and discard secondary alignments with AS <95% of the best (highest AS) alignment since these likely represent misalignments or reads from unannotated isoforms. Build the read-transcript compatibility index from surviving alignments and run the EM loop (as in RSEM/kallisto/salmon) to convergence to distribute multi-mapping reads. Emit per-transcript raw abundance (summing to 1), est_count (raw abundance x number of primary alignments) and tpm (raw abundance x 1M).

## Related tools

- **NanoCount** (Performs DRS-specific alignment filtering and expectation-maximisation quantification of multi-mapping reads into raw/est_count/tpm abundances) — https://github.com/a-slide/NanoCount
- **minimap2** (Aligns DRS reads to the transcriptome with -ax map-ont -p 0 to retain secondary mappings required by the EM step) — https://github.com/lh3/minimap2
- **samtools** (Converts minimap2 SAM output to BAM (samtools view -bh) feeding NanoCount)
- **nanopore-rna-analysis** (Jared Simpson's nanopore_transcript_abundance.py, the EM-transcript-quantification approach that inspired NanoCount) — https://github.com/jts/nanopore-rna-analysis

## Examples

```
minimap2 -ax map-ont -p 0 -N 100 transcriptome.fasta reads.fastq | samtools view -bh > aligned_reads.bam; nanocount -i aligned_reads.bam -o sirv_transcript_counts.tsv
```

## Evaluation signals

- Output TSV contains transcript_name/raw/est_count/tpm fields
- Raw abundance values sum to 1 across transcripts
- tpm values equal raw abundance x 1,000,000 and est_count equals raw abundance x number of primary alignments
- Table covers all expected reference transcripts (69 SIRV transcripts in the validation run)
- BAM fed to NanoCount retains secondary alignments (minimap2 run with -p 0 / -N 100), otherwise multi-mapping reads cannot be resolved

## Limitations

- Secondary alignments with AS <95% of the best are discarded and may include reads from genuinely unannotated isoforms, not only misalignments
- Modest DRS sequencing depth makes it challenging to detect and quantify lowly expressed genes and isoforms; DE results likely miss genes/isoforms revealed at higher depth
- Many incomplete reads are largely consigned to 3'UTRs and cannot specify the expressed isoform; extrapolating validation from spike-ins needs caution given the lower complexity of the sequin/SIRV transcriptome vs human

## Evidence

- [results] NanoCount performs DRS specific filtering steps to improve transcript assignment prior to quantifying DRS reads using an expectation-maximisation algorithm: "NanoCount performs DRS specific filtering steps to improve transcript assignment prior to quantifying DRS reads using an expectation-maximisation algorithm"
- [results] discard read alignments with 3' ends >50nt away from the transcript 3' end: "discard read alignments with 3′ ends >50nt away from the transcript 3′ end"
- [results] secondary alignments with an AS <95% that of the best alignment (highest AS), as these likely represent misalignments or reads from unannotated isoforms: "and secondary alignments with an AS <95% that of the best alignment (highest AS), as these likely represent misalignments or reads from unannotated isoforms"
- [readme] NanoCount estimates transcripts abundance from Oxford Nanopore direct-RNA sequencing datasets, using an expectation-maximization approach like RSEM, Kallisto, salmon, etc: "NanoCount estimates transcripts abundance from Oxford Nanopore *direct-RNA sequencing* datasets, using an expectation-maximization approach like RSEM, Kallisto, salmon, etc"
- [readme] The reads must be mapped to the transcript set using minimap2 with the -p0 flag to retain all secondary mappings: [evidence span withheld — anchor status: external_doc]
