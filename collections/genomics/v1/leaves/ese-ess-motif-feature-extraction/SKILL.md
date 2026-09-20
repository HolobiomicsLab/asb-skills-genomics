---
name: ese-ess-motif-feature-extraction
description: 'Use when building splicing-impact features from variant-affected exons:
  you have WT and mutant exon sequences plus flanking regions and need pESE/pESS enhancer/silencer
  motif scores for ΔΨ prediction.'
when_to_use_negative:
- The input is already a completed feature table (e.g. features.tsv with the 42 PredPSI-SVR
  columns) — motif scoring has already been done
- You need splice-site strength or intronic features rather than exonic enhancer/silencer
  motifs — use MaxEntScan MES5/MES3 deltas or intronic features instead
- You need branch point or effective pre-mRNA secondary structure features — the paper
  states these are not included effectively due to the small sample size
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: PredPSI-SVR
  role: Pipeline that packages the ESE/ESS feature extraction (tools/ese3/ese3_mod.py)
    into the ΔΨ prediction workflow
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: SilVA
  role: Source package of the script from which PredPSI-SVR's ese3_mod.py (pESE/pESS
    scoring) is modified
  repo: https://github.com/buske/silva
- name: samtools
  role: faidx-indexed hg19 FASTA retrieval of exon and flanking sequences used as
    motif-scoring input
  repo: http://www.htslib.org/download/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:c6695c9b7b87b5b5a53b771bb56043cf48e719346026d7426224d905b9593930
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - MaxEntScan
  - Perl
  - PredPSI-SVR
  - SPANR
  - SilVA
  - samtools
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# ESE/ESS motif feature extraction

## Summary

Computes exonic splicing enhancer (ESE) and silencer (ESS) scores for wild-type and mutant vex-seq exon sequences with flanking regions, using ESEfinder SF2/ASF and FAS-ESS matrices plus the SilVA-derived ese3_mod.py script for pESE/pESS scores, as part of the 42-feature PredPSI-SVR ΔΨ prediction table.

## When to use

Use when building splicing-impact features from variant-affected exons: you have WT and mutant exon sequences plus flanking regions and need pESE/pESS enhancer/silencer motif scores for ΔΨ prediction.

## When NOT to use

- The input is already a completed feature table (e.g. features.tsv with the 42 PredPSI-SVR columns) — motif scoring has already been done
- You need splice-site strength or intronic features rather than exonic enhancer/silencer motifs — use MaxEntScan MES5/MES3 deltas or intronic features instead
- You need branch point or effective pre-mRNA secondary structure features — the paper states these are not included effectively due to the small sample size

## Inputs

- Wild-type and mutant exon nucleotide sequences with flanking regions (extracted from samtools-faidx-indexed hg19 FASTA)
- Filtered VCF subset containing variants within 200 bp of splice sites (annotated by ANNOVAR ensGene)
- ESEfinder SF2/ASF position-weight matrix
- FAS-ESS motif matrices
- SilVA-derived ese3_mod.py script (PredPSI-SVR/tools/ese3)

## Outputs

- Per-variant exonic enhancer/silencer feature columns (ESEfinder SF2/ASF, FAS-ESS, pESE/pESS scores) for WT and mutant alleles
- Contribution to the 42-column features.tsv splicing-feature table used by PredPSI-SVR

## How to apply

Extract wild-type and mutant exon sequences (plus flanking regions) from the samtools-faidx-indexed hg19 FASTA after ANNOVAR ensGene-based VCF filtering retains variants within 200 bp of splice sites. Score exonic enhancers with ESEfinder using the SF2/ASF matrix and detect silencers with FAS-ESS matrices; additionally compute pESE/pESS scores with the ese3_mod.py script in PredPSI-SVR/tools/ese3, which is modified from the SilVA package. Compare WT and mutant values so the resulting columns capture the variant-induced change in enhancer/silencer content. The rationale is that ESE/ESS motif disruption is a key driver of exon inclusion changes, and these features feed the support vector regression for ΔΨ; greedy feature selection later identified the eight most contributing features. Concatenate these features with MaxEntScan splice site deltas, secondary structure energies (UNAFold ddG, ViennaRNA), SPIDEX dPsi, and CADD annotations into the 42-column features.tsv.

## Related tools

- **PredPSI-SVR** (Pipeline that packages the ESE/ESS feature extraction (tools/ese3/ese3_mod.py) into the ΔΨ prediction workflow) — https://github.com/chenkenbio/PredPSI-SVR
- **SilVA** (Source package of the script from which PredPSI-SVR's ese3_mod.py (pESE/pESS scoring) is modified) — https://github.com/buske/silva
- **samtools** (faidx-indexed hg19 FASTA retrieval of exon and flanking sequences used as motif-scoring input) — http://www.htslib.org/download/

## Examples

```
cd ~/PredPSI-SVR/ && ./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- The assembled feature table contains exactly the 42 described feature columns, with the ESE/ESS columns (ESEfinder SF2/ASF, FAS-ESS, pESE/pESS) present for every filtered variant
- ESE/ESS values are computed for both wild-type and mutant alleles so WT-to-mutant differences can be derived
- Every scored variant passes the splice-proximity filter (within 200 bp of a splice site, annotated via ANNOVAR ensGene), and the variant count matches the VCF subset
- ese3_mod.py output is consistent with SilVA's pESE/pESS scoring behavior, since it is a modified SilVA script

## Limitations

- Greedy feature selection found only eight features contributing most to the prediction, so not all ESE/ESS features may be retained in the final model
- The method was trained on the small vex-seq CAGI 5 challenge dataset, limiting model power and preventing deep learning alternatives
- Variants farther than 200 bp from splice sites or in intergenic regions are filtered out and never scored

## Evidence
