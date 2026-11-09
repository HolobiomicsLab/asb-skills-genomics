---
name: mhc-reference-genome-preparation
description: Reference for preparing a noChr6 GRCh38 reference augmented with HLA-I/II
  genomic alleles for long-read (PacBio HiFi, ONT) WGS alignment prior to HLAminer
  HPRA prediction.
when_to_use_negative:
- Input is Illumina-sized short reads, where the HPTASR targeted-assembly route (TASR
  k-mer recruitment, de novo assembly, reciprocal BLASTN) is preferred over read alignment
  to this reference
- The reference must retain a full, unmasked chr6 HLA locus, which would collapse
  allelic HLA reads onto the single reference haplotype instead of resolving alleles
- The goal is transcriptome-only HLA typing (e.g., MCF-7 PacBio RNA-seq demo) rather
  than WGS long-read genomic alignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: HLAminer
  role: Consumes streamed SAM alignments against the HLA-augmented reference to score
    and report HLA-I/II allele predictions
  repo: https://github.com/bcgsc/HLAminer
- name: minimap2
  role: Aligns long reads (map-hifi preset, --MD) to the noChr6 + HLA-I/II reference
    and streams SAM output into HLAminer
  repo: https://github.com/lh3/minimap2
- name: TASR
  role: Reads known IMGT/HLA class I/II reference sequences to build k-mer hash structures
    used in the HPTASR alternative and reference database preparation
  repo: https://github.com/warrenlr/TASR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:26ac02ede4199a35d050d15506d69d7d4d0f3baf12406a9cb7323dae66b7a4f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - TASR
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# mhc-reference-genome-preparation

## Summary

Prepares a GRCh38-based (GCA_000001405.15) reference FASTA in which chr6 is excluded (or HLA-masked) and replaced with HLA class I/II genomic allele sequences from IMGT/HLA, so that long-read alignments stream directly into HLAminer for HLA-I/II prediction.

## When to use

Reference for preparing a noChr6 GRCh38 reference augmented with HLA-I/II genomic alleles for long-read (PacBio HiFi, ONT) WGS alignment prior to HLAminer HPRA prediction.

## When NOT to use

- Input is Illumina-sized short reads, where the HPTASR targeted-assembly route (TASR k-mer recruitment, de novo assembly, reciprocal BLASTN) is preferred over read alignment to this reference
- The reference must retain a full, unmasked chr6 HLA locus, which would collapse allelic HLA reads onto the single reference haplotype instead of resolving alleles
- The goal is transcriptome-only HLA typing (e.g., MCF-7 PacBio RNA-seq demo) rather than WGS long-read genomic alignment

## Inputs

- GRCh38 genomic FASTA (GCA_000001405.15), chromosome-only, with chr6 excluded or HLA-masked
- IMGT/HLA class I and II genomic allele sequences (HLA-I_II_GEN.fasta)
- HLA P-designation file hla_nom_p.txt from hla.alleles.org
- Concatenated whole-genome shotgun long-read FASTQ (e.g., 39 NA24385 PacBio CCS/HiFi subread files)

## Outputs

- HLA-augmented, noChr6 reference FASTA: GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz
- Indexed reference and HLA database files for minimap2/HLAminer HPRA streaming

## How to apply

Build the reference by taking the GRCh38 genomic sequence (GCA_000001405.15) with all chromosomes except (or with masked) chr6, and concatenating the HLA-I and HLA-II genomic allele sequences so alignments map reads to allelic references instead of collapsing onto the chr6 HLA locus. Source HLA class I/II sequences from the IMGT/HLA public repository (download and index scripts exist in the HLAminer ./database/ directory); the resulting file is named GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz. Also obtain the P-designation file hla_nom_p.txt (http://hla.alleles.org/wmda/hla_nom_p.txt) so that alleles sharing identical antigen-binding domains collapse to the P-designated allele in the summary. Use this reference with minimap2 -ax map-hifi --MD for PacBio HiFi reads, piping SAM output straight into HLAminer.pl HPRA mode (-a stream, -h ../database/HLA-I_II_GEN.fasta, -s 500, -q 1, -i 1, -p ../database/hla_nom_p.txt) to avoid writing bulky .sam files to disk.

## Related tools

- **HLAminer** (Consumes streamed SAM alignments against the HLA-augmented reference to score and report HLA-I/II allele predictions) — https://github.com/bcgsc/HLAminer
- **minimap2** (Aligns long reads (map-hifi preset, --MD) to the noChr6 + HLA-I/II reference and streams SAM output into HLAminer) — https://github.com/lh3/minimap2
- **TASR** (Reads known IMGT/HLA class I/II reference sequences to build k-mer hash structures used in the HPTASR alternative and reference database preparation) — https://github.com/warrenlr/TASR

## Examples

```
minimap2 -ax map-hifi --MD GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz NA24385_HiFi.fq | ../bin/HLAminer.pl -a stream -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt
```

## Evaluation signals

- The assembled reference FASTA contains all GRCh38 chromosomes except chr6 plus HLA-I/II genomic alleles (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz)
- The HLA database is built from the current IMGT/HLA repository download, indexed via the scripts in HLAminer's ./database/ directory
- Downstream HPRA run on 30x NA24385 PacBio HiFi reads completes in roughly 1 hr 14 min, consistent with the reported benchmark
- HLAminer_HPRA.csv reports top-two predictions per gene with P-designated alleles (e.g., B*35:01, DRB1*04:03P) matching clinically confirmed types for NA24385
- No bulky intermediate .sam alignment files are written to disk because SAM is streamed from minimap2 into HLAminer.pl

## Limitations

- Predictions are allele-approximate: NA24385 predictions such as B*35:01 and DRB1*04:03P were close but not identical to the clinically typed B*35:08 and DRB1*04:02P
- Robustness was demonstrated on relatively low (10x) and older-chemistry nanopore data; extremely low coverage or new HLA alleles absent from IMGT/HLA may be missed
- Larger TASR k values increase specificity at the possible expense of sensitivity, so database preparation choices trade off prediction sensitivity

## Evidence

- [methods] Known HLA class I/class II reference sequences available from the IMGT/HLA public repository are read by TASR using default options (Warren and Holt 2011): "Known HLA class I/class II reference sequences available from the IMGT/HLA public repository are read by TASR using default options"
- [methods] GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz is the prepared reference for long-read alignments: "GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz"
- [methods] Follow these instructions to download updated HLA sequences from ebi/imgt (shell scripts to automatically download and format the databases exist in ./database/): "Follow these instructions to download updated HLA sequences from ebi/imgt (shell scripts to automatically download and format the databases exist in ./database/)"
- [abstract] The streaming design eliminates bulky SAM files on disk: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [readme] HLAminer v1.4 supports streaming aligner SAM output directly into the tool: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
