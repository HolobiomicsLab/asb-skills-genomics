---
name: paralog-classification-by-genomic-loci
description: Use when blastp hits for novel predicted coding genes show multiple high-identity
  matches and you must decide whether the query is a missing single-copy gene, a paralog,
  or a novel homolog based on genomic coordinates.
when_to_use_negative:
- Input transcripts are already present in the NCBI reference annotation (GTF reconciliation
  handles them; the homology/paralog scheme targets genes absent from reference annotations)
- Hits have <90% homology or are on different loci — they do not meet the paralog
  criterion and would be misclassified
- The organism lacks an assembled genome or NCBI GTF annotation, since genomic locus
  positions cannot be compared
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: ncbi-blast+
  role: Runs blastp of novel predicted proteins against the UniProt Gallus gallus
    proteome with -max_hsps 1 -max_target_seqs 1 to produce single best hits per query
- name: eggNOG-mapper
  role: Intersected with the blastp hit table to assign gene names and protein descriptions
    to novel coding genes
- name: annotate_my_genomes
  role: Upstream pipeline whose add-ncbi-annotation run produces the novel-prot.fa
    protein sequences and coding/noncoding annotations input to the homology analysis
  repo: https://github.com/cfarkas/annotate_my_genomes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:3dd06a15eefc2ae9d7c222b9051a854c225ada7588884c040120be984b7afa61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - annotate_my_genomes
  - eggNOG-mapper
  - ncbi-blast+
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Paralog classification by genomic loci

## Summary

A homology-based classification step, from the annotate_my_genomes workflow, that distinguishes single-copy homologs from paralogs among blastp hits of novel predicted proteins against the UniProt Gallus gallus proteome by checking whether 90-100% identity matches map to the same genomic locus (<0.5 Mb apart).

## When to use

Use when blastp hits for novel predicted coding genes show multiple high-identity matches and you must decide whether the query is a missing single-copy gene, a paralog, or a novel homolog based on genomic coordinates.

## When NOT to use

- Input transcripts are already present in the NCBI reference annotation (GTF reconciliation handles them; the homology/paralog scheme targets genes absent from reference annotations)
- Hits have <90% homology or are on different loci — they do not meet the paralog criterion and would be misclassified
- The organism lacks an assembled genome or NCBI GTF annotation, since genomic locus positions cannot be compared

## Inputs

- novel-prot.fa — novel predicted protein sequences from coding genes absent from the reference annotation (from add-ncbi-annotation run on the SCO StringTie assembly)
- UniProt Gallus gallus proteome (taxid 9031, 34,814 sequences)
- blastp hit table with single best hit per query (-max_hsps 1 -max_target_seqs 1)
- eggNOG-mapper functional annotation table
- Genomic coordinates of hit loci (e.g. galGal6 / bGalGal1.mat.broiler.GRCg7b annotations)

## Outputs

- Homolog annotation tables classifying each novel coding gene as single-copy homolog or paralog
- Paralog assignments (e.g. A2ML3 paralog) for queries with 90-100% identity matches within <0.5 Mb loci
- Gene names and protein descriptions attached to novel coding genes via eggNOG-mapper intersection
- Recovered missing-gene assignments (e.g. USP53, AASDH, VCP homolog on chromosome Z) reproducible against Supplementary Tables 3-4

## How to apply

First run blastp (NCBI BLAST+) of the novel predicted proteins (novel-prot.fa) against the UniProt Gallus gallus proteome (taxid 9031, 34,814 sequences) with -max_hsps 1 -max_target_seqs 1 to retain a single best hit per query. If two matches with 90-100% homology are found located within the same locus (<0.5 Mb apart), classify them as paralogs; otherwise assign the query as a single-copy homolog. Intersect the blastp hit table with eggNOG-mapper functional annotations to attach gene names and protein descriptions. Finally, cross-check genomic positions (e.g. the novel VCP homolog on chromosome Z) to confirm the assignment.

## Related tools

- **ncbi-blast+** (Runs blastp of novel predicted proteins against the UniProt Gallus gallus proteome with -max_hsps 1 -max_target_seqs 1 to produce single best hits per query)
- **eggNOG-mapper** (Intersected with the blastp hit table to assign gene names and protein descriptions to novel coding genes)
- **annotate_my_genomes** (Upstream pipeline whose add-ncbi-annotation run produces the novel-prot.fa protein sequences and coding/noncoding annotations input to the homology analysis) — https://github.com/cfarkas/annotate_my_genomes

## Examples

```
blastp -query novel-prot.fa -db uniprot_gallus_9031 -max_hsps 1 -max_target_seqs 1 -outfmt 6 -out blastp_hits.tsv
```

## Evaluation signals

- Each novel gene in the output table carries a UniProt hit description and is correctly classified as single-copy vs paralog status
- Paralog calls are supported by two matches with 90-100% homology within the same locus (<0.5 Mb)
- Output table reproduces the homolog assignments reported in Supplementary Tables 3-4 (USP53, AASDH, A2ML3 paralog, novel VCP homolog on chromosome Z)
- USP53 recovered as present in Ensembl but absent in NCBI annotation; AASDH missing in both databases
- blastp output has at most one HSP and one target per query, consistent with -max_hsps 1 -max_target_seqs 1

## Limitations

- The pipeline's homology scheme applies only to organisms with an assembled genome and an NCBI GTF annotation available
- Protein products of assembled SSPO isoforms were not validated, illustrating that transcript-level homology assignments do not confirm protein existence
- Paralog criterion is heuristic: two matches with 90-100% homology within the same locus (<0.5 Mb) — calls outside these bounds are ambiguous

## Evidence

- [discussion] blastp with -max_hsps 1 -max_target_seqs 1: "with the setting -max_hsps 1 -max_target_seqs 1 in blastp command [72]"
- [discussion] Paralog classification criterion: "If two matches with 90–100% homology were found within the same loci (<0.5 Mb), we considered them as paralogs"
- [abstract] Homology-based identification of missing genes: "including the identification of missing genes in the chicken reference annotations by homology assignments"
- [discussion] Pipeline scope restriction: "This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format"
- [methods] Pipeline availability and reproducibility: "git clone https://github.com/cfarkas/annotate_my_genomes.git"
