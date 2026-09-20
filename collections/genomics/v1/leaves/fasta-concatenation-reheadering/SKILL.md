---
name: fasta-concatenation-reheadering
description: Use when building a combined human-genome-plus-HLA reference for long-read
  (nanopore/PacBio) alignment in the HLAminer protocol, or when HLA allele FASTA headers
  must be reduced to allele names before concatenation.
when_to_use_negative:
- The reference already exists as GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz
  in a validated deposit (Zenodo doi 10.5281/zenodo.14751276 / bcgsc.ca downloads)
  — reuse it instead of rebuilding.
- The task is short-read HLAminer prediction via HPTASR/HPRA against the IMGT/HLA
  allele database only, not whole-genome long-read alignment, which does not require
  the concatenated genome reference.
tools:
- name: HLAminer
  role: Pipeline whose long-read alignment reference this skill constructs; its ./database/
    shell scripts automate downloading and formatting the HLA databases
  repo: https://github.com/bcgsc/HLAminer
- name: NCBI BLAST
  role: formatdb is used to format the concatenated/reheadered HLA allele FASTA (formatdb
    -p F -i HLA_ABC_GEN.fasta) for reciprocal BLASTN searches
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:60379b7ace16836e43c3fa62113436d8ca04b6f84627ae67de781a41ef9fdc03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - NCBI BLAST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# fasta-concatenation-reheadering

## Summary

Prepares the single FASTA reference used for HLAminer long-read alignments by reheadering IMGT/HLA allele genomic FASTA files to allele names only, concatenating them with the GRCh38 chromosome-only genome lacking chr6, and compressing everything into one .fa.gz.

## When to use

Use when building a combined human-genome-plus-HLA reference for long-read (nanopore/PacBio) alignment in the HLAminer protocol, or when HLA allele FASTA headers must be reduced to allele names before concatenation.

## When NOT to use

- The reference already exists as GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz in a validated deposit (Zenodo doi 10.5281/zenodo.14751276 / bcgsc.ca downloads) — reuse it instead of rebuilding.
- The task is short-read HLAminer prediction via HPTASR/HPRA against the IMGT/HLA allele database only, not whole-genome long-read alignment, which does not require the concatenated genome reference.

## Inputs

- IMGT/HLA class I allele genomic FASTA files (A_gen.fasta, B_gen.fasta, C_gen.fasta)
- IMGT/HLA class II gen FASTA files
- GRCh38 chromosome-only genome without chr6: GCA_000001405.15_GRCh38_genomic.chr-only-noChr6.fa.gz
- hla_nom_p.txt P-designation file from http://hla.alleles.org/wmda/hla_nom_p.txt (for accompanying validation)

## Outputs

- HLA_ABC_GEN.fasta (reheadered class I allele sequences, allele-name-only headers)
- HLA-I_II_GEN.fasta (combined class I + class II genomic sequences)
- GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz (single gzipped alignment reference)
- BLAST and bwa indexes (formatdb -p F; bwa index -a is) built on the allele FASTA

## How to apply

Download the allele genomic FASTA files (A_gen.fasta, B_gen.fasta, C_gen.fasta and class II gen FASTA) from ftp://ftp.ebi.ac.uk/pub/databases/imgt/mhc/hla/fasta/ and reheader them with the perl one-liner 'if(/\>\S+\s+(\S+)/){print ">$1\n";}' so only allele names remain, writing HLA_ABC_GEN.fasta. Combine class I and class II genomic sequences into HLA-I_II_GEN.fasta and build search indexes (formatdb -p F -i HLA_ABC_GEN.fasta; bwa index -a is). Fetch the GRCh38 chromosome-only genome without chr6 (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6.fa.gz) from https://www.bcgsc.ca/downloads/btl/hlaminer/, then concatenate it with HLA-I_II_GEN.fasta using pigz to produce GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz. Optionally repeat with the chr6hlamasked genome masking the extended MHC region (GRCh38 coordinates 28510120-33480577), and validate the rebuilt reference and the hla_nom_p.txt P-designation file against the static January 2025 deposit (Zenodo doi 10.5281/zenodo.14751276).

## Related tools

- **HLAminer** (Pipeline whose long-read alignment reference this skill constructs; its ./database/ shell scripts automate downloading and formatting the HLA databases) — https://github.com/bcgsc/HLAminer
- **NCBI BLAST** (formatdb is used to format the concatenated/reheadered HLA allele FASTA (formatdb -p F -i HLA_ABC_GEN.fasta) for reciprocal BLASTN searches)

## Examples

```
cat GCA_000001405.15_GRCh38_genomic.chr-only-noChr6.fa.gz HLA-I_II_GEN.fasta | pigz > GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz
```

## Evaluation signals

- output_matches_reference: the rebuilt GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz matches the static January 2025 deposit (Zenodo doi 10.5281/zenodo.14751276 / bcgsc.ca downloads)
- Reheadered FASTA headers contain only allele names (perl one-liner stripped the second header field)
- The compressed reference contains both the chr-less-6 GRCh38 chromosomes and the HLA-I_II_GEN allele sequences
- Search indexes build without error (formatdb -p F -i HLA_ABC_GEN.fasta; bwa index -a is) on the concatenated files

## Limitations

- HLA sequence databases must be continually refreshed from IMGT/HLA; the protocol relies on shell scripts in ./database/ to fetch updated sequences.
- An optional chr6hlamasked variant masks the extended MHC region at GRCh38 coordinates 28510120-33480577; choosing between the masked and unmasked reference is an explicit decision point.
- In the authors' tests, blast 2.2.22 may be faster than blast+ (2.2.28) while producing accurate results, so index/search tool version can affect downstream HPTASR performance.

## Evidence

- [methods] Reheadered allele FASTA construction: [evidence span withheld — anchor status: not_found]
- [methods] Index building on combined FASTA: [evidence span withheld — anchor status: not_found]
- [methods] Genome acquisition and final concatenation: [evidence span withheld — anchor status: not_found]
- [methods] Optional masked MHC variant: [evidence span withheld — anchor status: not_found]
- [methods] Automated database download scripts: "Follow these instructions to download updated HLA sequences from ebi/imgt (shell scripts to automatically download and format the databases exist in ./database/)"
