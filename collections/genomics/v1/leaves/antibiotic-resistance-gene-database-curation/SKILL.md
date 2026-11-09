---
name: antibiotic-resistance-gene-database-curation
description: Use when constructing or updating a curated ARG reference database for
  read-based environmental surveillance (e.g., SARG+ for Argo/Lynx) from CARD, NDARO,
  SARG, and RefSeq PGAP-annotated proteins.
when_to_use_negative:
- The goal is de novo identification of ARG genomic contexts or hosts in novel, understudied
  environments — the article notes assembly-based (metaFlye + Kraken2) and binning-based
  (SemiBin2 + GTDB-Tk) methods are better suited for that.
- Applications requiring ORF-based annotation or detailed ARG sequence typing (e.g.,
  distinguishing blaOXA-1 from blaOXA-1042), since the clustering step deliberately
  merges highly similar ARGs into single subtypes.
- Detection of point-mutation-derived ARGs is required — such ARGs are consistently
  excluded from the database because read-based profiling cannot reliably distinguish
  SNPs from sequencing errors.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0102
tools:
- name: MMseqs2
  role: easy-cluster deduplication of the reference set (-s 7.5 -c 0.995 --min-seq-id
    0.995 --cov-mode 0 --cluster-reassign) and the extension set (-c 0.95 --min-seq-id
    0.95)
- name: diamond
  role: makedb over refseq_protein FASTA and over the final prot.fa/sarg.fa for Argo
    indexing
- name: blast
  role: blastdbcmd extraction of nr/env_nr sequences into FASTA during header standardization
- name: kofamscan
  role: KOfam-based annotation support during curation
- name: seqkit
  role: sequence file manipulation across curation notebooks
- name: Argo
  role: downstream long-read ARG profiler whose default database is SARG+; embedding
    validates the curated files
  repo: https://github.com/xinehc/argo
- name: Lynx
  role: downstream short-read ARG profiler embedding SARG+ (beta-testing)
  repo: https://github.com/xinehc/lynx
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:2c76cc8c37d0d71c92717e3623044c0fc859c2b6e0d12b6503c4656fcbacc13a
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
  - Lynx
  - MMseqs2
  - blast
  - diamond
  - kofamscan
  - seqkit
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# antibiotic-resistance-gene-database-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curates SARG+, a species-level antibiotic resistance gene (ARG) protein reference database, by merging CARD, NDARO, and SARG into a consistent type/subtype/sequence hierarchy, extending it with RefSeq homologs sharing PGAP annotation evidence, and deduplicating via MMseqs2 clustering to yield 104,529 sequences.

## When to use

Use when constructing or updating a curated ARG reference database for read-based environmental surveillance (e.g., SARG+ for Argo/Lynx) from CARD, NDARO, SARG, and RefSeq PGAP-annotated proteins.

## When NOT to use

- The goal is de novo identification of ARG genomic contexts or hosts in novel, understudied environments — the article notes assembly-based (metaFlye + Kraken2) and binning-based (SemiBin2 + GTDB-Tk) methods are better suited for that.
- Applications requiring ORF-based annotation or detailed ARG sequence typing (e.g., distinguishing blaOXA-1 from blaOXA-1042), since the clustering step deliberately merges highly similar ARGs into single subtypes.
- Detection of point-mutation-derived ARGs is required — such ARGs are consistently excluded from the database because read-based profiling cannot reliably distinguish SNPs from sequencing errors.

## Inputs

- CARD v3.2.9 protein reference sequences
- NDARO ver. 2024-05-02.2 (ReferenceGeneCatalog.txt and AMRProt.fa from NCBI AMRFinderPlus)
- SARG v3.2.1 protein sequences
- NCBI refseq_protein release (wp_protein FASTA and gpff GenBank records for plasmid/archaea/bacteria kingdoms)
- NCBI nr and env_nr protein databases (BLAST DB format)
- KEGG KOfam profiles (ko_list and profiles.tar.gz)
- sarg.json and reference/reference.fasta specifying new ARG types, literature, and sequences

## Outputs

- prot.fa (curated protein FASTA for Argo)
- sarg.fa (combined SARG+ reference + extension)
- sarg_ref.fa (experimentally validated reference component)
- sarg_ext.fa (computationally derived extension component)
- hierarchy table recording type/subtype/sequence assignments per cluster
- misc/summary.tsv (types, subtypes, sources) and misc/evidence.tsv (evidence used for the extension)
- misc/sarg.txt, misc/sarg_ref.txt, misc/sarg_ext.txt sequence counts

## How to apply

Download protein references from CARD (v3.2.9), NDARO (AMRProt.fa + ReferenceGeneCatalog.txt, ver. 2024-05-02.2), and SARG (v3.2.1), then merge them into a consistent hierarchy of type (class/family), subtype (subclass/gene), and sequence, labeling sequences belonging to more than one subtype with an asterisk (e.g., cml*). Remove regulators (activators/repressors), housekeeping genes, and ARGs arising from point mutations to form a core reference of 39 types, 1039 subtypes, and 7041 sequences. Augment with all RefSeq protein sequences annotated by the NCBI PGAP pipeline through the same evidence (BlastRules or Hidden Markov Models) as experimentally validated ARGs, producing an extension of 476,546 sequences; evidence accessions are parsed from RefSeq GPFF 'Evidence-For-Name-Assignment' structured comments. Deduplicate with MMseqs2 v15.6f45266 easy-cluster using '-s 7.5 -c 0.995 --min-seq-id 0.995 --cov-mode 0 --cluster-reassign' for the reference set and '-s 7.5 -c 0.95 --min-seq-id 0.95 --cov-mode 0 --cluster-reassign' for the extension set, since highly similar ARGs (e.g., blaOXA-1 vs blaOXA-1042, a single R101G change) are grouped into one subtype to avoid ambiguous read assignment. In the sarg-curation repo this is executed as Jupyter notebooks n1-parse-refs, n2-merge-headers, n3-parse-evidence, and n4-remove-dups, which write sarg.fa, sarg_ref.fa, sarg_ext.fa, and a hierarchy/summary table (misc/summary.tsv, misc/evidence.tsv).

## Related tools

- **MMseqs2** (easy-cluster deduplication of the reference set (-s 7.5 -c 0.995 --min-seq-id 0.995 --cov-mode 0 --cluster-reassign) and the extension set (-c 0.95 --min-seq-id 0.95))
- **diamond** (makedb over refseq_protein FASTA and over the final prot.fa/sarg.fa for Argo indexing)
- **blast** (blastdbcmd extraction of nr/env_nr sequences into FASTA during header standardization)
- **kofamscan** (KOfam-based annotation support during curation)
- **seqkit** (sequence file manipulation across curation notebooks)
- **Argo** (downstream long-read ARG profiler whose default database is SARG+; embedding validates the curated files) — https://github.com/xinehc/argo
- **Lynx** (downstream short-read ARG profiler embedding SARG+ (beta-testing)) — https://github.com/xinehc/lynx

## Examples

```
mmseqs easy-cluster sarg_ref.fa sarg_ref_cluster tmp -s 7.5 -c 0.995 --min-seq-id 0.995 --cov-mode 0 --cluster-reassign
```

## Evaluation signals

- Final deduplicated database contains exactly 104,529 protein sequences spanning 39 types and 1053 subtypes.
- Exactly 81 subtypes are classified as Risk Rank I.
- Core reference (before extension) contains 39 types, 1039 subtypes, and 7041 sequences; extension contributes 476,546 sequences before clustering.
- Sequences belonging to more than one subtype are consistently labeled with an asterisk (e.g., cml*).
- Output files sarg.fa / sarg_ref.fa / sarg_ext.fa match the counts in misc/sarg.txt, misc/sarg_ref.txt, and misc/sarg_ext.txt; cluster representatives and hierarchy table agree (one representative per subtype cluster).

## Limitations

- ARGs arising from point mutations are excluded because read-based profiling cannot reliably distinguish SNPs from sequencing errors.
- Grouping highly similar ARGs (e.g., blaOXA-1 and blaOXA-1042, differing by a single R101G change) into single subtypes lowers resolution for ORF-based annotation or detailed sequence typing.
- Extracting sequences and annotation evidence from nr/env_nr and RefSeq can be time-consuming; README advises running blastdbcmd and diamond makedb in parallel across cores.

## Evidence

- [readme] SARG+ purpose: [evidence span withheld — anchor status: not_found]
- [readme] RefSeq PGAP evidence extension: [evidence span withheld — anchor status: not_found]
- [other] final database scope: [evidence span withheld — anchor status: not_found]
- [other] MMseqs2 dedup parameters: [evidence span withheld — anchor status: not_found]
- [discussion] clustering caveat: "This clustering lowers the resolution of ARGs and could be a drawback for applications requiring open reading frame (ORF)-based annotation or detailed sequence typing."
