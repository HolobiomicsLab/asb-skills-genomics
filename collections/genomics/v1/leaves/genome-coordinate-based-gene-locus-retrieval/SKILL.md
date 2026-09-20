---
name: genome-coordinate-based-gene-locus-retrieval
description: Use when you have a genome-guided annotated GTF (e.g. annotate_my_genomes
  final_annotated.gtf) and need to isolate all transcripts of one gene locus for isoform
  or protein-level verification.
when_to_use_negative:
- The input is a raw StringTie or gffcompare GTF without reconciled gene_id attributes
  and reference-coordinate context — run it through annotate_my_genomes first
- No reference genome or NCBI GTF annotation is available for the organism, since
  the pipeline approach assumes an assembled genome and NCBI-available GTF annotation
- The goal is genome-wide annotation quality assessment (F1/precision/recall benchmarking)
  rather than retrieval of a single locus
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_0622
tools:
- name: annotate_my_genomes
  role: Produces the annotated/reconciled GTF (final_annotated.gtf) from which the
    locus is filtered
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: AGAT
  role: GTF/GFF toolkit used within the pipeline for annotation manipulation and reconciliation
  repo: https://github.com/NBISweden/AGAT
- name: NCBI BLAST+
  role: blastp validation of the protein extracted from the retrieved locus against
    a reference (e.g. AJ866919)
- name: SAMtools
  role: BAM handling in the upstream hybrid assembly feeding the annotated GTF
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:ecd256c84e7e393dda2ddb7d9d319d838fdb829c884fd426d2426fb4201652c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - AGAT
  - Anaconda
  - NCBI BLAST+
  - SAMtools
  - annotate_my_genomes
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# genome-coordinate-based-gene-locus-retrieval

## Summary

Extracts all transcript/exon records for a target gene (e.g. the chicken SSPO/SCO-spondin locus) from an annotated GTF by combining gene_id matching with genomic coordinate overlap filtering, enabling downstream isoform-level validation (exon counts, CDS/protein extraction).

## When to use

Use when you have a genome-guided annotated GTF (e.g. annotate_my_genomes final_annotated.gtf) and need to isolate all transcripts of one gene locus for isoform or protein-level verification.

## When NOT to use

- The input is a raw StringTie or gffcompare GTF without reconciled gene_id attributes and reference-coordinate context — run it through annotate_my_genomes first
- No reference genome or NCBI GTF annotation is available for the organism, since the pipeline approach assumes an assembled genome and NCBI-available GTF annotation
- The goal is genome-wide annotation quality assessment (F1/precision/recall benchmarking) rather than retrieval of a single locus

## Inputs

- Annotated GTF file from annotate_my_genomes (final_annotated.gtf), reconciled against a reference annotation (e.g. galGal6.gtf)
- Reference genome FASTA (e.g. galGal6.fa, GCA_000002315.5)
- Target gene identifier/name and/or genomic coordinates of the locus (e.g. SSPO / SCO-spondin)
- Pipeline CDS and protein FASTA outputs (cds.fa, prot.fa)

## Outputs

- Filtered GTF subset containing all exon/transcript/CDS features of the target gene locus
- Per-isoform exon counts (e.g. 106/106/105/17/25 for SSPO)
- Extracted CDS and protein FASTA sequences per isoform
- Compiled validation table (e.g. sspo_isoform_validation.csv) with exon counts, protein lengths, and BLAST identity

## How to apply

Load the pipeline output GTF (produced by running ./annotate-my-genomes -a stringtie.gtf -r galGal6.gtf -g galGal6.fa -c gawn_config.sh -t 20 -o output1) and filter transcript records using the gene_id field together with genomic coordinate overlap to pull all features belonging to the target locus, e.g. the SSPO/SCO-spondin gene on the galGal6 chicken genome. For each retrieved transcript, parse exon features and count exons to confirm the expected isoform structure (in the SSPO case: two 106-exon isoforms plus 105-, 17-, and 25-exon isoforms). Then join the filtered transcript IDs to the pipeline's cds.fa and prot.fa outputs to extract the corresponding CDS and protein sequences, and run blastp with -max_hsps 1 -max_target_seqs 1 to validate protein identity against a reference. The rationale is that gene_id alone can be inconsistent in merged/reconciled GTFs, so coordinate overlap acts as a second constraint to recover the complete locus including unannotated exons.

## Related tools

- **annotate_my_genomes** (Produces the annotated/reconciled GTF (final_annotated.gtf) from which the locus is filtered) — https://github.com/cfarkas/annotate_my_genomes
- **AGAT** (GTF/GFF toolkit used within the pipeline for annotation manipulation and reconciliation) — https://github.com/NBISweden/AGAT
- **NCBI BLAST+** (blastp validation of the protein extracted from the retrieved locus against a reference (e.g. AJ866919))
- **SAMtools** (BAM handling in the upstream hybrid assembly feeding the annotated GTF)

## Examples

```
./annotate-my-genomes -a stringtie.gtf -r galGal6.gtf -g galGal6.fa -c gawn_config.sh -t 20 -o output1 && grep 'SSPO' output1/final_annotated.gtf > sspo_locus.gtf
```

## Evaluation signals

- The filtered GTF must contain the expected isoform set (e.g. five coding SSPO isoforms with exon counts 106/106/105/17/25)
- Extracted protein for the longest isoform matches the expected length (5270 aa for SSPO transcript 1)
- blastp of the extracted protein against the reference (AJ866919) with -max_hsps 1 -max_target_seqs 1 yields 98.88% identity
- All retrieved transcripts are classified as protein-coding, not lncRNA, consistent with the FEELnc/TransDecoder classification in the pipeline output
- Every filtered feature's coordinates fall within a single contiguous genomic locus overlapping the target gene region

## Limitations

- Protein products of some assembled SSPO isoforms (e.g. 89-exon and 25-exon variants) remain to be validated experimentally
- Isoform-protein correspondence for smaller isoforms (200–50 kD bands) was inferred but not confirmed
- The approach depends on the organism having an assembled genome and an NCBI GTF annotation; no result is expected for organisms lacking these

## Evidence

- [methods] filter transcript records to the SSPO/SCO-spondin locus using the gene_id field and genomic coordinate overlap: [evidence span withheld — anchor status: not_found]
- [abstract] correctly assembling and annotating all exons from the chicken SCO-spondin gene (containing more than 105 exons): "correctly assembling and annotating all exons from the chicken SCO-spondin gene (containing more than 105 exons)"
- [methods] For each SSPO transcript, parse exon features from the GTF and count exons, confirming the expected isoform set of two 106-exon isoforms plus 105-, 17-, and 25-exon isoforms: [evidence span withheld — anchor status: not_found]
- [readme] All relevant commands to reproduce these analyses are available here: https://github.com/cfarkas/annotate_my_genomes/wiki: "All relevant commands to reproduce these analyses are available here: https://github.com/cfarkas/annotate_my_genomes/wiki"
