---
name: alternative-polyadenylation-event-quantification
description: Use when you have tail-trimmed PacBio CCS reads from full-length cDNA
  (e.g., PAIso−seq of mouse GV oocytes) and need to count how many poly(A) sites each
  gene uses and quantify the per-gene APA site distribution.
when_to_use_negative:
- Input reads are short Illumina reads without full-length cDNA and poly(A) tails
  — tail clipping and TAPIS poly(A) site detection cannot be applied
- The goal is quantifying poly(A) tail length or non-A residues within tails rather
  than counting distinct polyadenylation sites per gene
- CCS reads lack the ≥15 nt poly(A) clip with ≥5 continuous adenosines required for
  site determination
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: GMAP
  role: Aligns tail-trimmed clean CCS reads to the mouse mm10 reference genome for
    downstream APA detection
  repo: https://github.com/PacificBiosciences/IsoSeq3
- name: TAPIS pipeline (run_tapis.py)
  role: Detects polyadenylation sites per aligned read that are then assigned to genes
    to count APA events
  repo: github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- name: minimap2 (v2.15-r905)
  role: Builds the splice-aware index (-x splice -d Mus_musculus.mmi) of the GRCm38/mm10
    reference used by the pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13228-9
    title: Poly(A) inclusive RNA isoform sequencing (PAIso−seq) reveals wide-spread
      non-adenosine residues within RNA poly(A) tails
  source_content_hash: sha256:a121897b90be7818c94efc92b5f73285b2004fe79d976a09be95da1ebb05f0d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/IsoSeq3
  tools:
  - GMAP
  - TAPIS pipeline (run_tapis.py)
  - minimap2 (v2.15-r905)
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# alternative-polyadenylation-event-quantification

## Summary

Detect per-gene alternative polyadenylation (APA) events from full-length PAIso−seq CCS reads by mapping with GMAP to mm10, extracting qualifying poly(A) tail clips, and running the TAPIS pipeline (run_tapis.py) to count distinct poly(A) sites per gene against Ensembl GRCm38 release-92 annotation.

## When to use

Use when you have tail-trimmed PacBio CCS reads from full-length cDNA (e.g., PAIso−seq of mouse GV oocytes) and need to count how many poly(A) sites each gene uses and quantify the per-gene APA site distribution.

## When NOT to use

- Input reads are short Illumina reads without full-length cDNA and poly(A) tails — tail clipping and TAPIS poly(A) site detection cannot be applied
- The goal is quantifying poly(A) tail length or non-A residues within tails rather than counting distinct polyadenylation sites per gene
- CCS reads lack the ≥15 nt poly(A) clip with ≥5 continuous adenosines required for site determination

## Inputs

- Tail-trimmed clean CCS reads (sample.out.fasta) from PAIso−seq
- CCS BAM file (movieX.ccs.bam) for pass-number collection (ccs.pass.txt)
- minimap2 splice index of the mm10 reference (Mus_musculus.mmi built from Mus_musculus.GRCm38.dna.toplevel.chromosome.fa)
- Ensembl Mus musculus GRCm38 genome and annotation (release 92)

## Outputs

- Per-gene poly(A) site counts and APA event distribution (e.g., 3511 genes with 2 sites, 762 with 3, 220 with >3)
- APA event table (apa_events.tsv)

## How to apply

Align tail-trimmed clean CCS reads (sample.out.fasta) to Mus_musculus.GRCm38.dna.toplevel.chromosome.fa with GMAP (minimap2 v2.15-r905 with -x splice was also used to build the reference index), then filter alignments by ignoring those carrying a supplementary alignment tag and discarding CCSs assigned to multiple genes. Extract 3'-end poly(A) tail clips only when the clipped sequence is ≥15 nt and contains at least five continuous adenosines, with non-A residues <50% and <20 in number. Run the TAPIS pipeline via run_tapis.pl on the aligned reads to determine poly(A) sites per read, then assign sites to genes using Ensembl Mus musculus GRCm38 (release 92) annotation and count distinct sites per gene. The expected distribution in mouse GV oocytes is 3511 genes with two poly(A) sites (1 APA), 762 genes with three sites (2 APAs), and 220 genes with more than three sites.

## Related tools

- **GMAP** (Aligns tail-trimmed clean CCS reads to the mouse mm10 reference genome for downstream APA detection) — https://github.com/PacificBiosciences/IsoSeq3
- **TAPIS pipeline (run_tapis.py)** (Detects polyadenylation sites per aligned read that are then assigned to genes to count APA events) — github.com/niehu2018/GV_oocyte_PAIsoSeqAnalysis
- **minimap2 (v2.15-r905)** (Builds the splice-aware index (-x splice -d Mus_musculus.mmi) of the GRCm38/mm10 reference used by the pipeline)

## Examples

```
perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread 10 --pass ccs.pass.txt &>run.log
```

## Evaluation signals

- Per-gene site-count distribution reproduces 3511 genes with two polyadenylation sites (1 APA), 762 genes with three sites (2 APAs), and 220 genes with more than three sites
- All retained alignments come from CCSs mapping to a single gene, and alignments with a supplementary alignment tag are absent
- Every poly(A) site derives from a clipped sequence ≥15 nt containing at least five continuous adenosines, with non-A residues <50% and <20
- The APA event table (apa_events.tsv) is produced with gene assignments consistent with Ensembl GRCm38 release-92 annotation
- CCS pass-number filtering (pass ≥10 for base-calling accuracy) is applied before site calling

## Limitations

- CCSs assigned to multiple genes are discarded, reducing read throughput for ambiguous or overlapping genes
- PacBio sequencing cost (~5 dollars per 1-K reads) limits achievable coverage; replicate correlation of APA detection would improve with deeper sequencing
- The pipeline depends on Ensembl release-92 GRCm38 annotation; results may shift with alternative annotations or PolyASite cluster versions (e.g., GRCm38-96)

## Evidence

- [results] Compared with annotated poly(A) sites in Ensembl mouse genome annotation (mm10, version 92), PAIso−seq identified 3511 genes with two polyadenylation sites (1 APA site), 762 genes with three polyadenylation sites: [evidence span withheld — anchor status: not_found]
- [methods] the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines: "the length of the clipped sequence was no <15 nt; (2) the clipped sequence contained at least five continuous adenosines"
- [discussion] CCSs assigned to multiple genes were discarded.: "CCSs assigned to multiple genes were discarded."
- [readme] minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa: "minimap2 -x splice -t 20 -d Mus_musculus.mmi Mus_musculus.GRCm38.dna.toplevel.chromosome.fa"
- [readme] perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi: "perl run.pl --ccs sample.out.fasta --sample sampleName --species mm10 --minimap2_index Mus_musculus.mmi --minimap2_thread  10 --pass ccs.pass.txt"
