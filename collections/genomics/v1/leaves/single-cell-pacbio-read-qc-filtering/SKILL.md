---
name: single-cell-pacbio-read-qc-filtering
description: Use when processing raw PacBio subreads from single-cell PAIso-seq libraries
  and you need quality-filtered, adapter-trimmed CCS reads with reliable poly(A) tail
  calls before isoform or tail-length analysis.
when_to_use_negative:
- The input is short-read Illumina data or bulk (non-single-cell) RNA-seq — the QC
  thresholds and library structure (TSO/barcode/adapter layout) are specific to PAIso-seq
  PacBio single-oocyte libraries.
- You only need genome annotation or isoform clustering rather than per-read poly(A)
  tail QC — use IsoSeq3 clustering tools or TAPIS instead.
- Reads fail the paper's QC criteria (pass number <10, tail <15 nt or lacking 5 continuous
  adenosines) and cannot be rescued; filtering further is not meaningful.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: ccs
  role: Generates circular consensus (HiFi) reads from PacBio subreads (version 3.4.1
    in the paper)
- name: minimap2 (v2.15-r905)
  role: Aligns trimmed CCS reads to the mm10 mouse genome using the -x splice preset
    via a prebuilt .mmi index
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: trim.py
  role: Demultiplexes single-oocyte samples and trims the 16-nt barcode and 25-nt
    adapter from CCS FASTA reads
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: GetCCSpass.pl
  role: Collects per-CCS-read pass numbers used to enforce the >=10 pass QC threshold
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: run.pl
  role: Perl pipeline that computes poly(A) tail lengths, calls non-A residues, and
    applies the paper's QC filters
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:5d8ee648e94ffed994138d8b6779c9f44c750f64f7411d10a218ad3a28097414
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
  tools:
  - GetCCSpass.pl
  - R
  - ccs
  - minimap2 (v2.15-r905)
  - run.pl
  - trim.py
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-pacbio-read-qc-filtering

## Summary

Apply the PAIso-seq per-read QC filters for single mouse GV oocyte PacBio data: generate CCS reads, demultiplex/trim barcodes and adapters, and retain only reads with pass number >=10, clipped tail >=15 nt with >=5 continuous adenosines, and non-A residues <50% and <20 in number.

## When to use

Use when processing raw PacBio subreads from single-cell PAIso-seq libraries and you need quality-filtered, adapter-trimmed CCS reads with reliable poly(A) tail calls before isoform or tail-length analysis.

## When NOT to use

- The input is short-read Illumina data or bulk (non-single-cell) RNA-seq — the QC thresholds and library structure (TSO/barcode/adapter layout) are specific to PAIso-seq PacBio single-oocyte libraries.
- You only need genome annotation or isoform clustering rather than per-read poly(A) tail QC — use IsoSeq3 clustering tools or TAPIS instead.
- Reads fail the paper's QC criteria (pass number <10, tail <15 nt or lacking 5 continuous adenosines) and cannot be rescued; filtering further is not meaningful.

## Inputs

- PacBio subreads BAM files from single GV oocyte SMRTbell libraries (movieX.subreads.bam)
- PAIso-seq library structure parameters: 30-nt TSO, 25-nt 3'-adapter, 16-nt barcode sequences
- Mouse reference genome mm10/GRCm38 FASTA (Mus_musculus.GRCm38.dna.toplevel.chromosome.fa) for building the minimap2 index

## Outputs

- movieX.ccs.bam circular consensus reads
- sample.out.fasta demultiplexed, adapter/barcode-trimmed reads per single oocyte sample
- ccs.pass.txt per-read pass-number table from GetCCSpass.pl
- QC-passed reads with poly(A) tail lengths and non-A residue calls (run.pl output) suitable for downstream R analysis

## How to apply

Start by generating circular consensus sequences from subreads with ccs, using --noPolish --minPasses 1 as in the original pipeline or the recommended --richQVs, then convert the CCS BAM to FASTA with bam2fasta. Demultiplex and trim with scripts/trim.py, supplying the 16-nt barcode plus 25-nt 3'-adapter sequence (GAGTGCTACTCTAGTAGTACTCTGCGTTGATACCACTGCTT) and offset 22. Run the Perl pipeline (run.pl) with the minimap2 -x splice mm10 index and per-read CCS pass counts from scripts/GetCCSpass.pl; apply the paper's QC gates: pass number >=10 to ensure base-calling accuracy, clipped tail length >=15 nt containing at least five continuous adenosines, non-A residues <50% of the tail and <20 in absolute number. Discard ambiguous reads classifiable into two or more samples and CCSs assigned to multiple genes, and ignore alignments carrying a supplementary alignment tag; concatemer reads with internal primers are detected and trimmed. The rationale is that high pass counts underpin HiFi base accuracy, while the tail composition filters keep only trustworthy poly(A) tail length and non-A residue measurements.

## Related tools

- **ccs** (Generates circular consensus (HiFi) reads from PacBio subreads (version 3.4.1 in the paper))
- **minimap2 (v2.15-r905)** (Aligns trimmed CCS reads to the mm10 mouse genome using the -x splice preset via a prebuilt .mmi index) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **trim.py** (Demultiplexes single-oocyte samples and trims the 16-nt barcode and 25-nt adapter from CCS FASTA reads) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **GetCCSpass.pl** (Collects per-CCS-read pass numbers used to enforce the >=10 pass QC threshold) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **run.pl** (Perl pipeline that computes poly(A) tail lengths, calls non-A residues, and applies the paper's QC filters) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- Every retained read has pass number >=10, clipped tail >=15 nt with at least five continuous adenosines, and non-A residues both <50% of tail length and <20 in count.
- Trimmed FASTA reads no longer contain the 3'-adapter sequence GTACTCTGCGTTGATACCACTGCTT or the barcode GAGTGCTACTCTAGTA.
- Reads ambiguous between two or more samples, and CCSs assigned to multiple genes, are absent from the output.
- cDNA read size distribution reflects the library design (cDNA plus poly(A) tail; size selection at >200 bp and >2 kb), and alignments use the splice preset against mm10 without supplementary-alignment-tagged records.
- Concatemer reads with internal primers are detected and trimmed rather than counted as full-length transcripts.

## Limitations

- Requiring pass number >=10 sacrifices reads and thus coverage; the authors note replicate correlation would improve with deeper sequencing given current coverage limits.
- PAIso-seq cannot accurately quantify non-A modifications at the very last 3'-end base, and a minimal number of additional A residues may be introduced during extension primer pairing.
- High PacBio sequencing cost (~$5 per 1-K reads, about 1000x Illumina per-read cost) limits achievable coverage of single-cell libraries.

## Evidence

- [discussion] pass number filter: "We require a pass number of 10 or more to ensure the accuracy of base calling."
- [discussion] tail length and composition filters: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [discussion] non-A residue limits: "the percentage of the non-A residues was <50% and the number of non-A residues were <20"
- [readme] trim.py invocation: "./scripts/trim.py movieX.ccs.fasta sample GAGTGCTACTCTAGTAGTACTCTGCGTTGATACCACTGCTT 22 2 1>sample.out.fasta 2>sample.err.fasta"
- [discussion] ambiguous read handling: "ambiguous reads that could be classified into two or more samples, were discarded"
