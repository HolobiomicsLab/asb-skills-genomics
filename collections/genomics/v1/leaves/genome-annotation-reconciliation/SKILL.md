---
name: genome-annotation-reconciliation
description: Use when you have a StringTie (or similar) merged assembly GTF from hybrid
  RNA-seq data and an assembled reference genome with an NCBI annotation in GTF format,
  and need to reconcile gene models, classify coding vs lncRNA, and flag novel genes
  absent from the reference.
when_to_use_negative:
- The organism lacks an assembled genome or an NCBI-available annotation in GTF format
  — the pipeline requires both
- Input is raw reads rather than an assembled transcriptome GTF; assembly (e.g., StringTie
  with -p 1 -j 2 -c 2 -v -a 4) must happen first
- The goal is de novo gene prediction from genome alone (use BRAKER2/3 or MAKER),
  not reconciliation of an existing transcriptome assembly against a reference annotation
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: annotate_my_genomes
  role: Core Nextflow/Anaconda/Docker pipeline performing coding/noncoding annotation
    and GTF reconciliation
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: GAWN
  role: Annotates all possible proteins from assembled transcripts using Blastx against
    Swissprot
  repo: https://github.com/enormandeau/gawn
- name: FEELnc
  role: Trained with known coding RNAs (NM_ prefix transcripts) to classify long non-coding
    RNAs
- name: TransDecoder
  role: Predicts coding regions and deduced proteins in transcripts left unclassified
    by coding/lncRNA assignment
  repo: https://github.com/TransDecoder/TransDecoder
- name: genome-download
  role: Nextflow utility that fetches the reference genome fasta and GTF annotations
    (e.g., galGal6)
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: AGAT
  role: Toolkit that must be installed for GTF processing in the pipeline
  repo: https://github.com/NBISweden/AGAT
- name: ncbi-blast+
  role: Blastx/Blastp homology searches (Swissprot, UniProt Gallus gallus taxid 9031)
    for protein annotation and novel-gene homology assignments
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:62e227d5a1cc9dac993c69bf6adcf13c3e7835233fc6f2e68b96b0261b95bf18
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
  - genome-download
  - ncbi-blast+
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# genome-annotation-reconciliation

## Summary

Reconcile a genome-guided StringTie transcriptome assembly against an existing GTF reference annotation (e.g., chicken galGal6 NCBI RefSeq) using the annotate_my_genosomes pipeline, producing a final_annotated.gtf with reconciled gene identities plus coding/lncRNA and novel-gene classification.

## When to use

Use when you have a StringTie (or similar) merged assembly GTF from hybrid RNA-seq data and an assembled reference genome with an NCBI annotation in GTF format, and need to reconcile gene models, classify coding vs lncRNA, and flag novel genes absent from the reference.

## When NOT to use

- The organism lacks an assembled genome or an NCBI-available annotation in GTF format — the pipeline requires both
- Input is raw reads rather than an assembled transcriptome GTF; assembly (e.g., StringTie with -p 1 -j 2 -c 2 -v -a 4) must happen first
- The goal is de novo gene prediction from genome alone (use BRAKER2/3 or MAKER), not reconciliation of an existing transcriptome assembly against a reference annotation

## Inputs

- Merged StringTie assembly GTF from hybrid (PacBio + Illumina) RNA-seq alignments
- Reference genome FASTA (e.g., galGal6 UCSC non-masked fasta, GCA_000002315.5)
- Reference coding gene GTF (galGal6)
- NCBI RefSeq GTF for the reference genome
- gawn_config.sh configuration (NCPUS=10 default)
- Swissprot database for Blastx protein annotation

## Outputs

- final_annotated.gtf with reconciled gene_id identities and novel-gene/lncRNA classification in the second field
- Reconciled gene and transcript counts (e.g., 19690 genes, 61940 transcripts)
- Novel candidate gene set absent from the reference annotation (e.g., 4292 genes)
- FEELnc-based lncRNA classification (e.g., 27% divergent lncRNAs, Supplementary Table 1)

## How to apply

Download the reference genome fasta and coding/NCBI RefSeq GTFs with the genome-download utility (e.g., galGal6, GCA_000002315.5), then run annotate-my-genomes.nf (Nextflow, 22.04_environment.yml conda env, or Docker/Anaconda) passing the merged StringTie assembly GTF (-a), the reference coding GTF (-r), the genome fasta (-g), and gawn_config.sh, with 20 threads (-t) and NCPUS=10 for blast processing. The pipeline annotates proteins via GAWN/Blastx against Swissprot (e.g., -evalue 1e-5 -qcov_hsp_perc 10), trains the FEELnc classifier with known coding RNAs (NM_ prefix transcripts) to classify lncRNAs, and predicts coding regions in remaining unclassified transcripts with TransDecoder. Inspect final_annotated.gtf: the gene_id field carries reconciled gene identities and the second field carries novel gene and lncRNA classification. Count reconciled genes/transcripts and compare against reported values (19690 reconciled genes, 4292 novel candidates not on GRCg6a, 61940 reconciled transcripts) to validate.

## Related tools

- **annotate_my_genomes** (Core Nextflow/Anaconda/Docker pipeline performing coding/noncoding annotation and GTF reconciliation) — https://github.com/cfarkas/annotate_my_genomes
- **GAWN** (Annotates all possible proteins from assembled transcripts using Blastx against Swissprot) — https://github.com/enormandeau/gawn
- **FEELnc** (Trained with known coding RNAs (NM_ prefix transcripts) to classify long non-coding RNAs)
- **TransDecoder** (Predicts coding regions and deduced proteins in transcripts left unclassified by coding/lncRNA assignment) — https://github.com/TransDecoder/TransDecoder
- **genome-download** (Nextflow utility that fetches the reference genome fasta and GTF annotations (e.g., galGal6)) — https://github.com/cfarkas/annotate_my_genomes
- **AGAT** (Toolkit that must be installed for GTF processing in the pipeline) — https://github.com/NBISweden/AGAT
- **ncbi-blast+** (Blastx/Blastp homology searches (Swissprot, UniProt Gallus gallus taxid 9031) for protein annotation and novel-gene homology assignments)

## Examples

```
nextflow run annotate-my-genomes.nf -profile docker --assembly merged_stringtie.gtf --reference galGal6_coding.gtf --genome galGal6.fa -w work -with-report --threads 20
```

## Evaluation signals

- final_annotated.gtf exists and parses as a valid GTF
- Gene and transcript counts in final_annotated.gtf match reported values (19690 reconciled genes, 4292 novel candidates, 61940 transcripts)
- Novel candidate genes show no overlap with the reference (GRCg6a) annotation but have homology support (e.g., Blastx/blastp hits against Swissprot or UniProt Gallus gallus proteome, taxid 9031)
- FEELnc lncRNA classification proportions match reported values (e.g., 27% divergent lncRNAs, Supplementary Table 1)
- Multi-exon genes such as chicken SCO-spondin (SSPO, >105 exons) are fully assembled and annotated across all exons

## Limitations

- Pipeline applicability is restricted to organisms with an assembled genome and an NCBI GTF annotation; no result is reported for organisms lacking these
- Protein products of some assembled isoforms (e.g., 89-exon and 25-exon SSPO isoforms) remain to be validated
- Precision/recall and F1-scores against reference are deferred to Supplementary Table 2 and depend on gffcompare benchmarking against simulated reads (IsoSeqSim)

## Evidence

- [abstract] distinguishes between coding and long non-coding RNAs by integration of several bioinformatic approaches, including gene reconciliation with previous annotations in GTF format: "distinguishes between coding and long non-coding RNAs by integration of several bioinformatic approaches, including gene reconciliation with previous annotations in GTF format"
- [discussion] We input assembled transcripts in GTF format to the annotate_my_genomes pipeline, obtaining coding/noncoding annotations and reconciled GTF file: [evidence span withheld — anchor status: not_found]
- [methods] git clone https://github.com/cfarkas/annotate_my_genomes.git: "git clone https://github.com/cfarkas/annotate_my_genomes.git"
- [discussion] setting the following parameters: -evalue 1e-5 -qcov_hsp_perc 10: "setting the following parameters: -evalue 1e-5 -qcov_hsp_perc 10 [61]"
- [discussion] This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format: "This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format"
