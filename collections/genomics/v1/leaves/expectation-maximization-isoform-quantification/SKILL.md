---
name: expectation-maximization-isoform-quantification
description: Use when nanopore direct-RNA or cDNA FASTQ reads have been aligned to
  a transcriptome with minimap2 retaining secondary alignments (-p 0) and per-transcript
  raw abundance, estimated counts, and TPM values are needed despite multi-mapping
  uncertainty.
when_to_use_negative:
- Reads are short Illumina reads or already assigned to transcripts — other quantifiers
  or simple counting suffice
- Only genomic (splice-aware) alignment is needed and transcript-level abundances
  are not required
- The transcriptome annotation is unknown/unannotated such that AS-score filtering
  against annotated isoforms would discard genuine unannotated isoform reads — pair
  with FLAIR-based novel isoform discovery instead
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: NanoCount
  role: Performs DRS-specific alignment filtering and EM-based transcript abundance
    estimation
  repo: https://github.com/a-slide/NanoCount
- name: minimap2
  role: Aligns DRS FASTQ reads to the transcriptome with map-ont preset and -p 0 to
    retain secondary mappings
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Converts minimap2 SAM output to BAM (samtools view -bh)
- name: nanopore-rna-analysis
  role: Inspiration for the EM approach; nanopore_transcript_abundance.py estimates
    abundance from PAF mappings
  repo: https://github.com/jts/nanopore-rna-analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:ceac4ddf05c108df49cd60f2cb7bb5884d3821976ea380790d6e7cea52d3a542
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-slide/NanoCount
  tools:
  - NanoCount
  - minimap2
  - nanopore-rna-analysis
  - samtools
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# expectation-maximization-isoform-quantification

## Summary

Quantify transcript isoform abundances from Oxford Nanopore direct RNA sequencing (DRS) reads using NanoCount, which applies DRS-specific alignment filtering followed by an expectation-maximisation (EM) algorithm to resolve multi-mapping reads, analogous to RSEM/Kallisto/Salmon.

## When to use

Use when nanopore direct-RNA or cDNA FASTQ reads have been aligned to a transcriptome with minimap2 retaining secondary alignments (-p 0) and per-transcript raw abundance, estimated counts, and TPM values are needed despite multi-mapping uncertainty.

## When NOT to use

- Reads are short Illumina reads or already assigned to transcripts — other quantifiers or simple counting suffice
- Only genomic (splice-aware) alignment is needed and transcript-level abundances are not required
- The transcriptome annotation is unknown/unannotated such that AS-score filtering against annotated isoforms would discard genuine unannotated isoform reads — pair with FLAIR-based novel isoform discovery instead

## Inputs

- FASTQ reads from Oxford Nanopore direct RNA sequencing (e.g. SIRV mix E0/E2 reads for SRR6058583 from ENA)
- Reference transcriptome FASTA (e.g. SIRV transcripts or GENCODE v31)
- minimap2 alignment BAM/PAF retaining secondary mappings (mapped with -ax map-ont -p 0 -N 100)

## Outputs

- sirv_transcript_counts.tsv with transcript_name, raw, est_count, and tpm columns for each transcript (e.g. the 69 SIRV transcripts)
- Raw abundance values per transcript summing to 1
- Optionally a filtered BAM of alignments surviving NanoCount filtering

## How to apply

Align reads to the transcriptome reference with minimap2 using -ax map-ont -p 0 -N 100 so that up to 100 secondary mappings per read are retained, piping through samtools view -bh to a BAM. Apply NanoCount's DRS-specific filtering: discard alignments with aligned fraction <0.5, alignment length <50 nt, negative-strand orientation, or 3' ends more than 50 nt from the transcript 3' end, and drop secondary alignments whose AS score is below 95% of the best alignment, since these likely represent misalignments or reads from unannotated isoforms. Build the read–transcript compatibility index from surviving alignments and run the EM loop to convergence to probabilistically assign multi-mapping reads. Emit per-transcript raw abundance (summing to 1), est_count (raw abundance × number of primary alignments), and tpm (raw abundance × 1M). The rationale is that DRS reads are 3'-end anchored and often truncated, so naive best-alignment assignment misattributes reads from similar isoforms.

## Related tools

- **NanoCount** (Performs DRS-specific alignment filtering and EM-based transcript abundance estimation) — https://github.com/a-slide/NanoCount
- **minimap2** (Aligns DRS FASTQ reads to the transcriptome with map-ont preset and -p 0 to retain secondary mappings) — https://github.com/lh3/minimap2
- **samtools** (Converts minimap2 SAM output to BAM (samtools view -bh))
- **nanopore-rna-analysis** (Inspiration for the EM approach; nanopore_transcript_abundance.py estimates abundance from PAF mappings) — https://github.com/jts/nanopore-rna-analysis

## Examples

```
minimap2 -ax map-ont -p 0 -N 100 sirv_transcriptome.fa SRR6058583.fastq | samtools view -bh > aligned_reads.bam && NanoCount -i aligned_reads.bam -o sirv_transcript_counts.tsv
```

## Evaluation signals

- Output TSV contains transcript_name, raw, est_count, and tpm columns
- Raw abundance values across all transcripts sum to exactly 1
- The table covers the expected transcript set (e.g. all 69 SIRV transcripts)
- Secondary alignments retained in the input BAM must have AS ≥ 95% of the best-alignment AS after filtering; alignment coverage fraction ≥ 0.5 and length ≥ 50 nt
- Quantification of synthetic spike-ins (SIRV/sequins) shows close agreement between known and measured fold changes

## Limitations

- Modest DRS sequencing depth limits detection and quantification of lowly expressed genes and isoforms; DE results likely miss isoforms revealed at higher depth
- Many incomplete reads are largely consigned to 3'UTRs and cannot specify the expressed isoform
- Caution extrapolating spike-in validation results due to the lower complexity of the sequin/SIRV transcriptome compared to the human one
- EM can only distribute multi-mapping reads among annotated transcripts; unannotated isoforms are filtered out as likely misalignments

## Evidence

- [results] NanoCount performs DRS specific filtering steps to improve transcript assignment prior to quantifying DRS reads using an expectation-maximisation algorithm: "NanoCount performs DRS specific filtering steps to improve transcript assignment prior to quantifying DRS reads using an expectation-maximisation algorithm"
- [results] 3' end filter: "discard read alignments with 3′ ends >50nt away from the transcript 3′ end"
- [results] AS filter: "and secondary alignments with an AS <95% that of the best alignment (highest AS), as these likely represent misalignments or reads from unannotated isoforms"
- [readme] EM approach: "NanoCount estimates transcripts abundance from Oxford Nanopore *direct-RNA sequencing* datasets, using an expectation-maximization approach like RSEM, Kallisto, salmon, etc"
- [readme] secondary mapping retention: [evidence span withheld — anchor status: external_doc]
