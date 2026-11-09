---
name: genomic-flanking-sequence-extraction
description: Use when variants (e.g., a VCF annotated by ANNOVAR ensGene) need underlying
  genomic sequence — exon plus flanks — retrieved from hg19 to build feature tables
  for splice-impact prediction.
when_to_use_negative:
- The input is already a computed feature table (e.g., features.tsv with the 42 columns)
  and no raw sequence retrieval is needed
- Variants are intergenic or farther than 200 bp from splice sites — the PredPSI-SVR
  pipeline filters these out before sequence extraction
- The target genome build is not hg19; the pipeline's ANNOVAR ensGene and SPIDEX annotations
  are build-specific
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0080
tools:
- name: samtools
  role: Indexes hg19.fasta with faidx and retrieves exon + flanking subsequences by
    coordinate for each WT/MT variant
  repo: http://www.htslib.org/download/
- name: PredPSI-SVR
  role: Pipeline that orchestrates the VCF filtering and sequence extraction step
    before feature scoring
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: ANNOVAR
  role: ensGene annotation used to determine which variants are exonic/splice-proximal
    and hence which sequences to extract
  repo: http://annovar.openbioinformatics.org/en/latest/user-guide/download/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:3a02e5962249729887c5c8657ece43ccd13ae8a418f4ba763a98dfc684647200
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ANNOVAR
  - MaxEntScan
  - Perl
  - PredPSI-SVR
  - SPANR
  - SilVA
  - samtools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# genomic-flanking-sequence-extraction

## Summary

Extract wild-type and mutant exon sequences plus flanking regions from a samtools-faidx-indexed hg19 FASTA so that splice site, splicing-regulatory-element, and secondary-structure features can be computed per vex-seq variant in the PredPSI-SVR pipeline.

## When to use

Use when variants (e.g., a VCF annotated by ANNOVAR ensGene) need underlying genomic sequence — exon plus flanks — retrieved from hg19 to build feature tables for splice-impact prediction.

## When NOT to use

- The input is already a computed feature table (e.g., features.tsv with the 42 columns) and no raw sequence retrieval is needed
- Variants are intergenic or farther than 200 bp from splice sites — the PredPSI-SVR pipeline filters these out before sequence extraction
- The target genome build is not hg19; the pipeline's ANNOVAR ensGene and SPIDEX annotations are build-specific

## Inputs

- samtools-faidx-indexed hg19 FASTA (hg19.fasta)
- Filtered VCF of variants within 200 bp of splice sites (ANNOVAR ensGene-annotated)
- Variant coordinates with ref/alt alleles (VCF CHROM/POS/REF/ALT fields)

## Outputs

- Wild-type exon sequence with flanking regions per variant
- Mutant (allele-substituted) exon sequence with flanking regions per variant
- Sequences ready for MaxEntScan, ESEfinder/FAS-ESS (pESE/pESS), and UNAFold/ViennaRNA scoring toward the 42-column features.tsv

## How to apply

First download the hg19 genome from UCSC goldenPath (chromFa.tar.gz), concatenate all chromosome FASTAs into a single hg19.fasta, and index it with `samtools faidx hg19.fasta` so random-access queries are possible. After filtering the input VCF with the PredPSI-SVR/ANNOVAR (ensGene) step to keep only variants within 200 bp of splice sites, use the index to pull the wild-type exon sequence with flanking regions for each variant. Generate the mutant counterpart by substituting the VCF ref/alt alleles at the corresponding coordinates. These WT/MT sequence pairs then feed downstream scorers — MaxEntScan for 5'/3' splice sites, ESEfinder/FAS-ESS plus the SilVA-derived ese3_mod.py for pESE/pESS, and UNAFold/ViennaRNA for secondary-structure energies — that produce the 42-feature table.

## Related tools

- **samtools** (Indexes hg19.fasta with faidx and retrieves exon + flanking subsequences by coordinate for each WT/MT variant) — http://www.htslib.org/download/
- **PredPSI-SVR** (Pipeline that orchestrates the VCF filtering and sequence extraction step before feature scoring) — https://github.com/chenkenbio/PredPSI-SVR
- **ANNOVAR** (ensGene annotation used to determine which variants are exonic/splice-proximal and hence which sequences to extract) — http://annovar.openbioinformatics.org/en/latest/user-guide/download/

## Examples

```
cat *.fa > hg19.fasta; $HOME/PredPSI-SVR/tools/samtools-1.9/samtools faidx hg19.fasta
```

## Evaluation signals

- hg19.fasta is indexed and a .fai companion file exists after `samtools faidx hg19.fasta`
- Every variant retained in OUTPUT.dpsi (i.e., within 200 bp of a splice site) has both WT and MT sequences extracted
- Extracted sequences are consistent with hg19 at the VCF coordinates — the REF base at POS matches the VCF ref allele
- The downstream feature table contains exactly the 42 described feature columns for every filtered variant
- Variants dropped from OUTPUT.psi relative to the input VCF correspond to intergenic or >200 bp-from-splice-site calls

## Limitations

- Extraction is limited to variants within 200 bp of splice sites; the pipeline silently removes intergenic and distant variants, so fewer sequences may result than VCF lines
- The hg19 FASTA must be assembled by concatenating all chromFa files; a missing or mis-concatenated chromosome yields failed or wrong-coordinate lookups
- Wild-type Ψ of exons is required by the full method; without experimental Ψ, downstream PCCs drop by about 0.1, though this does not affect sequence retrieval itself

## Evidence
