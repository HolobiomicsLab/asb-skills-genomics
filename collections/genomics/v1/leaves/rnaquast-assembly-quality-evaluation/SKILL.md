---
name: rnaquast-assembly-quality-evaluation
description: Use when you have de novo transcriptome contigs (e.g. rnaSPAdes/Trinity
  output) plus a reference annotation and need quality metrics such as 95%-assembled
  genes, isoforms, database coverage, misassemblies, or duplication ratio.
when_to_use_negative:
- Evaluating metatranscriptome assemblies — the paper states no appropriate quality-evaluation
  pipeline for metatranscriptomic assemblies was designed and designing one is future
  work
- Judging assembly contiguity from database coverage or nucleotide recall alone, since
  these metrics can be maximized by very fragmented assemblies or raw reads
- Comparing against Detonate scores expecting perfect agreement — the absence of a
  mapped-fraction cutoff in rnaQUAST means the two metrics do not perfectly correlate
edam_operation: http://edamontology.org/operation_0544
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: rnaQUAST
  role: Quality evaluation tool for de novo transcriptome assemblies; computes 95%-assembled
    genes/isoforms, database coverage, misassemblies, and duplication ratio against
    a reference transcriptome
- name: rnaSPAdes
  role: 'Produces the transcriptome assembly under evaluation (default multi-k schedule:
    lower k ≈ read_length/3 min 29, upper k = largest odd number ≤ read_length/2 −
    1)'
  repo: github.com/ablab/spades
- name: SPAdes
  role: Baseline genome assembler run in single-cell mode (k = 21, 33, 55) and hybrid
    mode (--pacbio/--nanopore) for comparison assemblies evaluated with the same rnaQUAST
    configuration
  repo: github.com/ablab/spades
- name: Trinity
  role: Competing de novo transcriptome assembler; its output is evaluated side-by-side
    with rnaQUAST under identical settings
- name: RNA-Bloom
  role: Competing de novo transcriptome assembler benchmarked with rnaQUAST; closest
    competitor after Trinity on 95%-assembled genes
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_001
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:98c24e2730fb701a1fee61b6ea7a90aeecd32fb1778583152a6ece36408e9365
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BinPacker
  - Bridger
  - IDBA-tran
  - RNA-Bloom
  - SOAPdenovo-Trans
  - SPAdes
  - Trans-ABySS
  - Trinity
  - rnaQUAST
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0544
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# rnaquast-assembly-quality-evaluation

## Summary

Evaluate de novo transcriptome assemblies with rnaQUAST against a reference transcriptome, extracting 95%-assembled genes/isoforms, database coverage, misassemblies, and duplication ratio. This is the benchmarking approach used to rank rnaSPAdes against Trinity, RNA-Bloom, Trans-ABySS and other assemblers in the rnaSPAdes paper.

## When to use

Use when you have de novo transcriptome contigs (e.g. rnaSPAdes/Trinity output) plus a reference annotation and need quality metrics such as 95%-assembled genes, isoforms, database coverage, misassemblies, or duplication ratio.

## When NOT to use

- Evaluating metatranscriptome assemblies — the paper states no appropriate quality-evaluation pipeline for metatranscriptomic assemblies was designed and designing one is future work
- Judging assembly contiguity from database coverage or nucleotide recall alone, since these metrics can be maximized by very fragmented assemblies or raw reads
- Comparing against Detonate scores expecting perfect agreement — the absence of a mapped-fraction cutoff in rnaQUAST means the two metrics do not perfectly correlate

## Inputs

- De novo transcriptome assembly FASTA (e.g. rnaSPAdes transcripts.fasta, Trinity output)
- Reference transcriptome / genome annotation for the target organism
- Optionally, simulated or real RNA-Seq reads (e.g. Human SRR5133163) used to produce the assemblies

## Outputs

- rnaQUAST report (per-assembly metrics: 95%-assembled genes, 95%-assembled isoforms, database coverage, misassemblies, duplication ratio)
- Comparative CSV table with one row per assembler mirroring the paper's Tables 3/4
- rnaquast_report.tsv / side-by-side baseline comparison reports

## How to apply

Run rnaQUAST on each assembler's transcript contigs against the organism's reference transcriptome, using the same evaluation configuration for every assembly so metrics are comparable. Extract the four benchmark metrics used in the paper (fraction of 95%-assembled genes, fraction of 95%-assembled isoforms, database coverage, number of misassemblies, plus duplication ratio) from the rnaQUAST reports and compile them into a CSV with one row per assembler. Judge completeness primarily by 95%-assembled genes/isoforms rather than database coverage alone, because the paper notes that high database coverage or nucleotide recall can be achieved by very fragmented assemblies or even raw reads. Apply paper-level anchors when reproducing the Human benchmarks: rnaSPAdes best-case values are 0.50 for 95%-assembled genes and 0.32 for isoforms, with Trinity (0.44/0.30) and RNA-Bloom (0.41/0.28) ranked second and third. Be aware that rnaQUAST applies no cutoff on mapped fraction of assembled sequences, which is a known difference from Detonate and explains imperfect correlation between contig recall and 95%-assembled isoforms.

## Related tools

- **rnaQUAST** (Quality evaluation tool for de novo transcriptome assemblies; computes 95%-assembled genes/isoforms, database coverage, misassemblies, and duplication ratio against a reference transcriptome)
- **rnaSPAdes** (Produces the transcriptome assembly under evaluation (default multi-k schedule: lower k ≈ read_length/3 min 29, upper k = largest odd number ≤ read_length/2 − 1)) — github.com/ablab/spades
- **SPAdes** (Baseline genome assembler run in single-cell mode (k = 21, 33, 55) and hybrid mode (--pacbio/--nanopore) for comparison assemblies evaluated with the same rnaQUAST configuration) — github.com/ablab/spades
- **Trinity** (Competing de novo transcriptome assembler; its output is evaluated side-by-side with rnaQUAST under identical settings)
- **RNA-Bloom** (Competing de novo transcriptome assembler benchmarked with rnaQUAST; closest competitor after Trinity on 95%-assembled genes)

## Examples

```
rnaQUAST.py --transcripts rnaSPAdes_transcripts.fasta --reference human_transcripts.fasta --gtf human_annotation.gtf -o rnaquast_out
```

## Evaluation signals

- On the Human simulated dataset, rnaQUAST-derived misassembly counts should match published anchors (rnaSPAdes 309, Trans-ABySS 126) and duplication ratios 1.74/1.93/2.19 for the leading assemblers
- On the Human real dataset (SRR5133163), rnaSPAdes should lead 95%-assembled genes with ~16%/31% more than RNA-Bloom and Trinity, and Trinity misassemblies should exceed 5,000
- The compiled CSV must contain one row per assembler (or per dataset per configuration) with the four rnaQUAST metric columns populated
- rnaSPAdes' best-case reference values should approach 0.50 (95%-assembled genes) and 0.32 (95%-assembled isoforms) across the paper's datasets
- All assemblies must be evaluated with identical rnaQUAST configuration so metric differences reflect the assemblies, not the evaluator settings

## Limitations

- High database coverage or nucleotide recall can be achieved by a very fragmented assembly (or even raw reads), so these metrics do not reflect assembly contiguity
- rnaQUAST applies no cutoff on mapped fraction of assembled sequences (unlike Detonate), so contig recall and 95%-assembled isoforms do not perfectly correlate
- No single assembler is the absolute leader across all quality metrics and all datasets; the choice of tool may require additional per-project benchmarks

## Evidence

- [intro] rnaQUAST for reference-based evaluation: "To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies"
- [discussion] best 95%-assembled genes/isoforms values: "rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively)."
- [discussion] fragmented assemblies can game coverage metrics: "high database coverage or nucleotide recall can be achieved by a very fragmented assembly (or even raw reads)"
- [discussion] Detonate vs rnaQUAST cutoff difference: "no cutoff is applied for mapped fraction of the assembled sequences in rnaQUAST. This difference between algorithms might explain the absence of perfect correlation"
- [abstract] no absolute leader across metrics: "it is not possible to detect the absolute leader according to all quality metrics and all used datasets"
