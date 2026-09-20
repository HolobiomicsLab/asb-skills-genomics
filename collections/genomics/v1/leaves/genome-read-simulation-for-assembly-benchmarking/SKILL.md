---
name: genome-read-simulation-for-assembly-benchmarking
description: Use when you need ground-truth error loci to measure assembly-evaluator
  recall/precision/F1 — e.g., benchmarking CRAQ CRE/CSE calls against QUAST-LG, Inspector,
  or Merqury on simulated data.
when_to_use_negative:
- You need to evaluate assemblies of species with higher ploidy levels — the article
  states evaluation of polyploid genomes may not be as accurate as the benchmarked
  diploid cases
- You lack a known reference genome, so simulated ground-truth error loci cannot be
  defined or matched against evaluator calls
- Your goal is quality assessment of a real draft assembly (use CRAQ directly for
  CRE/CSE/AQI reporting rather than the simulation benchmark)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: CRAQ
  role: Reference-free evaluator whose CRE/CSE calls are benchmarked against the simulated
    error ground truth; runs on Minimap2 alignments filtered with samtools
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: PBSIM
  role: Simulates HiFi-like long reads from hg38 and hg38_sim1 (--depth 40 --method
    qshmm --length-mean 10000 --length-sd 2000 --accuracy-min 0.95)
- name: Wgsim
  role: Simulates Illumina-like paired-end 150 bp reads (-e 0.0001 -r 0.0001 -R 0.0001
    -s 1 -1 150 -2 150) used for Merqury and short-read CRAQ mapping
- name: Minimap2
  role: Maps simulated reads back to hg38_sim2 using '-ax sr' (short reads) and '-ax
    map-hifi' (HiFi long reads)
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments before CRAQ with 'samtools view -F 1796 -q 20' (removes
    unmapped/secondary/QC-fail/duplicate and MAPQ<20)
- name: QUAST
  role: Baseline evaluator run against the true hg38 reference on the simulated assembly
- name: Inspector
  role: Baseline evaluator run with the simulated long reads
- name: Merqury
  role: k-mer-based baseline evaluator (k=21) run with the simulated Illumina reads;
    errors validated by boundary or 21-bp flanking overlap with simulated loci
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:a207af306846c20c40d01a7fb1304ab903273be62460d8b7792145ae8d8d6b8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Inspector
  - Merqury
  - Minimap2
  - PBSIM
  - QUAST
  - SAMtools
  - Wgsim
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Genome read simulation for assembly benchmarking

## Summary

Construct a simulated benchmark genome (e.g., hg38_sim1/hg38_sim2 from hg38 GCF_000001405.26) by embedding known heterozygous variants, regional/structural errors, and repeat errors, then simulate matching short- and long-read datasets (PBSIM, Wgsim) so that error-detection recall, precision, and F1 of evaluators such as CRAQ, QUAST-LG, Inspector, and Merqury can be computed against ground-truth loci (Table 1).

## When to use

Use when you need ground-truth error loci to measure assembly-evaluator recall/precision/F1 — e.g., benchmarking CRAQ CRE/CSE calls against QUAST-LG, Inspector, or Merqury on simulated data.

## When NOT to use

- You need to evaluate assemblies of species with higher ploidy levels — the article states evaluation of polyploid genomes may not be as accurate as the benchmarked diploid cases
- You lack a known reference genome, so simulated ground-truth error loci cannot be defined or matched against evaluator calls
- Your goal is quality assessment of a real draft assembly (use CRAQ directly for CRE/CSE/AQI reporting rather than the simulation benchmark)

## Inputs

- Human reference genome hg38 (GCF_000001405.26, 22 autosomes + X, FASTA)
- RepeatBrowser2020 hg38 repeat database (hgdownload.soe.ucsc.edu/hubs/RepeatBrowser2020/hg38/)
- Simulated error-annotated assemblies hg38_sim1 (11,000 heterozygous variants) and hg38_sim2 (7,000 regional/structural + 1,200 repeat errors)

## Outputs

- Simulated HiFi-like long reads (PBSIM) and Illumina-like paired-end 150 bp reads (Wgsim) from hg38 and hg38_sim1
- Per-evaluator error call sets matched to simulated loci with recall, precision, and F1 per error class (benchmark table analogous to Table 1: CRAQ CRE F1 97.463%, CSE F1 97.067%, Inspector CRE F1 ~96% with CSE recall 28%, Merqury F1 87.7%)

## How to apply

Start from hg38 (GCF_000001405.26, 22 autosomes + X), split into contigs at 'N' bases and drop fragments <500 kb. Build hg38_sim1 by embedding 11,000 heterozygous variants (10,000 small indels + 1000 structural variants), and hg38_sim2 by introducing 7,000 regional/structural errors (6000 regional indels, 200 fragment indels, 400 contig misjoins, 400 inversions) plus 1,200 repeat errors (1100 small repeat collapses/extensions, 100 large fragment repeats) selected from the RepeatBrowser2020 hg38 repeat database. Simulate HiFi-like reads with PBSIM (--depth 40 --method qshmm --length-mean 10000 --length-sd 2000 --accuracy-min 0.95) and Illumina-like reads with Wgsim (-e 0.0001 -r 0.0001 -R 0.0001 -s 1 -1 150 -2 150). Map reads to hg38_sim2 with Minimap2 ('-ax sr' for short reads, '-ax map-hifi' for HiFi), filter with samtools view '-F 1796 -q 20', and run CRAQ to call CREs/CSEs; run QUAST-LG (against true hg38), Inspector (long reads), and Merqury (k=21, Illumina reads) on the same inputs. Count an evaluator's error as a true positive when its reported locus overlaps a simulated locus (Merqury: boundary or 21-bp flanking overlap; CRAQ: breakpoint within SV boundary or 50 bp adjacent) and compute recall, precision, and F1 per error class.

## Related tools

- **CRAQ** (Reference-free evaluator whose CRE/CSE calls are benchmarked against the simulated error ground truth; runs on Minimap2 alignments filtered with samtools) — https://github.com/JiaoLaboratory/CRAQ
- **PBSIM** (Simulates HiFi-like long reads from hg38 and hg38_sim1 (--depth 40 --method qshmm --length-mean 10000 --length-sd 2000 --accuracy-min 0.95))
- **Wgsim** (Simulates Illumina-like paired-end 150 bp reads (-e 0.0001 -r 0.0001 -R 0.0001 -s 1 -1 150 -2 150) used for Merqury and short-read CRAQ mapping)
- **Minimap2** (Maps simulated reads back to hg38_sim2 using '-ax sr' (short reads) and '-ax map-hifi' (HiFi long reads)) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments before CRAQ with 'samtools view -F 1796 -q 20' (removes unmapped/secondary/QC-fail/duplicate and MAPQ<20))
- **QUAST** (Baseline evaluator run against the true hg38 reference on the simulated assembly)
- **Inspector** (Baseline evaluator run with the simulated long reads)
- **Merqury** (k-mer-based baseline evaluator (k=21) run with the simulated Illumina reads; errors validated by boundary or 21-bp flanking overlap with simulated loci)

## Examples

```
PBSIM --depth 40 --method qshmm --length-mean 10000 --length-sd 2000 --accuracy-min 0.95 hg38_sim1.fa; wgsim -e 0.0001 -r 0.0001 -R 0.0001 -s 1 -1 150 -2 150 hg38.fa rd1.fq rd2.fq
```

## Evaluation signals

- CRAQ achieves F1 >97% on simulated CRE/CSE errors — CRE F1 97.463% and CSE F1 97.067% (accept ±2 percentage points)
- Inspector reproduces CRE F1 ~96% with CSE recall 28%, and Merqury reproduces F1 87.7% on the same inputs
- Simulated hg38_sim2 contains exactly 7,000 regional/structural errors (6000 regional indels, 200 fragment indels, 400 contig misjoins, 400 inversions) and 1,200 repeat errors (1100 small, 100 large), and hg38_sim1 contains 11,000 heterozygous variants (10,000 small indels + 1000 SVs)
- CRAQ correctly distinguishes the 11,000 embedded heterozygous loci from true errors (clipping ratio within default heterozygous cutoff h = 0.4–0.6, error breakpoint ratio exceeding f = 0.75)
- Recall/precision/F1 are computed only where reported loci satisfy the overlap rule (Merqury: boundary or 21-bp flank overlap; CRAQ: breakpoint within SV boundary or 50 bp adjacent)

## Limitations

- CRAQ accuracy for polyploid genomes is not established: evaluation of species with higher ploidy levels may not be as accurate as the benchmarked cases
- Error breakpoint ratios can fall below 100% in practice due to sequencing errors or inaccurate read mapping, complicating classification
- Key benchmarking results (Table 1, Supplementary Fig. 13) are deferred to supplementary materials; sequencing/assembly data sources appear only in Supplementary Data 1

## Evidence

- [discussion] Simulation design grounding: "we simulated structural and small-scale local assembly errors, as well as heterozygous regions, in the human reference genome hg38"
- [discussion] Read mapping parameters: "the ‘-ax sr’ and ‘-ax map-pb/hifi/ont’ options were employed for genomic short-read and different types of long-read mapping"
- [discussion] Alignment filtering before evaluator runs: [evidence span withheld — anchor status: not_found]
- [readme] What CRAQ reports on the simulated benchmark: "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs)"
- [discussion] Heterozygous-vs-error decision rule: "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
