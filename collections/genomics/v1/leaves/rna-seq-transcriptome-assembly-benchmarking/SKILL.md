---
name: rna-seq-transcriptome-assembly-benchmarking
description: Use when comparing de novo transcriptome assemblers across datasets with
  different gene-detection depths, especially when raw counts of assembled genes are
  unfair and a Kallisto-normalized fraction of 95%-assembled genes is needed for cross-dataset
  ranking.
when_to_use_negative:
- Evaluating metatranscriptome assemblies — the article notes the approach was designed
  for RNA-Seq from a single organism and that an appropriate metatranscriptomic quality-evaluation
  pipeline is future work
- Ranking assemblers by contiguity-only metrics such as database coverage or nucleotide
  recall — the article warns these can be achieved by very fragmented assemblies or
  even raw reads
- Datasets lacking a reference genome and gene annotations, since rnaQUAST-based 95%-assembled
  gene counting is reference-dependent
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: rnaSPAdes
  role: De novo transcriptome assembler being benchmarked (top performer on 95%-assembled
    genes/isoforms)
  repo: github.com/ablab/spades
- name: SPAdes
  role: Genome assembler applied to RNA-Seq without modification as a baseline in
    the benchmark
  repo: github.com/ablab/spades
- name: Trinity
  role: State-of-the-art transcriptome assembler used as a benchmark competitor
- name: RNA-Bloom
  role: Transcriptome assembler used as a benchmark competitor
- name: rnaQUAST
  role: Quality-evaluation tool that counts 95%-assembled genes per assembly against
    reference gene annotations
- name: Kallisto
  role: Quantification tool whose estimated fragment counts define the per-gene detection
    denominator (nucleotide coverage > 5)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:a3d9396421fdc5edaac8034b1a115b13194ee4924a80dcc89220c753ede3ffff
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
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# rna-seq-transcriptome-assembly-benchmarking

## Summary

Benchmarks de novo transcriptome assemblers (rnaSPAdes, SPAdes, Trinity, RNA-Bloom) across multiple real RNA-Seq datasets by combining rnaQUAST assembly-completeness counts (95%-assembled genes) with Kallisto-based gene detection, yielding a detection-normalized fraction that is comparable across datasets and assemblers.

## When to use

Use when comparing de novo transcriptome assemblers across datasets with different gene-detection depths, especially when raw counts of assembled genes are unfair and a Kallisto-normalized fraction of 95%-assembled genes is needed for cross-dataset ranking.

## When NOT to use

- Evaluating metatranscriptome assemblies — the article notes the approach was designed for RNA-Seq from a single organism and that an appropriate metatranscriptomic quality-evaluation pipeline is future work
- Ranking assemblers by contiguity-only metrics such as database coverage or nucleotide recall — the article warns these can be achieved by very fragmented assemblies or even raw reads
- Datasets lacking a reference genome and gene annotations, since rnaQUAST-based 95%-assembled gene counting is reference-dependent

## Inputs

- Real RNA-Seq datasets from NCBI SRA (FASTQ): SRR5133163, SRR1957703, SRR1957706, SRX648736, SRR1560107, SRR1588569, SRR5344669, SRR5344670
- De novo transcriptome assemblies (contigs/scaffolds, FASTA) from rnaSPAdes, SPAdes, Trinity, and RNA-Bloom
- Reference genome and gene annotations (GTF/GFF) for rnaQUAST evaluation

## Outputs

- Per-dataset counts of 95%-assembled genes from rnaQUAST
- Per-gene Kallisto estimated fragment counts and detection flags (nucleotide coverage > 5)
- assembled_gene_fractions.csv with per-dataset normalized fractions and cross-dataset mean fraction per assembler

## How to apply

Download the real RNA-Seq datasets from NCBI SRA (e.g. Human SRR5133163; Human large SRR1957703/SRR1957706; Mouse SRX648736; Worm SRR1560107; Corn SRR1588569; Arabidopsis SRR5344669/SRR5344670) and the de novo assemblies produced by each assembler. Run rnaQUAST against the reference gene annotations to count 95%-assembled genes per assembler-dataset pair, and run Kallisto quantification per dataset, flagging a gene as detected when its nucleotide coverage from estimated fragment counts exceeds 5. Divide the number of 95%-assembled genes by the number of Kallisto-detected genes to obtain a normalized fraction per assembler-dataset pair; this normalization is the key rationale, because it adjusts for dataset-specific gene detectability and enables fair cross-dataset comparison. Average the per-dataset fractions per assembler and write per-dataset values and cross-dataset means to a CSV (assembled_gene_fractions.csv). Validate by checking that computed means reproduce the published benchmark values (rnaSPAdes 0.500, Trinity 0.438, RNA-Bloom 0.406, SPAdes 0.397).

## Related tools

- **rnaSPAdes** (De novo transcriptome assembler being benchmarked (top performer on 95%-assembled genes/isoforms)) — github.com/ablab/spades
- **SPAdes** (Genome assembler applied to RNA-Seq without modification as a baseline in the benchmark) — github.com/ablab/spades
- **Trinity** (State-of-the-art transcriptome assembler used as a benchmark competitor)
- **RNA-Bloom** (Transcriptome assembler used as a benchmark competitor)
- **rnaQUAST** (Quality-evaluation tool that counts 95%-assembled genes per assembly against reference gene annotations)
- **Kallisto** (Quantification tool whose estimated fragment counts define the per-gene detection denominator (nucleotide coverage > 5))

## Examples

```
rnaQUAST.py --transcripts rnaspades_transcripts.fasta --reference genome.fa --gtf genes.gtf --output-dir rnaquast_out
```

## Evaluation signals

- Computed cross-dataset mean fractions match the reported benchmark values: rnaSPAdes 0.500, Trinity 0.438, RNA-Bloom 0.406, SPAdes 0.397
- Every assembler-dataset pair has a finite normalized fraction, i.e. the Kallisto-detected gene count (coverage > 5) is nonzero and ≥ the count of 95%-assembled genes so fractions stay ≤ 1
- rnaQUAST output reports a nonzero number of 95%-assembled genes for each of the 8 datasets per assembler
- assembled_gene_fractions.csv contains per-dataset fractions plus one mean per assembler and no missing assembler-dataset combinations

## Limitations

- No single assembler is the absolute leader across all quality metrics and all datasets — assembler choice may require additional per-project benchmarks
- The Kallisto-normalized gene metric does not capture duplication or precision: SPAdes has a smaller mean duplication ratio (2% vs 32% for rnaSPAdes) and higher nucleotide precision
- Small k-mer sizes (e.g. k=21 in the first iteration) help assemble extremely low-covered genes but carry pitfalls for transcriptome assembly, explaining rare underperformance on low-coverage data
- Metric differences between Detonate and rnaQUAST (rnaQUAST applies no cutoff on mapped fraction) mean 95%-assembled isoform counts do not perfectly correlate across evaluation tools

## Evidence

- [discussion] rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively).: "In comparison with other assemblers, rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively)."
- [intro] To perform the benchmarking we have used the rnaQUAST tool: "To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies"
- [methods] Human: SRR5133163: "Human large: SRR1957703, SRR1957706"
- [readme] SPAdes is a versatile toolkit designed for assembly and analysis of sequencing data.: "SPAdes is a versatile toolkit designed for assembly and analysis of sequencing data. SPAdes is primarily developed for Illumina sequencing data, but can be used for IonTorrent as well."
- [abstract] it is not possible to detect the absolute leader according to all quality metrics and all used datasets: "it is not possible to detect the absolute leader according to all quality metrics and all used datasets"
