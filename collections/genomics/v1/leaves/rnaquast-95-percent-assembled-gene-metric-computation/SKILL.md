---
name: rnaquast-95-percent-assembled-gene-metric-computation
description: Use when benchmarking de novo transcriptome assemblers (e.g., rnaSPAdes,
  SPAdes, Trinity, RNA-Bloom) on real RNA-Seq datasets and you need a completeness
  metric comparable across organisms and datasets.
when_to_use_negative:
- The goal is contiguity assessment alone — high database coverage or nucleotide recall
  can be achieved by very fragmented assemblies (or even raw reads), so this completeness
  metric must not be used as a proxy for assembly contiguity
- No reference gene annotations are available — rnaQUAST-based 95%-assembled gene
  counting is a reference-based evaluation
- Evaluating metatranscriptome assemblies — the approach was designed for RNA-Seq
  data from a single organism, and metatranscriptomic quality evaluation remains future
  work
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: rnaQUAST
  role: Counts 95%-assembled genes by evaluating each assembler's contigs/scaffolds
    against reference gene annotations
- name: Kallisto
  role: Quantifies per-gene estimated fragment counts to define the denominator (genes
    with nucleotide coverage > 5) for cross-dataset normalization
- name: rnaSPAdes
  role: De novo transcriptome assembler being benchmarked; achieved the highest average
    fraction of 95%-assembled genes (0.500)
  repo: github.com/ablab/spades
- name: Trinity
  role: Competing transcriptome assembler benchmarked on the same datasets (normalized
    fraction 0.438)
- name: RNA-Bloom
  role: Competing transcriptome assembler benchmarked on the same datasets (normalized
    fraction 0.406)
- name: SPAdes
  role: Unmodified genome assembler used as a baseline comparison (normalized fraction
    0.397)
  repo: github.com/ablab/spades
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:47d8e49fc545c933811a73b0ec857f61365c534d0ebb5400a9af11b65b059c2b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Kallisto
  - RNA-Bloom
  - SPAdes
  - Trinity
  - rnaQUAST
  - rnaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# rnaQUAST 95%-assembled gene metric computation with Kallisto-normalized cross-dataset comparison

## Summary

Computes the fraction of 95%-assembled genes for each de novo transcriptome assembly using rnaQUAST against reference gene annotations, then normalizes per dataset by the number of Kallisto-detected genes (nucleotide coverage > 5) to enable fair cross-dataset assembler comparison.

## When to use

Use when benchmarking de novo transcriptome assemblers (e.g., rnaSPAdes, SPAdes, Trinity, RNA-Bloom) on real RNA-Seq datasets and you need a completeness metric comparable across organisms and datasets.

## When NOT to use

- The goal is contiguity assessment alone — high database coverage or nucleotide recall can be achieved by very fragmented assemblies (or even raw reads), so this completeness metric must not be used as a proxy for assembly contiguity
- No reference gene annotations are available — rnaQUAST-based 95%-assembled gene counting is a reference-based evaluation
- Evaluating metatranscriptome assemblies — the approach was designed for RNA-Seq data from a single organism, and metatranscriptomic quality evaluation remains future work

## Inputs

- De novo transcriptome assemblies (contigs/scaffolds FASTA) from rnaSPAdes, SPAdes, Trinity, and RNA-Bloom
- Reference gene annotations for each organism (used by rnaQUAST)
- 8 real RNA-Seq datasets from NCBI SRA: SRR5133163 (Human), SRR1957703/SRR1957706 (Human large), SRX648736 (Mouse), SRR1560107 (Worm), SRR1588569 (Corn), SRR5344669/SRR5344670 (Arabidopsis)

## Outputs

- Per-dataset fractions of 95%-assembled genes normalized by Kallisto-detected genes per assembler (assembled_gene_fractions.csv)
- Cross-dataset mean fraction of 95%-assembled genes per assembler
- rnaQUAST reports of 95%-assembled gene counts per assembly

## How to apply

For each assembler-dataset pair, run rnaQUAST against the reference gene annotations to count 95%-assembled genes, since rnaQUAST was the benchmarking tool used for quality evaluation of de novo assemblies in this study. Because raw counts of assembled genes are not comparable across datasets with different gene repertoires, run Kallisto quantification on each RNA-Seq dataset to obtain per-gene estimated fragment counts, and flag a gene as detected when its nucleotide coverage from estimated counts exceeds 5. Divide the number of 95%-assembled genes by the number of Kallisto-detected genes to obtain the normalized fraction per assembler-dataset pair. Average these fractions across the 8 real datasets (Human SRR5133163; Human large SRR1957703/SRR1957706; Mouse SRX648736; Worm SRR1560107; Corn SRR1588569; Arabidopsis SRR5344669/SRR5344670) to rank assemblers by assembly completeness.

## Related tools

- **rnaQUAST** (Counts 95%-assembled genes by evaluating each assembler's contigs/scaffolds against reference gene annotations)
- **Kallisto** (Quantifies per-gene estimated fragment counts to define the denominator (genes with nucleotide coverage > 5) for cross-dataset normalization)
- **rnaSPAdes** (De novo transcriptome assembler being benchmarked; achieved the highest average fraction of 95%-assembled genes (0.500)) — github.com/ablab/spades
- **Trinity** (Competing transcriptome assembler benchmarked on the same datasets (normalized fraction 0.438))
- **RNA-Bloom** (Competing transcriptome assembler benchmarked on the same datasets (normalized fraction 0.406))
- **SPAdes** (Unmodified genome assembler used as a baseline comparison (normalized fraction 0.397)) — github.com/ablab/spades

## Evaluation signals

- Computed cross-dataset mean fractions must match the reported benchmark values: rnaSPAdes 0.500, Trinity 0.438, RNA-Bloom 0.406, SPAdes 0.397
- Each normalized fraction must fall in [0, 1] since 95%-assembled genes cannot exceed Kallisto-detected genes
- A gene is counted as detected only when its nucleotide coverage from Kallisto estimated counts exceeds 5 — verify the threshold was applied before normalizing
- rnaQUAST output must report 95%-assembled gene counts for every assembler-dataset pair across all 8 datasets
- rnaSPAdes should rank first on this metric, consistent with its reported 26% higher average fraction of 95%-assembled genes than original SPAdes

## Limitations

- No single assembler is the absolute leader across all quality metrics and all datasets — this metric captures completeness only and assembler selection may require additional per-project benchmarks
- High database coverage or nucleotide recall can be achieved by a very fragmented assembly (or even raw reads), so this fraction must be interpreted alongside contiguity and precision metrics
- Mean duplication ratios differ sharply between assemblers (e.g., SPAdes 2% vs rnaSPAdes 32%), so a high assembled-gene fraction may coexist with more duplicated sequences
- rnaQUAST applies no cutoff on mapped fraction of assembled sequences, which (unlike Detonate) means 95%-assembled isoform counts may not perfectly correlate with contig recall

## Evidence

- [intro] rnaQUAST benchmarking: "To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies"
- [discussion] assembled-gene leader values: "In comparison with other assemblers, rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively)."
- [discussion] rnaSPAdes vs SPAdes completeness gap: "it has significantly better assembly completeness metrics: 26% higher average fraction of 95%-assembled genes, 18% larger database coverage"
- [methods] SPAdes project repository: "Project home page: cab.spbu.ru/software/rnaspades/, github.com/ablab/spades"
