---
name: hla-allele-prediction-from-long-reads
description: Use when WGS long-read FASTQ data (Oxford Nanopore or PacBio) is available
  and clinically confirmed HLA-I/-II allele/group predictions are needed, especially
  at low (~10x) coverage or with older nanopore chemistry.
when_to_use_negative:
- Input reads are Illumina-sized short reads (100–150 bp) — the README indicates best
  short-read results come from the HPTASR targeted-assembly mode, not the streamed
  long-read HPRA workflow
- The aligner cannot emit SAM output with MD tags, or a strictly allele-level (not
  group-level) clinical call is required — HLAminer's streamed mode yields fairly
  accurate group/allele predictions with the noted ambiguities
- Reads are targeted amplicons or RNA-seq rather than whole-genome shotgun long reads
  where a chr6-stripped GRCh38 reference alignment is the intended input
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3577
tools:
- name: HLAminer
  role: Pipeline that scores streamed SAM alignments to derive HLA class I and II
    predictions at group and allele resolution (HPRA mode)
  repo: https://github.com/bcgsc/HLAminer
- name: minimap2
  role: Long-read aligner (-ax map-ont --MD) producing the SAM stream piped into HLAminer;
    dominates pipeline runtime and RAM
- name: HLAminer.pl
  role: Perl driver invoked in stream mode (-a stream) with HLA-I_II_GEN.fasta, minimum
    score -s 500, phred-like E-value -q 1, identity -i 1, and hla_nom_p.txt to collapse
    P-designated alleles
- name: TASR
  role: Underlying component that builds the 15-nt k-mer hash table from IMGT/HLA
    reference sequences used in HPTASR-mode prediction
  repo: https://github.com/warrenlr/TASR
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:99e9335ee8aa7174c1cb14603a6157d9c8307fa5066956b8aeb02a48d40f84a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/HLAminer
  tools:
  - HLAminer
  - HLAminer.pl
  - TASR
  - minimap2
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hla-allele-prediction-from-long-reads

## Summary

Predict HLA class I and class II allele groups from whole-genome shotgun long-read (ONT/PacBio) data by streaming SAM alignments (with MD tags) directly from minimap2 into HLAminer v1.4's HPRA mode, avoiding intermediate SAM files on disk.

## When to use

Use when WGS long-read FASTQ data (Oxford Nanopore or PacBio) is available and clinically confirmed HLA-I/-II allele/group predictions are needed, especially at low (~10x) coverage or with older nanopore chemistry.

## When NOT to use

- Input reads are Illumina-sized short reads (100–150 bp) — the README indicates best short-read results come from the HPTASR targeted-assembly mode, not the streamed long-read HPRA workflow
- The aligner cannot emit SAM output with MD tags, or a strictly allele-level (not group-level) clinical call is required — HLAminer's streamed mode yields fairly accurate group/allele predictions with the noted ambiguities
- Reads are targeted amplicons or RNA-seq rather than whole-genome shotgun long reads where a chr6-stripped GRCh38 reference alignment is the intended input

## Inputs

- Whole-genome shotgun long-read FASTQ file (e.g. ONT PromethION ERR2585115.fastq.gz, ~10x coverage NA19240)
- Reference genome FASTA: GRCh38 (GCA_000001405.15) chromosome-only with chr6 removed and HLA-I/II GEN sequences appended
- IMGT/HLA reference sequence database (HLA-I_II_GEN.fasta) built via the ./database/ download scripts
- IMGT P designation file hla_nom_p.txt (http://hla.alleles.org/wmda/hla_nom_p.txt)

## Outputs

- HLAminer_HPRA.csv — concise ranked per-gene HLA prediction summary (top two highest-scoring predictions per gene, P-designated alleles)
- HLAminer_HPRA.log — full log listing all scored predictions
- Optional GNU time profile (.time file) with 'Elapsed (wall clock) time' and 'Maximum resident set size'

## How to apply

Download a human WGS long-read FASTQ (e.g. NA19240 PromethION ERR2585115 from ENA PRJEB26791) and build a GRCh38 reference with chromosome 6 HLA region removed but HLA-I/II GEN sequences appended (GCA_000001405.15_GRCh38_genomic.chr-only-noChr6-HLA-I_II_GEN.fa.gz). Align with minimap2 using '-t 60 -ax map-ont --MD' and pipe the SAM stream directly into './HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream' so no bulky SAM file is written to disk; the HLAminer.pl HPRA scorer emits ranked per-gene predictions. Accept predictions keeping the top two highest-scoring alleles per gene, collapsing to P-designated alleles sharing the same antigen binding domain. Validate by checking that predicted allele groups are concordant with clinically confirmed types (e.g. NA19240: A*30/A*68, B*35/B*57, C*04/C*18) and that runtime scales with read count while minimap2 dominates RAM (~35 GB peak, ~40 min for the 10x ONT dataset).

## Related tools

- **HLAminer** (Pipeline that scores streamed SAM alignments to derive HLA class I and II predictions at group and allele resolution (HPRA mode)) — https://github.com/bcgsc/HLAminer
- **minimap2** (Long-read aligner (-ax map-ont --MD) producing the SAM stream piped into HLAminer; dominates pipeline runtime and RAM)
- **HLAminer.pl** (Perl driver invoked in stream mode (-a stream) with HLA-I_II_GEN.fasta, minimum score -s 500, phred-like E-value -q 1, identity -i 1, and hla_nom_p.txt to collapse P-designated alleles)
- **TASR** (Underlying component that builds the 15-nt k-mer hash table from IMGT/HLA reference sequences used in HPTASR-mode prediction) — https://github.com/warrenlr/TASR

## Examples

```
/usr/bin/time -v -o run.time minimap2 -t 60 -ax map-ont --MD GRCh38-noChr6-HLA-I_II_GEN.fa ERR2585115.fastq.gz | ./HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream
```

## Evaluation signals

- HLAminer_HPRA.csv exists and parses as a ranked per-gene prediction table with exactly the top two highest-scoring alleles per gene
- Predicted allele groups are concordant with clinically confirmed HLA types for the sample (e.g. NA19240: A*30/A*68, B*35/B*57, C*04/C*18); a known minimap2-baseline run on dataset A yields groups A26 and A33 (A*26:33, A*33:24)
- GNU time profile shows wall-clock runtime ~40 minutes and peak RSS within the reported up-to-35-GB envelope for the ~10x ONT dataset A
- No bulky intermediate .sam file is written to disk — alignments stream through the Unix pipe directly into HLAminer.pl
- Database scripts successfully downloaded and formatted current IMGT/HLA sequences plus hla_nom_p.txt before the run

## Limitations

- Predictions were demonstrated robust only down to relatively low (~10x) coverage and with older, less accurate 2018-chemistry PromethION WGS data; lower quality or depth may degrade allele-level resolution
- Blast 2.2.22 may be faster than blast+ 2.2.28; larger TASR k values increase specificity at the possible expense of sensitivity
- HPTASR with short reads (50 bp) can produce ambiguous 4-digit allele predictions; streamed HPRA allele calls may be ambiguous at 4-digit resolution
- Minimap2 dominates the pipeline's runtime and RAM (~35 GB), so memory-constrained hosts may struggle with large WGS inputs

## Evidence

- [methods] Streamed workflow benchmarks ~40 min for 10x ONT WGS: [evidence span withheld — anchor status: not_found]
- [abstract] Streamed aligner requirements: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [abstract] Robustness at low coverage and old chemistry: "demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage"
- [readme] Streaming capability added in v1.4: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
- [readme] Output summary keeps top two per gene: "Keeps top two [highest-scoring by HLA group] predictions per gene and only the 'P' designated allele"
