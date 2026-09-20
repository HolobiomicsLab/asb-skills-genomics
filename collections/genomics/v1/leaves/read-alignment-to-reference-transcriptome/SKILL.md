---
name: read-alignment-to-reference-transcriptome
description: Use when you have basecalled Nanopore direct RNA FASTQ reads (e.g., test.fastq
  from Guppy) and need them mapped to a reference transcript FASTA for downstream
  signal-to-sequence feature extraction.
when_to_use_negative:
- Reads have not yet been basecalled — basecalling with guppy_basecaller (rna_r9.4.1_70bps_hac.cfg)
  must precede alignment
- The only available reference is a genome FASTA rather than a transcript/gene FASTA
  — RedNano's resquiggle and alignment steps expect the reference gene fasta, not
  a genome file
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: v2.17 (-ax map-ont) is used to map nanopore DRS reads to the reference transcriptome
  repo: https://github.com/Derryxu/RedNano
- name: samtools
  role: v1.7 converts minimap2 output to BAM (samtools view -hSb), sorts it (samtools
    sort -@ 30), indexes it (samtools index), and filters records (-F 3844) before
    TSV conversion
  repo: https://github.com/Derryxu/RedNano
- name: jvarkit
  role: sam2tsv converts the filtered BAM alignment to a TSV format file used for
    downstream feature extraction
  repo: https://github.com/lindenb/jvarkit
- name: Guppy
  role: v3.1.5 produces the basecalled FASTQ (test.fastq) that is the input to transcriptome
    alignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:d1e52fa3574c46a411f470061f32667dcfbd7fa75ed672ad1127802064383ec6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - Guppy
  - RedNano
  - jvarkit
  - minimap2
  - ont-tombo
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# read-alignment-to-reference-transcriptome

## Summary

Align basecalled Nanopore DRS reads to a reference transcriptome using minimap2, sort/index the BAM with samtools, and optionally convert to TSV via jvarkit sam2tsv — a prerequisite step in RedNano's preprocessing so raw signals can be re-squiggled to transcript coordinates.

## When to use

Use when you have basecalled Nanopore direct RNA FASTQ reads (e.g., test.fastq from Guppy) and need them mapped to a reference transcript FASTA for downstream signal-to-sequence feature extraction.

## When NOT to use

- Reads have not yet been basecalled — basecalling with guppy_basecaller (rna_r9.4.1_70bps_hac.cfg) must precede alignment
- The only available reference is a genome FASTA rather than a transcript/gene FASTA — RedNano's resquiggle and alignment steps expect the reference gene fasta, not a genome file

## Inputs

- Merged basecalled FASTQ file (e.g., test.fastq) from Guppy basecalling
- Reference transcriptome FASTA file (referance.transcript.fa), not a genome FASTA
- Reference transcriptome length file with header (e.g., cc.bed) for downstream feature extraction

## Outputs

- Sorted, indexed BAM file of transcriptome-aligned reads (test.bam plus test.bam.bai)
- Alignment TSV file (test.tsv) generated via samtools view -F 3844 and jvarkit sam2tsv

## How to apply

Run minimap2 v2.17 with '-t 30 -ax map-ont' to align the merged FASTQ (test.fastq) to the reference transcriptome FASTA (referance.transcript.fa), piping output through 'samtools view -hSb' and 'samtools sort -@ 30' to produce a sorted BAM (test.bam). Index the sorted BAM with 'samtools index'. For RedNano feature extraction, additionally convert the BAM to TSV using samtools view with '-F 3844' (filtering secondary/supplementary/unmapped/multimapped records) and jvarkit's sam2tsv against the transcriptome FASTA, yielding test.tsv with per-read alignment details. Use the reference gene (transcript) fasta, not a genome file, consistent with the Tombo resquiggle requirement.

## Related tools

- **minimap2** (v2.17 (-ax map-ont) is used to map nanopore DRS reads to the reference transcriptome) — https://github.com/Derryxu/RedNano
- **samtools** (v1.7 converts minimap2 output to BAM (samtools view -hSb), sorts it (samtools sort -@ 30), indexes it (samtools index), and filters records (-F 3844) before TSV conversion) — https://github.com/Derryxu/RedNano
- **jvarkit** (sam2tsv converts the filtered BAM alignment to a TSV format file used for downstream feature extraction) — https://github.com/lindenb/jvarkit
- **Guppy** (v3.1.5 produces the basecalled FASTQ (test.fastq) that is the input to transcriptome alignment)

## Examples

```
minimap2 -t 30 -ax map-ont /path/to/referance.transcript.fa /test.fastq | samtools view -hSb | samtools sort -@ 30 -o test.bam; samtools index test.bam
```

## Evaluation signals

- test.bam exists and is a valid sorted BAM; 'samtools index test.bam' completes without error
- The minimap2 command uses the ONT preset (-ax map-ont) and the transcriptome FASTA (not a genome) as reference
- The TSV conversion command filters with 'samtools view -h -F 3844' so only primary mapped alignments feed feature extraction
- test.tsv is generated with sam2tsv against referance.transcript.fa and can be split per-read into the tmp/ folder for downstream parallel feature extraction

## Limitations

- Alignment must target a transcript/gene FASTA, not a genome FASTA; the README warns the tombo resquiggle reference 'should not be genome file, it should be the referance gene fasta file'
- Guppy v3.1.5 and jvarkit are external requirements not bundled in the RedNano conda environment
- The sorted BAM alone is insufficient for RedNano — signal-to-reference mapping additionally requires the Tombo resquiggle workspace (RawGenomeCorrected_001)

## Evidence

- [methods] In RedNano preprocessing, each basecalled read is aligned to the reference transcriptome using minimap2 (v2.17): [evidence span withheld — anchor status: not_found]
- [readme] Map the nanopore reads to reference transcriptome using minimap2 (v2.17-r941): "Map the nanopore reads to reference transcriptome  using minimap2 (v2.17-r941)"
- [readme] minimap2 -t 30 -ax map-ont /path/to/referance.transcript.fa /test.fastq | samtools view -hSb | samtools sort -@ 30 -o test.bam: "minimap2 -t 30 -ax map-ont /path/to/referance.transcript.fa /test.fastq | samtools view -hSb | samtools sort -@ 30 -o test.bam"
- [readme] The tombo (v1.5.1) resquiggle referance.transcript.fa should not be genome file, it should be the referance gene fasta file.: "The tombo (v1.5.1) resquiggle referance.transcript.fa should not be genome file, it should be the referance gene fasta file."
