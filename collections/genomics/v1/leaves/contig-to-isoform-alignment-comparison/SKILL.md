---
name: contig-to-isoform-alignment-comparison
description: Use when you need to assess whether short-read assembly recovers complex
  isoforms at multi-isoform loci that long-read RNAseq (ONT/Mandalorion) has already
  defined, or to flag assembly contigs with no long-read counterpart.
when_to_use_negative:
- When the goal is standard transcript quantification rather than testing assembly
  completeness against long-read isoforms.
- When reads are raw ONT/R2C2 reads rather than assembled contigs — Mandalorion's
  isoform definition pipeline (C3POa/R2C2 consensus) is the appropriate path, not
  contig-to-isoform comparison.
- When the gene locus is single-isoform and simple; the comparison adds value mainly
  for multi-isoform loci like CD37 where short reads cannot resolve full-length isoforms.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Trinity
  role: De novo assembly of per-cell Illumina RNAseq reads into contigs
- name: minimap2
  role: Splice-aware alignment of Trinity contigs to ONT-identified isoform sequences
    (-ax splice)
  repo: https://github.com/lh3/minimap2
- name: blat
  role: Cross-check of contig-to-isoform alignments
- name: Mandalorion
  role: Pipeline that produced the ONT isoform set used as the reference for comparison
  repo: https://github.com/christopher-vollmers/Mandalorion
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms16027
    title: Nanopore long-read RNAseq reveals widespread transcriptional variation
      among the surface receptors of individual B cells
  source_content_hash: sha256:dffde1dfa1739527f65b07683313a08a232d698dce65f74cfe86869842b52372
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Mandalorion
  - Trinity
  - blat
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# contig-to-isoform-alignment-comparison

## Summary

A benchmarking skill that tests whether a short-read de novo assembly (e.g., Trinity on per-cell Illumina RNAseq) can reproduce long-read-identified isoforms (from ONT RNAseq/Mandalorion) by splice-aligning contigs to isoform sequences and classifying coverage as complete, partial, or absent.

## When to use

Use when you need to assess whether short-read assembly recovers complex isoforms at multi-isoform loci that long-read RNAseq (ONT/Mandalorion) has already defined, or to flag assembly contigs with no long-read counterpart.

## When NOT to use

- When the goal is standard transcript quantification rather than testing assembly completeness against long-read isoforms.
- When reads are raw ONT/R2C2 reads rather than assembled contigs — Mandalorion's isoform definition pipeline (C3POa/R2C2 consensus) is the appropriate path, not contig-to-isoform comparison.
- When the gene locus is single-isoform and simple; the comparison adds value mainly for multi-isoform loci like CD37 where short reads cannot resolve full-length isoforms.

## Inputs

- Per-cell Illumina RNAseq FASTQ files (e.g., from SRA/ENA study SRP082530)
- ONT-identified isoform sequences for the target locus extracted from Mandalorion isoform output (built on hg38.fasta and gencodeV29.gtf)
- Trinity contigs (FASTA) from de novo assembly of each cell's short reads

## Outputs

- Per-isoform coverage classification table (complete / partial / no contig) for the locus
- List of Trinity contigs with no counterpart in the ONT isoform set
- Summary report, e.g., trinity_vs_ont_cd37_report.tsv

## How to apply

Assemble each cell's Illumina RNAseq FASTQ reads independently with Trinity (e.g., v2.2.0, default k-mer and minimum contig length settings) to produce per-cell contigs. Extract the long-read isoform consensus sequences for the target locus (e.g., CD37 isoforms from Mandalorion output run with '-G hg38.fasta' and '-g gencodeV29.gtf'). Splice-align Trinity contigs to the ONT isoforms with minimap2 (e.g., -ax splice settings) and cross-check the alignments with blat to reduce mapper artifacts. For each ONT isoform, record whether any contig spans it completely or partially, and separately flag contigs with no counterpart in the ONT isoform set. The key decision point is coverage classification per isoform — incomplete or missing contigs indicate the short-read assembler failed at that multi-isoform locus.

## Related tools

- **Trinity** (De novo assembly of per-cell Illumina RNAseq reads into contigs)
- **minimap2** (Splice-aware alignment of Trinity contigs to ONT-identified isoform sequences (-ax splice)) — https://github.com/lh3/minimap2
- **blat** (Cross-check of contig-to-isoform alignments)
- **Mandalorion** (Pipeline that produced the ONT isoform set used as the reference for comparison) — https://github.com/christopher-vollmers/Mandalorion

## Examples

```
./minimap2 -ax splice ont_cd37_isoforms.fasta trinity_contigs.fasta > contigs_vs_isoforms.sam; blat ont_cd37_isoforms.fasta trinity_contigs.fasta contigs_vs_isoforms.psl
```

## Evaluation signals

- Report reproduces the paper's finding: Trinity fails to form complete contigs covering the ONT CD37 isoforms, or yields contigs not observed in the ONT isoform set (qualitative concordance check).
- Every ONT isoform at the locus receives an explicit classification (complete / partial / unrepresented) in the report.
- Contigs with no ONT counterpart are flagged rather than silently dropped.
- minimap2 splice alignments are corroborated by blat for each contig-isoform pair before classification.
- Report is machine-readable TSV (e.g., trinity_vs_ont_cd37_report.tsv) covering all per-cell assemblies.

## Limitations

- Short-read assembly cannot reliably reproduce complex multi-isoform loci: Trinity may fail to form complete contigs or produce contigs absent from the ONT isoform set.
- Illumina reads do not sequence full-length cDNA copies of RNA molecules, so isoform-level structure across a locus is inherently unresolvable by short reads alone.
- Per-cell Illumina depth is low; some events are closer to the detection limits in either technology, so absent support may reflect coverage rather than biology.

## Evidence

- [full_text] Trinity assembly of per-cell Illumina reads mostly failed to reproduce the ONT-identified CD37 isoforms: it either could not form complete contigs or produced contigs not detected by ONT RNAseq.: [evidence span withheld — anchor status: not_found]
- [abstract] Short-read RNAseq is limited in its ability to resolve complex isoforms because it fails to sequence full-length cDNA copies of RNA molecules: "Short-read RNAseq is limited in its ability to resolve complex isoforms because it fails to sequence full-length cDNA copies of RNA molecules"
- [discussion] CD19, CD20 (Ms4a1), IGH, CD45 (B220 or Ptprc), CD2, CD79b and CD37 were expressed as multiple complex isoforms: "CD19, CD20 (Ms4a1), IGH, CD45 (B220 or Ptprc), CD2, CD79b and CD37 were expressed as multiple complex isoforms"
- [readme] minimap2 -ax splice ref.fa rna-reads.fa > aln.sam       # spliced long reads (strand unknown): [evidence span withheld — anchor status: external_doc]
- [methods] python3 Mandalorion.py -c config_file -p . -g gencodeV29.gtf -G hg38.fasta -a adapters.fasta -f R2C2_consensi.fasta -b R2C2_subreads.fastq: "python3 Mandalorion.py -c config_file -p . -g gencodeV29.gtf -G hg38.fasta -a adapters.fasta -f R2C2_consensi.fasta -b R2C2_subreads.fastq"
