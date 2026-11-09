---
name: lncrna-coding-potential-classification
description: Use when a genome-guided transcriptome assembly (StringTie GTF from hybrid
  PacBio/Illumina RNA-seq) must be split into coding and lncRNA classes with a reconciled,
  annotated GTF output.
when_to_use_negative:
- The organism lacks an assembled genome and an NCBI-available annotation in GTF format
  — the article explicitly restricts the pipeline to such organisms
- Input is raw sequencing reads rather than a genome-guided assembled GTF — reads
  must first be aligned (minimap2/HISAT2) and assembled with StringTie
- De novo transcriptome assembly without a reference genome — the pipeline is genome-guided
  and requires a genome FASTA as a guide
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0204
tools:
- name: annotate_my_genomes
  role: Host pipeline that integrates reconciliation, FEELnc lncRNA classification,
    and coding/noncoding annotation into final_annotated.gtf
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: FEELnc
  role: lncRNA classifier trained with known coding RNAs ('NM_' prefix chicken transcripts)
    to label transcripts coding vs long non-coding, including divergent classes
- name: GAWN
  role: Genome annotation workflow that runs Blastx against Swissprot to identify
    protein-coding transcripts by homology
  repo: https://github.com/enormandeau/gawn
- name: TransDecoder
  role: Predicts coding regions and deduced proteins in transcripts left unclassified
    after reconciliation, Blastx, and FEELnc
  repo: https://github.com/TransDecoder/TransDecoder
- name: ncbi-blast+
  role: Blastx (-evalue 1e-5 -qcov_hsp_perc 10) and blastp (-max_hsps 1 -max_target_seqs
    1) homology searches underpinning coding assignment
- name: AGAT
  role: GTF toolkit that must be installed as a dependency for the pipeline
  repo: https://github.com/NBISweden/AGAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:7398df587ff27d37280f0ae55b93da3a055f8a38e3bc2af2b987466c4bff2b99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - AGAT
  - Anaconda
  - Docker
  - FEELnc
  - GAWN
  - SAMtools
  - TransDecoder
  - annotate_my_genomes
  - ncbi-blast+
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# lncRNA-coding-potential-classification

## Summary

Classifies assembled transcripts as coding versus long non-coding RNAs by reconciling the StringTie GTF against existing annotations, running GAWN/Blastx against Swissprot, training the FEELnc lncRNA classifier with known coding RNAs, and using TransDecoder to recover coding regions in remaining unclassified transcripts.

## When to use

Use when a genome-guided transcriptome assembly (StringTie GTF from hybrid PacBio/Illumina RNA-seq) must be split into coding and lncRNA classes with a reconciled, annotated GTF output.

## When NOT to use

- The organism lacks an assembled genome and an NCBI-available annotation in GTF format — the article explicitly restricts the pipeline to such organisms
- Input is raw sequencing reads rather than a genome-guided assembled GTF — reads must first be aligned (minimap2/HISAT2) and assembled with StringTie
- De novo transcriptome assembly without a reference genome — the pipeline is genome-guided and requires a genome FASTA as a guide

## Inputs

- Genome-guided StringTie transcriptome assembly in GTF format (merged PacBio/Illumina alignments)
- Reference genome FASTA (e.g. galGal6, non-masked)
- Reference coding gene annotation GTF (e.g. galGal6 coding GTF)
- NCBI RefSeq GTF annotation for reconciliation
- Swissprot protein database for Blastx

## Outputs

- final_annotated.gtf with reconciled gene_id identities and coding/novel/lncRNA classification per transcript
- FEELnc lncRNA classification table (coding/putative-noncoding, divergent/overlapping classifications)
- TransDecoder-predicted coding sequences and deduced proteins for unclassified transcripts
- Counts of reconciled genes, novel candidate genes, and lncRNA classes (e.g. 27% divergent lncRNA)

## How to apply

First reconcile the input StringTie assembly GTF against the reference coding GTF (e.g. galGal6) and genome FASTA to mark transcripts already present in the annotation. For remaining transcripts, annotate proteins with GAWN using Blastx against Swissprot with -evalue 1e-5 -qcov_hsp_perc 10 to capture known coding transcripts by homology. Then train the FEELnc classifier with known coding RNAs from the species ('NM_' prefix RefSeq transcripts in chicken) and classify the remaining transcripts as lncRNAs (including divergent lncRNAs relative to nearby coding genes). Finally, run TransDecoder on the unclassified transcripts to predict coding regions and deduced proteins, confirming any coding potential FEELnc missed. Results are consolidated in final_annotated.gtf, where the gene_id field carries reconciled gene identities and the second field carries novel gene and lncRNA classification.

## Related tools

- **annotate_my_genomes** (Host pipeline that integrates reconciliation, FEELnc lncRNA classification, and coding/noncoding annotation into final_annotated.gtf) — https://github.com/cfarkas/annotate_my_genomes
- **FEELnc** (lncRNA classifier trained with known coding RNAs ('NM_' prefix chicken transcripts) to label transcripts coding vs long non-coding, including divergent classes)
- **GAWN** (Genome annotation workflow that runs Blastx against Swissprot to identify protein-coding transcripts by homology) — https://github.com/enormandeau/gawn
- **TransDecoder** (Predicts coding regions and deduced proteins in transcripts left unclassified after reconciliation, Blastx, and FEELnc) — https://github.com/TransDecoder/TransDecoder
- **ncbi-blast+** (Blastx (-evalue 1e-5 -qcov_hsp_perc 10) and blastp (-max_hsps 1 -max_target_seqs 1) homology searches underpinning coding assignment)
- **AGAT** (GTF toolkit that must be installed as a dependency for the pipeline) — https://github.com/NBISweden/AGAT

## Examples

```
annotate-my-genomes.nf -a merged_stringtie.gtf -r galGal6_coding.gtf -g galGal6.fna -t 20
```

## Evaluation signals

- final_annotated.gtf exists, parses as a valid GTF, and its gene/transcript counts match the reported 19690 reconciled genes (4292 novel) and 61940 transcripts for the chicken SCO merged assembly
- FEELnc classification summary reproduces the reported 27% divergent lncRNA proportion (Figure 1D–1G, Supplementary Table 1)
- Transcripts matching known coding annotations (NM_ RefSeq or Swissprot hits at evalue 1e-5, qcov ≥10%) are labeled coding, not lncRNA
- Novel predicted proteins blast back against the UniProt Gallus gallus proteome (taxid 9031) with genomic positions consistent with the assembly
- Every transcript in the input assembly receives exactly one class (coding, lncRNA subclass, or TransDecoder-coded) in the final GTF

## Limitations

- Protein products of predicted non-canonical isoforms (e.g. SSPO isoforms) remain unvalidated by the pipeline
- Applicability restricted to organisms with an assembled genome and an NCBI GTF annotation; no results reported for organisms lacking these
- StringTie GTF assembly input is required for the reported performance gains over BRAKER1/2 and TSEBRA; other inputs were not claimed to match this

## Evidence

- [abstract] distinguishes between coding and long non-coding RNAs by integration of several bioinformatic approaches, including gene reconciliation with previous annotations in GTF format: "distinguishes between coding and long non-coding RNAs by integration of several bioinformatic approaches, including gene reconciliation with previous annotations in GTF format"
- [discussion] we assessed long noncoding RNAs training FEELnc classifier [34] with known coding RNAs from chicken (“NM_” prefix transcripts): "we assessed long noncoding RNAs training FEELnc classifier [34] with known coding RNAs from chicken (“NM_” prefix transcripts)"
- [discussion] setting the following parameters: -evalue 1e-5 -qcov_hsp_perc 10 [61]: "setting the following parameters: -evalue 1e-5 -qcov_hsp_perc 10 [61]"
- [discussion] the remaining unclassified transcripts were assessed to predicted coding regions and deduced proteins using TransDecoder gene prediction program: "the remaining unclassified transcripts were assessed to predicted coding regions and deduced proteins using TransDecoder gene prediction program"
- [readme] aims to annotate genome-guided transcriptome assemblies from StringTie, coming from long read RNA-Seq alignments in vertebrate genomes: "aims to annotate genome-guided transcriptome assemblies from StringTie, coming from long read RNA-Seq alignments in vertebrate genomes"
