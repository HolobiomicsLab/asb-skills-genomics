---
name: nanopore-drs-transcriptome-alignment
description: Use when input is Oxford Nanopore direct-RNA FASTQ reads plus a transcriptome
  FASTA and the goal is to produce BAM alignments that preserve secondary mappings
  for EM-based isoform quantification (NanoCount, Salmon, LIQA).
when_to_use_negative:
- The goal is spliced whole-genome alignment of DRS reads rather than transcript-level
  assignment — minimap2's splice preset (-ax splice -uf -k14 for noisy Nanopore direct
  RNA) would be the appropriate mode instead of map-ont against a transcriptome.
- Downstream quantification needs only a single best alignment per read — retaining
  -N secondary mappings and -p 0 would inflate the alignment file unnecessarily.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Aligns DRS FASTQ reads to the reference transcriptome using the map-ont preset
    with -a, -p 0 and -N secondary-mapping retention to emit SAM
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Converts minimap2 SAM output to compressed BAM (samtools view -bh) and drops
    unmapped reads when paired with --sam-hit-only
- name: NanoCount
  role: 'Downstream consumer: performs DRS-specific filtering and expectation-maximisation
    quantification on the multi-mapping BAM produced by this alignment step'
  repo: https://github.com/a-slide/NanoCount
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:0c0065aaebe2291bc4537d3ded8d20ec20ce5733a41034adfe5425430738f13a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - FLAIR
  - LIQA
  - NanoCount
  - SciPy
  - StringTie2
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# nanopore-drs-transcriptome-alignment

## Summary

Aligns Oxford Nanopore direct RNA sequencing (DRS) reads to a reference transcriptome with minimap2 (map-ont preset), deliberately retaining multiple secondary mappings (-p 0, -N 10 or higher) and converting to BAM with samtools, because downstream EM-based transcript quantification (e.g. NanoCount) needs the full multi-mapping compatibility set per read.

## When to use

Use when input is Oxford Nanopore direct-RNA FASTQ reads plus a transcriptome FASTA and the goal is to produce BAM alignments that preserve secondary mappings for EM-based isoform quantification (NanoCount, Salmon, LIQA).

## When NOT to use

- The goal is spliced whole-genome alignment of DRS reads rather than transcript-level assignment — minimap2's splice preset (-ax splice -uf -k14 for noisy Nanopore direct RNA) would be the appropriate mode instead of map-ont against a transcriptome.
- Downstream quantification needs only a single best alignment per read — retaining -N secondary mappings and -p 0 would inflate the alignment file unnecessarily.

## Inputs

- Oxford Nanopore direct RNA sequencing FASTQ reads (e.g. ENA accession SRR6058583, study PRJEB39347)
- Reference transcriptome FASTA (e.g. SIRV spike-in transcriptome or GENCODE v31)

## Outputs

- BAM file of transcriptome alignments retaining secondary mappings (aligned_reads.bam)
- Optionally the raw SAM stream piped from minimap2

## How to apply

Run minimap2 with the map-ont preset and the -a flag to emit SAM against the transcriptome reference (not the genome), because NanoCount-style quantification assigns reads to transcript isoforms directly. Critically, keep secondary alignments: use -p 0 to disable the secondary score ratio cutoff and -N 10 (or up to -N 100 for the SIRV benchmark) so that ambiguous multi-mapping reads retain their alternative transcript alignments, since the downstream expectation-maximisation loop needs the complete read-transcript compatibility index. Pipe the SAM through samtools view -bh to produce aligned_reads.bam, ideally adding --sam-hit-only to drop unmapped reads. The resulting BAM then feeds directly into NanoCount's DRS-specific filtering (aligned fraction >=0.5, length >=50 nt, 3' end within 50 nt of transcript end, secondary AS >=95% of best), which operates on the multi-mapping alignments it receives.

## Related tools

- **minimap2** (Aligns DRS FASTQ reads to the reference transcriptome using the map-ont preset with -a, -p 0 and -N secondary-mapping retention to emit SAM) — https://github.com/lh3/minimap2
- **samtools** (Converts minimap2 SAM output to compressed BAM (samtools view -bh) and drops unmapped reads when paired with --sam-hit-only)
- **NanoCount** (Downstream consumer: performs DRS-specific filtering and expectation-maximisation quantification on the multi-mapping BAM produced by this alignment step) — https://github.com/a-slide/NanoCount

## Examples

```
minimap2 -ax map-ont -p 0 -N 10 transcriptome.fasta reads.fastq | samtools view -bh > aligned_reads.bam
```

## Evaluation signals

- The BAM contains secondary alignments: verify the minimap2 command used -p 0 and -N >= 10, since omitting them collapses multi-mapping reads and breaks the downstream EM compatibility index
- Alignment is against the transcriptome reference, not the genome; read coordinates should fall within transcript sequences
- Downstream NanoCount filtering retains a plausible fraction of reads (aligned fraction >=0.5, length >=50 nt, 3' end within 50 nt of the transcript 3' end), indicating alignments are transcript-complete rather than fragmented
- After quantification, NanoCount raw abundances sum to 1 and cover all 69 SIRV transcripts on the SIRV E0/E2 benchmark
- Unmapped reads are removed (--sam-hit-only or samtools filtering) before downstream tools, as required by BamSlam and NanoCount

## Limitations

- Secondary alignments below the -N cap are discarded, so reads mapping to more transcripts than -N lose part of their compatibility set; task_001 used -N 100 on the 69-transcript SIRV reference while the benchmark used -N 10
- Alignments to a transcriptome cannot place reads across splice junctions of unannotated isoforms; NanoCount's own filtering discards reads whose 3' end is >50 nt from the transcript 3' end because they may come from unannotated isoforms
- For noisy Nanopore direct RNA reads against a genome, the map-ont transcriptome preset is not splice-aware — minimap2 offers the splice -uf -k14 preset for that case

## Evidence

- [results] Reads from each sample were aligned using minimap2 (29) to the human and sequin reference genome and transcriptome.: "Reads from each sample were aligned using minimap2 (29) to the human and sequin reference genome and transcriptome."
- [readme] The reads must be mapped to the transcript set using minimap2 with the `-p0` flag to retain all secondary mappings.: [evidence span withheld — anchor status: external_doc]
- [readme] minimap2 -ax map-ont --sam-hit-only transcriptome.fasta reads.fastq | samtools view -bh > aligned_reads.bam: [evidence span withheld — anchor status: external_doc]
- [readme] minimap2 -ax splice -uf -k14 ref.fa reads.fa ref# noisy Nanopore direct RNA-seq: [evidence span withheld — anchor status: not_found]
