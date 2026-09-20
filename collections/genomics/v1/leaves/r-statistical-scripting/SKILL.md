---
name: r-statistical-scripting
description: Use when read-level PAIso-seq poly(A) tail lengths with isoform assignments
  need per-group summaries, t-tests of tail-length differences between isoforms (e.g.,
  Ccnb1 APA1/APA2/APA3, Wee2, Tcl1), or R/Bioconductor enrichment analyses.
when_to_use_negative:
- Reads have not passed the PAIso-seq QC filters (pass number <10, tail <15 nt or
  <5 continuous adenosines, non-A residues >=50% or >=20) — statistics on unfiltered
  tails will be biased
- Reads cannot be unambiguously assigned to a single gene/isoform (CCSs assigned to
  multiple genes or ambiguous sample assignment are discarded upstream)
- The goal is poly(A) site detection or alignment itself — that is handled upstream
  by minimap2/GMAP and the TAPIS pipeline (run_tapis.py), not the R statistical layer
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Statistical scripting environment (v3.5.1) used to compute per-isoform mean
    tail lengths and Student's t-tests comparing tail-length distributions between
    isoforms
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: clusterProfiler
  role: R/Bioconductor package whose compareCluster function performs enrichment analysis
    of gene lists against the Mus musculus annotation background
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:02a0a9d7e7d6a9862a5bb6820663ca5be63a3d47eab21677142368a77d981903
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
  tools:
  - R
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# r-statistical-scripting

## Summary

R-based statistical analysis and scripting layer of the PAIso-seq pipeline (R 3.5.1 in the GV_oocyte_PAIsoSeqAnalysis repo), used to compute per-isoform poly(A) tail-length summaries, apply Student's t-tests between isoforms (e.g., Ccnb1 APA1/APA2 vs APA3, p=0.0067; Wee2 APA isoforms, p=3.9e-12; Tcl1 splice isoforms a/b/c/e, p=0.038 and p=0.014), and run R/Bioconductor enrichment via clusterProfiler.

## When to use

Use when read-level PAIso-seq poly(A) tail lengths with isoform assignments need per-group summaries, t-tests of tail-length differences between isoforms (e.g., Ccnb1 APA1/APA2/APA3, Wee2, Tcl1), or R/Bioconductor enrichment analyses.

## When NOT to use

- Reads have not passed the PAIso-seq QC filters (pass number <10, tail <15 nt or <5 continuous adenosines, non-A residues >=50% or >=20) — statistics on unfiltered tails will be biased
- Reads cannot be unambiguously assigned to a single gene/isoform (CCSs assigned to multiple genes or ambiguous sample assignment are discarded upstream)
- The goal is poly(A) site detection or alignment itself — that is handled upstream by minimap2/GMAP and the TAPIS pipeline (run_tapis.py), not the R statistical layer

## Inputs

- Per-read poly(A) tail lengths with APA/splice isoform assignments from the PAIso-seq pipeline (run.pl on trimmed CCS fasta from scripts/trim.py, aligned to mm10/GRCm38 with minimap2 -x splice)
- QC-passed CCS reads (pass number >=10, clipped tail >=15 nt with >=5 continuous adenosines, non-A residues <50% and <20 in number)
- Gene list for enrichment analysis with Mus musculus annotation database background

## Outputs

- Per-isoform poly(A) tail-length summary table with Student's t-test p-values (isoform_tail_length_summary.tsv)
- Gene-set enrichment results from clusterProfiler compareCluster

## How to apply

Load the per-read poly(A) tail lengths already filtered by the pipeline's QC criteria (pass number >=10, clipped tail >=15 nt with >=5 continuous adenosines, non-A residues <50% and <20 in number) and group tail lengths by isoform for each gene (Ccnb1 APA1/APA2/APA3, Wee2's two APA isoforms, Tcl1 splice isoforms a/b/c/e). In R 3.5.1, compute per-isoform mean tail lengths and apply Student's t-tests (t.test) to compare tail-length distributions between isoforms of each gene, writing the summary table with p-values (e.g., to isoform_tail_length_summary.tsv). For functional interpretation of gene sets, use the compareCluster function in the R/Bioconductor package clusterProfiler with the Mus musculus annotation database as background. The analysis pipeline (Perl v5.26.2 and R 3.5.1) is implemented in github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis, which produces the ccs.pass.txt and run.pl outputs consumed upstream of the R statistics.

## Related tools

- **R** (Statistical scripting environment (v3.5.1) used to compute per-isoform mean tail lengths and Student's t-tests comparing tail-length distributions between isoforms) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **clusterProfiler** (R/Bioconductor package whose compareCluster function performs enrichment analysis of gene lists against the Mus musculus annotation background)

## Examples

```
R> t.test(tail_len[isoform=='APA1'], tail_len[isoform=='APA3'])  # Ccnb1 APA isoform tail-length comparison, per-isoform means reported to isoform_tail_length_summary.tsv
```

## Evaluation signals

- Per-isoform mean tail lengths reproduce published comparisons: Ccnb1 APA1 and APA2 similar, APA3 significantly longer (p=0.0067); Wee2 APA isoforms differ (p=3.9e-12); Tcl1 isoforms a/b/c/e differ (p=0.038, p=0.014)
- Only reads meeting all four QC criteria (pass number >=10; clipped >=15 nt with >=5 continuous adenosines; non-A <50% and <20 in number) are included in the t-tests
- Output table contains one row per isoform per gene with mean tail length and t-test p-value, written as isoform_tail_length_summary.tsv
- t-tests are run per gene between isoforms (e.g., APA1 vs APA3), never pooling across genes
- Enrichment analyses use a background list of all proteins in the Mus musculus annotation database, not the input gene list itself

## Limitations

- PacBio sequencing cost (about 5 dollars per 1-K reads) limits coverage, so replicate correlation — and hence statistical power for isoform comparisons — is expected to improve only with deeper sequencing
- Very long tails (e.g., a 13-kb tail observed in only one replicate) represent very low abundance events not independently verified and can distort tail-length distribution statistics
- PAIso-seq cannot accurately quantify non-A modifications at the very last 3'-end base, so tail-length and composition summaries in R carry this terminal-base bias

## Evidence

- [readme] README pipeline languages and versions: "This analysis pipeline is implemented in Perl and R. Packages and software Versions used are listed below"
- [discussion] clusterProfiler usage: "using the compareCluster function in the R/Bioconductor package clusterProfiler"
- [discussion] QC thresholds feeding the statistics: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [discussion] TAPIS polyadenylation analysis: "Polyadenylation analysis was performed by using the TAPIS pipeline44"
