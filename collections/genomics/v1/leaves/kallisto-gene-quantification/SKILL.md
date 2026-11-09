---
name: kallisto-gene-quantification
description: Use when comparing de novo transcriptome assemblies across multiple RNA-Seq
  datasets and raw counts of assembled genes are not comparable because each dataset/organism
  expresses a different number of genes.
when_to_use_negative:
- When the quantity of interest is a metric other than assembled gene/isoform completeness
  — e.g. mean duplication ratio, where the article reports SPAdes is better (2% vs
  32% for rnaSPAdes) — no kallisto-based normalization is defined for it.
- 'For metatranscriptome assemblies: the underlying rnaSPAdes/evaluation approach
  was designed for RNA-Seq from a single organism and only ''can potentially be applied
  for metatranscriptome assembly''.'
- When a single reference-gene denominator suffices — this skill exists specifically
  because datasets differ in expressed gene content, so it is wasted effort on a single
  simulated dataset with a fixed known gene set.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Kallisto
  role: Pseudoaligns RNA-Seq reads to the reference transcriptome to produce per-gene
    estimated fragment counts used to flag detected genes (coverage > 5) as the normalization
    denominator
- name: rnaQUAST
  role: Evaluates each assembler's de novo assembly against reference gene annotations
    to count 95%-assembled genes, the normalization numerator
- name: rnaSPAdes
  role: One of the assemblers whose output contigs/scaffolds are quantified and evaluated;
    benchmark leader on the normalized metric (0.500)
  repo: https://github.com/ablab/spades
- name: SPAdes
  role: Genome assembler applied unmodified to transcriptomic data; its assemblies
    are quantified and evaluated as a baseline (normalized mean 0.397)
  repo: https://github.com/ablab/spades
- name: Trinity
  role: Competing transcriptome assembler whose assemblies are evaluated with the
    same normalized metric (cross-dataset mean 0.438, ranked second)
- name: RNA-Bloom
  role: Competing transcriptome assembler evaluated with the same normalized metric
    (cross-dataset mean 0.406, ranked third)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:cee936c5c9703985523c40eabd70c150e6bcfb61921303671838f77690bdcc3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/spades
  tools:
  - Kallisto
  - RNA-Bloom
  - SPAdes
  - Trinity
  - rnaQUAST
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# kallisto-gene-quantification

## Summary

Quantifies per-gene RNA-Seq abundance with kallisto to define a dataset-specific set of detectable genes (nucleotide coverage from estimated counts > 5), which normalizes rnaQUAST 95%-assembled gene counts for fair cross-dataset comparison of de novo transcriptome assemblers.

## When to use

Use when comparing de novo transcriptome assemblies across multiple RNA-Seq datasets and raw counts of assembled genes are not comparable because each dataset/organism expresses a different number of genes.

## When NOT to use

- When the quantity of interest is a metric other than assembled gene/isoform completeness — e.g. mean duplication ratio, where the article reports SPAdes is better (2% vs 32% for rnaSPAdes) — no kallisto-based normalization is defined for it.
- For metatranscriptome assemblies: the underlying rnaSPAdes/evaluation approach was designed for RNA-Seq from a single organism and only 'can potentially be applied for metatranscriptome assembly'.
- When a single reference-gene denominator suffices — this skill exists specifically because datasets differ in expressed gene content, so it is wasted effort on a single simulated dataset with a fixed known gene set.

## Inputs

- RNA-Seq FASTQ reads from NCBI SRA (Human SRR5133163; Human large SRR1957703/SRR1957706; Mouse SRX648736; Worm SRR1560107; Corn SRR1588569; Arabidopsis SRR5344669/SRR5344670)
- Reference transcriptome (FASTA) for building the kallisto index, and reference gene annotations for rnaQUAST
- De novo transcriptome assemblies (contigs/scaffolds) from rnaSPAdes, SPAdes, Trinity, and RNA-Bloom

## Outputs

- Per-gene estimated fragment counts per dataset from kallisto
- Per assembler-dataset normalized fraction of 95%-assembled genes (numerator from rnaQUAST divided by kallisto-detected genes)
- assembled_gene_fractions.csv containing per-dataset fractions and cross-dataset mean fractions per assembler

## How to apply

For each of the 8 real RNA-Seq datasets (Human SRR5133163; Human large SRR1957703/SRR1957706; Mouse SRX648736; Worm SRR1560107; Corn SRR1588569; Arabidopsis SRR5344669/SRR5344670), run kallisto quantification against the reference transcriptome to obtain per-gene estimated fragment counts, and flag a gene as detected when its nucleotide coverage from estimated counts exceeds 5. In parallel, run rnaQUAST on each assembler's contigs/scaffolds against the reference gene annotations to count 95%-assembled genes. For each assembler-dataset pair, divide the number of 95%-assembled genes by the number of kallisto-detected genes to obtain the normalized fraction; this rescales completeness by what is actually expressed in that sample rather than by reference gene count. Compute the cross-dataset mean fraction per assembler and write per-dataset fractions and means to assembled_gene_fractions.csv.

## Related tools

- **Kallisto** (Pseudoaligns RNA-Seq reads to the reference transcriptome to produce per-gene estimated fragment counts used to flag detected genes (coverage > 5) as the normalization denominator)
- **rnaQUAST** (Evaluates each assembler's de novo assembly against reference gene annotations to count 95%-assembled genes, the normalization numerator)
- **rnaSPAdes** (One of the assemblers whose output contigs/scaffolds are quantified and evaluated; benchmark leader on the normalized metric (0.500)) — https://github.com/ablab/spades
- **SPAdes** (Genome assembler applied unmodified to transcriptomic data; its assemblies are quantified and evaluated as a baseline (normalized mean 0.397)) — https://github.com/ablab/spades
- **Trinity** (Competing transcriptome assembler whose assemblies are evaluated with the same normalized metric (cross-dataset mean 0.438, ranked second))
- **RNA-Bloom** (Competing transcriptome assembler evaluated with the same normalized metric (cross-dataset mean 0.406, ranked third))

## Examples

```
kallisto quant -i transcripts.idx -o srr5133163_out SRR5133163_R1.fastq.gz SRR5133163_R2.fastq.gz
```

## Evaluation signals

- Cross-dataset mean normalized fractions must reproduce the reported benchmark values: rnaSPAdes 0.500, Trinity 0.438, RNA-Bloom 0.406, SPAdes 0.397, confirming rnaSPAdes as the leader.
- Every gene counted as detected has nucleotide coverage from kallisto estimated counts strictly exceeding 5.
- Per-dataset fractions are bounded in [0, 1] since they are 95%-assembled gene counts divided by kallisto-detected gene counts.
- Per-dataset fractions vary across the 8 datasets (Human, Human large, Mouse, Worm, Corn, Arabidopsis), demonstrating the denominator is dataset-specific rather than a fixed reference count.
- assembled_gene_fractions.csv contains both per-dataset fractions and cross-dataset means, and means equal the arithmetic mean of the 8 per-dataset fractions.

## Limitations

- The detection threshold (nucleotide coverage from estimated counts > 5) is an arbitrary cutoff; a different threshold changes the denominator and thus all normalized fractions.
- This metric only reflects assembly completeness: the article notes high database coverage or nucleotide recall can be achieved by very fragmented assemblies, and no single assembler leads on all metrics or datasets.
- rnaQUAST applies no cutoff on mapped fraction, which the authors say explains the absence of perfect correlation with metrics like Detonate's, so 95%-assembled gene fractions should not be conflated with other recall measures.
- Reported normalized means are averages over only 8 real datasets from 6 organisms; the article stresses that choosing a suitable assembly tool 'remains a non-trivial problem and may require additional' per-project benchmarking.

## Evidence

- [discussion] In comparison with other assemblers, rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively).: "In comparison with other assemblers, rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively)."
- [discussion] Trinity (0.44 and 0.30) and RNA-Bloom (0.41 and 0.28) are ranked second and the third according to these metrics (Supplementary Fig. S3).: "Trinity (0.44 and 0.30) and RNA-Bloom (0.41 and 0.28) are ranked second and the third according to these metrics (Supplementary Fig. S3)."
- [intro] To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies: "To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies"
- [abstract] it is not possible to detect the absolute leader according to all quality metrics and all used datasets: "it is not possible to detect the absolute leader according to all quality metrics and all used datasets"
- [discussion] no cutoff is applied for mapped fraction of the assembled sequences in rnaQUAST. This difference between algorithms might explain the absence of perfect correlation: "no cutoff is applied for mapped fraction of the assembled sequences in rnaQUAST. This difference between algorithms might explain the absence of perfect correlation"
