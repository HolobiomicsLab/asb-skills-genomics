---
name: protein-sequence-homology-blast-verification
description: Use when validating that deduced proteins from an assembled transcriptome
  (e.g. GAWN/TransDecoder prot.fa outputs) match known homologs, or when confirming
  identity of a reconstructed protein like chicken SSPO against AJ866919.
when_to_use_negative:
- When the organism has no assembled genome or NCBI GTF annotation available — the
  upstream pipeline this verification depends on is explicitly restricted to such
  cases
- When transcripts are predicted as long non-coding RNAs by FEELnc and have no deduced
  protein product to blast
- When the goal is comprehensive homolog discovery rather than single-best-hit verification
  — forcing -max_hsps 1 -max_target_seqs 1 discards secondary hits
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0091
tools:
- name: ncbi-blast+
  role: Runs blastx (Swissprot functional assignment) and blastp (protein-to-reference
    identity verification) with the article's constrained parameters
- name: annotate_my_genomes
  role: Upstream Nextflow pipeline producing cds.fa and prot.fa inputs for the BLAST
    verification step
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: GAWN
  role: Genome annotation step that deduces transcript proteins later blasted against
    Swissprot and the UniProt Gallus gallus proteome
  repo: https://github.com/enormandeau/gawn
- name: TransDecoder
  role: Predicts coding regions and deduced proteins from unclassified transcripts
    before homology assignment
  repo: https://github.com/TransDecoder/TransDecoder
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:a21b4e4b08b345cff63ff1c2c42fac1e665f4885c91affe4cf33eed3ffc4b8c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - AGAT
  - Anaconda
  - GAWN
  - SAMtools
  - TransDecoder
  - annotate_my_genomes
  - ncbi-blast+
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# protein-sequence-homology-blast-verification

## Summary

Verifies assembled transcript-derived proteins by BLAST homology against reference databases, using constrained blastx/blastp settings (e.g. -max_hsps 1 -max_target_seqs 1) to recover a single top hit and quantify identity to a known protein such as the chicken SSPO reference AJ866919.

## When to use

Use when validating that deduced proteins from an assembled transcriptome (e.g. GAWN/TransDecoder prot.fa outputs) match known homologs, or when confirming identity of a reconstructed protein like chicken SSPO against AJ866919.

## When NOT to use

- When the organism has no assembled genome or NCBI GTF annotation available — the upstream pipeline this verification depends on is explicitly restricted to such cases
- When transcripts are predicted as long non-coding RNAs by FEELnc and have no deduced protein product to blast
- When the goal is comprehensive homolog discovery rather than single-best-hit verification — forcing -max_hsps 1 -max_target_seqs 1 discards secondary hits

## Inputs

- prot.fa — deduced protein sequences from the annotate_my_genomes/GAWN outputs
- cds.fa — predicted CDS sequences for blastx functional assignment
- Reference protein or proteome FASTA (e.g. AJ866919-deduced SSPO protein; UniProt Gallus gallus proteome taxid 9031; Swissprot database)

## Outputs

- BLAST hit table with percent identity and single top target per query
- Verified protein-to-reference identity value (e.g. 98.88% for the 5270-aa SSPO protein vs AJ866919)
- Compiled validation table (e.g. sspo_isoform_validation.csv) of protein lengths and BLAST identities per isoform

## How to apply

After the annotate_my_genomes pipeline yields cds.fa and prot.fa, run blastx of predicted CDSs against Swissprot with the article's filter settings (-evalue 1e-5 -qcov_hsp_perc 10) to assign functional homology, and blastp of deduced proteins against a reference proteome (e.g. UniProt Gallus gallus, taxid 9031) using -max_hsps 1 -max_target_seqs 1 to force a single best target and a single HSP. For isoform-level verification, extract the protein of the longest transcript (e.g. the 5270-aa SSPO isoform from transcript 1), align it to the AJ866919-deduced reference protein, and read off percent identity — the article expects 98.88% for a correctly assembled >105-exon SSPO. Compile per-isoform protein lengths and BLAST identities (e.g. into sspo_isoform_validation.csv) as the correctness record.

## Related tools

- **ncbi-blast+** (Runs blastx (Swissprot functional assignment) and blastp (protein-to-reference identity verification) with the article's constrained parameters)
- **annotate_my_genomes** (Upstream Nextflow pipeline producing cds.fa and prot.fa inputs for the BLAST verification step) — https://github.com/cfarkas/annotate_my_genomes
- **GAWN** (Genome annotation step that deduces transcript proteins later blasted against Swissprot and the UniProt Gallus gallus proteome) — https://github.com/enormandeau/gawn
- **TransDecoder** (Predicts coding regions and deduced proteins from unclassified transcripts before homology assignment) — https://github.com/TransDecoder/TransDecoder

## Examples

```
blastp -query sspo_transcript1_prot.fa -subject AJ866919_sspo_prot.fa -max_hsps 1 -max_target_seqs 1
```

## Evaluation signals

- blastp of the 5270-aa SSPO transcript-1 protein against AJ866919 returns 98.88% identity with -max_hsps 1 -max_target_seqs 1
- Exactly one HSP and one target per query is reported, consistent with the constrained blastp settings
- blastx against Swissprot respects the -evalue 1e-5 -qcov_hsp_perc 10 thresholds for retained hits
- Validation table (sspo_isoform_validation.csv) reports all five coding SSPO isoforms (106/106/105/17/25 exons) with non-empty protein lengths and identities
- Homologous SSPO transcripts classified as protein-coding, not lncRNA, by the combined FEELnc/TransDecoder/BLAST assignment

## Limitations

- Pipeline applicability is restricted to organisms with an assembled genome and an NCBI GTF annotation — no result is reported for organisms lacking these
- Identity to a reference does not by itself validate protein products in vivo: protein products of smaller SSPO isoforms (e.g. 25-exon) were inferred from lower molecular weight bands but remain to be validated
- Isoform-to-protein correspondence for smaller isoforms is unconfirmed, so BLAST identity of the longest protein does not extend to all isoforms

## Evidence

- [discussion] with the setting -max_hsps 1 -max_target_seqs 1 in blastp command: "with the setting -max_hsps 1 -max_target_seqs 1 in blastp command [72]"
- [discussion] setting the following parameters: -evalue 1e-5 -qcov_hsp_perc 10: "setting the following parameters: -evalue 1e-5 -qcov_hsp_perc 10 [61]"
- [discussion] we blasted the novel predicted proteins from these cds against the UniProt Gallus gallus proteome (taxid 9031): "we blasted the novel predicted proteins from these cds against the UniProt Gallus gallus proteome (taxid 9031)"
- [intro] correctly assembling all exons from the chicken SSPO gene (containing more than 105 exons): "correctly assembling all exons from the chicken SSPO gene (containing more than 105 exons)"
