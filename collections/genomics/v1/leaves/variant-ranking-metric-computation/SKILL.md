---
name: variant-ranking-metric-computation
description: Use when comparing causal variant prioritization tools on trio WES data
  with known spike-in causal variants, needing recall, false-positive counts, and
  rank-of-causal-variant distributions per inheritance mode.
when_to_use_negative:
- 'Real disease cohorts without known causal variants: no ground truth exists to compute
  recall or causal-variant rank'
- 'CNV or structural variant prioritization: CNV analysis is not yet available in
  eDiVA (integration pending)'
- 'Single-sample analysis with no pedigree or inheritance model: eDiVA-Prioritize
  requires pedigree info and a selectable inheritance mode (dominant_inherited, recessive,
  dominant_denovo, Xlinked, compound)'
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: eDiVA
  role: Produces the ranked candidate shortlists (eDiVA-Annotate + eDiVA-Score + eDiVA-Prioritize)
    whose causal-variant ranks are benchmarked; variants sorted by eDiVA-Score
  repo: https://github.com/mbosio85/ediva
- name: PhenoDB
  role: Benchmark comparator run via https://phenodb.org/ with AF < 0.01 and recessive/dominant-new-mutation
    analysis types; results have no prioritization score, so default chromosome/position
    order is used
- name: Phen–Gen
  role: Benchmark comparator run locally ('Recessive' with allow_de_novo = 0; 'Dominant'
    with allow_de_novo = 1); candidates sorted by DCOD-score
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:65b67946d12e3a197fa630a27f79215743c50d138ff2ae6fb424ef5008a218eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - PhenoDB
  - Phen–Gen
  - eDiVA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# variant-ranking-metric-computation

## Summary

Benchmark protocol for computing recall, average false positives, and causal-variant rank distributions for variant prioritization tools (eDiVA-Prioritize vs. PhenoDB, Phen-Gen) on 6,811 semisynthetic ClinVar spike-in trios across recessive homozygous, compound heterozygous, and dominant de novo inheritance modes.

## When to use

Use when comparing causal variant prioritization tools on trio WES data with known spike-in causal variants, needing recall, false-positive counts, and rank-of-causal-variant distributions per inheritance mode.

## When NOT to use

- Real disease cohorts without known causal variants: no ground truth exists to compute recall or causal-variant rank
- CNV or structural variant prioritization: CNV analysis is not yet available in eDiVA (integration pending)
- Single-sample analysis with no pedigree or inheritance model: eDiVA-Prioritize requires pedigree info and a selectable inheritance mode (dominant_inherited, recessive, dominant_denovo, Xlinked, compound)

## Inputs

- 6,811 semisynthetic trio multisample VCFs from the CEPH trio (NA12878/NA12891/NA12892) with ClinVar spike-in variants
- Pedigree files (pedigree.tree) for each trio
- Per-variant inheritance-mode and HPO metadata
- Per-tool candidate variant lists (e.g., combined.variants.supplement.filtered.<mode>.csv sorted by eDiVA-Score; Phen-Gen DCOD-scored output; PhenoDB web results)

## Outputs

- prioritization_benchmark_results.csv with per-tool/per-mode recall, average false positives, and causal-variant rank distribution
- Rank distribution summaries (top-5 and top-25 statistics) per inheritance mode

## How to apply

Run each tool on the 6,811 semisynthetic trios (built from the 1000 Genomes CEPH trio NA12878/NA12891/NA12892 with 3,353 recessive homozygous, 2,592 dominant de novo, and 866 compound heterozygous ClinVar spike-ins), using the pedigree file and the lenient default filter setting (Table S1), with and without HPO IDs. Sort each tool's candidate list by its native score: eDiVA variants by eDiVA-Score (the random-forest pathogenicity probability from 1000 trees, five-fold cross validation), Phen-Gen by DCOD-score, and PhenoDB results in default chromosome/position order since no prioritization score is provided. For each tool and inheritance mode compute recall (causal variant present in the output list), the average number of false positives across all cases, and the rank distribution of the causal variant. Write all metrics to prioritization_benchmark_results.csv. Validation: detection rate and precision must be equal to or better than Phen-Gen and PhenoDB; expected reference behavior is causal recessive variants within the top five candidates (median = 1) and dominant de novo within the top 25 (median = 4).

## Related tools

- **eDiVA** (Produces the ranked candidate shortlists (eDiVA-Annotate + eDiVA-Score + eDiVA-Prioritize) whose causal-variant ranks are benchmarked; variants sorted by eDiVA-Score) — https://github.com/mbosio85/ediva
- **PhenoDB** (Benchmark comparator run via https://phenodb.org/ with AF < 0.01 and recessive/dominant-new-mutation analysis types; results have no prioritization score, so default chromosome/position order is used)
- **Phen–Gen** (Benchmark comparator run locally ('Recessive' with allow_de_novo = 0; 'Dominant' with allow_de_novo = 1); candidates sorted by DCOD-score)

## Examples

```
nextflow run eDiVA-predict.nf --OUTF ./ --ANNOTATED combined.variants.supplement.ranked.csv --MODE standard --PEDIGREE pedigree.tree --FAMILY_TYPE trio --INHERITANCE recessive
```

## Evaluation signals

- Recall computed per tool and inheritance mode as 'causal variant present in output list'
- Causal recessive homozygous and compound heterozygous variants appear within the top five candidates (median = 1) for eDiVA
- Dominant de novo variants appear within the top 25 of reported candidates (median = 4) for eDiVA
- Detection rate and precision equal to or better than Phen-Gen and PhenoDB (Figure 3a-c benchmark comparison)
- All metrics recorded in prioritization_benchmark_results.csv with one row per tool/inheritance mode

## Limitations

- Benchmark relies on semisynthetic trios (real CEPH WES data plus ClinVar spike-ins) rather than fully real disease cohorts
- PhenoDB provides no prioritization score, so its results can only be evaluated in default chromosome/position order
- Incomplete maps of known genotype–phenotype (or gene–phenotype) relations in public databases lead to a mild reduction in HPO-based prioritization recall
- CNV analysis is not yet available in eDiVA, so the benchmark covers only SNV/indel point-mutation prioritization

## Evidence

- [abstract] Benchmarking against state-of-the-art competitors demonstrates that eDiVA consistently performed as a good or better than existing approach in terms of detection rate and precision.: "Benchmarking against state-of-the-art competitors demonstrates that eDiVA consistently performed as a good or better than existing approach in terms of detection rate and precision."
- [intro] Using the pathogenicity probability computed by eDiVA-Score, variants are ranked such that better candidates appear on top of the result list: [evidence span withheld — anchor status: not_found]
- [results] eDiVA consistently reported causal recessive homozygous and compound heterozygous variants within the top five candidates (median = 1): [evidence span withheld — anchor status: not_found]
- [readme] combined.variants.supplement.filtered.dominant_inherited.csv : Only the variants passing the filters are reported here, sorted by eDiVA-Score: "combined.variants.supplement.filtered.dominant_inherited.csv : Only the variants passing the filters are reported here, sorted by eDiVA-Score"
