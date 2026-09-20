---
name: long-read-read-alignment-to-custom-hla-reference
description: Use when predicting HLA class I/II alleles from raw long-read WGS FASTQ
  data and a SAM-with-MD-tag alignment compatible with HLAminer's stream mode is needed
  as input.
when_to_use_negative:
- Input reads are Illumina-sized short reads — the HPTASR targeted-assembly route
  (reads 100–150bp) is preferred for short-read data
- You need full BAM/SAM alignment files preserved for downstream inspection other
  than HLA prediction — streaming mode explicitly eliminates intermediate alignment
  files on disk
- The reference genome already contains the native chromosome 6 HLA region — reads
  will align there instead of the appended HLA allele references, defeating allele
  discrimination
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: minimap2
  role: Long-read aligner run with '-t 60 -ax map-ont --MD' against the custom HLA
    reference to produce streamable SAM output
  repo: https://github.com/lh3/minimap2
- name: HLAminer
  role: Receives the streamed SAM alignments and performs HPRA HLA-I/-II allele prediction
  repo: https://github.com/bcgsc/HLAminer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:2d87d39c3b1b96651d226b00f58e4fe6bb3886a9f2f9a994ae1bbb671fff9baa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HLAminer
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Long-read read alignment to custom HLA reference

## Summary

Align whole-genome shotgun long reads (ONT PromethION or PacBio) against a GRCh38 reference modified to include HLA class I/II allele sequences, generating MD-tagged SAM output that can be piped directly into HLAminer for HPRA prediction.

## When to use

Use when predicting HLA class I/II alleles from raw long-read WGS FASTQ data and a SAM-with-MD-tag alignment compatible with HLAminer's stream mode is needed as input.

## When NOT to use

- Input reads are Illumina-sized short reads — the HPTASR targeted-assembly route (reads 100–150bp) is preferred for short-read data
- You need full BAM/SAM alignment files preserved for downstream inspection other than HLA prediction — streaming mode explicitly eliminates intermediate alignment files on disk
- The reference genome already contains the native chromosome 6 HLA region — reads will align there instead of the appended HLA allele references, defeating allele discrimination

## Inputs

- Raw WGS long-read FASTQ (e.g. ONT PromethION reads such as ERR2585115 from NA19240, ENA PRJEB26791)
- Custom GRCh38 reference FASTA with HLA-I/II allele sequences and chromosome 6 HLA region removed (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz)
- IMGT/HLA reference sequence database (downloadable via ./database/ scripts) and hla_nom_p.txt P designation file

## Outputs

- MD-tagged SAM alignment records (streamed, not stored on disk)
- HLAminer_HPRA.csv ranked per-gene HLA allele/group prediction table
- HLAminer_HPRA.log listing all predictions

## How to apply

First prepare the reference by appending known HLA-I/II sequences from the IMGT/HLA repository (hla_nom_p.txt P designations) to the chromosome-only GRCh38 genome while excluding the native HLA region on chromosome 6, producing a file like GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz. Then align the nanopore FASTQ reads (e.g. ERR2585115 from ENA accession PRJEB26791) with minimap2 run as '-t 60 -ax map-ont --MD' so that the SAM output carries MD tags describing mismatches, which HLAminer uses to score allelic differences. Rather than writing bulky .sam files to disk, stream the alignment output directly into HLAminer.pl via a Unix pipe using options '-h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream'. The rationale is that MD tags preserve the read-versus-reference sequence detail needed to discriminate HLA alleles at group and 4-digit resolution, and the streaming design works with any aligner producing SAM with MD tags.

## Related tools

- **minimap2** (Long-read aligner run with '-t 60 -ax map-ont --MD' against the custom HLA reference to produce streamable SAM output) — https://github.com/lh3/minimap2
- **HLAminer** (Receives the streamed SAM alignments and performs HPRA HLA-I/-II allele prediction) — https://github.com/bcgsc/HLAminer

## Examples

```
minimap2 -t 60 -ax map-ont --MD GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz ERR2585115.fastq.gz | ../bin/HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream
```

## Evaluation signals

- HLAminer_HPRA.csv exists and parses as a ranked per-gene prediction table keeping the top two highest-scoring predictions per HLA gene
- Predicted allele groups are concordant with clinically confirmed NA19240 types (e.g. A*30/A*68, B*35/B*57, C*04/C*18)
- The SAM stream contains MD tags — verify the aligner was invoked with '--MD' (minimap2 does not emit MD by default in all modes)
- Predictions remain robust at relatively low (~10x) sequencing coverage even with older-chemistry nanopore data
- The custom reference includes appended IMGT/HLA allele sequences while excluding the native chromosome 6 HLA region

## Limitations

- The protocol was demonstrated on older-chemistry (2018) ONT PromethION WGS data; higher error rates degrade but do not eliminate prediction accuracy
- Larger k values for TASR-type assembly increase specificity at the possible expense of sensitivity, so parameter tuning trades off specificity and sensitivity
- blast 2.2.22 may be faster than blast+ (2.2.28) while producing accurate results, so BLAST version choice affects runtime

## Evidence

- [abstract] a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer: "a streamlined protocol for predicting HLA signatures from whole-genome shotgun (WGS) long-read sequencing data by directly streaming sequence alignments into HLAminer"
- [abstract] works with any read aligner compatible with the SAM file format: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [methods] GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz: "GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz"
- [methods] Known HLA class I/class II reference sequences available from the IMGT/HLA public repository are read by TASR: "Known HLA class I/class II reference sequences available from the IMGT/HLA public repository are read by TASR using default options (Warren and Holt 2011)"
- [readme] Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer.: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
