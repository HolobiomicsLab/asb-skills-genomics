---
name: arg-reference-hierarchy-standardization
description: Use when building or updating an ARG protein reference database (e.g.,
  from CARD, NDARO, SARG) that needs a uniform type/subtype hierarchy and deduplicated
  sequences for read-based metagenomic AMR surveillance.
when_to_use_negative:
- The goal is allele-level or SNP-level ARG sequence typing — SARG+ deliberately clusters
  highly similar ARGs (e.g., blaOXA-1/blaOXA-1042) and excludes point-mutation-derived
  ARGs, lowering resolution for ORF-based annotation.
- You need de novo identification of ARG genomic contexts or hosts in unstudied environments
  — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) approaches
  are better suited than reference curation.
- The input FASTA already follows the SARG+ type/subtype hierarchy and the target
  MMseqs2 clustering thresholds; re-curation would only duplicate the published database.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MMseqs2
  role: Deduplicates merged reference and extension sets via easy-cluster at min-seq-id/cov
    0.995 (reference) and 0.95 (extension)
- name: DIAMOND
  role: Builds a protein database (diamond makedb) from RefSeq .faa to extract annotation
    evidence for the SARG+ extension
- name: BLAST+ (blastdbcmd)
  role: Extracts sequences from nr and env_nr BLAST databases used to standardize
    SARG+ headers
- name: Argo
  role: Downstream long-read ARG profiler that consumes the curated SARG+ database
    (sarg.fa/prot.fa) as its default reference
  repo: https://github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:2f6baa96f2a6eb791d7f9670a42ec68c3ccd5d096c4c2bd859640471f85ad7c0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - BLAST+ (blastdbcmd)
  - DIAMOND
  - MMseqs2
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# arg-reference-hierarchy-standardization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates the SARG+ ARG reference database by merging CARD, NDARO, and SARG protein sequences into a consistent type (class/family) → subtype (subclass/gene) → sequence hierarchy, then extending and deduplicating it with MMseqs2 so that species-level read-based profilers like Argo can use stringent cutoffs without losing sensitivity.

## When to use

Use when building or updating an ARG protein reference database (e.g., from CARD, NDARO, SARG) that needs a uniform type/subtype hierarchy and deduplicated sequences for read-based metagenomic AMR surveillance.

## When NOT to use

- The goal is allele-level or SNP-level ARG sequence typing — SARG+ deliberately clusters highly similar ARGs (e.g., blaOXA-1/blaOXA-1042) and excludes point-mutation-derived ARGs, lowering resolution for ORF-based annotation.
- You need de novo identification of ARG genomic contexts or hosts in unstudied environments — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) approaches are better suited than reference curation.
- The input FASTA already follows the SARG+ type/subtype hierarchy and the target MMseqs2 clustering thresholds; re-curation would only duplicate the published database.

## Inputs

- CARD v3.2.9 protein reference sequences (FASTA)
- NDARO 2024-05-02.2 ReferenceGeneCatalog.txt and AMRProt.fa
- SARG v3.2.1 reference sequences (FASTA)
- NCBI RefSeq release wp_protein .gpff/.faa for plasmid, archaea, bacteria kingdoms
- NCBI nr and env_nr protein databases (via blastdbcmd extraction)
- KEGG KOfam ko_list and profiles
- sarg.json curation specification (type assignments, literature, rationales) and reference/reference.fasta for new ARGs

## Outputs

- sarg.fa — combined SARG+ reference and extension protein FASTA
- sarg_ref.fa — experimentally validated reference component (7041 sequences before extension clustering)
- sarg_ext.fa — computationally derived homolog extension component
- Hierarchy table recording type/subtype/sequence assignments per cluster (prot.fa headers)
- misc/summary.tsv (types, subtypes, sources of reference sequences), misc/evidence.tsv (evidence used for the extension), misc/sarg.txt, misc/sarg_ref.txt, misc/sarg_ext.txt count files

## How to apply

Download protein references from CARD v3.2.9, NDARO (2024-05-02.2; ReferenceGeneCatalog.txt and AMRProt.fa via AMRFinderPlus), and SARG v3.2.1, then run the sarg-curation notebooks: n1-parse-refs.ipynb parses NDARO and curates against sarg.json to build the initial reference; n2-merge-headers.ipynb standardizes headers against nr/env_nr. Remove regulators (activators/repressors), housekeeping genes, and point-mutation-derived ARGs — yielding a core of 39 types, 1039 subtypes, 7041 sequences — and collapse highly similar ARGs into single subtypes (e.g., blaOXA-1 and blaOXA-1042, which differ by one R101G change, become blaOXA), asterisk-labeling sequences spanning multiple subtypes (e.g., cml*). n3-parse-evidence.ipynb extends the database with RefSeq proteins annotated through the same PGAP evidence (BlastRules or Hidden Markov Models) as experimentally validated ARGs (~476,546 sequences). n4-remove-dups.ipynb deduplicates with MMseqs2 v15.6f45266 easy-cluster using '-s 7.5 -c 0.995 --min-seq-id 0.995 --cov-mode 0 --cluster-reassign' for the reference set and 0.95 thresholds for the extension set, writing the final protein FASTA (prot.fa/sarg.fa) plus a hierarchy table of type/subtype/sequence assignments per cluster.

## Related tools

- **MMseqs2** (Deduplicates merged reference and extension sets via easy-cluster at min-seq-id/cov 0.995 (reference) and 0.95 (extension))
- **DIAMOND** (Builds a protein database (diamond makedb) from RefSeq .faa to extract annotation evidence for the SARG+ extension)
- **BLAST+ (blastdbcmd)** (Extracts sequences from nr and env_nr BLAST databases used to standardize SARG+ headers)
- **Argo** (Downstream long-read ARG profiler that consumes the curated SARG+ database (sarg.fa/prot.fa) as its default reference) — https://github.com/xinehc/argo

## Examples

```
mmseqs easy-cluster sarg_ref.fa ref_cluster tmp --min-seq-id 0.995 -c 0.995 --cov-mode 0 --cluster-reassign -s 7.5
```

## Evaluation signals

- Final curated database contains exactly 104,529 protein sequences spanning 39 types and 1053 subtypes, with 81 subtypes classified as Risk Rank I.
- Pre-extension core database has 39 types, 1039 subtypes, and 7041 sequences after removing regulators, housekeeping genes, and point-mutation ARGs.
- Reference-set clusters were produced with MMseqs2 '-s 7.5 -c 0.995 --min-seq-id 0.995 --cov-mode 0 --cluster-reassign' and extension-set clusters with the 0.95 variants of the same flags.
- Sequences belonging to more than one subtype carry an asterisk-labeled subtype (e.g., cml*), and highly similar variants (blaOXA-1/blaOXA-1042) resolve to a single subtype.
- Counts in misc/sarg.txt, misc/sarg_ref.txt, and misc/sarg_ext.txt are consistent with sequence counts in sarg.fa, sarg_ref.fa, and sarg_ext.fa.

## Limitations

- Clustering highly similar ARGs into single subtypes lowers resolution and is a drawback for ORF-based annotation or detailed sequence typing.
- ARGs originating from point mutations are consistently excluded because read-based profiling cannot reliably distinguish SNPs from sequencing errors.
- SARG+ is reference-based; new ARGs can only be added by editing sarg.json and reference/reference.fasta, and evidence extraction from RefSeq/nr downloads is time-consuming (parallelize blastdbcmd and diamond makedb).

## Evidence

- [readme] SARG+ is a manually curated database of Antibiotic Resistance Genes (ARGs), designed to enhance read-based environmental surveillance at species-level resolution.: [evidence span withheld — anchor status: not_found]
- [readme] Finds sequences annotated through the same evidence (BlastRules and Hidden Markov Models) as SARG+ reference sequences.: [evidence span withheld — anchor status: external_doc]
- [readme] Removes duplicated and cross-mapped sequences by clustering.: [evidence span withheld — anchor status: external_doc]
