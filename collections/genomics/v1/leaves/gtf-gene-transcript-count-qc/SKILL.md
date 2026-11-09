---
name: gtf-gene-transcript-count-qc
description: Use when a pipeline run has produced a reconciled GTF (final_annotated.gtf)
  and you must validate that its gene/transcript counts and novel-gene classification
  match reported values.
when_to_use_negative:
- Input is a raw StringTie or BRAKER GTF rather than a pipeline-reconciled final_annotated.gtf
  — counts would not reflect reconciled gene identities
- Organism lacks an assembled genome and an NCBI GTF annotation, which the annotate_my_genomes
  pipeline requires to produce the reconciled output
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: annotate_my_genomes
  role: Produces the reconciled final_annotated.gtf whose gene_id field and second
    field carry gene identities and novel gene/lncRNA classification to be counted
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: AGAT
  role: Toolkit installed for GTF/GFF validation and parsing support in the pipeline's
    annotation workflow
  repo: https://github.com/NBISweden/AGAT
- name: GAWN
  role: Genome annotation pipeline used within annotate_my_genomes that contributes
    to producing the reconciled GTF output
  repo: https://github.com/enormandeau/gawn
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:882ab2497b4f821af6dd19a37977057ed737097964ada4525587247d5c585e77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - AGAT
  - Anaconda
  - Docker
  - GAWN
  - SAMtools
  - annotate_my_genomes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# gtf-gene-transcript-count-qc

## Summary

Quality-control skill that parses a reconciled annotation GTF (e.g. final_annotated.gtf from annotate_my_genomes), counts unique reconciled genes and transcripts via the gene_id field, and checks the counts against reference values (19690 genes with 4292 novel candidates and 61940 transcripts for the chicken SCO transcriptome).

## When to use

Use when a pipeline run has produced a reconciled GTF (final_annotated.gtf) and you must validate that its gene/transcript counts and novel-gene classification match reported values.

## When NOT to use

- Input is a raw StringTie or BRAKER GTF rather than a pipeline-reconciled final_annotated.gtf — counts would not reflect reconciled gene identities
- Organism lacks an assembled genome and an NCBI GTF annotation, which the annotate_my_genomes pipeline requires to produce the reconciled output

## Inputs

- Reconciled annotation GTF file (final_annotated.gtf) with gene_id attribute and classification in the second field
- Reference reported values for comparison (e.g. 19690 genes / 4292 novel / 61940 transcripts)

## Outputs

- Unique reconciled gene count from the GTF gene_id field
- Unique reconciled transcript count
- Count of novel candidate genes not present in the reference annotation (e.g. galGal6 GRCg6a)
- LncRNA classification summary (e.g. divergent lncRNA proportion)

## How to apply

Parse final_annotated.gtf as a GTF, extracting the gene_id attribute to enumerate unique reconciled gene identities and the second field to count transcripts and read novel-gene/lncRNA classifications. Count unique genes and transcripts (expected: 19690 reconciled genes, of which 4292 are candidates absent from the chicken GRCg6a/galGal6 reference annotation, and 61940 reconciled transcripts for the merged-assembly SCO transcriptome). Validate the file parses as a well-formed GTF before counting, and cross-check against reported values rather than only confirming file existence. Optionally summarize the FEELnc-based lncRNA classification (expected ~27% divergent lncRNA proportion) to confirm classification counts accompany the gene-level counts.

## Related tools

- **annotate_my_genomes** (Produces the reconciled final_annotated.gtf whose gene_id field and second field carry gene identities and novel gene/lncRNA classification to be counted) — https://github.com/cfarkas/annotate_my_genomes
- **AGAT** (Toolkit installed for GTF/GFF validation and parsing support in the pipeline's annotation workflow) — https://github.com/NBISweden/AGAT
- **GAWN** (Genome annotation pipeline used within annotate_my_genomes that contributes to producing the reconciled GTF output) — https://github.com/enormandeau/gawn

## Examples

```
grep -oP 'gene_id "\K[^"]+' final_annotated.gtf | sort -u | wc -l
```

## Evaluation signals

- final_annotated.gtf exists and parses as a valid GTF
- Unique gene_id count equals 19690 reconciled genes
- Novel-gene subset equals 4292 candidate genes absent from the chicken GRCg6a reference annotation
- Unique transcript count equals 61940 reconciled transcripts
- FEELnc-based lncRNA summary shows ~27% divergent lncRNA proportion consistent with the reported Figure 1D-1G / Supplementary Table 1

## Limitations

- Counts are specific to the merged-assembly SCO transcriptome and galGal6 reference; other inputs will not match 19690/4292/61940
- Reported precision/recall and F1-score comparisons are deferred to Supplementary Table 2 and are not verifiable from the main GTF alone
- The pipeline assumes an assembled genome plus an available NCBI annotation in GTF format; it is not applicable without both

## Evidence

- [methods] Inspect final_annotated.gtf gene_id field: [evidence span withheld — anchor status: not_found]
- [methods] Compare counts against reported values: [evidence span withheld — anchor status: not_found]
- [discussion] GAWN pipeline used for annotation: "we used the GAWN pipeline (https://github.com/enormandeau/gawn)"
- [methods] AGAT toolkit requirement: "the AGAT toolkit (https://github.com/NBISweden/AGAT) must be installed"
- [discussion] Pipeline scope statement: "This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format"
