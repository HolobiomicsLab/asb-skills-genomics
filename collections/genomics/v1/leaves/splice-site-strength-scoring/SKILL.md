---
name: splice-site-strength-scoring
description: Use when variant effect on splicing must be quantified via splice site
  motif strength — e.g., building features from exon+flanking sequences or benchmarking
  against SPANR on VCF variants near splice sites.
when_to_use_negative:
- Variants farther than 200 bp from splice sites or in intergenic regions — PredPSI-SVR's
  VCF filter removes these before splice-site scoring
- Variant impact assessment for branch point regions — the paper notes branch point
  sequence features are not included due to small sample numbers
- Input already contains a completed feature table; splice-site scoring would duplicate
  existing MES columns
edam_operation: http://edamontology.org/operation_0433
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3511
tools:
- name: MaxEntScan
  role: Scores 5' (MES5) and 3' (MES3) splice site motif strength for both alleles;
    deltas feed the 42-feature PredPSI-SVR table and serve as comparison scores
- name: samtools
  role: faidx-indexes the hg19 FASTA so wild-type and mutant exon+flanking sequences
    can be extracted for scoring
- name: ANNOVAR
  role: ensGene annotation used to filter the VCF to non-intergenic variants within
    200 bp of splice sites before scoring
- name: PredPSI-SVR
  role: Pipeline that orchestrates splice-site scoring and integrates the resulting
    features into the ΔΨ prediction
  repo: https://github.com/chenkenbio/PredPSI-SVR
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:2405de30d0d50457c75a8189e66e36e6ef2d5b08912cc1ba120f5ba3d66ddfe9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - ANNOVAR
  - MaxEntScan
  - Perl
  - PredPSI-SVR
  - SPANR
  - SilVA
  - samtools
  edam_operation: http://edamontology.org/operation_0433
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# splice-site-strength-scoring

## Summary

Score 5' (donor) and 3' (acceptor) splice site strength with MaxEntScan (MES5/MES3) for both wild-type and mutant alleles, then compute WT-to-MT deltas as splicing-impact features for ΔΨ prediction models like PredPSI-SVR.

## When to use

Use when variant effect on splicing must be quantified via splice site motif strength — e.g., building features from exon+flanking sequences or benchmarking against SPANR on VCF variants near splice sites.

## When NOT to use

- Variants farther than 200 bp from splice sites or in intergenic regions — PredPSI-SVR's VCF filter removes these before splice-site scoring
- Variant impact assessment for branch point regions — the paper notes branch point sequence features are not included due to small sample numbers
- Input already contains a completed feature table; splice-site scoring would duplicate existing MES columns

## Inputs

- VCF of variants already filtered to within 200 bp of splice sites (non-intergenic, ANNOVAR ensGene-annotated)
- samtools-faidx-indexed hg19 genome FASTA (hg19.fasta + hg19.fasta.fai)
- Wild-type and mutant exon sequence with 300 nt flanking regions

## Outputs

- Per-variant MES5 and MES3 scores for wild-type and mutant alleles
- WT-to-MT splice site deltas (ΔMES5, ΔMES3) written as columns of the 42-column features.tsv feature table
- Optional summed score |ΔMES5|+|ΔMES3| used as a comparison metric against SPANR/PredPSI-SVR

## How to apply

Extract the wild-type and mutant exon sequences plus 300 nt of flanking intronic/exonic sequence from the samtools-faidx-indexed hg19 FASTA. Run MaxEntScan to score the 5' splice site (MES5) and 3' splice site (MES3) for both alleles, then compute the WT-to-MT deltas for each. In PredPSI-SVR these six splice-site motif features (WT and mutant MES5/MES3 plus deltas) form part of the 42-feature table; MaxEntScan features also serve as a standalone comparison score via the summed absolute change |ΔMES5|+|ΔMES3|. Before scoring, filter the input VCF to discard intergenic variants and variants farther than 200 bp from splice sites, since distant variants lack scorable splice-site context.

## Related tools

- **MaxEntScan** (Scores 5' (MES5) and 3' (MES3) splice site motif strength for both alleles; deltas feed the 42-feature PredPSI-SVR table and serve as comparison scores)
- **samtools** (faidx-indexes the hg19 FASTA so wild-type and mutant exon+flanking sequences can be extracted for scoring)
- **ANNOVAR** (ensGene annotation used to filter the VCF to non-intergenic variants within 200 bp of splice sites before scoring)
- **PredPSI-SVR** (Pipeline that orchestrates splice-site scoring and integrates the resulting features into the ΔΨ prediction) — https://github.com/chenkenbio/PredPSI-SVR

## Examples

```
$HOME/PredPSI-SVR/tools/samtools-1.9/samtools faidx hg19.fasta && ./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- Feature table contains exactly the 42 described feature columns, including the six splice-site motif features, for every variant passing the 200 bp splice-proximity filter
- Filtered variant count matches the VCF subset within 200 bp of splice sites (README warns output can contain fewer variants than input)
- WT and mutant MES5/MES3 scores are finite numeric values for every scored variant; deltas equal mutant minus wild-type scores
- On SynonMut benchmarks, the summed |ΔMES5|+|ΔMES3| score should reproduce reported MaxEntScan AUC comparisons (PredPSI-SVR was 6.6% higher on SynonMut-psi)

## Limitations

- MaxEntScan cannot score all variants — in the SynonMut analysis, 11024 mutations fell outside MaxEntScan's scanning scope
- Splice-site features alone underperform: PredPSI-SVR AUC on SynonMut-psi was 0.579 vs MaxEntScan, but without Ψ input (noPSI) performance dropped to 0.508
- Scores depend on hg19 reference sequence; the pipeline requires the samtools-indexed hg19 FASTA downloaded from UCSC goldenPath chromFa

## Evidence

- [other] Score 5' and 3' splice sites with MaxEntScan (MES5/MES3) for both alleles and compute the WT-to-MT deltas.: [evidence span withheld — anchor status: not_found]
- [other] From the exon sequence plus 300nt flanking regions, 42 features were extracted including six splice site motif features: [evidence span withheld — anchor status: not_found]
- [other] Score the same variants with SPANR (SPIDEX ΔΨ) and with MaxEntScan using the summed absolute change |ΔMES5|+|ΔMES3|.: [evidence span withheld — anchor status: not_found]
