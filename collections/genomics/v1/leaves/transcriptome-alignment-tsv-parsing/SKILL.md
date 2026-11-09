---
name: transcriptome-alignment-tsv-parsing
description: Use when raw Nanopore DRS reads must be aligned to a reference transcriptome
  (not a genome) and converted into a per-base alignment TSV that RedNano's extract_features.py
  can consume as basecalling-error input.
when_to_use_negative:
- Reads should be aligned to a whole genome rather than a reference transcriptome
  — tombo resquiggle and this workflow expect the transcript/gene FASTA
- The feature table (test.features.sort.tsv or aggregated site-level TSV) already
  exists — proceed to aggre_features_to_site_level.py instead of re-aligning
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: aligns guppy-basecalled DRS reads to the reference transcriptome (-ax map-ont)
  repo: https://github.com/Derryxu/RedNano
- name: samtools
  role: sorts, indexes, and filters the BAM (-F 3844) before TSV conversion
  repo: https://github.com/Derryxu/RedNano
- name: jvarkit
  role: sam2tsv.jar converts the filtered BAM into a per-base alignment TSV used as
    the basecalling-error source
  repo: https://github.com/lindenb/jvarkit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:2029340e021a1d7ce691a3ed6c89679de4951b9209a6e84e87ae3c77d748f77a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - Python
  - RedNano
  - h5py
  - jvarkit
  - minimap2
  - ont-tombo
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# transcriptome-alignment-tsv-parsing

## Summary

Maps guppy-basecalled Nanopore DRS reads to a reference transcriptome with minimap2, converts the sorted BAM to a per-base TSV via samtools and jvarkit's sam2tsv, and splits that TSV per read so RedNano feature extraction can run in parallel.

## When to use

Use when raw Nanopore DRS reads must be aligned to a reference transcriptome (not a genome) and converted into a per-base alignment TSV that RedNano's extract_features.py can consume as basecalling-error input.

## When NOT to use

- Reads should be aligned to a whole genome rather than a reference transcriptome — tombo resquiggle and this workflow expect the transcript/gene FASTA
- The feature table (test.features.sort.tsv or aggregated site-level TSV) already exists — proceed to aggre_features_to_site_level.py instead of re-aligning

## Inputs

- guppy FASTQ from Nanopore DRS reads (merged test.fastq)
- reference transcriptome FASTA (referance.transcript.fa)
- BAM sorted with samtools (test.bam)
- minimap2/samtools per-base alignment TSV (test.tsv)

## Outputs

- coordinate-sorted and indexed transcriptome BAM (test.bam, test.bam.bai)
- per-base alignment TSV (test.tsv) from sam2tsv.jar
- per-read split alignment TSVs (tmp/<read>.txt) with header retained

## How to apply

Basecall the single-read FAST5 files with guppy v3.1.5 (rna_r9.4.1_70bps_hac.cfg) and merge the FASTQs, then align with minimap2 v2.17 (-t 30 -ax map-ont) against reference.transcript.fa — the tombo resquiggle reference must be the transcript/gene FASTA, not a genome file. Sort and index the BAM with samtools v1.7, filter with 'samtools view -h -F 3844' to drop unmapped/secondary/supplementary records, and convert to a per-base TSV using jvarkit's sam2tsv.jar with the transcriptome FASTA as -r. Finally, split test.tsv by column 2 (read) with an awk one-liner into a tmp/ folder so each read's alignment snippet becomes a separate --errors_dir input for parallel feature extraction (k=5, s=65, -n 30). The header line of the TSV must be preserved and written to each split file.

## Related tools

- **minimap2** (aligns guppy-basecalled DRS reads to the reference transcriptome (-ax map-ont)) — https://github.com/Derryxu/RedNano
- **samtools** (sorts, indexes, and filters the BAM (-F 3844) before TSV conversion) — https://github.com/Derryxu/RedNano
- **jvarkit** (sam2tsv.jar converts the filtered BAM into a per-base alignment TSV used as the basecalling-error source) — https://github.com/lindenb/jvarkit

## Examples

```
minimap2 -t 30 -ax map-ont referance.transcript.fa test.fastq | samtools view -hSb | samtools sort -@ 30 -o test.bam; samtools index test.bam; samtools view -h -F 3844 test.bam | java -jar sam2tsv.jar -r referance.transcript.fa > test.tsv
```

## Evaluation signals

- test.bam is sorted and indexed without errors after minimap2 | samtools sort -@ 30
- The sam2tsv TSV contains a header line plus one row per aligned base, split files tmp/<read>.txt each carry the header
- Every split alignment TSN read name maps to reads present in the resquiggle-corrected FAST5 workspace (RawGenomeCorrected_001)
- Downstream extract_features.py runs complete and output rows carry all required columns (chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele)
- minimap2 v2.17 and samtools v1.7 versions match the RedNano conda environment specification

## Limitations

- The tombo resquiggle reference must be the transcript/gene FASTA, not a genome file — using the wrong reference breaks the pipeline
- The awk split of test.tsv requires column 2 to hold the read name; alignment records must not be interleaved across reads
- sam2tsv requires jvarkit as an external Java dependency in addition to the conda environment

## Evidence

- [methods] Map the nanopore reads to reference transcriptome: "**Map reads to reference transcriptome**"
- [readme] samtools view -h -F 3844 test.bam | java -jar sam2tsv.jar -r referance.transcript.fa > test.tsv: "samtools view -h -F 3844 test.bam | java -jar sam2tsv.jar -r referance.transcript.fa > test.tsv"
- [readme] The tombo (v1.5.1) resquiggle referance.transcript.fa should not be genome file, it should be the referance gene fasta file.: "The tombo (v1.5.1) resquiggle referance.transcript.fa should not be genome file, it should be the referance gene fasta file."
- [readme] To allow downstream parallelisation of the feature extraction, the generated test.tsv is split into the different reads in a tmp folder.: "To allow downstream parallelisation of the feature extraction, the generated test.tsv is split into the different reads in a tmp folder."
