---
name: t30vn-internal-priming-anchoring-classification
description: Use when analyzing full-length PacBio Iso-seq/PAIso-seq CCS reads to
  test whether non-adenosine residues occur inside poly(A) tail bodies, by classifying
  T30VN RT-primer anchoring as terminal versus internal (non-A) positions.
when_to_use_negative:
- Input reads lack CCS pass counts or fail the >=10 passes requirement, since base-calling
  accuracy of tail non-A residues cannot be trusted
- The goal is to quantify non-A modifications at the very last 3'-end tail base, which
  PAIso-seq cannot accurately measure because terminal G/C/U pair to the adaptor and
  are lost
- Data are short-read Illumina-based (e.g., PAL-seq on the discontinued Illumina Genome
  Analyzer II) rather than full-length PacBio cDNA reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: ccs
  role: Generates circular consensus sequences from subreads BAM (recommended --richQVs;
    historical --noPolish --minPasses 1) as input to the anchoring pipeline
  repo: https://github.com/PacificBiosciences/IsoSeq3
- name: bam2fasta
  role: Converts the CCS BAM to FASTA (-u -o ccs) for demultiplexing and adapter trimming
  repo: https://github.com/PacificBiosciences/IsoSeq
- name: minimap2
  role: Aligns full-length trimmed CCS reads to the reference genome with a splice
    index to define transcript bodies and clipped poly(A) tails
- name: PAIso-Seq Perl pipeline (run.pl, trim.py, GetCCSpass.pl)
  role: Trims barcode/adapter, enforces pass and tail filters, computes tail lengths,
    calls non-A residues, and supports T30VN anchoring classification
  repo: https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:a07abe4c607331a98c539bdd62d9bf5618baac4dec717795e5d03cbd500d446f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/IsoSeq3
  tools:
  - PAIso-Seq Perl pipeline (run.pl, trim.py, GetCCSpass.pl)
  - Perl
  - bam2fasta
  - ccs
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# t30vn-internal-priming-anchoring-classification

## Summary

Classifies PAIso-seq reads by where the T30VN reverse-transcription primer anchored on the poly(A) tail, distinguishing terminal anchoring from internal anchoring on non-adenosine residues within the poly(A) tail body. This validates the existence of non-A residues inside tails in independent datasets such as the pepper Iso-seq data (CRA001412), where ~5% of transcripts show VN base pairing with internal tail nucleotides.

## When to use

Use when analyzing full-length PacBio Iso-seq/PAIso-seq CCS reads to test whether non-adenosine residues occur inside poly(A) tail bodies, by classifying T30VN RT-primer anchoring as terminal versus internal (non-A) positions.

## When NOT to use

- Input reads lack CCS pass counts or fail the >=10 passes requirement, since base-calling accuracy of tail non-A residues cannot be trusted
- The goal is to quantify non-A modifications at the very last 3'-end tail base, which PAIso-seq cannot accurately measure because terminal G/C/U pair to the adaptor and are lost
- Data are short-read Illumina-based (e.g., PAL-seq on the discontinued Illumina Genome Analyzer II) rather than full-length PacBio cDNA reads

## Inputs

- PacBio subreads BAM (e.g., pepper Iso-seq experiment CRX041331, accession CRA001412)
- CCS BAM produced by ccs
- CCS FASTA converted with bam2fasta
- Pepper reference genome FASTA for minimap2 splice index
- CCS pass-count file (ccs.pass.txt) from GetCCSpass.pl

## Outputs

- Per-read T30VN anchoring classification (terminal vs. internal non-A anchoring)
- Percentage of transcripts with internal (VN) base pairing inside poly(A) tail bodies
- Counts/frequencies of single vs. double non-A internal anchoring events
- Filtered table of poly(A) tail lengths and called non-A residues per transcript

## How to apply

First generate circular consensus sequences from the pepper subreads BAM with ccs (recommended: --richQVs; historical run used --noPolish --minPasses 1), convert the CCS BAM to FASTA with bam2fasta (-u -o ccs), then demultiplex and trim with scripts/trim.py using the 16-nt barcode GAGTGCTACTCTAGTA and 25-nt adapter GTACTCTGCGTTGATACCACTGCTT at offsets 22 and 2. Build a minimap2 splice index of the pepper reference (-x splice -t 20) and collect per-CCS pass counts with scripts/GetCCSpass.pl. Run the Perl pipeline 'perl run.pl --ccs sample.out.fasta --sample ... --species ... --minimap2_index ... --minimap2_thread 10 --pass ccs.pass.txt' to align full-length reads, compute poly(A) tail lengths, and call non-A residues, keeping only reads with >=10 CCS passes, clipped tails >=15 nt containing >=5 continuous adenosines, and non-A residues that are both <50% and <20 per tail. Finally, for each retained read, determine the position at which the T30VN primer base-paired during reverse transcription: reads whose VN positions pair with non-A nucleotides in the middle of the poly(A) tail body are 'internal anchoring' reads; tabulate the percentage of internal-anchoring transcripts and compare single versus double non-A anchoring frequencies.

## Related tools

- **ccs** (Generates circular consensus sequences from subreads BAM (recommended --richQVs; historical --noPolish --minPasses 1) as input to the anchoring pipeline) — https://github.com/PacificBiosciences/IsoSeq3
- **bam2fasta** (Converts the CCS BAM to FASTA (-u -o ccs) for demultiplexing and adapter trimming) — https://github.com/PacificBiosciences/IsoSeq
- **minimap2** (Aligns full-length trimmed CCS reads to the reference genome with a splice index to define transcript bodies and clipped poly(A) tails)
- **PAIso-Seq Perl pipeline (run.pl, trim.py, GetCCSpass.pl)** (Trims barcode/adapter, enforces pass and tail filters, computes tail lengths, calls non-A residues, and supports T30VN anchoring classification) — https://github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample pepper --species pepper --minimap2_index pepper.mmi --minimap2_thread 10 --pass ccs.pass.txt
```

## Evaluation signals

- The computed percentage of transcripts with internal anchoring approximates the reported ~5% in the pepper Iso-seq dataset
- Single non-A anchoring events are more frequent than double non-A anchoring events
- All retained reads satisfy the filters: >=10 CCS passes, clipped tail >=15 nt with >=5 continuous adenosines, non-A residues <50% and <20 per tail
- No retained read is classified into two or more samples or assigned to multiple genes, and alignments with supplementary alignment tags are ignored
- Detected non-A positions lie within the poly(A) tail body, not just at the annotated tail terminus

## Limitations

- PAIso-seq cannot accurately quantify non-A modifications at the very last 3'-end base, since terminal G/C/U pair to the adaptor and are lost
- Additional A residues may be introduced during pairing of the extension primer, a terminal base-calling bias
- CCSs assigned to multiple genes or ambiguous reads classifiable into two or more samples are discarded, reducing usable reads
- Very low-abundance events (e.g., a 13-kb tail seen in only one replicate) may not be independently verified

## Evidence

- [discussion] Filter steps: >=10 CCS passes and clipped tail requirements: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [discussion] Non-A residue acceptance criteria: "the percentage of the non-A residues was <50% and the number of non-A residues were <20"
- [readme] Recommended ccs invocation: "ccs movieX.subreads.bam movieX.ccs.bam --richQVs &>ccs.log"
- [readme] Barcode and adapter sequences with offsets for trim.py: "./scripts/trim.py movieX.ccs.fasta sample GAGTGCTACTCTAGTAGTACTCTGCGTTGATACCACTGCTT 22 2 1>sample.out.fasta 2>sample.err.fasta"
- [readme] Pipeline run command tying together alignment, tail length, and non-A calling: "perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread  10 --pass ccs.pass.txt &>run.log"
- [discussion] Pass-number accuracy requirement: "We require a pass number of 10 or more to ensure the accuracy of base calling."
