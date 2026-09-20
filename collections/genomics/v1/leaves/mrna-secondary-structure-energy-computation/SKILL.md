---
name: mrna-secondary-structure-energy-computation
description: Use when building splicing-prediction features for vex-seq/ΔΨ variants
  and you need wild-type-to-mutant mRNA secondary-structure energy changes as predictors.
when_to_use_negative:
- The input is already an assembled feature table (e.g. features.tsv with the 42 PredPSI-SVR
  columns)
- Variants are intergenic or farther than 200 bp from splice sites — the upstream
  PredPSI-SVR VCF filter removes them before feature extraction
- The model needs branch point or strong secondary-structure signal — the authors
  state these features are not included effectively due to small sample numbers
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_0097
tools:
- name: UNAFold
  role: Computes the ddG mRNA folding free-energy change between wild-type and mutant
    sequences
  repo: http://mfold.rna.albany.edu/?q=DINAMelt/software
- name: ViennaRNA
  role: Computes the dD secondary-structure energy feature comparing WT and mutant
    sequences
  repo: http://www.tbi.univie.ac.at/~ronny/RNA/
- name: samtools
  role: faidx-indexes the hg19 FASTA used to extract WT and mutant exon/flanking sequences
    before energy computation
  repo: http://www.htslib.org/download/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:dfe09eff41e365f993974f8e0e0a6748781d3604d524519c567a04fdc8c77b91
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MaxEntScan
  - Perl
  - PredPSI-SVR
  - SPANR
  - SilVA
  - UNAFold
  - ViennaRNA
  - samtools
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# mRNA secondary-structure energy computation

## Summary

Computes wild-type vs. mutant mRNA folding-energy features (UNAFold ddG and ViennaRNA dD) for exon-plus-flanking sequences as part of the PredPSI-SVR 42-feature splicing feature table.

## When to use

Use when building splicing-prediction features for vex-seq/ΔΨ variants and you need wild-type-to-mutant mRNA secondary-structure energy changes as predictors.

## When NOT to use

- The input is already an assembled feature table (e.g. features.tsv with the 42 PredPSI-SVR columns)
- Variants are intergenic or farther than 200 bp from splice sites — the upstream PredPSI-SVR VCF filter removes them before feature extraction
- The model needs branch point or strong secondary-structure signal — the authors state these features are not included effectively due to small sample numbers

## Inputs

- Wild-type exon sequence with flanking regions extracted from hg19 FASTA (samtools faidx-indexed)
- Mutant (variant-carrying) exon sequence with flanking regions
- Variants passing the splice-proximity filter (within 200 bp of splice sites, non-intergenic via ANNOVAR ensGene)

## Outputs

- UNAFold ddG delta (WT vs. mutant) per variant
- ViennaRNA dD delta (WT vs. mutant) per variant
- Two secondary-structure feature columns contributing to the 42-column features.tsv

## How to apply

Within the PredPSI-SVR pipeline, first extract the wild-type and mutant exon sequence with flanking regions from the samtools-faidx-indexed hg19 FASTA (exon plus 300nt flanks, after the ANNOVAR ensGene-based VCF filter removes variants farther than 200 bp from splice sites). Compute the folding free energy with UNAFold (ddG) and ViennaRNA (dD) for both alleles, then record the WT-to-MT energy deltas as two of the 42 feature columns in features.tsv. These secondary-structure features mirror the mRNA folding free energy features used by SilVA, from which PredPSI-SVR's ese3_mod.py script was adapted. Note the authors' caveat that, due to the small vex-seq sample size, this feature type was not used effectively in the final model.

## Related tools

- **UNAFold** (Computes the ddG mRNA folding free-energy change between wild-type and mutant sequences) — http://mfold.rna.albany.edu/?q=DINAMelt/software
- **ViennaRNA** (Computes the dD secondary-structure energy feature comparing WT and mutant sequences) — http://www.tbi.univie.ac.at/~ronny/RNA/
- **samtools** (faidx-indexes the hg19 FASTA used to extract WT and mutant exon/flanking sequences before energy computation) — http://www.htslib.org/download/

## Evaluation signals

- The feature table contains exactly two secondary-structure energy columns (UNAFold ddG and ViennaRNA dD) among the 42 described features
- Each ddG/dD value is a WT-to-MT delta computed on the same exon-plus-flanking sequence window for both alleles
- All variants scored have passed the splice-proximity filter (non-intergenic, within 200 bp of splice sites)
- Energy values are finite numbers for every filtered variant, matching the filtered variant count from the VCF subset

## Limitations

- PredPSI-SVR does not include features of branch point sequences and pre-mRNA secondary structure effectively due to the limited number of samples
- The method requires wild-type Ψ of exons as input; without experimental Ψ, performance (PCC) drops by about 0.1
- Only Human BodyMap Project 2.0 RNA-seq data were used, and many exons could not be analyzed due to sequencing depth

## Evidence

- [methods] Compute mRNA secondary-structure energy features (UNAFold ddG and ViennaRNA dD) comparing WT and mutant sequences.: [evidence span withheld — anchor status: not_found]
