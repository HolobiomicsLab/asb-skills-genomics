---
name: minimap2-splice-alignment-to-mm10
description: Use when adapter-trimmed PacBio CCS FASTA reads from PAIso-seq (or similar
  full-length cDNA) libraries must be mapped to mouse GRCm38/mm10 in splice-aware
  mode before gene assignment or poly(A) tail measurement.
when_to_use_negative:
- Input reads are short Illumina paired-end reads rather than full-length PacBio CCS
  cDNA (use a standard short-read splice aligner such as STAR/HISAT2 instead)
- The target species is not mouse GRCm38/mm10 (a species-matched genome index is required;
  e.g., the pepper Iso-seq dataset used its own genome index)
- Reads have not been demultiplexed and adapter/barcode trimmed — untrimmed concatemers
  and adapters will corrupt splice alignment
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: minimap2
  role: Splice-aware alignment of clean CCS reads to the mouse GRCm38/mm10 reference
    genome (v2.15-r905), invoked inside the Perl run.pl pipeline with 10 threads
  repo: https://github.com/PacificBiosciences/IsoSeq3/blob/master/README_v3.2.md
- name: ccs
  role: Generates the circular consensus sequences from PacBio subreads that become
    the minimap2 alignment input
  repo: https://github.com/PacificBiosciences/IsoSeq
- name: run.pl (GV_oocyte_PAIsoSeqAnalysis)
  role: Perl driver that splits concatemer reads, calls minimap2 with the splice index,
    and assigns aligned full-length reads to genes
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:80523f73f4f9219457a520ccc773d1c7f2c3173df07b0ee5458f96e2211e90d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Perl
  - R
  - bam2fasta
  - ccs
  - minimap2
  - run.pl (GV_oocyte_PAIsoSeqAnalysis)
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# minimap2-splice-alignment-to-mm10

## Summary

Align clean, adapter-trimmed PAIso-seq CCS reads to the mouse GRCm38/mm10 reference genome using minimap2 in splice mode (v2.15-r905) within the Perl run.pl pipeline, as a prerequisite step for gene assignment and poly(A) tail quantification.

## When to use

Use when adapter-trimmed PacBio CCS FASTA reads from PAIso-seq (or similar full-length cDNA) libraries must be mapped to mouse GRCm38/mm10 in splice-aware mode before gene assignment or poly(A) tail measurement.

## When NOT to use

- Input reads are short Illumina paired-end reads rather than full-length PacBio CCS cDNA (use a standard short-read splice aligner such as STAR/HISAT2 instead)
- The target species is not mouse GRCm38/mm10 (a species-matched genome index is required; e.g., the pepper Iso-seq dataset used its own genome index)
- Reads have not been demultiplexed and adapter/barcode trimmed — untrimmed concatemers and adapters will corrupt splice alignment

## Inputs

- Adapter-trimmed and demultiplexed CCS FASTA (sample.out.fasta) from trim.py
- Mus_musculus.GRCm38.dna.toplevel.chromosome.fa (Ensembl GRCm38/mm10 genome)
- CCS pass counts file (ccs.pass.txt) from GetCCSpass.pl
- minimap2 splice index (Mus_musculus.mmi)

## Outputs

- Splice-aware alignments of clean CCS reads to the mouse mm10 genome
- Gene-assigned full-length reads (via Ensembl GRCm38 exons) and per-replicate detected transcript/summary tables
- Poly(A) tail length and non-A residue calls per transcript

## How to apply

Build a splice-mode minimap2 index with `minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa` using the Ensembl GRCm38 (release 92) toplevel chromosome FASTA. After CCS calling (>=10 passes), bam2fasta conversion, and trim.py demultiplexing with the 16-nt barcode GAGTGCTACTCTAGTA plus 25-nt adapter GTACTCTGCGTTGATACCACTGCTT, invoke the Perl pipeline (`perl run.pl --ccs sample.out.fasta --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt`) so that concatemer reads with internal primers are split and clean CCS reads are aligned to mm10. Ignore alignments carrying a supplementary alignment tag to avoid chimeric or multi-locus artifacts. Aligned full-length reads are then assigned to genes via Ensembl GRCm38 exons, discarding CCSs assigned to multiple genes, before poly(A) tail length and non-A residue calling.

## Related tools

- **minimap2** (Splice-aware alignment of clean CCS reads to the mouse GRCm38/mm10 reference genome (v2.15-r905), invoked inside the Perl run.pl pipeline with 10 threads) — https://github.com/PacificBiosciences/IsoSeq3/blob/master/README_v3.2.md
- **ccs** (Generates the circular consensus sequences from PacBio subreads that become the minimap2 alignment input) — https://github.com/PacificBiosciences/IsoSeq
- **run.pl (GV_oocyte_PAIsoSeqAnalysis)** (Perl driver that splits concatemer reads, calls minimap2 with the splice index, and assigns aligned full-length reads to genes) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt
```

## Evaluation signals

- Alignment runs against a minimap2 splice index built exactly with `-x splice -t 20` on Mus_musculus.GRCm38.dna.toplevel.chromosome.fa
- Reads carrying supplementary alignment tags are ignored (count of retained primary-only alignments is consistent)
- Per-replicate mapped transcript counts reproduce published values: 79,994 transcripts (rep 1), 227,902 transcripts (rep 2), 191,023 transcripts for 15 single cells combined, with 11,538 genes covered
- CCSs assigned to multiple genes are discarded from the gene assignment output
- Only reads with >=10 CCS passes (from ccs.pass.txt) contribute to downstream tail and gene quantification

## Limitations

- Alignment and tail quantification are restricted to reads with >=10 polymerase passes, limiting sensitivity for low-coverage molecules; replicate correlation is expected to improve with deeper sequencing
- Very long tails (>231 nt, e.g., one 13-kb tail) were detected in only one replicate and are of very low abundance, not independently verified
- Supplementary alignments are ignored, so multi-mapping or chimeric molecules yield no usable gene assignment

## Evidence

- [discussion] Clean CCS reads were aligned to mouse reference genome (mm10) by using minimap2 (v2.15-r905) with the following parameters: "Clean CCS reads were aligned to mouse reference genome (mm10) by using minimap2 (v2.15-r905) with the following parameters"
- [readme] minimap2 index build command: "minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa"
- [readme] perl run.pl invocation: "perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread  10 --pass ccs.pass.txt"
- [discussion] supplementary alignment filter: "Alignments with supplementary alignment tag were ignored."
- [discussion] pass-number requirement: "We require a pass number of 10 or more to ensure the accuracy of base calling."
