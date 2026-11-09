---
name: semisynthetic-trio-benchmark-evaluation
description: Use when a variant prioritization or inheritance-model filtering tool
  needs quantitative benchmarking with known ground-truth causal variants in trio/family
  WES data.
when_to_use_negative:
- The goal is clinical diagnosis of a real undiagnosed family rather than benchmarking
  against known ground-truth variants
- CNV or structural variant prioritization must be evaluated — CNV analysis is not
  yet available in eDiVA
- Phenotype-based prioritization recall is critical and the disease's genotype–phenotype
  map is poorly covered in public databases, since incomplete HPO mappings reduce
  recall
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_0622
tools:
- name: eDiVA
  role: 'Tool under test: eDiVA-Prioritize filters spike-in variants by inheritance
    mode and pedigree, ranks by eDiVA-Score, optionally with HPO soft filtering'
  repo: https://github.com/mbosio85/ediva
- name: PhenoDB
  role: Competitor benchmark tool run from https://phenodb.org/ with AF < 0.01, dbSNP
    variants included, and analysis type set per inheritance mode
- name: Phen–Gen
  role: Competitor benchmark tool run locally with 'Recessive'/allow_de_novo=0 or
    'Dominant'/allow_de_novo=1, candidates ranked by DCOD-score
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:532dd74f681d9c66f14b2505ef3de1dc3a792348d180bd94c198853cb12247d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - PhenoDB
  - Phen–Gen
  - eDiVA
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# semisynthetic-trio-benchmark-evaluation

## Summary

Evaluate causal variant prioritization tools by building semisynthetic parent–child trios from real WES data (1000 Genomes CEPH trio NA12878/NA12891/NA12892) spiked with known ClinVar disease variants, then measuring recall, false positives, and causal-variant rank per inheritance mode.

## When to use

Use when a variant prioritization or inheritance-model filtering tool needs quantitative benchmarking with known ground-truth causal variants in trio/family WES data.

## When NOT to use

- The goal is clinical diagnosis of a real undiagnosed family rather than benchmarking against known ground-truth variants
- CNV or structural variant prioritization must be evaluated — CNV analysis is not yet available in eDiVA
- Phenotype-based prioritization recall is critical and the disease's genotype–phenotype map is poorly covered in public databases, since incomplete HPO mappings reduce recall

## Inputs

- Semisynthetic trio multisample VCFs (e.g. 6,811 trios built from the CEPH trio NA12878/NA12891/NA12892 with 3,353 recessive homozygous, 2,592 dominant de novo, and 866 recessive compound heterozygous ClinVar spike-ins)
- Pedigree files for each trio
- Per-variant inheritance-mode metadata and HPO ID metadata
- Competitor tool outputs (PhenoDB web runs, Phen-Gen local runs)

## Outputs

- Per-tool, per-inheritance-mode recall values
- Average number of false positives across all cases
- Rank distribution of causal variants (e.g. median rank = 1 for recessive modes, median = 4 for dominant de novo)
- prioritization_benchmark_results.csv

## How to apply

Construct semisynthetic multisample VCFs by spike-ing ClinVar pathogenic variants into real WES data from a parent–child trio, tagging each variant with its inheritance mode (recessive homozygous, compound heterozygous, dominant de novo) and HPO terms. Run the tool under test (e.g. eDiVA-Prioritize with pedigree file, lenient default filter setting from Table S1, and each relevant inheritance mode) alongside competitors (PhenoDB via its web interface with AF < 0.01; Phen-Gen with 'Recessive'/allow_de_novo=0 or 'Dominant'/allow_de_novo=1), both without and with HPO-based phenotype prioritization. Sort candidate lists per tool using its native score (eDiVA-Score for eDiVA, DCOD-score for Phen-Gen) and compute recall, average false positives, and causal-variant rank distribution per inheritance mode, writing metrics to a results CSV. The rationale is that semisynthetic trios retain real sequencing noise and background variant burden while providing exact ground truth for rank and recall evaluation.

## Related tools

- **eDiVA** (Tool under test: eDiVA-Prioritize filters spike-in variants by inheritance mode and pedigree, ranks by eDiVA-Score, optionally with HPO soft filtering) — https://github.com/mbosio85/ediva
- **PhenoDB** (Competitor benchmark tool run from https://phenodb.org/ with AF < 0.01, dbSNP variants included, and analysis type set per inheritance mode)
- **Phen–Gen** (Competitor benchmark tool run locally with 'Recessive'/allow_de_novo=0 or 'Dominant'/allow_de_novo=1, candidates ranked by DCOD-score)

## Examples

```
nextflow run eDiVA-predict.nf --OUTF ./ --ANNOTATED combined.variants.supplement.ranked.csv --MODE standard --PEDIGREE pedigree.tree --FAMILY_TYPE trio --HPO hpo_list.txt --INHERITANCE recessive
```

## Evaluation signals

- eDiVA must report causal recessive homozygous and compound heterozygous variants within the top five candidates (median rank = 1)
- Dominant de novo causal variants must rank within the top 25 reported candidates (median = 4)
- Detection rate and precision must equal or better Phen-Gen and PhenoDB across the three inheritance modes (Figure 3a–c comparison)
- The benchmark results CSV contains recall, average false positives, and rank distribution rows for every tool × inheritance-mode combination
- With HPO IDs supplied, recall degradation vs. the no-phenotype configuration should be mild (attributable to incomplete genotype–phenotype maps in public databases)

## Limitations

- The benchmark uses semisynthetic trios built from a single CEPH trio rather than fully real disease cohorts
- Incomplete maps of genotype–phenotype (gene–phenotype) relations in public databases cause a mild reduction in HPO-based prioritization recall
- PhenoDB provides no prioritization score, so its candidates can only be evaluated in default chromosome/position order
- CNV analysis is not available in eDiVA, so the benchmark covers SNVs/small indels only

## Evidence

- [discussion] a highly realistic benchmark test combining real WES data of a parent–child trio with thousands of pathogenic ClinVar variants to generate 6,811 semisynthetic disease trios: "a highly realistic benchmark test combining real WES data of a parent–child trio with thousands of pathogenic ClinVar variants to generate 6,811 semisynthetic disease trios"
- [methods] By default, eDiVA applies a lenient filter setting defined in Table S1: "By default, eDiVA applies a lenient filter setting defined in Table S1"
- [methods] removal of all variants that do not segregate according to the selected inheritance mode; (c) filtering based on functional and population genomic features: "removal of all variants that do not segregate according to the selected inheritance mode; (c) filtering based on functional and population genomic features"
- [intro] real WES data from the 1000 Genomes Project CEPH parent–offspring trio of European ancestry (NA12878, NA12891, and NA12892): "real WES data from the 1000 Genomes Project CEPH parent–offspring trio of European ancestry (NA12878, NA12891, and NA12892)"
- [readme] Only the variants passing the filters are reported here, sorted by eDiVA-Score: "combined.variants.supplement.filtered.dominant_inherited.csv : Only the variants passing the filters are reported here, sorted by eDiVA-Score"
