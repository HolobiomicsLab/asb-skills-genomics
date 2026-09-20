---
name: polyadenylation-site-detection-tapis
description: 'Use when full-length PacBio CCS reads (PAIso−seq or similar) must be
  analyzed for alternative polyadenylation: counting distinct poly(A) sites per gene
  and reporting per-gene APA site-count distributions.'
when_to_use_negative:
- Input reads are short Illumina reads rather than full-length PacBio CCS reads, since
  poly(A) tails must be captured within the same molecule as the cDNA
- The goal is quantifying non-A residues within poly(A) tail bodies rather than detecting
  APA sites (use the tail-composition skill instead)
- The organism/reference is not mouse GRCm38/mm10 and no equivalent Ensembl annotation
  is available for gene assignment
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0659
tools:
- name: GMAP
  role: Aligns tail-trimmed clean CCS reads to the mm10/GRCm38 mouse reference genome
    before APA detection
- name: TAPIS pipeline (run_tapis.py)
  role: Detects polyadenylation sites per aligned read and feeds per-gene APA site
    counting
  repo: github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: minimap2 (v2.15-r905)
  role: Alternative splice-aware aligner used to build the Mus_musculus.mmi index
    for the PAIso-Seq pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:597cafd57ec050b5a33a752eb335753c8f99897ad2d990cb8386277ffe51fd99
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GMAP
  - TAPIS pipeline (run_tapis.py)
  - minimap2 (v2.15-r905)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# polyadenylation-site-detection-tapis

## Summary

Detects per-read polyadenylation (APA) sites from full-length PAIso−seq CCS reads by aligning tail-trimmed reads to mm10 with GMAP, extracting qualifying 3' poly(A) tail clips, and running the TAPIS pipeline (run_tapis.py) to count distinct poly(A) sites per gene against Ensembl GRCm38 release 92 annotation.

## When to use

Use when full-length PacBio CCS reads (PAIso−seq or similar) must be analyzed for alternative polyadenylation: counting distinct poly(A) sites per gene and reporting per-gene APA site-count distributions.

## When NOT to use

- Input reads are short Illumina reads rather than full-length PacBio CCS reads, since poly(A) tails must be captured within the same molecule as the cDNA
- The goal is quantifying non-A residues within poly(A) tail bodies rather than detecting APA sites (use the tail-composition skill instead)
- The organism/reference is not mouse GRCm38/mm10 and no equivalent Ensembl annotation is available for gene assignment

## Inputs

- Tail-trimmed clean CCS reads in FASTA (sample.out.fasta)
- Mus_musculus.GRCm38.dna.toplevel.chromosome.fa (mm10/GRCm38 reference genome)
- Ensembl Mus musculus GRCm38 genome annotation (release 92)
- CCS BAM file (movieX.ccs.bam) for pass-number collection

## Outputs

- Per-gene poly(A) site-count distribution (3511 genes with 2 sites, 762 with 3 sites, 220 with >3 sites expected)
- APA event table (apa_events.tsv)
- Per-read polyadenylation site calls from TAPIS

## How to apply

Align tail-trimmed clean CCS reads (sample.out.fasta from trim.py) to the Mus_musculus.GRCm38.dna.toplevel.chromosome.fa reference with GMAP. Ignore alignments carrying a supplementary alignment tag and discard CCSs assigned to multiple genes. Extract 3'-end poly(A) tail clips passing QC: clipped sequence >=15 nt containing at least five continuous adenosines, non-A residues <50% and <20 in number. Run the TAPIS pipeline via run_tapis.py on the aligned reads to call poly(A) sites per read, then assign sites to genes with the Ensembl Mus musculus GRCm38 (release 92) annotation and count distinct sites per gene. Correct application reproduces 3511 genes with two poly(A) sites (1 APA), 762 genes with three sites (2 APAs), and 220 genes with more than three sites (>=3 APAs) in mouse GV oocytes.

## Related tools

- **GMAP** (Aligns tail-trimmed clean CCS reads to the mm10/GRCm38 mouse reference genome before APA detection)
- **TAPIS pipeline (run_tapis.py)** (Detects polyadenylation sites per aligned read and feeds per-gene APA site counting) — github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **minimap2 (v2.15-r905)** (Alternative splice-aware aligner used to build the Mus_musculus.mmi index for the PAIso-Seq pipeline)

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- Per-gene site-count distribution reproduces 3511 genes with two poly(A) sites, 762 genes with three sites, and 220 genes with more than three sites
- apa_events.tsv is produced with per-gene APA site assignments against Ensembl GRCm38 release 92
- All retained CCS reads pass the QC gate: clipped tail >=15 nt with >=5 continuous adenosines, non-A residues <50% and <20 in count
- No CCS read is retained if assigned to multiple genes, and alignments with a supplementary alignment tag are excluded
- Alignment runs against Mus_musculus.GRCm38.dna.toplevel.chromosome.fa complete without supplementary-tagged alignments entering site calling

## Limitations

- CCSs assigned to multiple genes are discarded, reducing yield for genes with ambiguous mapping
- PAIso−seq cannot accurately quantify non-A modifications at the very last 3′-end base, which can bias terminal site composition
- Current PacBio sequencing cost (about 5 dollars per 1-K reads) limits achievable coverage, and replicate correlation is expected to improve only with deeper sequencing
- Requires pass number >=10 for reliable base calling, excluding low-pass CCS reads

## Evidence

- [results] Compared with annotated poly(A) sites in Ensembl mouse genome annotation (mm10, version 92): [evidence span withheld — anchor status: not_found]
- [methods] TAPIS QC thresholds: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [discussion] Alignment filtering: "CCSs assigned to multiple genes were discarded."
- [discussion] TAPIS pipeline usage: "aligned to mouse reference genome (mm10) by using GMAP; APA events were then analyzed by using run_tapis.py script"
- [readme] README run.pl invocation: "perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread  10 --pass ccs.pass.txt"
