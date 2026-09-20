---
name: transcriptome-assembly-parameter-tradeoff-analysis
description: Use when configuring a de novo transcriptome assembler (rnaSPAdes, SPAdes,
  Trinity, RNA-Bloom, Trans-ABySS) for RNA-Seq reads of known length and you must
  decide between automatic vs. manually specified k-mer iterations, or when explaining
  assembly completeness-vs-accuracy differences attributable.
when_to_use_negative:
- The input is a metatranscriptome — the approach was designed for RNA-Seq from a
  single organism and metatranscriptome assembly poses unaddressed challenges (repetitive
  patterns in homologous genes, extreme mRNA quantity differences)
- The goal is picking a single best assembler outright — no absolute leader exists
  across all quality metrics and all datasets, and tool choice may require additional
  per-project benchmarks
- The input is long-read data intended to be used directly as complete transcripts
  — high Oxford Nanopore/PacBio error rates prevent this, so this k-mer-schedule skill
  does not apply
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
tools:
- name: rnaSPAdes
  role: Target transcriptome assembler whose k-mer sizes are computed automatically
    from the input read length; the schedule rules implemented here reproduce its
    defaults
  repo: github.com/ablab/spades
- name: SPAdes
  role: Baseline genome assembler applied to RNA-Seq data using fixed default k-values
    chosen by developers (iterative series such as 21, 33, 55, 77), used as the manual-schedule
    contrast
  repo: github.com/ablab/spades
- name: rnaQUAST
  role: Quality-evaluation tool used to benchmark the resulting assemblies and quantify
    the completeness/accuracy tradeoff across parameter choices
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:61b05c0c8f0994254305e427d44f5a9750620ff1bd0a719b956382fcf3fced04
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPAdes
  - rnaQUAST
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# transcriptome-assembly-parameter-tradeoff-analysis

## Summary

Derives and justifies de novo transcriptome assembly k-mer schedules from RNA-Seq read length, contrasting rnaSPAdes' automatic two-value k computation with fixed developer-chosen k-series (e.g., 21, 33, 55, 77 in SPAdes), and weighs the completeness/accuracy tradeoff each choice induces.

## When to use

Use when configuring a de novo transcriptome assembler (rnaSPAdes, SPAdes, Trinity, RNA-Bloom, Trans-ABySS) for RNA-Seq reads of known length and you must decide between automatic vs. manually specified k-mer iterations, or when explaining assembly completeness-vs-accuracy differences attributable to k-mer size.

## When NOT to use

- The input is a metatranscriptome — the approach was designed for RNA-Seq from a single organism and metatranscriptome assembly poses unaddressed challenges (repetitive patterns in homologous genes, extreme mRNA quantity differences)
- The goal is picking a single best assembler outright — no absolute leader exists across all quality metrics and all datasets, and tool choice may require additional per-project benchmarks
- The input is long-read data intended to be used directly as complete transcripts — high Oxford Nanopore/PacBio error rates prevent this, so this k-mer-schedule skill does not apply

## Inputs

- RNA-Seq FASTQ reads (or SRA accessions) with known read length for each dataset
- Assembled contigs/transcripts from competing assemblers (rnaSPAdes, SPAdes, Trinity, RNA-Bloom, Trans-ABySS)
- Reference transcriptome/genome annotations for reference-based evaluation (rnaQUAST)

## Outputs

- Per-dataset two-value k-mer schedule (lower k ≈ read_length/3, min 29; upper k = largest odd number ≤ read_length/2 - 1)
- Summary table mapping read length to computed k-mer schedule per dataset
- rnaQUAST evaluation metrics (fraction of 95%-assembled genes and isoforms, mean duplication ratio, misassemblies) for comparing assembler/parameter tradeoffs

## How to apply

First extract each dataset's read length from SRA metadata (e.g., Human SRR5133163, Human large SRR1957703/SRR1957706, Mouse SRX648736, Worm SRR1560107, Corn SRR1588569, Arabidopsis SRR5344669/SRR5344670). Then implement rnaSPAdes' schedule rules: the upper k is the largest odd number not exceeding read_length/2 - 1 (nearly half the read length), and the lower k is approximately read_length/3 with a minimum of 29, because k < 29 dramatically increases erroneous contigs from transcripts sharing the same k-mer. Apply this to every read length to produce the two-value k schedule rnaSPAdes uses instead of SPAdes' longer iterative series such as 21, 33, 55, 77, and assemble results into a summary table cross-checked against rnaSPAdes' known default behavior. When interpreting outcomes, note the tradeoff: SPAdes' first iteration at small k = 21 recovers extremely low-covered genes where read overlaps are short, but this was implicated in rnaSPAdes' rare underperformance on low-coverage simulated data. Evaluate resulting assemblies with rnaQUAST (95%-assembled genes/isoforms, duplication ratio, misassemblies), remembering that transcript filtration thresholds (Supplementary Table S14) also shift assembly properties.

## Related tools

- **rnaSPAdes** (Target transcriptome assembler whose k-mer sizes are computed automatically from the input read length; the schedule rules implemented here reproduce its defaults) — github.com/ablab/spades
- **SPAdes** (Baseline genome assembler applied to RNA-Seq data using fixed default k-values chosen by developers (iterative series such as 21, 33, 55, 77), used as the manual-schedule contrast) — github.com/ablab/spades
- **rnaQUAST** (Quality-evaluation tool used to benchmark the resulting assemblies and quantify the completeness/accuracy tradeoff across parameter choices)

## Examples

```
python -c "rl=150; upper=max(k for k in range(29,rl,2) if k<=rl//2-1); lower=max(29,rl//3); print('k-schedule:',lower,upper)"
```

## Evaluation signals

- For every dataset read length, the computed upper k equals the largest odd number ≤ read_length/2 - 1, with no dataset left unmatched
- The computed lower k satisfies lower k ≈ read_length/3 and is never below 29 (small k < 29 inflates erroneous contigs from transcripts sharing the same k-mer)
- The schedule contains exactly two k-values per dataset, replacing SPAdes' longer iterative series (e.g., 21, 33, 55, 77)
- Each computed schedule reproduces rnaSPAdes' known default k-values for that read length (verifiable from the assembler's console output/configuration)
- Downstream rnaQUAST comparison shows the expected tradeoff direction: small first-iteration k (e.g., 21) recovers extremely low-covered genes but increases erroneous/duplicated contigs

## Limitations

- Using small k-mer sizes (< 29) in transcriptome assembly dramatically increases erroneous contigs from transcripts sharing the same k-mer; rnaSPAdes' rare underperformance on the low-coverage simulated Mouse dataset was attributed to small k usage
- High database coverage or nucleotide recall can be achieved by very fragmented assemblies (or even raw reads), so these metrics do not reflect assembly contiguity and can mislead parameter tradeoff judgments
- Metric differences between Detonate and rnaQUAST (cutoff on mapped fraction) mean contig recall and 95%-assembled isoforms do not perfectly correlate, complicating cross-tool evaluation of parameter choices

## Evidence

- [discussion] By using small k = 21 during the first iteration SPAdes manages to assemble extremely low-covered genes, where overlaps between reads are short.: "By using small k = 21 during the first iteration SPAdes manages to assemble extremely low-covered genes, where overlaps between reads are short."
- [methods] the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction: "the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction"
- [discussion] various thresholds for transcript filtration in rnaSPAdes (Table S14 in the Supplementary material) result in assemblies with different properties.: "various thresholds for transcript filtration in rnaSPAdes (Table S14 in the Supplementary material) result in assemblies with different properties."
- [discussion] Even with the aid of specially developed tools, such as Transrate, DETONATE, BUSCO, and rnaQUAST, the choice of a suitable assembly tool remains a non-trivial problem and may require additional: "Even with the aid of specially developed tools, such as Transrate, DETONATE, BUSCO, and rnaQUAST, the choice of a suitable assembly tool remains a non-trivial problem and may require additional"
- [readme] SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data.: "SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data."
