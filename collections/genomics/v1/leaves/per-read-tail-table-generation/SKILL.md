---
name: per-read-tail-table-generation
description: Use when PAIso-seq clean CCS reads have been aligned with minimap2 -x
  splice to mm10/GRCm38 and per-read poly(A) tail lengths must be called from terminal
  clipped sequences under explicit length/purity thresholds.
when_to_use_negative:
- Input alignments are chimeric or carry a supplementary alignment tag, since such
  alignments are explicitly ignored by the pipeline
- The task is 3' end poly(A) site annotation or APA analysis (handled by the TAPIS/run_tapis.py
  pipeline with GMAP), not per-read tail-length measurement
- Clipped candidates shorter than 15 nt or lacking five continuous adenosines exist
  and no candidate tail can satisfy the filter — no tail should be forced onto such
  reads
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: v2.15-r905 with -x splice aligns clean CCS reads to mm10/GRCm38; its terminal
    soft-clipped sequences are the candidate poly(A) tails
- name: ccs
  role: Generates circular consensus sequences (ccs 3.4.1; --richQVs suggested in
    README) that are trimmed and aligned upstream of tail extraction
  repo: https://github.com/PacificBiosciences/IsoSeq3/blob/master/README_v3.2.md
- name: GV_oocyte_PAIsoSeqAnalysis pipeline (Perl run.pl)
  role: Implements the tail-acceptance filter and writes the per-read tail-length
    table; dependencies Perl v5.26.2, R 3.5.1, minimap2 2.15-r905
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:dae55a8b983eaa064d2f90c03a1fc94499c1732f4b999bd0092f3a7d142394f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GV_oocyte_PAIsoSeqAnalysis pipeline (Perl run.pl)
  - Perl
  - ccs
  - minimap2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# per-read-tail-table-generation

## Summary

Extracts the terminal soft-clipped sequence from minimap2 (v2.15-r905, -x splice) alignments of clean CCS reads, applies the PAIso-seq four-criterion poly(A) tail filter, and writes per-read tail lengths (e.g. polya_tail_lengths.csv) used for downstream non-A residue and tail-length analysis.

## When to use

Use when PAIso-seq clean CCS reads have been aligned with minimap2 -x splice to mm10/GRCm38 and per-read poly(A) tail lengths must be called from terminal clipped sequences under explicit length/purity thresholds.

## When NOT to use

- Input alignments are chimeric or carry a supplementary alignment tag, since such alignments are explicitly ignored by the pipeline
- The task is 3' end poly(A) site annotation or APA analysis (handled by the TAPIS/run_tapis.py pipeline with GMAP), not per-read tail-length measurement
- Clipped candidates shorter than 15 nt or lacking five continuous adenosines exist and no candidate tail can satisfy the filter — no tail should be forced onto such reads

## Inputs

- minimap2 -x splice alignment records (SAM/BAM) of clean PAIso-seq CCS reads against mm10/GRCm38
- Trimmed/demultiplexed clean CCS FASTA reads (e.g. sample.out.fasta from trim.py)
- CCS pass counts (ccs.pass.txt from GetCCSpass.pl)

## Outputs

- Per-read poly(A) tail-length table (polya_tail_lengths.csv) for reads passing all four tail criteria
- Non-A residue calls within the body of accepted poly(A) tails

## How to apply

Parse minimap2 splice alignments of clean CCS reads against the mm10/GRCm38 reference and skip alignments carrying a supplementary alignment tag so only primary, non-chimeric alignments remain. Extract each alignment's terminal soft-clipped sequence as the candidate tail and accept it only if it is at least 15 nt long and contains at least five continuous adenosines. For accepted tails, count non-A residues and retain the read only if non-A residues are fewer than 20 and below 50% of the tail; the tail length is then recorded per read. The rationale is that strict length, continuous-adenosine, and non-A purity criteria prevent genomic-clipping artifacts and internally divergent sequences from being scored as tails. Implement the filter in Perl (as in the GV_oocyte_PAIsoSeqAnalysis pipeline driven by run.pl) and emit a per-read tail-length table (polya_tail_lengths.csv) where every retained read carries a computed tail-length value.

## Related tools

- **minimap2** (v2.15-r905 with -x splice aligns clean CCS reads to mm10/GRCm38; its terminal soft-clipped sequences are the candidate poly(A) tails)
- **ccs** (Generates circular consensus sequences (ccs 3.4.1; --richQVs suggested in README) that are trimmed and aligned upstream of tail extraction) — https://github.com/PacificBiosciences/IsoSeq3/blob/master/README_v3.2.md
- **GV_oocyte_PAIsoSeqAnalysis pipeline (Perl run.pl)** (Implements the tail-acceptance filter and writes the per-read tail-length table; dependencies Perl v5.26.2, R 3.5.1, minimap2 2.15-r905) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- Every accepted tail satisfies all four criteria: length >=15 nt, >=5 continuous adenosines, <50% non-A residues, and <20 non-A residues
- No read whose alignment carries a supplementary alignment tag appears in the output tail table
- Each retained read in polya_tail_lengths.csv carries a computed tail-length value (no missing/null lengths)
- Total read count after filtering matches reads passing the CCS pass-number >=10 accuracy requirement
- Recorded tail lengths are consistent with the clipped-sequence lengths in the source minimap2 alignments

## Limitations

- PAIso-seq cannot accurately quantify non-A modifications at the very last 3'-end base because terminal G/C/U pair to the adaptor and are lost, and additional A residues may be introduced during extension primer pairing
- Reliable detection requires >=10 CCS passes for base-calling accuracy, and high PacBio sequencing cost (~$5 per 1-K reads) limits achievable coverage, so replicate correlation is coverage-dependent
- Reads classified into multiple samples or assigned to multiple genes must be discarded, reducing usable reads feeding the tail filter

## Evidence

- [discussion] length >=15 nt and five continuous adenosines: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [discussion] non-A purity thresholds: "the percentage of the non-A residues was <50% and the number of non-A residues were <20"
- [discussion] supplementary alignments ignored: "Alignments with supplementary alignment tag were ignored."
- [discussion] alignment setup: "Clean CCS reads were aligned to mouse reference genome (mm10) by using minimap2 (v2.15-r905) with the following parameters"
- [readme] index build command: "minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa"
