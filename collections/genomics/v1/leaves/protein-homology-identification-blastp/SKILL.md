---
name: protein-homology-identification-blastp
description: Use when coding genes predicted from an assembled transcriptome are absent
  from reference annotations (e.g. NCBI/Ensembl GTF) and their identity must be recovered
  by protein-level homology against a same-species UniProt proteome.
when_to_use_negative:
- The organism has no assembled genome or no NCBI GTF annotation available, since
  the pipeline requires both to define which genes are missing
- Coding genes are already present in the reference annotation (NCBI or Ensembl),
  so no novel proteins need homology assignment
- Only nucleotide-level (not protein-level) homology is needed — the method uses blastp
  on deduced proteins, not blastn
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0621
tools:
- name: ncbi-blast+
  role: Runs blastp of novel predicted proteins against the UniProt Gallus gallus
    proteome with -max_hsps 1 -max_target_seqs 1
- name: eggNOG-mapper
  role: Provides functional annotations intersected with the blastp hit table to assign
    gene names and protein descriptions
- name: annotate_my_genomes
  role: Pipeline (add-ncbi-annotation run) that produces the novel predicted protein
    sequences for coding genes absent from the reference annotation
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: TransDecoder
  role: Predicts coding regions and deduced proteins for unclassified transcripts
    prior to blastp
  repo: https://github.com/TransDecoder/TransDecoder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:20b47d9cab8355401ae93dcaeb342cd776df818a5202ae9e05ac1bc9d2ac0d21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - TransDecoder
  - annotate_my_genomes
  - eggNOG-mapper
  - ncbi-blast+
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# protein-homology-identification-blastp

## Summary

A method for recovering genes missing from a reference genome annotation by blasting novel predicted protein sequences (e.g. from a StringTie SCO transcriptome assembly) against the species' UniProt proteome and classifying hits as homologs or paralogs using genomic position and percent identity criteria.

## When to use

Use when coding genes predicted from an assembled transcriptome are absent from reference annotations (e.g. NCBI/Ensembl GTF) and their identity must be recovered by protein-level homology against a same-species UniProt proteome.

## When NOT to use

- The organism has no assembled genome or no NCBI GTF annotation available, since the pipeline requires both to define which genes are missing
- Coding genes are already present in the reference annotation (NCBI or Ensembl), so no novel proteins need homology assignment
- Only nucleotide-level (not protein-level) homology is needed — the method uses blastp on deduced proteins, not blastn

## Inputs

- novel predicted protein sequences in FASTA format (novel-prot.fa) from TransDecoder/add-ncbi-annotation on a StringTie assembly
- UniProt Gallus gallus proteome, taxid 9031 (34,814 sequences)
- eggNOG-mapper functional annotation table
- Genomic coordinates (GTF/annotation) of novel genes and blastp hits for paralog positional comparison

## Outputs

- blastp hit table with a single best hit per query (max_hsps 1, max_target_seqs 1)
- Homolog annotation table assigning UniProt gene names and protein descriptions to novel coding genes
- Single-copy homolog vs paralog classification (paralogs: 90–100% homology, same locus <0.5 Mb)

## How to apply

Extract the novel predicted protein sequences (novel-prot.fa) for coding genes absent from the reference annotation, produced by the add-ncbi-annotation run on the StringTie assembly. Run blastp (NCBI BLAST+) of these proteins against the UniProt Gallus gallus proteome (taxid 9031, 34,814 sequences) with -max_hsps 1 -max_target_seqs 1 to retain a single best hit per query. Classify pairs of hits with 90–100% homology located within the same locus (<0.5 Mb) as paralogs, versus single-copy homologs. Intersect the blastp hit table with eggNOG-mapper functional annotations to assign gene names and protein descriptions, assembling homolog annotation tables (e.g. USP53, AASDH, A2ML3 paralog, novel VCP homolog on chromosome Z).

## Related tools

- **ncbi-blast+** (Runs blastp of novel predicted proteins against the UniProt Gallus gallus proteome with -max_hsps 1 -max_target_seqs 1)
- **eggNOG-mapper** (Provides functional annotations intersected with the blastp hit table to assign gene names and protein descriptions)
- **annotate_my_genomes** (Pipeline (add-ncbi-annotation run) that produces the novel predicted protein sequences for coding genes absent from the reference annotation) — https://github.com/cfarkas/annotate_my_genomes
- **TransDecoder** (Predicts coding regions and deduced proteins for unclassified transcripts prior to blastp) — https://github.com/TransDecoder/TransDecoder

## Examples

```
blastp -query novel-prot.fa -db uniprot_gallus_9031 -max_hsps 1 -max_target_seqs 1 -outfmt 6 -out homolog_hits.tsv
```

## Evaluation signals

- Each novel query protein has exactly one best hit (max_hsps 1, max_target_seqs 1) with a UniProt description
- Paralog calls satisfy the criterion: two matches with 90–100% homology within the same locus (<0.5 Mb)
- Recovered homologs reproduce the assignments reported in Supplementary Tables 3–4 (USP53, AASDH, A2ML3 paralog, novel VCP homolog on chromosome Z)
- USP53 is recovered as absent in NCBI annotation but present in Ensembl; AASDH is missing in both databases
- Every novel coding gene in the output table carries a UniProt hit description and a correct single-copy vs paralog status

## Limitations

- Homology assignments depend on the completeness of the queried UniProt proteome; genes absent from both NCBI and Ensembl (like AASDH) can only be recovered via same-species protein homology
- The paralog criterion (90–100% identity, <0.5 Mb same locus) may misclassify very recent duplications or divergent paralogs outside these thresholds
- Protein products of some assembled isoforms (e.g. 89-exon and 25-exon SSPO isoforms) remain unvalidated

## Evidence

- [discussion] blastp with -max_hsps 1 -max_target_seqs 1: "with the setting -max_hsps 1 -max_target_seqs 1 in blastp command [72]"
- [discussion] paralog classification criterion: "If two matches with 90–100% homology were found within the same loci (<0.5 Mb), we considered them as paralogs"
- [discussion] UniProt proteome blast source: "blasted the novel predicted proteins from these cds against the UniProt Gallus gallus proteome (taxid 9031)"
- [abstract] missing-gene finding: "including the identification of missing genes in the chicken reference annotations by homology assignments"
- [discussion] pipeline scope restriction: "This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format"
