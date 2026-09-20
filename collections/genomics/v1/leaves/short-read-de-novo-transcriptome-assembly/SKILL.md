---
name: short-read-de-novo-transcriptome-assembly
description: Use when short-read FASTQ data must be assembled de novo and the resulting
  contigs need to be checked against a long-read-defined isoform set to assess whether
  complex/multi-isoform loci are recovered.
when_to_use_negative:
- The input reads are ONT long reads — Mandalorion is intended for R2C2/C3POa or PacBio
  ccs/lima consensus data and 'is not tested for use with regular ONT reads'; a different
  pipeline is required.
- The goal is gene-level expression quantification rather than isoform structure recovery
  — ONT RNAseq alone quantifies known genes with high correlation to Illumina, so
  no assembly step is needed.
- The locus is known to express a single simple isoform — the skill targets multi-isoform
  surface receptor loci such as CD37, where assembly failure is expected.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: Trinity
  role: De novo assembly of per-cell Illumina RNAseq reads into contigs (v2.2.0, default
    k-mer and minimum contig length settings)
- name: minimap2
  role: Splice-aware alignment of Trinity contigs to the ONT-identified isoform sequences
    (e.g. '-ax splice' preset)
  repo: https://github.com/lh3/minimap2
- name: blat
  role: Independent cross-check of the minimap2 contig-to-isoform alignments
  repo: https://users.soe.ucsc.edu/~kent/src/blatSrc35.zip
- name: Mandalorion
  role: Source of the ONT-identified CD37 isoform reference set used as the alignment
    target
  repo: https://github.com/christopher-vollmers/Mandalorion
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms16027
    title: Nanopore long-read RNAseq reveals widespread transcriptional variation
      among the surface receptors of individual B cells
  source_content_hash: sha256:8cffcc873c84f6245d7f6016d1b96d30e29203d1051c5b597a589eca0feb7241
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Mandalorion
  - Trinity
  - blat
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# short-read-de-novo-transcriptome-assembly

## Summary

Assembles per-cell Illumina RNAseq reads de novo with Trinity to test whether short-read assembly can recover complex isoforms (e.g. the multi-isoform CD37 locus) identified by ONT long-read RNAseq, then aligns Trinity contigs against the ONT isoform set for concordance checking.

## When to use

Use when short-read FASTQ data must be assembled de novo and the resulting contigs need to be checked against a long-read-defined isoform set to assess whether complex/multi-isoform loci are recovered.

## When NOT to use

- The input reads are ONT long reads — Mandalorion is intended for R2C2/C3POa or PacBio ccs/lima consensus data and 'is not tested for use with regular ONT reads'; a different pipeline is required.
- The goal is gene-level expression quantification rather than isoform structure recovery — ONT RNAseq alone quantifies known genes with high correlation to Illumina, so no assembly step is needed.
- The locus is known to express a single simple isoform — the skill targets multi-isoform surface receptor loci such as CD37, where assembly failure is expected.

## Inputs

- Per-cell Illumina RNAseq FASTQ files (paired-end short reads, e.g. from SRA study SRP082530)
- ONT-identified isoform FASTA sequences extracted from Mandalorion output (built with '-G hg38.fasta' and '-g gencodeV29.gtf')
- Optionally, hg38 genome FASTA and gencodeV29.gtf annotation for context/coordinate checking

## Outputs

- Trinity de novo transcriptome contigs (FASTA) per cell
- minimap2 splice-aware alignments of Trinity contigs against ONT isoforms (SAM/PSL; blat used as a cross-check)
- Comparison report, e.g. trinity_vs_ont_cd37_report.tsv, listing per-isoform coverage (complete/partial/none) and unexplained Trinity contigs

## How to apply

Download per-cell Illumina RNAseq FASTQ files (e.g. study SRP082530) from SRA/ENA and run Trinity (v2.2.0 in the reference study, default k-mer and minimum contig length settings) on each cell's reads separately to produce contigs. Build the reference isoform set from the long-read pipeline output: extract ONT-identified isoforms (e.g. CD37) from the Mandalorion isoform output, which was produced with 'python3 Mandalorion.py -c config_file -p . -g gencodeV29.gtf -G hg38.fasta -a adapters.fasta -f R2C2_consensi.fasta -b R2C2_subreads.fastq' against the hg38 genome and gencodeV29 annotation. Align Trinity contigs to the ONT isoforms using minimap2 in splice-aware mode (e.g. '-ax splice' presets, appropriate for spliced mRNA-to-mRNA comparisons per the minimap2 README) and cross-check the alignments with blat. For each ONT isoform, record whether any Trinity contig spans it completely or partially, and flag Trinity contigs with no counterpart in the ONT isoform set; write the per-isoform summary to a TSV report. The rationale is that short-read RNAseq 'is limited in its ability to resolve complex isoforms because it fails to sequence full-length cDNA copies of RNA molecules', so de novo assembly of short reads is expected to fail at multi-isoform loci, and the comparison quantifies that failure.

## Related tools

- **Trinity** (De novo assembly of per-cell Illumina RNAseq reads into contigs (v2.2.0, default k-mer and minimum contig length settings))
- **minimap2** (Splice-aware alignment of Trinity contigs to the ONT-identified isoform sequences (e.g. '-ax splice' preset)) — https://github.com/lh3/minimap2
- **blat** (Independent cross-check of the minimap2 contig-to-isoform alignments) — https://users.soe.ucsc.edu/~kent/src/blatSrc35.zip
- **Mandalorion** (Source of the ONT-identified CD37 isoform reference set used as the alignment target) — https://github.com/christopher-vollmers/Mandalorion

## Examples

```
minimap2 -ax splice ONT_CD37_isoforms.fasta trinity_contigs.fasta > trinity_vs_ont_cd37.sam
```

## Evaluation signals

- The report TSV reproduces the qualitative finding: Trinity either fails to form complete contigs covering the ONT CD37 isoforms or yields contigs not observed in the ONT isoform set.
- Every ONT CD37 isoform has an explicit record (complete / partial / no Trinity counterpart) in the report.
- Each Trinity contig classified as unexplained is corroborated by both minimap2 splice-aware alignment and an independent blat cross-check showing no counterpart in the ONT isoform set.
- Contigs below Trinity's default minimum contig length have been filtered before comparison, so only assembly-produced contigs are evaluated.

## Limitations

- Short-read assembly on individual cells operates at low per-cell sequencing depth, so absence of an isoform in Trinity output can reflect coverage rather than assembly capability alone.
- Trinity is not expected to span full-length multi-isoform transcripts because Illumina short reads 'fail to sequence full-length cDNA copies of RNA molecules'; partial contigs are the typical best case.
- The result is a qualitative concordance check, not a base-level isoform reconstruction; splice site and TSS/TES positions from the ONT side are identified in ~20 bp bins due to ONT error rate.

## Evidence

- [abstract] Short-read RNAseq is limited in its ability to resolve complex isoforms because it fails to sequence full-length cDNA copies of RNA molecules: "Short-read RNAseq is limited in its ability to resolve complex isoforms because it fails to sequence full-length cDNA copies of RNA molecules"
- [methods] Mandalorion command line with genome and annotation inputs: "python3 Mandalorion.py -c config_file -p . -g gencodeV29.gtf -G hg38.fasta -a adapters.fasta -f R2C2_consensi.fasta -b R2C2_subreads.fastq"
- [discussion] CD37 expressed as multiple complex isoforms: "CD19, CD20 (Ms4a1), IGH, CD45 (B220 or Ptprc), CD2, CD79b and CD37 were expressed as multiple complex isoforms"
- [readme] minimap2 splice-aware alignment preset: [evidence span withheld — anchor status: external_doc]
