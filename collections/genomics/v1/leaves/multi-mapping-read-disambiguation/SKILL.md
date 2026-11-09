---
name: multi-mapping-read-disambiguation
description: Use when nanopore DRS reads aligned with minimap2 map to multiple transcript
  isoforms and you need probabilistic EM-based multi-mapping disambiguation for transcript-level
  abundance quantification.
when_to_use_negative:
- Reads were aligned to a whole genome rather than a transcriptome with retained secondary
  mappings — the EM compatibility index requires multi-mapping read-to-transcript
  alignments
- The goal is novel isoform discovery/annotation rather than abundance estimation
  of annotated transcripts (pair NanoCount with FLAIR for discovery of novel isoforms)
- Data is short-read RNA-seq or already has a resolved count matrix — use Salmon/tximport-style
  downstream tools instead of DRS-specific filtering
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NanoCount
  role: Performs DRS-specific filtering of alignments and runs the expectation-maximisation
    loop to quantify transcript abundances from multi-mapping reads
  repo: https://github.com/a-slide/NanoCount
- name: minimap2
  role: Aligns direct-RNA reads to the reference transcriptome with map-ont preset
    and -p 0 -N 100 to retain all secondary mappings needed by the EM step
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Converts minimap2 SAM output to binary BAM (samtools view -bh) for filtering
    and EM input
- name: nanopore_transcript_abundance.py
  role: Jared Simpson's EM-based predecessor script that inspired NanoCount's abundance
    estimation approach
  repo: https://github.com/jts/nanopore-rna-analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:6969f158cf8edc41cd74bfc6a16f926bf85db5cffde994ca1ecf8b276e33743a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-slide/NanoCount
  tools:
  - NanoCount
  - minimap2
  - nanopore_transcript_abundance.py
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# multi-mapping-read-disambiguation

## Summary

Resolves reads that map to multiple transcript isoforms in Oxford Nanopore direct RNA sequencing (DRS) by applying DRS-specific alignment filters and an expectation-maximisation algorithm (NanoCount) to estimate per-transcript abundances.

## When to use

Use when nanopore DRS reads aligned with minimap2 map to multiple transcript isoforms and you need probabilistic EM-based multi-mapping disambiguation for transcript-level abundance quantification.

## When NOT to use

- Reads were aligned to a whole genome rather than a transcriptome with retained secondary mappings — the EM compatibility index requires multi-mapping read-to-transcript alignments
- The goal is novel isoform discovery/annotation rather than abundance estimation of annotated transcripts (pair NanoCount with FLAIR for discovery of novel isoforms)
- Data is short-read RNA-seq or already has a resolved count matrix — use Salmon/tximport-style downstream tools instead of DRS-specific filtering

## Inputs

- Oxford Nanopore direct-RNA FASTQ reads (e.g. SIRV mix E0/E2 SRR6058583 from ENA)
- Reference transcriptome FASTA (e.g. SIRV transcriptome or GENCODE v31)
- minimap2 alignments in SAM/BAM format retaining up to 100 secondary mappings per read (-p 0 -N 100)

## Outputs

- Per-transcript abundance table (e.g. sirv_transcript_counts.tsv) with transcript_name, raw, est_count and tpm columns
- Raw abundance values that sum to 1 across transcripts
- EM-resolved est_count per transcript (raw abundance × number of primary alignments)

## How to apply

First align direct-RNA FASTQ reads to the reference transcriptome with minimap2 using `-ax map-ont` plus `-p 0 -N 100`, so that up to 100 secondary mappings per read are retained and piped through `samtools view -bh` into a BAM; retaining secondary alignments is essential because the EM step needs the full read–transcript compatibility set. Then apply NanoCount's DRS-specific filtering: discard alignments with aligned fraction <0.5, alignment length <50 nt, negative-strand orientation, or 3' ends more than 50 nt from the transcript 3' end, and discard secondary alignments with an AS score below 95% of the best alignment's AS. Build the read–transcript compatibility index from surviving alignments and run the expectation-maximisation loop to convergence to probabilistically distribute multi-mapping reads across isoforms. Emit per-transcript raw abundance (summing to 1), est_count (raw abundance × number of primary alignments) and tpm (raw abundance × 1M).

## Related tools

- **NanoCount** (Performs DRS-specific filtering of alignments and runs the expectation-maximisation loop to quantify transcript abundances from multi-mapping reads) — https://github.com/a-slide/NanoCount
- **minimap2** (Aligns direct-RNA reads to the reference transcriptome with map-ont preset and -p 0 -N 100 to retain all secondary mappings needed by the EM step) — https://github.com/lh3/minimap2
- **samtools** (Converts minimap2 SAM output to binary BAM (samtools view -bh) for filtering and EM input)
- **nanopore_transcript_abundance.py** (Jared Simpson's EM-based predecessor script that inspired NanoCount's abundance estimation approach) — https://github.com/jts/nanopore-rna-analysis

## Examples

```
minimap2 -ax map-ont -p 0 -N 100 transcriptome.fasta reads.fastq | samtools view -bh > aligned_reads.bam
```

## Evaluation signals

- Raw abundance values in the output TSV sum to 1 across all transcripts
- tpm column equals raw abundance × 1,000,000 for every transcript
- est_count equals raw abundance multiplied by the number of primary alignments for each read
- No surviving secondary alignment has an AS score below 95% of the best alignment's AS for its read
- Output covers the expected transcript set (e.g. all 69 SIRV transcripts) with transcript_name/raw/est_count/tpm fields present

## Limitations

- Many incomplete DRS reads are largely consigned to 3'UTRs and cannot specify the expressed isoform, so EM cannot fully resolve some reads
- Modest DRS sequencing depth makes it challenging to detect and quantify lowly expressed genes and isoforms; results likely miss DE genes/isoforms that higher depths would reveal
- Filtered-out secondary alignments with AS <95% of the best alignment may represent reads from genuinely unannotated isoforms, not just misalignments

## Evidence

- [results] NanoCount performs DRS specific filtering steps to improve transcript assignment prior to quantifying DRS reads using an expectation-maximisation algorithm: "NanoCount performs DRS specific filtering steps to improve transcript assignment prior to quantifying DRS reads using an expectation-maximisation algorithm"
- [results] 3' end distance filter: "discard read alignments with 3′ ends >50nt away from the transcript 3′ end"
- [results] secondary alignment AS filter: "and secondary alignments with an AS <95% that of the best alignment (highest AS), as these likely represent misalignments"
- [discussion] filtering rationale: "NanoCount performs a DRS specific filtering step that eliminates many incorrect alignments prior to transcript quantification"
- [readme] EM purpose in NanoCount: "using an expectation-maximization approach like RSEM, Kallisto, salmon, etc to handle the uncertainty of multi-mapping reads"
- [readme] predecessor tool requirement for secondary mappings: [evidence span withheld — anchor status: external_doc]
