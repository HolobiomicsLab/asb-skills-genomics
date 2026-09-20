---
name: salmon-transcript-quantification-exn50-calculation
description: Use when comparing multiple transcriptome assembly strategies (e.g. IsoSeq-only,
  merged PacBio+Illumina, Illumina-only) and raw N50 alone would be mis without expression-weighting.
when_to_use_negative:
- When the goal is coding/noncoding annotation of a single assembly rather than cross-assembly
  comparison of contiguity and completeness
- When assemblies differ only marginally and raw N50 with rnaQUAST/SQANTI statistics
  suffices
- When no expression quantification is feasible (e.g. de novo assemblies without adequate
  read support)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: Salmon
  role: Quantify transcript abundance in each assembly for expression-weighted ExN50
    computation
- name: StringTie
  role: Assemble the transcriptomes being compared using settings -p 1 -j 2 -c 2 -v
    -a 4
- name: minimap2
  role: Align PacBio IsoSeq reads against galGal6 at ~34x depth as input to IsoSeq
    assembly
- name: HISAT2
  role: Align fastp-trimmed Illumina reads against galGal6 for the Illumina-only and
    merged assemblies
- name: BUSCO
  role: Provide the completed-BUSCO (aves_odb10) completeness metric tabulated alongside
    Ex90N50
- name: rnaQUAST
  role: Complementary assembly quality assessment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:72a7ec98ff3caf2e28fed323cd52fd3925e251770defbf292cbcf8d7c77c5553
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  - HISAT2
  - IsoSeq
  - SQANTI
  - Salmon
  - StringTie
  - fastp
  - minimap2
  - rnaQUAST
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Salmon transcript quantification and ExN50 calculation

## Summary

Quantify transcript abundance in each candidate transcriptome assembly with Salmon and compute the expression-weighted Ex90N50 — the N50 of transcripts contributing 90% of total expressed signal — to compare assembly contiguity on equal footing.

## When to use

Use when comparing multiple transcriptome assembly strategies (e.g. IsoSeq-only, merged PacBio+Illumina, Illumina-only) and raw N50 alone would be misleading without expression-weighting.

## When NOT to use

- When the goal is coding/noncoding annotation of a single assembly rather than cross-assembly comparison of contiguity and completeness
- When assemblies differ only marginally and raw N50 with rnaQUAST/SQANTI statistics suffices
- When no expression quantification is feasible (e.g. de novo assemblies without adequate read support)

## Inputs

- StringTie GTF assemblies (IsoSeq-only, merged PacBio+Illumina, Illumina-only) built with -p 1 -j 2 -c 2 -v -a 4
- Chicken brain RNA-seq reads from ENA PRJEB36569 (PacBio) and PRJEB36584 (Illumina)
- Reference genome galGal6 (GCA_000002315.5) BAM alignments from minimap2 and HISAT2

## Outputs

- Salmon per-transcript abundance quantification for each of the three assemblies
- Ex90N50 statistic per assembly (expression-weighted N50 at 90% of total expressed signal)
- ex90n50_busco_metrics.csv tabulating Ex90N50 and completed-BUSCO values per assembly

## How to apply

After building each assembly with StringTie using the fixed parameter set -p 1 -j 2 -c 2 -v -a 4 (from IsoSeq PacBio alignments at ~34x depth via minimap2 against galGal6, fastp/HISAT2-trimmed Illumina alignments, or their merged BAM), run Salmon on each assembly to obtain per-transcript abundance estimates. Feed the Salmon quantification plus assembly transcript sequences/lengths into the Salmon-based ExN50 script, which computes the N50 restricted to the set of transcripts contributing 90% of total expressed signal. Tabulate the resulting Ex90N50 per assembly alongside completed-BUSCO counts (aves_odb10) in a metrics CSV (ex90n50_busco_metrics.csv) to reproduce the Figure 1B–1C comparison. Rationale: expression-weighting penalizes long, lowly-expressed transcripts that inflate plain N50, giving a fairer contiguity metric across assemblies with different completeness.

## Related tools

- **Salmon** (Quantify transcript abundance in each assembly for expression-weighted ExN50 computation)
- **StringTie** (Assemble the transcriptomes being compared using settings -p 1 -j 2 -c 2 -v -a 4)
- **minimap2** (Align PacBio IsoSeq reads against galGal6 at ~34x depth as input to IsoSeq assembly)
- **HISAT2** (Align fastp-trimmed Illumina reads against galGal6 for the Illumina-only and merged assemblies)
- **BUSCO** (Provide the completed-BUSCO (aves_odb10) completeness metric tabulated alongside Ex90N50)
- **rnaQUAST** (Complementary assembly quality assessment)

## Examples

```
stringtie -p 1 -j 2 -c 2 -v -a 4 merged_alignments.bam -o merged.gtf; salmon quant -i index -l A -1 reads_1.fq -2 reads_2.fq -o salmon_quant; Rscript ExN50_salmon.R salmon_quant merged.gtf
```

## Evaluation signals

- The merged PacBio+Illumina assembly must show a completed-BUSCO count approximately 1% higher than the Illumina-only assembly, matching the improvement reported in Figure 1B–1C
- The IsoSeq PacBio-only assembly must show superior Ex90N50 statistics relative to the Illumina and merged assemblies
- Ex90N50 values must be computed from Salmon quantification restricted to transcripts contributing 90% of total expressed signal, not plain N50
- A row per assembly (IsoSeq, merged, Illumina-only) must be present in ex90n50_busco_metrics.csv with both Ex90N50 and completed-BUSCO columns populated

## Limitations

- ExN50 weighting depends on the quality of Salmon abundance estimates; low-coverage or poorly-quantified assemblies can yield unstable Ex90N50 values
- Ex90N50 and BUSCO completeness can rank assemblies in opposite directions (as here: IsoSeq wins Ex90N50, Illumina/merged win BUSCO), so the metric must be reported jointly with completeness measures
- Results are specific to galGal6-guided assemblies of chicken brain SCO transcriptome data; parameters and rankings may not transfer to other genomes or tissues

## Evidence

- [other] Quantify transcript abundance in each assembly with Salmon and run the Salmon-based ExN50 script to compute the expression-weighted Ex90N50 (N50 of transcripts contributing 90% of total expressed signal): [evidence span withheld — anchor status: not_found]
- [discussion] StringTie assembly parameters: "using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4"
- [other] Merged PacBio+Illumina assembly completeness: [evidence span withheld — anchor status: not_found]
- [other] IsoSeq Ex90N50 vs BUSCO tradeoff: [evidence span withheld — anchor status: not_found]
