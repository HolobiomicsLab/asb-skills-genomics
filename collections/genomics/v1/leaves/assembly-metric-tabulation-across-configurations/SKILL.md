---
name: assembly-metric-tabulation-across-configurations
description: Use when benchmarking how a single k-mer size deviates from rnaSPAdes'
  default automatically calculated multi-k schedule and you need rnaQUAST metrics
  (95%-assembled genes/isoforms, database coverage, misassemblies) tabulated per dataset
  per configuration.
when_to_use_negative:
- The goal is comparing assemblers rather than ablating k-mer schedules within rnaSPAdes
  — a different benchmarking design (Supplementary Fig. S3-style comparisons vs Trinity,
  RNA-Bloom, Trans-ABySS) is needed.
- The question is how transcript filtration thresholds (Table S14 in the Supplementary
  material) affect assembly properties, not how k-mer size does.
- Input is simulated data or a single dataset only; the ablation's conclusions about
  the completeness-vs-misassemblies trade-off rest on the multi-dataset real SRA set.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: rnaSPAdes
  role: de novo transcriptome assembler run once with the default multi-k schedule
    and once with a single k-mer size per dataset
  repo: github.com/ablab/spades
- name: rnaQUAST
  role: quality evaluation of the de novo assemblies, extracting 95%-assembled genes/isoforms,
    database coverage, and misassemblies
- name: SPAdes
  role: base assembler whose iterative de Bruijn graph construction rnaSPAdes builds
    on for multi-k assemblies
  repo: github.com/ablab/spades
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:e506c7d4fabbf83688d9f126d33e852b827919ef9d59fafa0894a63c04aae692
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPAdes
  - rnaQUAST
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# assembly-metric-tabulation-across-configurations

## Summary

Reproduces a controlled ablation of rnaSPAdes k-mer schedules (default multi-k vs single-k) across real RNA-Seq datasets and tabulates rnaQUAST-derived metrics per dataset per configuration into a side-by-side CSV comparison.

## When to use

Use when benchmarking how a single k-mer size deviates from rnaSPAdes' default automatically calculated multi-k schedule and you need rnaQUAST metrics (95%-assembled genes/isoforms, database coverage, misassemblies) tabulated per dataset per configuration.

## When NOT to use

- The goal is comparing assemblers rather than ablating k-mer schedules within rnaSPAdes — a different benchmarking design (Supplementary Fig. S3-style comparisons vs Trinity, RNA-Bloom, Trans-ABySS) is needed.
- The question is how transcript filtration thresholds (Table S14 in the Supplementary material) affect assembly properties, not how k-mer size does.
- Input is simulated data or a single dataset only; the ablation's conclusions about the completeness-vs-misassemblies trade-off rest on the multi-dataset real SRA set.

## Inputs

- FASTQ RNA-Seq reads from NCBI SRA (SRR5133163, SRR1957703, SRR1957706, SRX648736, SRR1560107, SRR1588569, SRR5344669, SRR5344670)
- read length per dataset (to derive the default k-mer schedule: upper k ≤ read_length/2 - 1, lower k ≈ read_length/3, min 29)
- reference gene/isoform databases used by rnaQUAST for evaluation

## Outputs

- per-dataset de novo transcriptome assemblies for the default multi-k and single-k configurations
- rnaQUAST evaluation reports with fraction of 95%-assembled genes, fraction of 95%-assembled isoforms, database coverage, and misassembly counts
- kmer_comparison.csv: one row per dataset per configuration with the rnaQUAST-derived metric columns

## How to apply

Download the real RNA-Seq datasets from NCBI SRA (Human SRR5133163; Human large SRR1957703/SRR1957706; Mouse SRX648736; Worm SRR1560107; Corn SRR1588569; Arabidopsis SRR5344669/SRR5344670). Run rnaSPAdes with the default k-mer schedule — upper k is the largest odd number not exceeding read_length/2 minus 1, lower k is approximately read_length/3 with a minimum of 29 — then re-run the same datasets with a single k (e.g. the upper k alone), relying on iterative de Bruijn graph construction over the specified k values. Evaluate every assembly with rnaQUAST, extracting the representative metrics reported in Supplementary Tables S9–S11: fraction of 95%-assembled genes, fraction of 95%-assembled isoforms, database coverage, and number of misassemblies. Tabulate one row per dataset per configuration (default multi-k vs single-k) with the rnaQUAST-derived columns plus assembled gene/isoform counts and misassemblies, and write the consolidated comparison to kmer_comparison.csv. The expected direction of the trade-off is that small k-mer sizes (e.g. k = 21 in the first iteration) enable assembly of extremely low-covered genes with short read overlaps, while single-k runs capture fewer genes and isoforms but produce fewer misassembled contigs.

## Related tools

- **rnaSPAdes** (de novo transcriptome assembler run once with the default multi-k schedule and once with a single k-mer size per dataset) — github.com/ablab/spades
- **rnaQUAST** (quality evaluation of the de novo assemblies, extracting 95%-assembled genes/isoforms, database coverage, and misassemblies)
- **SPAdes** (base assembler whose iterative de Bruijn graph construction rnaSPAdes builds on for multi-k assemblies) — github.com/ablab/spades

## Examples

```
rnaspades.py -1 SRR5133163_R1.fastq.gz -2 SRR5133163_R2.fastq.gz -o human_singlek -k 55
```

## Evaluation signals

- kmer_comparison.csv contains exactly one row per dataset per configuration (6 datasets × 2 configurations) with the rnaQUAST-derived columns populated.
- Single-k rows show fewer assembled genes and isoforms than the corresponding default multi-k rows, matching the reported trade-off.
- Single-k rows show fewer misassemblies than default multi-k rows (with known exceptions such as Corn SS and Human large datasets where this pattern can flip).
- Default multi-k schedule values are consistent with the rule: upper k = largest odd number ≤ read_length/2 − 1, lower k ≈ read_length/3 with a minimum of 29.
- All metric values are traceable to rnaQUAST output and reproducible against Supplementary Tables S9–S11.

## Limitations

- Single-k assemblies yield fewer captured genes and isoforms, so completeness suffers relative to the default multi-k schedule.
- Small k-mer sizes (e.g. k = 21) carry known pitfalls in transcriptome assembly even though they recover extremely low-covered genes with short read overlaps.
- Misassembly counts do not uniformly favor single-k: datasets such as Corn SS and Human large are exceptions where multi-k is not worse.
- rnaQUAST reports no cutoff on mapped fraction, so these tabulated metrics differ from Detonate-based recall and do not perfectly correlate across tools.

## Evidence

- [intro] To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies: "To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies"
- [methods] iterative de Bruijn graph construction is the core multi-k mechanism: "the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction"
- [discussion] small k enables recovery of extremely low-covered genes: "By using small k = 21 during the first iteration SPAdes manages to assemble extremely low-covered genes, where overlaps between reads are short."
- [discussion] rnaQUAST representative metric values reported in the paper: "rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively)."
- [readme] SPAdes package provides the transcriptomic assembly pipeline used here: "SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data."
