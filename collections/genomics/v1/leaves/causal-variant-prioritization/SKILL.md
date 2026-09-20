---
name: causal-variant-prioritization
description: Use when analyzing multisample VCFs from small related sample groups
  (parent-child trios or families) with pedigree and inheritance-mode metadata, to
  shortlist candidate causal variants for rare Mendelian disease diagnosis.
when_to_use_negative:
- The candidate variants are CNVs — CNV analysis is not yet integrated in eDiVA, which
  currently handles SNPs/indels only
- Samples are unrelated singletons without pedigree or inheritance-model information,
  so segregation-based filtering cannot be applied
- A prioritization ranking already exists (e.g. a precomputed ranked.csv) and the
  task is only annotation or downstream interpretation
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0102
tools:
- name: eDiVA
  role: End-to-end pipeline (eDiVA-Predict, eDiVA-Annotate, eDiVA-Score, eDiVA-Prioritize)
    performing segregation filtering, rare-variant filtering (AF ≤ 0.01), and random-forest
    eDiVA-Score ranking of candidate variants
  repo: https://github.com/mbosio85/ediva
- name: PhenoDB
  role: Benchmark comparator run from phenodb.org (AF < 0.01, autosomal recessive
    homozygous / compound heterozygous / dominant new mutation analyses); provides
    no prioritization score
- name: Phen–Gen
  role: Benchmark comparator run locally with 'Recessive' (allow_de_novo = 0) or 'Dominant'
    (allow_de_novo = 1); candidates ranked by DCOD-score
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:1340608cb1e3cc8d087965f500212cf5ad785b737d2051ea66ba6008f9a3d0c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - PhenoDB
  - Phen–Gen
  - eDiVA
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# causal-variant-prioritization

## Summary

Rank causal coding variants in familial WES/WGS disease studies using eDiVA-Prioritize, which filters variants by inheritance-model segregation (dominant de novo, dominant inherited, recessive homozygous, compound heterozygous, X-linked), applies functional/population-genomic filters (AF ≤ 0.01), and prioritizes by the random-forest eDiVA-Score with optional HPO phenotype soft-filtering.

## When to use

Use when analyzing multisample VCFs from small related sample groups (parent-child trios or families) with pedigree and inheritance-mode metadata, to shortlist candidate causal variants for rare Mendelian disease diagnosis.

## When NOT to use

- The candidate variants are CNVs — CNV analysis is not yet integrated in eDiVA, which currently handles SNPs/indels only
- Samples are unrelated singletons without pedigree or inheritance-model information, so segregation-based filtering cannot be applied
- A prioritization ranking already exists (e.g. a precomputed ranked.csv) and the task is only annotation or downstream interpretation

## Inputs

- Joint multisample VCF (combined.variants.supplement.vcf) from a trio or family
- Pedigree file (pedigree.tree) with affected status
- family_config text file (concatenated sample_info.txt lines with AFFECTED=0/1 flags)
- HPO term list file (hpo_list.txt) for phenotype-based prioritization (optional; can be an empty list)
- Gene exclusion list (Resource/gene_exclusion_list.txt)

## Outputs

- combined.variants.supplement.filtered.<inheritance>.csv: variants passing filters, sorted by eDiVA-Score
- combined.variants.supplement.<inheritance>.csv: all annotated variants with an inheritance-compatibility column
- variant_prioritization_report.xlsx: one spreadsheet per inheritance type with OMIM/ClinVar annotations
- Rank distribution of causal variants (e.g. prioritization_benchmark_results.csv in benchmark settings)

## How to apply

First produce a joint multisample VCF (eDiVA-Predict with GATK best practices, then eDiVA_prepare_for_annotation to generate combined.variants.supplement.vcf and pedigree.tree). Annotate and score variants with eDiVA-Annotate.nf, which runs the 1000-tree random-forest eDiVA-Score (trained on 15,000 ClinVar pathogenic variants with five-fold cross validation) and outputs combined.variants.supplement.ranked.csv. Then run eDiVA-Prioritize with --PEDIGREE pedigree.tree, --FAMILY_TYPE trio, --MODE standard (the lenient default filter setting from Table S1), an HPO term list for phenotype-based soft filtering, and the gene EXCLUSIONLIST to drop recurrent false-positive genes. Choose an --INHERITANCE mode (dominant_inherited, recessive, dominant_denovo, Xlinked, compound); eDiVA removes all variants that do not segregate according to the selected inheritance mode, keeps rare variants (AF ≤ 0.01), and sorts survivors by eDiVA-Score so better candidates appear on top.

## Related tools

- **eDiVA** (End-to-end pipeline (eDiVA-Predict, eDiVA-Annotate, eDiVA-Score, eDiVA-Prioritize) performing segregation filtering, rare-variant filtering (AF ≤ 0.01), and random-forest eDiVA-Score ranking of candidate variants) — https://github.com/mbosio85/ediva
- **PhenoDB** (Benchmark comparator run from phenodb.org (AF < 0.01, autosomal recessive homozygous / compound heterozygous / dominant new mutation analyses); provides no prioritization score)
- **Phen–Gen** (Benchmark comparator run locally with 'Recessive' (allow_de_novo = 0) or 'Dominant' (allow_de_novo = 1); candidates ranked by DCOD-score)

## Examples

```
nextflow run eDiVA-predict.nf --OUTF ./ --ANNOTATED combined.variants.supplement.ranked.csv --MODE standard --PEDIGREE pedigree.tree --FAMILY_TYPE trio --HPO hpo_list.txt --INHERITANCE choose
```

## Evaluation signals

- Causal recessive homozygous and compound heterozygous variants appear within the top five candidates (benchmark median rank = 1); dominant de novo variants within the top 25 (median = 4)
- Detection rate and precision are equal to or better than Phen-Gen and PhenoDB across all three inheritance modes (Figure 3a–c)
- Output filtered.<inheritance>.csv contains only variants compatible with the selected inheritance pattern, sorted by eDiVA-Score
- All retained variants have AF ≤ 0.01 in population frequency databases (dbSNP, 1000 Genomes, Exome Variant Server, GnomAD exomes, discovEHR)
- variant_prioritization_report.xlsx is populated with one spreadsheet per inheritance type run

## Limitations

- Benchmark relies on semisynthetic trios (6,811 ClinVar spike-ins into real CEPH trio WES data) rather than fully real disease cohorts
- Incomplete genotype–phenotype (gene–phenotype) maps in public databases lead to a mild reduction in HPO-based prioritization recall
- CNV analysis is not yet available in eDiVA; WES analyses may miss CNVs rarely processed in combination with point mutations
- Pipeline stack requires legacy dependencies: Python 2.7, GATK 3.3+, Picard 1.119, bwa 0.7.10, samtools v1.5

## Evidence

- [results] eDiVA consistently reported causal recessive homozygous and compound heterozygous variants within the top five candidates (median = 1): [evidence span withheld — anchor status: not_found]
- [methods] removal of all variants that do not segregate according to the selected inheritance mode: "(b) removal of all variants that do not segregate according to the selected inheritance mode; (c) filtering based on functional and population genomic features"
- [methods] lenient default filter setting defined in Table S1: "By default, eDiVA applies a lenient filter setting defined in Table S1"
- [discussion] incomplete genotype–phenotype maps reduce HPO recall: "incomplete maps of known genotype–phenotype (or gene–phenotype) relations in public databases led to a mild reduction in recall."
- [readme] README: filtered output sorted by eDiVA-Score: "combined.variants.supplement.filtered.dominant_inherited.csv : Only the variants passing the filters are reported here, sorted by eDiVA-Score"
