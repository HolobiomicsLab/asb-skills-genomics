---
name: concatemer-read-splitting
description: Use when processing PacBio CCS reads from PAIso-seq libraries where single
  molecules may contain internal TSO (AAGCAGTGGTATCAACGCAGAGTACATGGG), barcode (GAGTGCTACTCTAGTA),
  or 3' adapter (GTACTCTGCGTTGATACCACTGCTT) primer occurrences, which signal concatenated
  templates that must be split before.
when_to_use_negative:
- Reads already split into single full-length molecules with no internal primer occurrences
  — splitting logic will find nothing to remove and only add overhead
- Datasets with a different library structure or primer design (e.g. PacBio IsoSeq3
  pipelines with UMI/cell-barcoded deduplication) where the TSO/barcode/adapter motif
  search would misidentify concatenation
- Short-read (Illumina) data where single reads cannot span a full-length cDNA plus
  poly(A) tail and concatemer structure is not applicable
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: Splice-aware aligner (v2.15-r905) that aligns the split clean CCS reads to
    the mouse reference genome (mm10/GRCm38) using a prebuilt index; alignments with
    supplementary tags are ignored
  repo: https://github.com/PacificBiosciences/IsoSeq3/blob/master/README_v3.2.md
- name: ccs
  role: Generates circular consensus reads (v3.4.1, --richQVs recommended) from PacBio
    movie subreads, the input molecules in which concatemers may occur
- name: GV_oocyte_PAIsoSeqAnalysis run.pl
  role: Perl pipeline that detects concatemer reads when primers are found in the
    middle of reads, splits them, aligns the resulting clean CCS reads, and assigns
    full-length reads to Ensembl GRCm38 genes
  repo: github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: GetCCSpass.pl
  role: Collects CCS pass-number information into ccs.pass.txt, supplied to run.pl
    via --pass so split reads retain accuracy metadata
  repo: github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:ae059f3de1f694f8ca195b2e41dd93ccec6e8f432d157c51c36cb2bae1808325
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GV_oocyte_PAIsoSeqAnalysis run.pl
  - GetCCSpass.pl
  - ccs
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# concatemer-read-splitting

## Summary

Detects and splits PAIso-seq concatemer reads, in which multiple full-length cDNA molecules (each flanked by TSO/barcode/adapter primer sequences) are concatenated within a single circular consensus read, so each transcript is counted and poly(A)-tailed individually rather than as one merged molecule.

## When to use

Use when processing PacBio CCS reads from PAIso-seq libraries where single molecules may contain internal TSO (AAGCAGTGGTATCAACGCAGAGTACATGGG), barcode (GAGTGCTACTCTAGTA), or 3' adapter (GTACTCTGCGTTGATACCACTGCTT) primer occurrences, which signal concatenated templates that must be split before alignment and gene assignment.

## When NOT to use

- Reads already split into single full-length molecules with no internal primer occurrences — splitting logic will find nothing to remove and only add overhead
- Datasets with a different library structure or primer design (e.g. PacBio IsoSeq3 pipelines with UMI/cell-barcoded deduplication) where the TSO/barcode/adapter motif search would misidentify concatenation
- Short-read (Illumina) data where single reads cannot span a full-length cDNA plus poly(A) tail and concatemer structure is not applicable

## Inputs

- Trimmed, demultiplexed CCS FASTA (e.g. sample.out.fasta from trim.py)
- CCS BAM files (movieX.ccs.bam) for pass-number collection via GetCCSpass.pl (ccs.pass.txt)
- Prebuilt minimap2 index (Mus_musculus.mmi) over Mus_musculus.GRCm38.dna.toplevel.chromosome.fa
- PAIso-seq primer sequences: 30-nt TSO (AAGCAGTGGTATCAACGCAGAGTACATGGG), 16-nt barcode (GAGTGCTACTCTAGTA), 25-nt 3' adapter (GTACTCTGCGTTGATACCACTGCTT)

## Outputs

- Split full-length (non-concatemer) CCS reads aligned to the GRCm38/mm10 mouse genome
- Per-transcript gene assignments from Ensembl GRCm38 exons (multi-gene CCSs discarded)
- Summary table of detected transcripts per replicate (e.g. 79,994 rep 1; 227,902 rep 2; 191,023 across 15 single cells; 11,538 genes covered)

## How to apply

After ccs generation (>=10 polymerase passes recommended for base-calling accuracy) and barcode/adapter trimming with trim.py (16-nt barcode plus 25-nt adapter, 22-unit matching configuration, ambiguous reads discarded), run the Perl pipeline run.pl with --species mm10 and a prebuilt minimap2 splice index (minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa). The pipeline scans each clean CCS read for primer sequences occurring in the middle of the read; such internal primers identify concatemers, which are split into their constituent full-length molecules before alignment. Alignments carrying a supplementary alignment tag are ignored, and CCSs assigned to multiple genes are discarded to keep one transcript per molecule. Supply the ccs.pass.txt pass information (from GetCCSpass.pl) so downstream poly(A) tail analysis retains pass-number metadata.

## Related tools

- **minimap2** (Splice-aware aligner (v2.15-r905) that aligns the split clean CCS reads to the mouse reference genome (mm10/GRCm38) using a prebuilt index; alignments with supplementary tags are ignored) — https://github.com/PacificBiosciences/IsoSeq3/blob/master/README_v3.2.md
- **ccs** (Generates circular consensus reads (v3.4.1, --richQVs recommended) from PacBio movie subreads, the input molecules in which concatemers may occur)
- **GV_oocyte_PAIsoSeqAnalysis run.pl** (Perl pipeline that detects concatemer reads when primers are found in the middle of reads, splits them, aligns the resulting clean CCS reads, and assigns full-length reads to Ensembl GRCm38 genes) — github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **GetCCSpass.pl** (Collects CCS pass-number information into ccs.pass.txt, supplied to run.pl via --pass so split reads retain accuracy metadata) — github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- Summary table reproduces reported counts: 79,994 transcripts (replicate 1), 227,902 transcripts (replicate 2), 191,023 transcripts (15 single cells combined), and 11,538 genes covered
- No retained read contains a primer sequence in its middle after splitting — internal TSO/barcode/adapter hits should be absent from the clean read set
- No retained alignment carries a supplementary alignment tag (such alignments are ignored)
- Every counted transcript maps to exactly one Ensembl GRCm38 gene; CCSs assigned to multiple genes must be discarded
- Each split read retains valid pass-number information from ccs.pass.txt (pass number >= 10 recommended for base-calling accuracy)

## Limitations

- PacBio sequencing cost (~5 dollars per 1-K reads on Sequel, ~1000x Illumina per-read cost) limits coverage, which caps replicate correlation and detectable transcript counts
- Concatemer detection relies on exact primer-motif structure; reads with sequencing errors inside primer sites may evade splitting or be misclassified
- Terminal poly(A) 3'-end bases (non-A at the very last position) pair with the adaptor and are lost, so split-read downstream tail analysis cannot accurately quantify terminal non-A modifications

## Evidence

- [readme] Concatemer reads detected by internal primers: "Concatemer reads were detected when primers were found in the middle of the reads"
- [readme] run.pl invocation with minimap2 index and pass file: "perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt"
- [methods] minimap2 index construction for the GRCm38 reference: "minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa"
- [discussion] Supplementary-alignment filter after splitting and alignment: "Alignments with supplementary alignment tag were ignored."
- [discussion] Pass-number threshold ensuring base-calling accuracy of the reads being split: "We require a pass number of 10 or more to ensure the accuracy of base calling."
