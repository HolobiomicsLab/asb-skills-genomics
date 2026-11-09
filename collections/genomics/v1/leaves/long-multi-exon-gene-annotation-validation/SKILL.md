---
name: long-multi-exon-gene-annotation-validation
description: Use when a merged long-read (PacBio) + short-read (Illumina) assembly
  is expected to recover a known large gene with >100 exons, and you must confirm
  exon counts, coding status, and protein-level fidelity against a reference.
when_to_use_negative:
- The organism lacks an assembled genome or an NCBI GTF annotation; the pipeline is
  restricted to organisms with both
- You only need raw transcript assembly (StringTie GTF) without protein-level validation
  of a specific locus
- Protein products of candidate isoforms require experimental validation (e.g. western/qPCR)
  rather than in-silico confirmation
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: annotate_my_genomes
  role: Generates the reconciled, coding/noncoding-annotated GTF (final_annotated.gtf)
    plus cds.fa and prot.fa used for isoform validation
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: StringTie
  role: Assembles transcripts from the merged PacBio+Illumina BAM (settings -p 1 -j
    2 -c 2 -v -a 4) providing the input GTF
- name: NCBI BLAST+ (blastp)
  role: Compares the deduced longest-isoform protein against the reference protein
    with -max_hsps 1 -max_target_seqs 1 to confirm identity
- name: AGAT
  role: GTF/GFF toolkit that must be installed for annotate_my_genomes annotation
    processing
  repo: https://github.com/NBISweden/AGAT
- name: GAWN
  role: Downstream genome annotation step producing protein annotations (gawn_config.sh)
    consumed in the validation workflow
  repo: https://github.com/enormandeau/gawn
- name: TransDecoder
  role: Predicts coding regions and deduced proteins for unclassified transcripts
    during coding/noncoding assignment
  repo: https://github.com/TransDecoder/TransDecoder
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:e6ad0e518ff412040242d70a4b2207a88d74534983c140545ed594ebec94a5eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - AGAT
  - Anaconda
  - GAWN
  - NCBI BLAST+ (blastp)
  - SAMtools
  - StringTie
  - TransDecoder
  - annotate_my_genomes
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Long multi-exon gene annotation validation

## Summary

Validates that a genome-guided transcriptome assembly recovers the full isoform structure of a very large, multi-exon gene (e.g. chicken SSPO/SCO-spondin, >105 exons) by counting exons per transcript, checking coding classification, verifying protein length, and confirming identity to a reference protein by blastp.

## When to use

Use when a merged long-read (PacBio) + short-read (Illumina) assembly is expected to recover a known large gene with >100 exons, and you must confirm exon counts, coding status, and protein-level fidelity against a reference.

## When NOT to use

- The organism lacks an assembled genome or an NCBI GTF annotation; the pipeline is restricted to organisms with both
- You only need raw transcript assembly (StringTie GTF) without protein-level validation of a specific locus
- Protein products of candidate isoforms require experimental validation (e.g. western/qPCR) rather than in-silico confirmation

## Inputs

- final_annotated.gtf from annotate_my_genomes (reconciled, coding/noncoding-annotated GTF)
- Reference genome annotation GTF (e.g. galGal6.gtf) for locus reconciliation
- Pipeline protein output prot.fa and CDS output cds.fa
- Reference protein sequence (e.g. AJ866919-deduced chicken SSPO protein) for blastp

## Outputs

- Per-isoform exon counts table confirming expected isoform set (e.g. 106/106/105/17/25 exons)
- Protein length per isoform (e.g. 5270 aa for transcript 1)
- blastp identity to reference protein (e.g. 98.88% to AJ866919)
- Coding vs lncRNA classification per isoform (e.g. five protein-coding SSPO isoforms)
- Compiled validation CSV (e.g. sspo_isoform_validation.csv)

## How to apply

Load the annotate_my_genomes output final_annotated.gtf (produced via ./annotate-my-genomes -a stringtie.gtf -r galGal6.gtf -g galGal6.fa -c gawn_config.sh -t 20 -o output1) and filter transcript records to the target locus using the gene_id field and genomic coordinate overlap. For each transcript, parse exon features from the GTF and count exons, confirming the expected isoform set (for chicken SSPO: two 106-exon isoforms plus 105-, 17-, and 25-exon isoforms, five coding isoforms in total). Extract CDS and protein sequences from the pipeline outputs cds.fa and prot.fa and verify the longest isoform's protein length (transcript 1 encodes 5270 aa). Align the deduced protein against the reference (AJ866919 SSPO protein) with blastp using -max_hsps 1 -max_target_seqs 1 and require identity consistent with prior reports (98.88% here). Compile exon counts, protein lengths, and blastp identity per isoform into a table such as sspo_isoform_validation.csv.

## Related tools

- **annotate_my_genomes** (Generates the reconciled, coding/noncoding-annotated GTF (final_annotated.gtf) plus cds.fa and prot.fa used for isoform validation) — https://github.com/cfarkas/annotate_my_genomes
- **StringTie** (Assembles transcripts from the merged PacBio+Illumina BAM (settings -p 1 -j 2 -c 2 -v -a 4) providing the input GTF)
- **NCBI BLAST+ (blastp)** (Compares the deduced longest-isoform protein against the reference protein with -max_hsps 1 -max_target_seqs 1 to confirm identity)
- **AGAT** (GTF/GFF toolkit that must be installed for annotate_my_genomes annotation processing) — https://github.com/NBISweden/AGAT
- **GAWN** (Downstream genome annotation step producing protein annotations (gawn_config.sh) consumed in the validation workflow) — https://github.com/enormandeau/gawn
- **TransDecoder** (Predicts coding regions and deduced proteins for unclassified transcripts during coding/noncoding assignment) — https://github.com/TransDecoder/TransDecoder

## Examples

```
./annotate-my-genomes -a stringtie.gtf -r galGal6.gtf -g galGal6.fa -c gawn_config.sh -t 20 -o output1 && blastp -query prot.fa -subject AJ866919_protein.fa -max_hsps 1 -max_target_seqs 1
```

## Evaluation signals

- Exon-count table reports the expected isoform set: 106/106/105/17/25 exons across five coding isoforms
- Longest isoform (transcript 1) protein length equals 5270 amino acids when extracted from prot.fa
- blastp (-max_hsps 1 -max_target_seqs 1) reports 98.88% identity to the AJ866919 reference protein
- All target-locus transcripts are classified as protein-coding, not lncRNA (FEELnc/TransDecoder-based classification)
- Final validation CSV (sspo_isoform_validation.csv) contains exon counts, protein lengths, and blastp identity per isoform

## Limitations

- Protein products of smaller SSPO isoforms (e.g. 25-exon and 89-exon transcripts) were not experimentally validated; lower 200-50 kDa bands were only inferred to correspond to smaller isoforms
- Pipeline applicability is restricted to organisms with an assembled genome and an NCBI GTF annotation; no results reported for organisms lacking these
- The method requires a reference genome as a guide; performance was demonstrated for chicken (galGal6, GCA_000002315.5) hybrid PacBio+Illumina data

## Evidence

- [abstract] correctly assembling and annotating all exons from the chicken SCO-spondin gene (containing more than 105 exons): "correctly assembling and annotating all exons from the chicken SCO-spondin gene (containing more than 105 exons)"
- [abstract] distinguishes between coding and long non-coding RNAs by integration of several bioinformatic approaches, including gene reconciliation with previous annotations in GTF format: "distinguishes between coding and long non-coding RNAs by integration of several bioinformatic approaches, including gene reconciliation with previous annotations in GTF format"
- [discussion] with the setting -max_hsps 1 -max_target_seqs 1 in blastp command: "with the setting -max_hsps 1 -max_target_seqs 1 in blastp command"
- [discussion] We did not discard the existence of these SSPO isoforms, but the protein products from these transcripts remains to be validated: "We did not discard the existence of these SSPO isoforms, but the protein products from these transcripts remains to be validated"
- [discussion] This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format: "This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format"
