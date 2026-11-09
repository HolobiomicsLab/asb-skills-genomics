---
name: hla-p-designation-file-handling
description: Use when setting up the HLAminer long-read (nanopore/PacBio) alignment
  reference or when the summary must report only P-designated alleles for identical
  antigen-binding domains.
when_to_use_negative:
- Running the HPTASR short-read targeted-assembly path, which aligns contigs to IMGT/HLA
  allele references directly rather than to the concatenated GRCh38-noChr6+HLA reference
- The genome-wide reference already exists, is current, and matches the static Zenodo/BCGSC
  January 2025 deposit
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: HLAminer
  role: protocol scripts (./database/) that download, reheader and format the updated
    IMGT/HLA sequence databases and assemble the concatenated long-read reference;
    uses hla_nom_p.txt to collapse P-designated alleles in summaries
  repo: https://github.com/bcgsc/HLAminer
- name: NCBI BLAST (blastall/formatdb)
  role: formatdb -p F -i builds the nucleotide search index over the reheadered HLA_ABC_GEN.fasta
    / HLA-I_II_GEN.fasta allele sequences
- name: bwa
  role: bwa index -a is builds the alignment index over the HLA allele reference sequences
- name: minimap2
  role: aligns long reads against the concatenated GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz
    reference, streaming SAM output into HLAminer
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:39128a7c8332c22b3761c69a8906b4d3084559d4da7c7947e0cd02fc94067885
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - NCBI BLAST
  - NCBI BLAST (blastall/formatdb)
  - bwa
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# hla-p-designation-file-handling

## Summary

Prepares the HLAminer long-read alignment reference by combining updated IMGT/HLA class I and II genomic allele sequences (HLA-I_II_GEN.fasta) with the GRCh38 chromosome-only genome lacking chr6, and fetches the hla_nom_p.txt P-designation file that collapses alleles sharing antigen-binding domains in the prediction summary.

## When to use

Use when setting up the HLAminer long-read (nanopore/PacBio) alignment reference or when the summary must report only P-designated alleles for identical antigen-binding domains.

## When NOT to use

- Running the HPTASR short-read targeted-assembly path, which aligns contigs to IMGT/HLA allele references directly rather than to the concatenated GRCh38-noChr6+HLA reference
- The genome-wide reference already exists, is current, and matches the static Zenodo/BCGSC January 2025 deposit

## Inputs

- IMGT/HLA class I and class II genomic allele FASTA files (A_gen.fasta, B_gen.fasta, C_gen.fasta, class II gen fasta files)
- GRCh38 chromosome-only genome without chr6 (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6.fa.gz)
- hla_nom_p.txt P-designation file (http://hla.alleles.org/wmda/hla_nom_p.txt)

## Outputs

- Concatenated, pigz-compressed reference GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz
- Reheadered HLA_ABC_GEN.fasta / HLA-I_II_GEN.fasta allele databases with BLAST (formatdb -p F) and bwa (index -a is) search indexes
- Local hla_nom_p.txt P-designation table used to collapse identical antigen-binding-domain alleles in the prediction summary

## How to apply

Download the A_gen, B_gen, C_gen and class II genomic fasta files from ftp://ftp.ebi.ac.uk/pub/databases/imgt/mhc/hla/fasta/ and reheader them with the perl one-liner 'if(/\>\S+\s+(\S+)/){print ">$1\n";}' so only allele names are kept, then concatenate class I and II sequences into HLA-I_II_GEN.fasta and build indexes (formatdb -p F -i; bwa index -a is). Fetch hla_nom_p.txt from http://hla.alleles.org/wmda/hla_nom_p.txt so HLAminer can collapse predictions to the P-designated allele when sequences share the same antigen-binding domain. Fetch the GRCh38 chromosome-only genome without chromosome 6 (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6.fa.gz) from https://www.bcgsc.ca/downloads/btl/hlaminer/ and concatenate it with HLA-I_II_GEN.fasta using pigz to produce GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz (optionally repeat with the chr6hlamasked genome masking GRCh38 coordinates 28510120-33480577). Validate that the rebuilt reference and P-designation file match the static January 2025 deposit (Zenodo doi 10.5281/zenodo.14751276).

## Related tools

- **HLAminer** (protocol scripts (./database/) that download, reheader and format the updated IMGT/HLA sequence databases and assemble the concatenated long-read reference; uses hla_nom_p.txt to collapse P-designated alleles in summaries) — https://github.com/bcgsc/HLAminer
- **NCBI BLAST (blastall/formatdb)** (formatdb -p F -i builds the nucleotide search index over the reheadered HLA_ABC_GEN.fasta / HLA-I_II_GEN.fasta allele sequences)
- **bwa** (bwa index -a is builds the alignment index over the HLA allele reference sequences)
- **minimap2** (aligns long reads against the concatenated GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz reference, streaming SAM output into HLAminer) — https://github.com/lh3/minimap2

## Examples

```
cat GCA_000001405.15_GRCh38_genomic.chr-only-noChr6.fa HLA-I_II_GEN.fasta | pigz > GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz; wget http://hla.alleles.org/wmda/hla_nom_p.txt
```

## Evaluation signals

- Rebuilt reference and P-designation file match the static January 2025 deposit (Zenodo doi 10.5281/zenodo.14751276 / bcgsc.ca downloads), i.e. output_matches_reference
- Output file is a single .fa.gz named GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz containing chr-only GRCh38 sequences except chr6 plus the HLA-I_II_GEN.fasta alleles
- Reheadered HLA fasta entries retain only allele names (perl 'if(/\>\S+\s+(\S+)/){print ">$1\n";}') with no annotation text in headers
- formatdb and bwa index complete without error on HLA_ABC_GEN.fasta / HLA-I_II_GEN.fasta
- hla_nom_p.txt parses and the HLAminer summary keeps only the P-designated allele when sequences share the same antigen-binding domain

## Limitations

- Larger k values for TASR assembly yield increased specificity at possible expense of sensitivity; k-mer choice affects downstream predictions
- blast 2.2.22 may be faster than blast+ (2.2.28) while producing accurate results; BLAST version choice affects indexing/alignment runtime
- Optionally the chr6hlamasked genome (extended MHC masked at GRCh38 coordinates 28510120-33480577) can be substituted, which changes which reads align to the native MHC region

## Evidence

- [methods] The final long-read alignment reference is built by concatenating the HLA-I_II_GEN.fasta allele sequences with the GRCh38 chromosome-only genome (no chr6): [evidence span withheld — anchor status: not_found]
- [methods] 3. Download the P designation file hla_nom_p.txt from http://hla.alleles.org/wmda/hla_nom_p.txt.: [evidence span withheld — anchor status: not_found]
- [methods] Follow these instructions to download updated HLA sequences from ebi/imgt (shell scripts to automatically download and format the databases exist in ./database/): "Follow these instructions to download updated HLA sequences from ebi/imgt (shell scripts to automatically download and format the databases exist in ./database/)"
- [readme] only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain.: "Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele when the summary include HLA Sequences reported to have the same antigen binding domain."
