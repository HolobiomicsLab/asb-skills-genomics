---
name: non-adenosine-residue-composition-filtering
description: Use when computing per-read poly(A) tail lengths from soft-clipped terminal
  sequences of minimap2 splice alignments of PAIso-seq CCS reads, to exclude spurious
  clips and retain tails with controlled non-A content.
when_to_use_negative:
- The candidate sequence is not a terminal soft-clip of a primary minimap2 alignment
  (e.g. it comes from a supplementary/chimeric alignment) — the filter is defined
  only on terminal clips of primary alignments.
- The analysis target is genome-wide polyadenylation site (APA) detection rather than
  tail length — that is handled by the TAPIS pipeline (run_tapis.py) with GMAP alignments
  in this paper.
- Input reads have fewer than 10 CCS passes, since base-calling accuracy is insufficient
  and the pipeline discards them before tail calling.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: minimap2 (v2.15-r905)
  role: Splice-aligns clean CCS reads to the mm10/GRCm38 genome (-x splice); its terminal
    soft-clipped sequences are the candidate poly(A) tails evaluated by the filter
- name: Perl pipeline (run.pl / scripts in GV_oocyte_PAIsoSeqAnalysis)
  role: Implements the four-criterion tail-acceptance filter (>=15 nt, >=5 continuous
    A, <50% non-A, <20 non-A) and records per-read tail length
  repo: github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: ccs (version 3.4.1)
  role: Generates the circular consensus sequences from subreads that are upstream
    inputs to alignment and tail calling
  repo: https://github.com/PacificBiosciences/IsoSeq3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:47ca61313a89e300a6b40732a6156fb768ae62fce4fb524659488613dd7e9c0c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Perl
  - Perl pipeline (run.pl / scripts in GV_oocyte_PAIsoSeqAnalysis)
  - ccs (version 3.4.1)
  - minimap2 (v2.15-r905)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# non-adenosine-residue-composition-filtering

## Summary

A four-criterion Perl-based filter that decides whether the terminal clipped sequence of a minimap2-aligned clean CCS read qualifies as a poly(A) tail in the PAIso-seq pipeline, and computes per-read tail length for reads that pass.

## When to use

Use when computing per-read poly(A) tail lengths from soft-clipped terminal sequences of minimap2 splice alignments of PAIso-seq CCS reads, to exclude spurious clips and retain tails with controlled non-A content.

## When NOT to use

- The candidate sequence is not a terminal soft-clip of a primary minimap2 alignment (e.g. it comes from a supplementary/chimeric alignment) — the filter is defined only on terminal clips of primary alignments.
- The analysis target is genome-wide polyadenylation site (APA) detection rather than tail length — that is handled by the TAPIS pipeline (run_tapis.py) with GMAP alignments in this paper.
- Input reads have fewer than 10 CCS passes, since base-calling accuracy is insufficient and the pipeline discards them before tail calling.

## Inputs

- clean CCS reads as FASTA (sample.out.fasta) after ccs calling and trim.py adapter/barcode trimming
- minimap2 splice alignments of clean CCS reads against the mm10/GRCm38 reference (Mus_musculus.GRCm38.dna.toplevel.chromosome.fa)
- CCS pass-count table (ccs.pass.txt) from GetCCSpass.pl, with reads requiring >=10 passes

## Outputs

- Per-read poly(A) tail-length table (polya_tail_lengths.csv) for reads passing all four tail-acceptance criteria
- Poly(A) tail sequences with non-adenosine residue composition retained for downstream tail-body non-A analysis

## How to apply

Parse minimap2 (v2.15-r905, -x splice) alignments of clean CCS reads against mm10/GRCm38 and skip any alignment carrying a supplementary alignment tag so only primary, non-chimeric alignments remain. Extract the terminal soft-clipped sequence of each alignment as the candidate poly(A) tail and require it to be >=15 nt long and to contain at least five continuous adenosines. Then count non-adenosine residues in the accepted tail and retain it only if the non-A percentage is <50% and the non-A count is <20; these composition bounds are what let the pipeline read non-A residues within poly(A) tail bodies while discarding low-quality clips. For each retained read, record the tail length and write the per-read tail-length table (polya_tail_lengths.csv). The filter is implemented in Perl within the run.pl pipeline of the GV_oocyte_PAIsoSeqAnalysis repository, downstream of CCS calling and adapter trimming.

## Related tools

- **minimap2 (v2.15-r905)** (Splice-aligns clean CCS reads to the mm10/GRCm38 genome (-x splice); its terminal soft-clipped sequences are the candidate poly(A) tails evaluated by the filter)
- **Perl pipeline (run.pl / scripts in GV_oocyte_PAIsoSeqAnalysis)** (Implements the four-criterion tail-acceptance filter (>=15 nt, >=5 continuous A, <50% non-A, <20 non-A) and records per-read tail length) — github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **ccs (version 3.4.1)** (Generates the circular consensus sequences from subreads that are upstream inputs to alignment and tail calling) — https://github.com/PacificBiosciences/IsoSeq3

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt
```

## Evaluation signals

- Every accepted tail in polya_tail_lengths.csv satisfies all four criteria: length >=15 nt, at least five continuous adenosines, <50% non-A residues, and <20 non-A residues.
- Every retained read carries a computed tail-length value; no read is emitted with a missing tail length.
- No alignment contributing a tail has a supplementary alignment tag (only primary, non-chimeric alignments are used).
- All reads contributing to the output have a CCS pass number >=10, consistent with the pipeline's accuracy requirement.
- Per-read tail lengths from replicate GV-oocyte libraries reproduce the published length distributions (e.g. tails beyond the ~231 nt TAIL-seq range are detected).

## Limitations

- PAIso-seq cannot accurately quantify the frequency of non-A modifications at the very last 3'-end base, because terminal G/C/U pair to the adaptor and are lost.
- A minimal number of additional A residues may be introduced during pairing of the extension primer, biasing terminal base composition.
- Concatemer reads with internal primers must be detected and trimmed first; undetected concatemers would corrupt terminal-clip extraction.
- Ambiguous reads classifiable into two or more samples and CCSs assigned to multiple genes are discarded, reducing usable reads.

## Evidence

- [discussion] Require >=15 nt clipped sequence and at least five continuous adenosines for poly(A) tail: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [discussion] Non-A residue composition bounds for accepted tails: "the percentage of the non-A residues was <50% and the number of non-A residues were <20"
- [discussion] Supplementary alignments are excluded before tail extraction: "Alignments with supplementary alignment tag were ignored."
- [discussion] Minimap2 alignment of clean CCS reads against the mouse genome: "Clean CCS reads were aligned to mouse reference genome (mm10) by using minimap2 (v2.15-r905)"
- [discussion] CCS pass threshold ensuring base-calling accuracy: "We require a pass number of 10 or more to ensure the accuracy of base calling."
