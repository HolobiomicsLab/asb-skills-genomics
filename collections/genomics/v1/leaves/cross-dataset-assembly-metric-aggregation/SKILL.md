---
name: cross-dataset-assembly-metric-aggregation
description: Use when comparing de novo transcriptome assembler quality across datasets
  whose gene-detection depth differs, and raw counts of 95%-assembled genes are not
  directly comparable without Kallisto-based normalization.
when_to_use_negative:
- The goal is metatranscriptome assembly evaluation — the paper states its approach
  was designed for RNA-Seq from a single organism and quality evaluation for metatranscriptomic
  assemblies is unaddressed future work
- Comparing a single assembler on a single dataset — normalization across datasets
  adds nothing without cross-dataset aggregation
- Evaluating contiguity or duplication quality (e.g. mean duplication ratio, BUSCOs)
  — this skill targets only the completeness metric of 95%-assembled genes
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3308
tools:
- name: rnaQUAST
  role: Counts 95%-assembled genes per assembler assembly against reference gene annotations
- name: Kallisto
  role: Quantifies per-gene estimated fragment counts used to flag detected genes
    (nucleotide coverage > 5) as the normalization denominator
- name: rnaSPAdes
  role: Source of one set of de novo transcriptome assemblies being compared; leader
    on the normalized metric (0.500)
  repo: github.com/ablab/spades
- name: SPAdes
  role: Source of comparison assemblies (unmodified genome assembler baseline); mean
    fraction 0.397
  repo: github.com/ablab/spades
- name: Trinity
  role: Source of comparison assemblies; second-ranked on the normalized metric (0.438)
- name: RNA-Bloom
  role: Source of comparison assemblies; third-ranked on the normalized metric (0.406)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:2475324d01e5fd3b0dde7c750289dabd9070291340c39e9f231c5a46b86f4447
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-dataset-assembly-metric-aggregation

## Summary

Normalizes the fraction of 95%-assembled genes (computed by rnaQUAST) per assembler–dataset pair by the number of Kallisto-detected genes, then averages across 8 real RNA-Seq datasets to enable fair cross-dataset comparison of de novo transcriptome assemblers (rnaSPAdes, SPAdes, Trinity, RNA-Bloom).

## When to use

Use when comparing de novo transcriptome assembler quality across datasets whose gene-detection depth differs, and raw counts of 95%-assembled genes are not directly comparable without Kallisto-based normalization.

## When NOT to use

- The goal is metatranscriptome assembly evaluation — the paper states its approach was designed for RNA-Seq from a single organism and quality evaluation for metatranscriptomic assemblies is unaddressed future work
- Comparing a single assembler on a single dataset — normalization across datasets adds nothing without cross-dataset aggregation
- Evaluating contiguity or duplication quality (e.g. mean duplication ratio, BUSCOs) — this skill targets only the completeness metric of 95%-assembled genes

## Inputs

- 8 real RNA-Seq datasets from NCBI SRA (SRR5133163; SRR1957703; SRR1957706; SRX648736; SRR1560107; SRR1588569; SRR5344669; SRR5344670)
- De novo assemblies (contigs/scaffolds) from rnaSPAdes, SPAdes, Trinity, and RNA-Bloom per dataset
- Reference gene annotations per organism for rnaQUAST evaluation
- Kallisto index and FASTQ read pairs for quantification

## Outputs

- Per-dataset normalized fraction of 95%-assembled genes per assembler
- Cross-dataset mean fraction per assembler (rnaSPAdes 0.500, Trinity 0.438, RNA-Bloom 0.406, SPAdes 0.397)
- assembled_gene_fractions.csv containing per-dataset fractions and cross-dataset means

## How to apply

Download the 8 real RNA-Seq datasets from NCBI SRA (Human SRR5133163; Human large SRR1957703/SRR1957706; Mouse SRX648736; Worm SRR1560107; Corn SRR1588569; Arabidopsis SRR5344669/SRR5344670) and obtain the de novo contig/scaffold assemblies from rnaSPAdes, SPAdes, Trinity, and RNA-Bloom. Run Kallisto quantification per dataset to get per-gene estimated fragment counts, flagging a gene as detected when its nucleotide coverage from estimated counts exceeds 5. Run rnaQUAST against reference gene annotations to count 95%-assembled genes per assembler–dataset pair. For each pair, divide 95%-assembled genes by Kallisto-detected genes, compute the cross-dataset mean per assembler, and write per-dataset fractions plus means to assembled_gene_fractions.csv. The rationale is that normalizing by the number of detectable genes removes dataset-specific detection-depth bias; expected validation means are rnaSPAdes 0.500, Trinity 0.438, RNA-Bloom 0.406, SPAdes 0.397.

## Related tools

- **rnaQUAST** (Counts 95%-assembled genes per assembler assembly against reference gene annotations)
- **Kallisto** (Quantifies per-gene estimated fragment counts used to flag detected genes (nucleotide coverage > 5) as the normalization denominator)
- **rnaSPAdes** (Source of one set of de novo transcriptome assemblies being compared; leader on the normalized metric (0.500)) — github.com/ablab/spades
- **SPAdes** (Source of comparison assemblies (unmodified genome assembler baseline); mean fraction 0.397) — github.com/ablab/spades
- **Trinity** (Source of comparison assemblies; second-ranked on the normalized metric (0.438))
- **RNA-Bloom** (Source of comparison assemblies; third-ranked on the normalized metric (0.406))

## Examples

```
kallisto quant -i transcripts.idx -o human_srr5133163 sample_R1.fastq sample_R2.fastq; python aggregate_assembled_gene_fractions.py --rnaquast_reports rq/*.tsv --kallisto_dirs human_srr5133163 human_large mouse worm corn arabidopsis --coverage_cutoff 5 --output assembled_gene_fractions.csv
```

## Evaluation signals

- Computed cross-dataset means must match the reported benchmark values: rnaSPAdes 0.500, Trinity 0.438, RNA-Bloom 0.406, SPAdes 0.397
- rnaSPAdes mean fraction must be ~26% higher than unmodified SPAdes on the normalized metric
- Per-gene detection flags must use the nucleotide coverage cutoff of 5 from Kallisto estimated counts, not raw read counts
- assembled_gene_fractions.csv must contain both per-dataset fractions and the cross-dataset mean column for all 4 assemblers
- All 8 datasets (2 human, 1 human large, 1 mouse, 1 worm, 1 corn, 2 arabidopsis) must be represented per assembler before averaging

## Limitations

- No single transcriptome assembler is the absolute leader across all quality metrics and all datasets, so this aggregation captures only the 95%-assembled-genes completeness axis
- rnaQUAST applies no cutoff on mapped fraction of assembled sequences (unlike Detonate), so contig recall and 95%-assembled isoforms do not perfectly correlate with other tools' metrics
- Erroneous and duplicated assembly sequences may negatively affect downstream analysis such as gene annotation, and are not penalized by this normalized completeness fraction

## Evidence

- [discussion] rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively).: "rnaSPAdes shows the highest fraction of 95%-assembled genes and isoforms (0.50 and 0.32, respectively)."
- [discussion] Trinity and RNA-Bloom rankings: "Trinity (0.44 and 0.30) and RNA-Bloom (0.41 and 0.28) are ranked second and the third according to these metrics (Supplementary Fig. S3)."
- [intro] rnaQUAST as the benchmarking tool: "To perform the benchmarking we have used the rnaQUAST tool [16], which was designed for quality evaluation of de novo assemblies"
- [discussion] 26% higher fraction over SPAdes: "it has significantly better assembly completeness metrics: 26% higher average fraction of 95%-assembled genes, 18% larger database coverage"
- [readme] SPAdes transcriptomic pipeline context: "SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data."
