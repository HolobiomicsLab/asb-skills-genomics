---
name: dual-strand-alignment-selection
description: Use when analyzing nanoEM (enzymatic C-to-T converted) nanopore long
  reads whose C-to-T (*_CT.fq.gz) and G-to-A (*_GA.fq.gz) converted FASTQs must each
  be mapped to the converted reference (output.fa) before strand-level alignments
  are merged per read.
when_to_use_negative:
- Input is native (unconverted) nanopore WGS reads — use direct signal-level methylation
  calling with nanopolish instead, which also yields ~4x longer N50 reads.
- Data are short-read bisulfite/enzymatic methyl-seq alignments, not long-read C-to-T/G-to-A
  converted nanopore reads requiring dual-strand selection.
- The goal is only methylation calling from already-selected alignments — use call_methylation.py
  on the pileups rather than re-running the mapping/selection step.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Maps each converted FASTQ (*_CT.fq.gz, *_GA.fq.gz) to the converted reference
    output.fa with -ax map-ont and --eqx so alignment blocks support strand selection
  repo: http://github.com/lh3/minimap2
- name: samtools
  role: Converts minimap2 SAM output to sorted, indexed BAMs (1.sorted.bam, 2.sorted.bam,
    output_CT.sorted.bam, output_GA.sorted.bam)
- name: pysam
  role: Python library underlying best_align.py for reading the two BAMs and selecting
    the best strand-specific alignment per read
- name: sambamba
  role: Downstream mpileup of selected alignments for methylation calling (sambamba
    mpileup output_CT.sorted.bam -L cpg_sites.bed)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:9b42a8e94546bde069f28f099eb660f4463e4b5a0aa6af32b92a07f78cca60d0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yos-sk/nanoEM
  tools:
  - Python3
  - minimap2
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# dual-strand-alignment-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns C-to-T and G-to-A converted nanoEM nanopore reads to a converted reference genome in two parallel minimap2 runs and selects the best strand-specific alignment per read with best_align.py. This is required because in silico conversion halves the alphabet on each strand, so a single mapping pass cannot recover alignments for both converted strands.

## When to use

Use when analyzing nanoEM (enzymatic C-to-T converted) nanopore long reads whose C-to-T (*_CT.fq.gz) and G-to-A (*_GA.fq.gz) converted FASTQs must each be mapped to the converted reference (output.fa) before strand-level best alignments are merged per read.

## When NOT to use

- Input is native (unconverted) nanopore WGS reads — use direct signal-level methylation calling with nanopolish instead, which also yields ~4x longer N50 reads.
- Data are short-read bisulfite/enzymatic methyl-seq alignments, not long-read C-to-T/G-to-A converted nanopore reads requiring dual-strand selection.
- The goal is only methylation calling from already-selected alignments — use call_methylation.py on the pileups rather than re-running the mapping/selection step.

## Inputs

- C-to-T converted nanopore FASTQ (*_CT.fq.gz) from convert_reads.py
- G-to-A converted nanopore FASTQ (*_GA.fq.gz) from convert_reads.py
- C-to-T converted reference genome FASTA (output.fa) from convert_ref.py
- Original nanoEM FASTQ runs (e.g. MB231 and BT474 cell lines, DDBJ DRA accession DRA011237)

## Outputs

- 1.sorted.bam — sorted, indexed CT-strand alignment BAM
- 2.sorted.bam — sorted, indexed GA-strand alignment BAM
- output_CT.sam — best CT-strand alignment per read
- output_GA.sam — best GA-strand alignment per read
- output_CT.sorted.bam / output_GA.sorted.bam with .bai indexes
- Per-run fraction of reads aligned, comparable in mapping_statistics.tsv

## How to apply

First in silico convert the reference genome (python src/convert_ref.py ref > output.fa) and each nanoEM FASTQ into paired C-to-T and G-to-A variants with convert_reads.py. Map each converted FASTQ separately to output.fa with minimap2 (-t 8, --split-prefix temp_sam1/temp_sam2, -ax map-ont, --eqx), piping through 'samtools view -b | samtools sort -@ 8' to yield 1.sorted.bam (CT) and 2.sorted.bam (GA), then 'samtools index' both. Run 'python src/best_align.py --bam1 1.sorted.bam --bam2 2.sorted.bam --fastq fastq', which uses pysam to pick the best strand-specific alignment per read, writing output_CT.sam and output_GA.sam; sort and index these with samtools. The --eqx flag is essential because the selection logic relies on exact-match (=X) blocks from the converted-read alignment. Per read, the better-scoring of the CT or GA alignment is retained, which is why the pipeline outperforms mapping unconverted reads with minimap2 alone.

## Related tools

- **minimap2** (Maps each converted FASTQ (*_CT.fq.gz, *_GA.fq.gz) to the converted reference output.fa with -ax map-ont and --eqx so alignment blocks support strand selection) — http://github.com/lh3/minimap2
- **samtools** (Converts minimap2 SAM output to sorted, indexed BAMs (1.sorted.bam, 2.sorted.bam, output_CT.sorted.bam, output_GA.sorted.bam))
- **pysam** (Python library underlying best_align.py for reading the two BAMs and selecting the best strand-specific alignment per read)
- **sambamba** (Downstream mpileup of selected alignments for methylation calling (sambamba mpileup output_CT.sorted.bam -L cpg_sites.bed))

## Examples

```
minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa *_CT.fq.gz --eqx | samtools view -b | samtools sort -@ 8 -o 1.sorted.bam; python src/best_align.py --bam1 1.sorted.bam --bam2 2.sorted.bam --fastq fastq
```

## Evaluation signals

- Fraction of reads aligned approaches the reported 90% (MB231) and 89% (BT474) versus the 81–84% baseline of default minimap2 on unconverted reads — a 6–8% improvement.
- Both output_CT.sam and output_GA.sam are produced and every read in the original FASTQ is represented in at least one of the two best-alignment files.
- All BAM outputs (1.sorted.bam, 2.sorted.bam, output_CT.sorted.bam, output_GA.sorted.bam) are sorted and have valid .bai indexes.
- minimap2 was run with --eqx so alignments contain exact-match (=) blocks required by the selection logic.
- Downstream sambamba mpileup on the selected BAMs against cpg_sites.bed yields pileup_CT.tsv and pileup_GA.tsv without empty output.

## Limitations

- No software version numbers are stated for minimap2, samtools, sambamba, or the pipeline scripts; the README only notes that --split-prefix is unnecessary for minimap2 cloned after April 7, 2023.
- nanoEM cannot distinguish between 5-methylcytosine and 5-hydroxymethylcytosine on the same DNA molecule, unlike direct nanopore WGS calling.
- The pipeline analyzes converted reads more efficiently than minimap2 alone particularly at CGIs; unconverted long reads may still be preferable for regions where direct signal-level analysis works.

## Evidence

- [results] mapping-rate improvement reported: [evidence span withheld — anchor status: not_found]
