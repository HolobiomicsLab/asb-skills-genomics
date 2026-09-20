---
name: de-bruijn-graph-k-mer-parameter-selection
description: 'Use when running rnaSPAdes (or SPAdes-family assemblers) on Illumina
  RNA-Seq reads and you must decide the multi-k schedule: lower k ≈ read_length/3
  (min 29) and upper k ≈ largest odd number ≤ read_length/2 − 1.'
when_to_use_negative:
- Input is long reads (Oxford Nanopore/PacBio) rather than short Illumina reads —
  the k-mer de Bruijn schedule is designed for short-read RNA-Seq
- Data is metatranscriptomic; the k selection logic was designed for single-organism
  RNA-Seq and the authors note the approach 'was not designed or evaluated for metatranscriptome
  assembly'
- Assembly quality is limited by transcript filtration thresholds rather than k coverage
  — tune filtration (Supplementary Table S14) instead of re-deriving the k schedule
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: rnaSPAdes
  role: Transcriptome assembler whose iterative de Bruijn graph construction consumes
    the chosen k-mer schedule on RNA-Seq reads
  repo: https://github.com/ablab/spades
- name: rnaQUAST
  role: Downstream evaluator used to verify that the selected k schedule yields competitive
    95%-assembled genes/isoforms metrics
- name: Bandage
  role: Optional visualization of the resulting SPAdes assembly graph to inspect graph
    structure produced under the chosen k values
  repo: https://github.com/rrwick/Bandage/wiki
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:74beb8597caddc97ecba2cfac1ca560db73caacb978dbf6f2029203aff9d9b54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/spades
  tools:
  - Bandage
  - rnaQUAST
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# de-bruijn-graph-k-mer-parameter-selection

## Summary

Selects the set of de Bruijn graph k-mer sizes (and the 200 bp contig cutoff) used when assembling transcriptomic reads with rnaSPAdes, since k choice directly governs recovery of low-coverage genes versus erroneous edges in RNA-Seq assemblies.

## When to use

Use when running rnaSPAdes (or SPAdes-family assemblers) on Illumina RNA-Seq reads and you must decide the multi-k schedule: lower k ≈ read_length/3 (min 29) and upper k ≈ largest odd number ≤ read_length/2 − 1.

## When NOT to use

- Input is long reads (Oxford Nanopore/PacBio) rather than short Illumina reads — the k-mer de Bruijn schedule is designed for short-read RNA-Seq
- Data is metatranscriptomic; the k selection logic was designed for single-organism RNA-Seq and the authors note the approach 'was not designed or evaluated for metatranscriptome assembly'
- Assembly quality is limited by transcript filtration thresholds rather than k coverage — tune filtration (Supplementary Table S14) instead of re-deriving the k schedule

## Inputs

- Paired-end Illumina RNA-Seq reads in FASTQ (e.g. NCBI SRA SRR5133163 Human dataset)
- Known read length used to derive lower and upper k-mer sizes

## Outputs

- rnaSPAdes assembly graph and transcript contigs assembled over the iterative k schedule
- Set of transcript contigs >= 200 bp cutoff for rnaQUAST evaluation

## How to apply

Compute the k-mer schedule from the read length instead of guessing: set the smallest k to approximately read_length/3 (minimum 29) and the largest k to the largest odd number not exceeding read_length/2 − 1, letting rnaSPAdes iterate its condensed de Bruijn graph construction through intermediate odd k values. Rationale: iterative multi-k construction is what allows SPAdes-family tools to handle the non-uniform coverage depth typical of transcriptomic data, and a small first k (e.g. k = 21) lets SPAdes assemble extremely low-covered genes where read overlaps are short, though small k-mers bring their own pitfalls in transcriptome assembly. Leave k-mer sizes at their defaults for benchmark-style runs (as in the paper's SRR5133163 Human dataset assembly with default parameters), and apply the specified contig cut-off (200 bp) before downstream evaluation with rnaQUAST. If assembly completeness is poor, vary transcript filtration thresholds rather than immediately rescaling k (Table S14 in the Supplementary shows filtration yields assemblies with different properties).

## Related tools

- **rnaSPAdes** (Transcriptome assembler whose iterative de Bruijn graph construction consumes the chosen k-mer schedule on RNA-Seq reads) — https://github.com/ablab/spades
- **rnaQUAST** (Downstream evaluator used to verify that the selected k schedule yields competitive 95%-assembled genes/isoforms metrics)
- **Bandage** (Optional visualization of the resulting SPAdes assembly graph to inspect graph structure produced under the chosen k values) — https://github.com/rrwick/Bandage/wiki

## Examples

```
rnaspades.py --pe1-1 SRR5133163_1.fastq --pe1-2 SRR5133163_2.fastq -o human_rnaspades -c 200
```

## Evaluation signals

- K-mer schedule matches the formulas: lower k ≈ read_length/3 (min 29), upper k = largest odd number ≤ read_length/2 − 1
- All output contigs below the 200 bp cut-off are discarded before rnaQUAST evaluation
- rnaQUAST on the Human real dataset should show rnaSPAdes with the highest fraction of 95%-assembled genes (0.50) and isoforms (0.32), ~16%/31% above RNA-Bloom/Trinity
- Assembly graph builds without coverage collapses; isolated edges with coverage < 2 and length ≤ read length are removed during simplification
- Extremely low-coverage genes are recovered (small first k keeps short read overlaps), while misassemblies stay low (Trinity, by contrast, exceeds 5,000 misassemblies)

## Limitations

- Small k-mer sizes (e.g. k = 21) in the first iteration help assemble low-covered genes but carry pitfalls for transcriptome assembly discussed only in the Methods
- No absolute-leader k setting exists: 'it is not possible to detect the absolute leader according to all quality metrics and all used datasets'
- High database coverage can come from fragmented assemblies, so completeness metrics alone do not validate k choice
- rnaSPAdes has a higher mean duplication ratio (32% vs 2% for SPAdes); small-k-driven sensitivity trades off against duplication

## Evidence

- [supplementary] default k-mer sizes (lower k ≈ read_length/3, min 29; upper k ≈ largest odd number ≤ read_length/2 − 1): [evidence span withheld — anchor status: not_found]
- [methods] Iterative de Bruijn graph construction rationale: "the key techniques that allows SPAdes to assemble contiguous genomic sequences from data with non-uniform coverage depth is the iterative de Bruijn graph construction"
- [discussion] Small k = 21 helps low-coverage genes: "By using small k = 21 during the first iteration SPAdes manages to assemble extremely low-covered genes, where overlaps between reads are short."
- [discussion] Filtration thresholds offer an alternative tuning axis: "various thresholds for transcript filtration in rnaSPAdes (Table S14 in the Supplementary material) result in assemblies with different properties."
- [methods] Isolated-edge removal depends on coverage and read length: "during graph simplification we additionally remove isolated edges that have both (i) low coverage (<2) and (ii) length smaller than or equal to read length"
- [discussion] rnaSPAdes best 95%-assembled metrics under benchmark configuration: "rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively)."
