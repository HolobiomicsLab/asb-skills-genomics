---
name: chicken-genome-coordinate-mapping
description: Use when coding genes predicted from a genome-guided transcriptome assembly
  (e.g. StringTie GTF of chicken SCO PacBio+Illumina data) are absent from NCBI/Ensembl
  reference annotations and you need homolog names, functional descriptions, and single-copy
  vs paralog assignment.
when_to_use_negative:
- The organism lacks an assembled genome and an NCBI-available annotation in GTF format,
  since the upstream pipeline requires both
- The goal is de novo gene prediction without a reference genome or homology evidence
  — use ab initio tools (BRAKER/AUGUSTUS) instead
- Transcripts are already present in the NCBI RefSeq or Ensembl reference annotations;
  reconciliation, not homology-based recovery, is the appropriate step
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: ncbi-blast+
  role: blastp of novel predicted proteins against the UniProt Gallus gallus proteome
    with -max_hsps 1 -max_target_seqs 1
- name: eggNOG-mapper
  role: functional annotation intersected with the blastp hit table to assign gene
    names and protein descriptions to novel coding genes
- name: annotate_my_genomes
  role: pipeline that produces the novel-prot.fa protein sequences and reconciled
    GTF via the add-ncbi-annotation run
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: StringTie
  role: assembles the genome-guided transcriptome GTF from merged PacBio and Illumina
    BAM alignments that feeds the annotation run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:2670b796f85b3ae2ac6f6ab3f45fd6f948c48b0c08d36ea682b417e86b489ba6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - StringTie
  - annotate_my_genomes
  - eggNOG-mapper
  - ncbi-blast+
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# chicken-genome-coordinate-mapping

## Summary

Recovers genes missing from chicken reference annotations by blasting novel predicted proteins from an assembled SCO transcriptome against the UniProt Gallus gallus proteome (taxid 9031) and mapping hits to genomic loci, classifying single-copy homologs versus paralogs based on homology percentage and locus proximity.

## When to use

Use when coding genes predicted from a genome-guided transcriptome assembly (e.g. StringTie GTF of chicken SCO PacBio+Illumina data) are absent from NCBI/Ensembl reference annotations and you need homolog names, functional descriptions, and single-copy vs paralog assignment.

## When NOT to use

- The organism lacks an assembled genome and an NCBI-available annotation in GTF format, since the upstream pipeline requires both
- The goal is de novo gene prediction without a reference genome or homology evidence — use ab initio tools (BRAKER/AUGUSTUS) instead
- Transcripts are already present in the NCBI RefSeq or Ensembl reference annotations; reconciliation, not homology-based recovery, is the appropriate step

## Inputs

- novel-prot.fa: deduced protein sequences of coding genes absent from the reference annotation (from add-ncbi-annotation output on the StringTie GTF assembly)
- UniProt Gallus gallus proteome (taxid 9031, 34,814 sequences)
- eggNOG-mapper functional annotation table
- Genomic coordinates of the assembled transcripts (GTF, e.g. galGal6-based) for locus-distance calculations

## Outputs

- Blastp hit table with single best UniProt hit per novel query (-max_hsps 1 -max_target_seqs 1)
- Homolog annotation tables (Supplementary Tables 3-4) assigning gene names, UniProt hit descriptions, and single-copy vs paralog status to each novel coding gene

## How to apply

Extract deduced protein sequences (novel-prot.fa) for coding genes absent from the reference annotation, produced by the add-ncbi-annotation run on the StringTie assembly. Run blastp (NCBI BLAST+) against the UniProt Gallus gallus proteome (taxid 9031, 34,814 sequences) with -max_hsps 1 -max_target_seqs 1 to retain a single best hit per query. Apply the paralog criterion: two matches with 90-100% homology located within the same locus (<0.5 Mb) are called paralogs. Intersect the blastp hit table with eggNOG-mapper functional annotations to assign gene names and protein descriptions to the novel coding genes. This recovered USP53 (absent in NCBI but present in Ensembl), AASDH (missing in both databases), an A2ML3 paralog, and a novel VCP homolog on chromosome Z.

## Related tools

- **ncbi-blast+** (blastp of novel predicted proteins against the UniProt Gallus gallus proteome with -max_hsps 1 -max_target_seqs 1)
- **eggNOG-mapper** (functional annotation intersected with the blastp hit table to assign gene names and protein descriptions to novel coding genes)
- **annotate_my_genomes** (pipeline that produces the novel-prot.fa protein sequences and reconciled GTF via the add-ncbi-annotation run) — https://github.com/cfarkas/annotate_my_genomes
- **StringTie** (assembles the genome-guided transcriptome GTF from merged PacBio and Illumina BAM alignments that feeds the annotation run)

## Examples

```
blastp -query novel-prot.fa -db uniprot_gallus_9031 -max_hsps 1 -max_target_seqs 1 -outfmt 6 -out novel_blastp_hits.tsv
```

## Evaluation signals

- Each novel query in the blastp table carries exactly one best hit (enforced by -max_hsps 1 -max_target_seqs 1)
- The output table reproduces the homolog assignments in Supplementary Tables 3-4, with each novel gene carrying a UniProt hit description
- Recovered genes match expected single-copy vs paralog classification: USP53 and AASDH recovered as missing genes, A2ML3 flagged as paralog under the 90-100% homology within <0.5 Mb criterion
- The novel VCP homolog maps to chromosome Z at coordinates consistent with the UniProt hit locus

## Limitations

- Paralog classification depends on the 90-100% homology / <0.5 Mb same-loci heuristic, which may misclassify distant duplicates
- Missing genes are identified only if transcript evidence exists in the assembled (SCO brain) transcriptome; genes not expressed in the sampled tissue remain undetected
- Pipeline applicability is restricted to organisms with an assembled genome and an NCBI GTF annotation; no result is reported for organisms lacking these

## Evidence

- [discussion] blastp single-best-hit settings: "with the setting -max_hsps 1 -max_target_seqs 1 in blastp command [72]"
- [discussion] paralog criterion: "If two matches with 90–100% homology were found within the same loci (<0.5 Mb), we considered them as paralogs"
- [discussion] UniProt proteome blast: "we blasted the novel predicted proteins from these cds against the UniProt Gallus gallus proteome (taxid 9031)"
- [abstract] missing genes finding: "including the identification of missing genes in the chicken reference annotations by homology assignments"
- [discussion] pipeline scope restriction: "This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format"
