---
name: cross-build-variant-liftover
description: Use when a variant catalog (e.g., dbDSM pathogenic mutations, 1000 Genomes
  normals with AF 0.1–0.9) is in hg38 but downstream annotation and scoring tools
  (ANNOVAR ensGene hg19, hg19_spidex/SPANR) require hg19 coordinates.
when_to_use_negative:
- The variant set is already in hg19 coordinates and directly compatible with hg19
  SPIDEX/ensGene resources — liftover would only introduce coordinate drift.
- The goal is ΔΨ prediction for variants lacking SPANR/SPIDEX scores or lying outside
  the splice-proximal window — the joined benchmark table cannot be assembled for
  them.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CrossMap
  role: Converts variants from hg38 to hg19 coordinates against the indexed hg19 reference,
    dropping variants that fail to lift over
- name: samtools
  role: Indexes the concatenated hg19 chromFa FASTA (samtools faidx) so lifted coordinates
    can be validated against the target assembly
  repo: http://www.htslib.org/download/
- name: ANNOVAR
  role: Annotates lifted hg19 variants against the ensGene hg19 database to determine
    splice-site proximity
  repo: http://annovar.openbioinformatics.org/en/latest/user-guide/download/
- name: SPIDEX
  role: Third-party hg19 database supplying SPANR ΔΨ scores joined to each lifted
    variant; mutations without scores are removed
  repo: http://www.openbioinformatics.org/annovar/spidex_download_form.php
- name: PredPSI-SVR
  role: Downstream consumer of the lifted benchmark; its VCF filter drops variants
    in intergenic regions or distant to splice sites (more than 200 bp)
  repo: https://github.com/chenkenbio/PredPSI-SVR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:2be8e80a93659f42946241c0e6b555dbae27665aaae668b3b1b72f32daa68c7d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ANNOVAR
  - CrossMap
  - PredPSI-SVR
  - SPANR
  - SPIDEX
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-build-variant-liftover

## Summary

Lift synonymous variant coordinates from hg38 to hg19 with CrossMap against an indexed UCSC hg19 reference so the variants can be joined to hg19-locked resources (ANNOVAR ensGene, SPANR/SPIDEX ΔΨ scores) used to compile the SynonMut-complete and SynonMut-psi benchmark tables for PredPSI-SVR evaluation.

## When to use

Use when a variant catalog (e.g., dbDSM pathogenic mutations, 1000 Genomes normals with AF 0.1–0.9) is in hg38 but downstream annotation and scoring tools (ANNOVAR ensGene hg19, hg19_spidex/SPANR) require hg19 coordinates.

## When NOT to use

- The variant set is already in hg19 coordinates and directly compatible with hg19 SPIDEX/ensGene resources — liftover would only introduce coordinate drift.
- The goal is ΔΨ prediction for variants lacking SPANR/SPIDEX scores or lying outside the splice-proximal window — the joined benchmark table cannot be assembled for them.

## Inputs

- hg38-coordinate synonymous variant catalog from dbDSM (pathogenic mutations)
- hg38-coordinate normal synonymous variants from the 1000 Genomes Project filtered to allele frequency 0.1–0.9
- hg19 reference genome FASTA (UCSC goldenPath chromFa) with samtools faidx index
- CrossMap liftover chain/tooling
- ANNOVAR ensGene hg19 annotation database (humandb)
- hg19_spidex SPIDEX database (unzipped into ANNOVAR humandb)

## Outputs

- hg19-coordinate variant table of synonymous mutations within 300 bp of splice sites, each carrying a SPANR/SPIDEX ΔΨ score
- SynonMut-complete CSV (757 pathogenic / 10822 normal mutations)
- SynonMut-psi CSV (87 pathogenic / 826 normal mutations with experimental Ψ)

## How to apply

First stage the hg19 target reference: download the UCSC goldenPath hg19 chromFa tarball, concatenate the contigs into hg19.fasta, and index it with samtools faidx so coordinate queries are fast and byte-addressable. Run CrossMap against this hg19 FASTA to convert the hg38 variants, dropping any variant that fails to lift over (intergenic-mapped or ambiguous lifts). Re-annotate the lifted variants with ANNOVAR against the hg19 ensGene database and retain only mutations within 300 bp of a splice site; note that PredPSI-SVR itself applies a stricter 200 bp splice-site-distance VCF filter at run time, so lifted variants between 200 and 300 bp may be discarded downstream. Finally join the hg19_spidex SPANR ΔΨ scores and remove mutations lacking a SPANR score, yielding the labeled tables SynonMut-complete (757 pathogenic / 10822 normal) and its Ψ-bearing subset SynonMut-psi (87 pathogenic / 826 normal).

## Related tools

- **CrossMap** (Converts variants from hg38 to hg19 coordinates against the indexed hg19 reference, dropping variants that fail to lift over)
- **samtools** (Indexes the concatenated hg19 chromFa FASTA (samtools faidx) so lifted coordinates can be validated against the target assembly) — http://www.htslib.org/download/
- **ANNOVAR** (Annotates lifted hg19 variants against the ensGene hg19 database to determine splice-site proximity) — http://annovar.openbioinformatics.org/en/latest/user-guide/download/
- **SPIDEX** (Third-party hg19 database supplying SPANR ΔΨ scores joined to each lifted variant; mutations without scores are removed) — http://www.openbioinformatics.org/annovar/spidex_download_form.php
- **PredPSI-SVR** (Downstream consumer of the lifted benchmark; its VCF filter drops variants in intergenic regions or distant to splice sites (more than 200 bp)) — https://github.com/chenkenbio/PredPSI-SVR

## Examples

```
cat *.fa > hg19.fasta; $HOME/PredPSI-SVR/tools/samtools-1.9/samtools faidx hg19.fasta
```

## Evaluation signals

- Every variant in the output table is in hg19 coordinates and lifts back cleanly against the samtools faidx-indexed hg19.fasta; liftover failures are logged and excluded.
- Every retained mutation is within 300 bp of a splice site per the ANNOVAR ensGene hg19 annotation.
- Every retained mutation carries a SPANR/SPIDEX ΔΨ score from the hg19_spidex database; rows without scores were removed.
- SynonMut-complete contains exactly 757 pathogenic and 10822 normal mutations.
- SynonMut-psi contains exactly 87 pathogenic and 826 normal mutations, all with experimental Ψ values.

## Limitations

- Variants that fail hg38→hg19 lift over with CrossMap are silently lost unless failures are explicitly tracked, biasing the assembled benchmark.
- PredPSI-SVR filters the input VCF to remove variants in intergenic regions or distant to splice sites (more than 200 bp), so benchmark variants in the 200–300 bp window will not survive a full PredPSI-SVR run.
- The hg19 reference must be indexed (samtools faidx) before liftover/coordinate checks; unindexed FASTA or missing contigs cause coordinate lookups to fail.

## Evidence

- [readme] Download hg19 genome: "wget -c http://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/chromFa.tar.gz"
- [readme] Index the hg19 FASTA with samtools faidx so lifted coordinates can be validated: "$HOME/PredPSI-SVR/tools/samtools-1.9/samtools faidx hg19.fasta"
- [methods] Annotate lifted hg19 variants against ensGene to assess splice-site proximity: "./annotate_variation.pl -buildver hg19 -downdb -webfrom annovar ensGene ./humandb/"
- [methods] hg19-locked SPIDEX database is the source of SPANR ΔΨ scores joined after liftover: "Download third-party database [SPIDEX](http://tools.genes.toronto.edu/) from http://www.openbioinformatics.org/annovar/spidex_download_form.php"
- [methods] Downstream tool applies its own splice-proximity filter to the VCF after liftover: "will filter the VCF file at first to remove variants in intergenic regions or distant to splice sites (more than 200 bp)"
