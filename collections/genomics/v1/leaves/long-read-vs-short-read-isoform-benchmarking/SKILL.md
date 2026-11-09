---
name: long-read-vs-short-read-isoform-benchmarking
description: Use when a long-read study reports isoforms for a complex, multi-isoform
  gene locus and you need to test whether per-cell short-read Illumina assembly recovers
  those same isoforms or produces contigs absent from the long-read set.
when_to_use_negative:
- When the goal is isoform detection itself from ONT data — use the Mandalorion pipeline
  rather than this benchmarking comparison
- When the study locus has only a single annotated isoform, since the benchmark targets
  multi-isoform loci (e.g. CD37) where short-read assembly is expected to fail
- When comparing long-read technologies (e.g. ONT MinION vs PacBio Sequel) rather
  than short-read de novo assembly vs long-read isoforms
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: Trinity
  role: De novo assembly of per-cell Illumina RNAseq reads into contigs (v2.2.0, default
    k-mer and minimum contig length settings) to test short-read isoform recovery
- name: minimap2
  role: Splice-aware alignment (e.g. -ax splice) of Trinity contigs against ONT-identified
    CD37 isoforms
  repo: https://github.com/lh3/minimap2
- name: blat
  role: Cross-check of minimap2 alignments between Trinity contigs and ONT isoforms
- name: Mandalorion
  role: Source of the ONT-identified CD37 isoform sequences used as the benchmark
    reference (run with -G hg38.fasta and -g gencodeV29.gtf)
  repo: https://github.com/christopher-vollmers/Mandalorion
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms16027
    title: Nanopore long-read RNAseq reveals widespread transcriptional variation
      among the surface receptors of individual B cells
  source_content_hash: sha256:8771a39aa486d7d849a71100680925f2a431324cc76f9d59b2afe104c164ba66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Mandalorion
  - Trinity
  - blat
  - minimap2
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# long-read-vs-short-read-isoform-benchmarking

## Summary

Benchmark whether de novo short-read Illumina assembly (Trinity) can recover the complex isoforms identified from ONT MinION long-read RNAseq (via the Mandalorion pipeline), testing the claim that short reads fail to resolve multi-isoform loci such as CD37 in individual B1a cells.

## When to use

Use when a long-read study reports isoforms for a complex, multi-isoform gene locus and you need to test whether per-cell short-read Illumina assembly recovers those same isoforms or produces contigs absent from the long-read set.

## When NOT to use

- When the goal is isoform detection itself from ONT data — use the Mandalorion pipeline rather than this benchmarking comparison
- When the study locus has only a single annotated isoform, since the benchmark targets multi-isoform loci (e.g. CD37) where short-read assembly is expected to fail
- When comparing long-read technologies (e.g. ONT MinION vs PacBio Sequel) rather than short-read de novo assembly vs long-read isoforms

## Inputs

- Per-cell Illumina RNAseq FASTQ files (e.g. SRP082530 from SRA/ENA)
- ONT-identified isoform FASTA sequences from the Mandalorion pipeline (R2C2 consensus-derived)
- hg38 genome fasta (hg38.fasta) and gencodeV29.gtf annotation used by Mandalorion
- Trinity de novo assembled contigs (FASTA) per cell

## Outputs

- trinity_vs_ont_cd37_report.tsv — per ONT isoform coverage status (complete/partial/not covered) and list of Trinity contigs with no ONT counterpart
- minimap2 splice-aware alignment SAM of Trinity contigs against ONT isoforms
- Qualitative concordance verdict vs the paper's reported finding

## How to apply

Download per-cell Illumina RNAseq FASTQ files (e.g. study SRP082530 from SRA/ENA) and run Trinity v2.2.0 de novo assembly on each cell's reads with default k-mer and minimum contig length settings. Extract the ONT-identified isoform sequences for the target gene (e.g. CD37) from the Mandalorion output, which was produced with 'python3 Mandalorion.py -c config_file -p . -g gencodeV29.gtf -G hg38.fasta' parameters on R2C2 consensus data. Align Trinity contigs to the ONT isoforms splice-aware with minimap2 (e.g. -ax splice) and cross-check with blat. For each ONT isoform, record whether any Trinity contig spans it completely or partially, and flag Trinity contigs with no counterpart in the ONT isoform set, summarizing in a comparison TSV. The rationale is that short reads fail to sequence full-length cDNA copies, so multi-isoform loci like CD37 should mostly fail to form complete contigs or yield contigs not detected by ONT RNAseq.

## Related tools

- **Trinity** (De novo assembly of per-cell Illumina RNAseq reads into contigs (v2.2.0, default k-mer and minimum contig length settings) to test short-read isoform recovery)
- **minimap2** (Splice-aware alignment (e.g. -ax splice) of Trinity contigs against ONT-identified CD37 isoforms) — https://github.com/lh3/minimap2
- **blat** (Cross-check of minimap2 alignments between Trinity contigs and ONT isoforms)
- **Mandalorion** (Source of the ONT-identified CD37 isoform sequences used as the benchmark reference (run with -G hg38.fasta and -g gencodeV29.gtf)) — https://github.com/christopher-vollmers/Mandalorion

## Examples

```
minimap2 -ax splice ont_cd37_isoforms.fa trinity_contigs.fa > trinity_vs_ont_cd37.sam
```

## Evaluation signals

- Report reproduces the paper's finding: Trinity either fails to form complete contigs spanning the ONT CD37 isoforms or yields contigs not observed in the ONT isoform set (qualitative concordance check)
- Every ONT CD37 isoform has an explicit row in trinity_vs_ont_cd37_report.tsv with complete/partial/not-covered status
- Trinity contigs lacking any minimap2/blat counterpart in the ONT isoform set are flagged rather than silently dropped
- minimap2 alignments use splice-aware settings (-ax splice) and are consistent with independent blat cross-checks
- Trinity assembly used default k-mer and minimum contig length settings (v2.2.0) so the benchmark matches the reported conditions

## Limitations

- Short-read Illumina RNAseq is limited in its ability to resolve complex isoforms because it fails to sequence full-length cDNA copies, so partial or non-reproducible contigs are expected outcomes rather than pipeline errors
- Some alternative splicing events were not supported by single-cell Illumina data, possibly due to low coverage and low Illumina sequencing depth per cell
- Mandalorion identifies and quantifies complex isoforms longer than ~500 bp; isoforms below this threshold (default -I minimum isoform length) cannot be benchmarked
- Trinity contigs not detected by ONT RNAseq may reflect ONT limitations (low read numbers and high error rates) rather than assembly artifacts

## Evidence

- [abstract] Short-read RNAseq is limited in its ability to resolve complex isoforms because it fails to sequence full-length cDNA copies of RNA molecules: "Short-read RNAseq is limited in its ability to resolve complex isoforms because it fails to sequence full-length cDNA copies of RNA molecules"
- [discussion] CD19, CD20 (Ms4a1), IGH, CD45 (B220 or Ptprc), CD2, CD79b and CD37 were expressed as multiple complex isoforms: "CD19, CD20 (Ms4a1), IGH, CD45 (B220 or Ptprc), CD2, CD79b and CD37 were expressed as multiple complex isoforms"
- [methods] python3 Mandalorion.py -c config_file -p . -g gencodeV29.gtf -G hg38.fasta -a adapters.fasta -f R2C2_consensi.fasta -b R2C2_subreads.fastq: "python3 Mandalorion.py -c config_file -p . -g gencodeV29.gtf -G hg38.fasta -a adapters.fasta -f R2C2_consensi.fasta -b R2C2_subreads.fastq"
- [readme] ./minimap2 -ax splice ref.fa rna-reads.fa > aln.sam       # spliced long reads (strand unknown): [evidence span withheld — anchor status: external_doc]
- [discussion] the events that were not supported were of significantly lower coverage, indicating they might be closer to the detection limits in either technology: "the events that were not supported were of significantly lower coverage, indicating they might be closer to the detection limits in either technology"
