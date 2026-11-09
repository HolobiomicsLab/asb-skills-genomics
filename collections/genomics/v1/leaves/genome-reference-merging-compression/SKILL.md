---
name: genome-reference-merging-compression
description: 'Use when preparing a reference genome for long-read (PacBio HiFi / ONT
  PromethION) WGS alignment for HLA typing: you need updated IMGT/HLA alleles appended
  to GRCh38 with chr6 removed.'
when_to_use_negative:
- Short-read Illumina HLA typing via HPTASR/HPRA — those paths align directly to IMGT/HLA
  allele databases and do not require the merged genome reference
- When the input dataset targets RNA-seq or exome data that never spans the genomic
  HLA loci requiring genome-background alignment
- When you need reads to align to native chromosome 6 — appending HLA alleles while
  keeping chr6 would create ambiguous mappings, so the noChr6 genome is essential
  to this design
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3534
tools:
- name: pigz
  role: Parallel compression used to concatenate the GRCh38 noChr6 genome with HLA-I_II_GEN.fasta
    into the single .fa.gz alignment reference
- name: HLAminer
  role: Protocol whose long-read alignment step consumes the merged reference; provides
    download scripts and the chr-only-noChr6 genome deposit
  repo: https://github.com/bcgsc/HLAminer
- name: NCBI BLAST (formatdb/blastall)
  role: formatdb builds the HLA_ABC_GEN.fasta nucleotide search index alongside the
    merged reference construction
- name: bwa
  role: bwa index -a is indexes the concatenated HLA allele fasta for downstream alignment
    use
- name: minimap2
  role: Long-read aligner that maps reads against the merged .fa.gz reference, streaming
    SAM output into HLAminer
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:75d179e7e6b91a8ccb7f2c3babd6f973586ee9bc0857494cc49a0f514160bb47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - NCBI BLAST
  - NCBI BLAST (formatdb/blastall)
  - bwa
  - minimap2
  - pigz
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3534
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# genome-reference-merging-compression

## Summary

Builds a single long-read alignment reference by concatenating updated IMGT/HLA class I and II genomic allele sequences (HLA-I_II_GEN.fasta) with the GRCh38 chromosome-only genome lacking chromosome 6, compressing the result with pigz into one .fa.gz file so reads spanning the MHC align to HLA alleles rather than missing chr6 sequence.

## When to use

Use when preparing a reference genome for long-read (PacBio HiFi / ONT PromethION) WGS alignment for HLA typing: you need updated IMGT/HLA alleles appended to GRCh38 with chr6 removed.

## When NOT to use

- Short-read Illumina HLA typing via HPTASR/HPRA — those paths align directly to IMGT/HLA allele databases and do not require the merged genome reference
- When the input dataset targets RNA-seq or exome data that never spans the genomic HLA loci requiring genome-background alignment
- When you need reads to align to native chromosome 6 — appending HLA alleles while keeping chr6 would create ambiguous mappings, so the noChr6 genome is essential to this design

## Inputs

- IMGT/HLA class I genomic allele fasta files (A_gen.fasta, B_gen.fasta, C_gen.fasta) from ftp://ftp.ebi.ac.uk/pub/databases/imgt/mhc/hla/fasta/
- IMGT/HLA class II genomic allele fasta files
- GRCh38 chromosome-only genome without chromosome 6: GCA_000001405.15_GRCh38_genomic.chr-only-noChr6.fa.gz (from https://www.bcgsc.ca/downloads/btl/hlaminer/)
- Optional: chr6hlamasked GRCh38 genome masking extended MHC region at coordinates 28510120-33480577

## Outputs

- HLA_ABC_GEN.fasta (reheadered concatenated class I allele genomic sequences)
- HLA-I_II_GEN.fasta (combined class I and class II genomic allele sequences)
- GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz (single pigz-compressed merged alignment reference)
- P-designation file hla_nom_p.txt (from http://hla.alleles.org/wmda/hla_nom_p.txt) for downstream predictions

## How to apply

Download A_gen.fasta, B_gen.fasta, C_gen.fasta (plus class II genomic fasta files) from the IMGT/HLA ftp site (ftp://ftp.ebi.ac.uk/pub/databases/imgt/mhc/hla/fasta/) and concatenate them, reheadering with the perl one-liner 'if(/\>\S+\s+(\S+)/){print ">$1\n";}' to retain only allele names, yielding HLA_ABC_GEN.fasta; combine class I and II sequences into HLA-I_II_GEN.fasta and index (formatdb -p F -i HLA_ABC_GEN.fasta; bwa index -a is). Separately fetch the chromosome-only GRCh38 genome without chromosome 6 (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6.fa.gz) from https://www.bcgsc.ca/downloads/btl/hlaminer/ — removing chr6 prevents reads mapping ambiguously to the native locus instead of the appended allele sequences. Concatenate the genome with HLA-I_II_GEN.fasta using pigz to produce GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz, the single reference used by minimap2 for streamed long-read alignments into HLAminer. Optionally repeat the concatenation with a chr6hlamasked genome that masks the extended MHC region at GRCh38 coordinates 28510120-33480577, and also obtain the hla_nom_p.txt P-designation file for downstream allele collapse.

## Related tools

- **pigz** (Parallel compression used to concatenate the GRCh38 noChr6 genome with HLA-I_II_GEN.fasta into the single .fa.gz alignment reference)
- **HLAminer** (Protocol whose long-read alignment step consumes the merged reference; provides download scripts and the chr-only-noChr6 genome deposit) — https://github.com/bcgsc/HLAminer
- **NCBI BLAST (formatdb/blastall)** (formatdb builds the HLA_ABC_GEN.fasta nucleotide search index alongside the merged reference construction)
- **bwa** (bwa index -a is indexes the concatenated HLA allele fasta for downstream alignment use)
- **minimap2** (Long-read aligner that maps reads against the merged .fa.gz reference, streaming SAM output into HLAminer) — https://github.com/lh3/minimap2

## Examples

```
cat GRCh38.chr-only-noChr6.fa HLA-I_II_GEN.fasta | pigz > GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz
```

## Evaluation signals

- Validation: confirm the rebuilt reference and P-designation file match the static January 2025 deposit (Zenodo doi 10.5281/zenodo.14751276 / bcgsc.ca downloads) — output_matches_reference
- The final output is a single compressed file named GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz containing both genome contigs and HLA allele sequences
- The reheadered HLA_ABC_GEN.fasta retains only allele names in headers (perl 'if(/\>\S+\s+(\S+)/){print ">$1\n";}') with no extraneous description fields
- The genome component must be chr-only and exclude chromosome 6; verify no chr6 contig remains in the merged reference
- Downstream smoke test: minimap2 alignments streamed into HLAminer v1.4 yield HLA-I and -II predictions, indicating the reference indexes correctly

## Limitations

- The merged reference is tied to a specific genome build and IMGT/HLA database snapshot — an older chr6hlamasked variant masks the extended MHC region at GRCh38 coordinates 28510120-33480577, and builds differ; the article validates against a static January 2025 deposit
- HLAminer protocol demonstration relied on older, less accurate ONT chemistry and relatively low (10x) coverage, so reference quality bounds what predictions can recover
- Removing chr6 is mandatory for this design; using a full genome with chr6 plus appended alleles would defeat the alignment strategy

## Evidence

- [methods] Final long-read alignment reference is built by concatenating HLA-I_II_GEN.fasta allele sequences with the GRCh38 chromosome-only genome (no chr6) and compressing them into a single .fa.gz reference.: [evidence span withheld — anchor status: not_found]
- [methods] Concatenate the genome file with HLA-I_II_GEN.fasta using pigz to produce GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz: [evidence span withheld — anchor status: not_found]
- [methods] Fetch the GRCh38 chromosome-only genome without chromosome 6, GCA_000001405.15_GRCh38_genomic.chr-only-noChr6.fa.gz, from https://www.bcgsc.ca/downloads/btl/hlaminer/.: [evidence span withheld — anchor status: not_found]
- [intro] This protocol enables the preparation of reference genome sequences for long‐read alignments: "This protocol enables the preparation of reference genome sequences for long‐read alignments and, using a single command at the Unix terminal, prediction of likely HLA class I and II alleles"
- [methods] GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz: "GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz"
