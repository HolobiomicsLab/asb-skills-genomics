---
name: novel-gene-annotation-in-transcriptome-assembly
description: Use when a genome-guided transcriptome assembly contains predicted coding
  genes missing from the reference GTF and you need homolog-based gene names, protein
  descriptions, and single-copy vs paralog status.
when_to_use_negative:
- The organism lacks an assembled genome and an NCBI annotation in GTF format — the
  underlying pipeline is restricted to such organisms
- The transcriptome assembly is de novo (not genome-guided), so no reference loci/coordinates
  exist for the <0.5 Mb paralog distance test
- The coding/noncoding status of the missing transcripts has not been established
  (e.g. no TransDecoder/FEELnc classification step was run)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: ncbi-blast+
  role: blastp of novel predicted proteins against the UniProt Gallus gallus proteome
    with -max_hsps 1 -max_target_seqs 1
- name: eggNOG-mapper
  role: intersected with the blastp hit table to assign gene names and protein descriptions
    to novel coding genes
- name: annotate_my_genomes
  role: produces the reconciled GTF and add-ncbi-annotation outputs (including novel-prot.fa)
    that feed this homology recovery step
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: TransDecoder
  role: predicts coding regions and deduced proteins of unclassified transcripts whose
    proteins become blastp queries
  repo: https://github.com/TransDecoder/TransDecoder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:9ee95dd30759f6e4d83f473bfa0e348ad7a679c930e955ab95e2a86553675b00
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
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# novel-gene-annotation-in-transcriptome-assembly

## Summary

Recover coding genes absent from the reference genome annotation by blasting novel predicted proteins from a genome-guided StringTie assembly against the UniProt Gallus gallus proteome (taxid 9031) and intersecting hits with eggNOG-mapper annotations to assign homolog/paralog names.

## When to use

Use when a genome-guided transcriptome assembly contains predicted coding genes missing from the reference GTF and you need homolog-based gene names, protein descriptions, and single-copy vs paralog status.

## When NOT to use

- The organism lacks an assembled genome and an NCBI annotation in GTF format — the underlying pipeline is restricted to such organisms
- The transcriptome assembly is de novo (not genome-guided), so no reference loci/coordinates exist for the <0.5 Mb paralog distance test
- The coding/noncoding status of the missing transcripts has not been established (e.g. no TransDecoder/FEELnc classification step was run)

## Inputs

- novel predicted protein sequences (novel-prot.fa) from coding genes absent from the reference annotation
- UniProt Gallus gallus proteome (taxid 9031, 34,814 sequences)
- eggNOG-mapper functional annotation table
- genomic coordinates of matched loci (for paralog distance testing)

## Outputs

- homolog annotation table mapping each novel coding gene to a UniProt hit description and single-copy vs paralog status
- blastp hit table with one best hit per query
- paralog classification (90-100% homology matches within <0.5 Mb of the same locus)

## How to apply

Extract the novel predicted protein sequences (novel-prot.fa) produced by the add-ncbi-annotation run on the StringTie assembly for coding genes absent from the reference annotation. Run blastp (NCBI BLAST+) against the UniProt Gallus gallus proteome (taxid 9031, 34,814 sequences) with -max_hsps 1 -max_target_seqs 1 to keep a single best hit per query. Classify a novel gene as a paralog when two matches with 90-100% homology fall within the same locus (<0.5 Mb); otherwise treat it as a single-copy homolog. Intersect the blastp hit table with eggNOG-mapper functional annotations to attach gene names and protein descriptions to the novel coding genes. The rationale is that a genome-guided assembly recovers transcripts the reference annotation missed, so homology against the same species' curated proteome recovers their identities.

## Related tools

- **ncbi-blast+** (blastp of novel predicted proteins against the UniProt Gallus gallus proteome with -max_hsps 1 -max_target_seqs 1)
- **eggNOG-mapper** (intersected with the blastp hit table to assign gene names and protein descriptions to novel coding genes)
- **annotate_my_genomes** (produces the reconciled GTF and add-ncbi-annotation outputs (including novel-prot.fa) that feed this homology recovery step) — https://github.com/cfarkas/annotate_my_genomes
- **TransDecoder** (predicts coding regions and deduced proteins of unclassified transcripts whose proteins become blastp queries) — https://github.com/TransDecoder/TransDecoder

## Examples

```
blastp -query novel-prot.fa -db uniprot_gallus_9031 -max_hsps 1 -max_target_seqs 1 -outfmt 6 -out novel_hits.tsv
```

## Evaluation signals

- Each query in the blastp table has exactly one hit (enforced by -max_hsps 1 -max_target_seqs 1)
- Recovered gene set reproduces the published assignments: USP53 (absent in NCBI annotation but present in Ensembl), AASDH (missing in both databases), the A2ML3 paralog, and the novel VCP homolog on chromosome Z
- Paralogs are called only when two matches share 90-100% homology within <0.5 Mb of the same locus
- Every novel gene in the output table carries a UniProt hit description from the eggNOG-mapper intersection
- Validation compares the homolog annotation table against Supplementary Tables 3-4 of the publication

## Limitations

- AASDH was missing in both NCBI and Ensembl chicken annotations, so recovery relies entirely on protein homology rather than reconciliation with any reference GTF
- Paralog calls depend on the <0.5 Mb same-locus criterion; genes outside that window or with weaker homology may be misclassified as single-copy
- Protein products of some assembled isoforms remain unvalidated; the method does not confirm protein existence experimentally

## Evidence

- [discussion] blasted the novel predicted proteins from these cds against the UniProt Gallus gallus proteome (taxid 9031): "blasted the novel predicted proteins from these cds against the UniProt Gallus gallus proteome (taxid 9031)"
- [discussion] blastp settings: "with the setting -max_hsps 1 -max_target_seqs 1 in blastp command [72]"
- [discussion] paralog criterion: "If two matches with 90–100% homology were found within the same loci (<0.5 Mb), we considered them as paralogs"
- [abstract] missing-gene finding: "including the identification of missing genes in the chicken reference annotations by homology assignments"
- [discussion] pipeline scope: "This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format"
