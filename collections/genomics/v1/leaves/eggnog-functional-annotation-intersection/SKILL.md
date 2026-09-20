---
name: eggnog-functional-annotation-intersection
description: Use when predicted proteins from coding genes absent from a reference
  GTF annotation (e.g. novel-prot.fa from an add-ncbi-annotation run) need homolog/paralog
  assignment and functional descriptions to recover missing genes.
when_to_use_negative:
- The organism lacks an assembled genome and an NCBI GTF annotation, since the workflow
  depends on reconciled genomic coordinates and a reference annotation (per the article's
  scope statement)
- The proteins already carry gene names and functional descriptions in the reference
  annotation, so homology-based recovery of missing genes is unnecessary
- A full multi-hit homology search is needed rather than a single best hit per query,
  since -max_target_seqs 1 suppresses secondary paralog matches outside the 90-100%
  same-locus rule
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0621
tools:
- name: ncbi-blast+
  role: blastp search of novel predicted proteins against the UniProt Gallus gallus
    proteome with -max_hsps 1 -max_target_seqs 1 to retain a single best hit per query
- name: eggNOG-mapper
  role: provides functional annotations that are intersected with the blastp hit table
    to assign gene names and protein descriptions to novel coding genes
- name: annotate_my_genomes
  role: pipeline whose add-ncbi-annotation run produces the novel-prot.fa protein
    sequences and reconciled GTF coordinates that feed the homology assignment
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: TransDecoder
  role: predicts the coding regions and deduced proteins for unclassified transcripts
    upstream of the homology assignment
  repo: https://github.com/TransDecoder/TransDecoder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:6f973e281502d7020851778bb3b3a757ce506568b7ea1fab32c6b7f54ec3aa36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - TransDecoder
  - annotate_my_genomes
  - eggNOG-mapper
  - ncbi-blast+
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# eggnog-functional-annotation-intersection

## Summary

Assigns gene names and functional descriptions to novel predicted coding proteins by intersecting a blastp hit table (single best hit against the UniProt Gallus gallus proteome, taxid 9031) with eggNOG-mapper functional annotations, and classifies hits as single-copy homologs vs paralogs (90-100% homology within <0.5 Mb loci).

## When to use

Use when predicted proteins from coding genes absent from a reference GTF annotation (e.g. novel-prot.fa from an add-ncbi-annotation run) need homolog/paralog assignment and functional descriptions to recover missing genes.

## When NOT to use

- The organism lacks an assembled genome and an NCBI GTF annotation, since the workflow depends on reconciled genomic coordinates and a reference annotation (per the article's scope statement)
- The proteins already carry gene names and functional descriptions in the reference annotation, so homology-based recovery of missing genes is unnecessary
- A full multi-hit homology search is needed rather than a single best hit per query, since -max_target_seqs 1 suppresses secondary paralog matches outside the 90-100% same-locus rule

## Inputs

- novel-prot.fa: predicted protein sequences of coding genes absent from the reference annotation (add-ncbi-annotation run on the SCO StringTie assembly)
- UniProt Gallus gallus proteome (taxid 9031, 34,814 sequences) as blastp target database
- eggNOG-mapper functional annotation table for the same proteins
- Genomic coordinates of hit loci (from the reconciled/NCBI GTF) to test the <0.5 Mb same-locus paralog criterion

## Outputs

- blastp hit table with one best hit per query (-max_hsps 1 -max_target_seqs 1)
- Homolog annotation tables (Supplementary Tables 3-4 equivalent) assigning gene names, UniProt hit descriptions, and single-copy vs paralog status to each novel coding gene

## How to apply

Extract the novel predicted protein sequences (novel-prot.fa) produced by the add-ncbi-annotation run on the StringTie assembly for coding genes absent from the reference annotation. Run blastp (NCBI BLAST+) against the UniProt Gallus gallus proteome (taxid 9031, 34,814 sequences) with -max_hsps 1 -max_target_seqs 1 so each query retains a single best hit. Apply the paralog criterion: two matches with 90-100% homology located within the same locus (<0.5 Mb) are called paralogs, otherwise hits are single-copy homologs. Intersect the blastp hit table with eggNOG-mapper functional annotations to transfer gene names and protein descriptions onto the novel coding genes, assembling homolog annotation tables that reproduce Supplementary Tables 3-4 (e.g. USP53, AASDH, the A2ML3 paralog, and the novel VCP homolog on chromosome Z).

## Related tools

- **ncbi-blast+** (blastp search of novel predicted proteins against the UniProt Gallus gallus proteome with -max_hsps 1 -max_target_seqs 1 to retain a single best hit per query)
- **eggNOG-mapper** (provides functional annotations that are intersected with the blastp hit table to assign gene names and protein descriptions to novel coding genes)
- **annotate_my_genomes** (pipeline whose add-ncbi-annotation run produces the novel-prot.fa protein sequences and reconciled GTF coordinates that feed the homology assignment) — https://github.com/cfarkas/annotate_my_genomes
- **TransDecoder** (predicts the coding regions and deduced proteins for unclassified transcripts upstream of the homology assignment) — https://github.com/TransDecoder/TransDecoder

## Examples

```
blastp -query novel-prot.fa -db uniprot_gallus_9031 -max_hsps 1 -max_target_seqs 1 -outfmt 6 -out blastp_hits.tsv
```

## Evaluation signals

- Every novel gene in the output table carries a UniProt hit description and a correctly classified single-copy vs paralog status
- The output table reproduces the homolog assignments reported in Supplementary Tables 3-4, including recovery of USP53, AASDH, the A2ML3 paralog, and the novel VCP homolog on chromosome Z
- Paralog calls strictly satisfy the rule that two matches with 90-100% homology lie within the same loci (<0.5 Mb); hits failing this are single-copy
- Each query in the blastp table has exactly one retained hit because -max_hsps 1 -max_target_seqs 1 was used
- USP53 is recovered as absent from NCBI annotation but present in Ensembl, whereas AASDH is missing in both databases, matching the reported finding

## Limitations

- The paralog criterion only detects 90-100% homology matches within <0.5 Mb of the same locus; more divergent or distally located paralogs are not classified
- Using -max_target_seqs 1 retains only the single best blastp hit per query, so secondary significant matches are discarded before intersection with eggNOG-mapper annotations
- Some missing genes remain unrecoverable by this scheme (AASDH was missing in both NCBI and Ensembl databases), and the pipeline is limited to organisms with an assembled genome and an NCBI annotation in GTF format

## Evidence

- [discussion] blastp with -max_hsps 1 -max_target_seqs 1: "with the setting -max_hsps 1 -max_target_seqs 1 in blastp command [72]"
- [discussion] paralog classification criterion: "If two matches with 90–100% homology were found within the same loci (<0.5 Mb), we considered them as paralogs"
- [discussion] UniProt Gallus gallus proteome target: "blasted the novel predicted proteins from these cds against the UniProt Gallus gallus proteome (taxid 9031)"
- [other] eggNOG-mapper intersection workflow: [evidence span withheld — anchor status: not_found]
- [other] USP53/AASDH finding: [evidence span withheld — anchor status: not_found]
