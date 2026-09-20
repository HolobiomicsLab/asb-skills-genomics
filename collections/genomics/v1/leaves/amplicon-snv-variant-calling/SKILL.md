---
name: amplicon-snv-variant-calling
description: Use when targeted nanopore amplicon reads (e.g. CYP2D6, HLA-A, HLA-B
  loci) must be mapped to a reference and SNVs called, and you need to quantify how
  the choice of mapper affects variant-calling precision and sensitivity against a
  truth set.
when_to_use_negative:
- Input reads are short Illumina reads — GraphMap and this benchmark target long,
  error-prone third-generation (MinION 1d/2d) reads
- Whole-genome nanopore resequencing where memory is limited — GraphMap's index requires
  large-memory machines (~100 GB for the human genome), which is disproportionate
  for small targeted amplicon workloads
- Structural-variant or indel-focused calling (100 bp–4 kbp events) — this skill is
  scoped to single-nucleotide variants
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: GraphMap
  role: Primary mapper whose alignments are used as the starting point for LoFreq
    and marginCaller SNV calling
  repo: https://github.com/isovic/graphmap
- name: BWA-MEM
  role: Comparison mapper for benchmarking SNV calling precision/sensitivity
- name: LAST
  role: Comparison mapper for benchmarking SNV calling precision/sensitivity
- name: BLASR
  role: Comparison mapper for benchmarking SNV calling precision/sensitivity
- name: DALIGNER
  role: Comparison mapper whose alignments feed LoFreq calling and serve as the 'next
    best mapper' baseline
- name: LoFreq
  role: SNV caller applied to GraphMap and DALIGNER alignments
- name: marginCaller
  role: Nanopore-tailored SNV caller (from marginAlign) applied to GraphMap alignments
  repo: https://github.com/isovic/marginAlign
- name: graphmap-reproduce
  role: Reproduction scripts that install evaluation dependencies (PyVCF, Tabix, VCFtools,
    LoFreq, samscripts, Mutatrix, Bamsurgeon) and run the real-data benchmark
  repo: https://github.com/isovic/graphmap-reproduce
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:ff8d874295aafa57d89e42f3de680afed426af1854ec3997ca0a317aad15cf28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - BLASR
  - BWA-MEM
  - DALIGNER
  - GCC/G++
  - GraphMap
  - LAST
  - LoFreq
  - graphmap-reproduce
  - marginAlign
  - marginCaller
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# amplicon-snv-variant-calling

## Summary

Benchmarking pipeline for single-nucleotide-variant (SNV) calling on targeted Oxford Nanopore amplicon sequencing of a diploid human genome, in which reads are mapped with multiple long-read mappers (GraphMap, BWA-MEM, LAST, BLASR, DALIGNER), variants are called with LoFreq and marginCaller, and per-mapper precision and sensitivity are computed against an amplicon truth set.

## When to use

Use when targeted nanopore amplicon reads (e.g. CYP2D6, HLA-A, HLA-B loci) must be mapped to a reference and SNVs called, and you need to quantify how the choice of mapper affects variant-calling precision and sensitivity against a truth set.

## When NOT to use

- Input reads are short Illumina reads — GraphMap and this benchmark target long, error-prone third-generation (MinION 1d/2d) reads
- Whole-genome nanopore resequencing where memory is limited — GraphMap's index requires large-memory machines (~100 GB for the human genome), which is disproportionate for small targeted amplicon workloads
- Structural-variant or indel-focused calling (100 bp–4 kbp events) — this skill is scoped to single-nucleotide variants

## Inputs

- Targeted Oxford Nanopore amplicon FASTQ reads (Ammar et al. CYP2D6, HLA-A, HLA-B amplicons)
- hg19 chr6 and chr22 reference sequences (FASTA)
- Ammar et al. amplicon SNV truth set (variant call format)
- Built mapper executables: GraphMap, BWA-MEM, LAST, BLASR, DALIGNER
- SNV callers: LoFreq and marginCaller (marginAlign package)

## Outputs

- One SAM/BAM alignment file per mapper per reference
- VCF SNV call sets from LoFreq (GraphMap and DALIGNER alignments) and marginCaller (GraphMap alignments)
- snv_precision_sensitivity.csv with per-mapper/caller precision and sensitivity
- Validated benchmark summary matching reported figures (GraphMap 96% precision, DALIGNER 93% precision, 15% sensitivity delta, marginCaller 36% precision)

## How to apply

Build GraphMap from source by initializing submodules with `make modules` then compiling with `make` (GCC/G++ >= 4.7; override compiler via GCC=/usr/local/bin/g++ make if needed). Map the Ammar et al. CYP2D6, HLA-A and HLA-B nanopore amplicon reads to hg19 chr6 and chr22 reference sequences with GraphMap, BWA-MEM, LAST, BLASR and DALIGNER, producing one alignment file per mapper. Call SNVs from the GraphMap and DALIGNER alignments using LoFreq, and call SNVs from the GraphMap alignments using marginCaller from the marginAlign package. Compare each caller's variant set against the Ammar et al. amplicon truth set and compute per-mapper precision and sensitivity. The rationale is that GraphMap's progressive candidate-alignment refinement and graph traversal produce mappings precise enough (>95%) that conventional variant callers succeed on nanopore data where they previously failed; the reproduction target is ~96% SNV precision for GraphMap with a 15% sensitivity gain over the next best mapper DALIGNER (93% precision), while marginCaller on GraphMap alignments reaches 36% precision.

## Related tools

- **GraphMap** (Primary mapper whose alignments are used as the starting point for LoFreq and marginCaller SNV calling) — https://github.com/isovic/graphmap
- **BWA-MEM** (Comparison mapper for benchmarking SNV calling precision/sensitivity)
- **LAST** (Comparison mapper for benchmarking SNV calling precision/sensitivity)
- **BLASR** (Comparison mapper for benchmarking SNV calling precision/sensitivity)
- **DALIGNER** (Comparison mapper whose alignments feed LoFreq calling and serve as the 'next best mapper' baseline)
- **LoFreq** (SNV caller applied to GraphMap and DALIGNER alignments)
- **marginCaller** (Nanopore-tailored SNV caller (from marginAlign) applied to GraphMap alignments) — https://github.com/isovic/marginAlign
- **graphmap-reproduce** (Reproduction scripts that install evaluation dependencies (PyVCF, Tabix, VCFtools, LoFreq, samscripts, Mutatrix, Bamsurgeon) and run the real-data benchmark) — https://github.com/isovic/graphmap-reproduce

## Examples

```
git clone https://github.com/isovic/graphmap.git; cd graphmap; make modules; make; ./bin/Linux-x64/graphmap align -r hg19_chr6.fa -d CYP2D6_reads.fastq -o graphmap_cyp2d6.sam
```

## Evaluation signals

- GraphMap SNV precision computed against the truth set should be ~96%
- DALIGNER-based SNV precision should be ~93%; GraphMap should show a 15% increase in sensitivity over DALIGNER
- marginCaller on GraphMap alignments should reach ~36% precision
- The computed values in snv_precision_sensitivity.csv must match the reported figures within tolerance
- GraphMap should map >95% of bases on MinION data, consistent with the paper's reported sensitivity

## Limitations

- GraphMap requires large-memory machines for mapping to large genomes (~100 GB for the human genome); the index is roughly ~14 x reference_size and peaks at ~42 GB for the human genome
- No custom nanopore-specific variant callers were available at the time; variant calling relied on existing tools, and marginCaller reaches only 36% precision even on GraphMap alignments
- After `git pull`, `make modules` must be run before `make` or the build fails (submodules not initialized)
- GraphMap's `--freq-percentile` seed filtering (default 0.99) slightly reduces sensitivity, though it plays a huge role when mapping to large references

## Evidence

- [abstract] GraphMap enables SNV calling on the human genome with 15% increased sensitivity over the next best mapper: "single-nucleotide variant calling on the human genome with increased sensitivity (15%) over the next best mapper"
- [methods] Build pipeline: submodules then compile: "For a Linux release version type:
make"
- [readme] Build prerequisite after git pull: "After running `git pull`, please `make modules` before `make`."
- [abstract] GraphMap precision basis for variant calling: "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [discussion] Memory limitation of GraphMap: "A drawback of the current implementation of GraphMap is the requirement of large-memory machines for mapping to large genomes (∼100 GB for the human genome)"
- [readme] Reproduction harness installs callers and evaluation tools: [evidence span withheld — anchor status: external_doc]
