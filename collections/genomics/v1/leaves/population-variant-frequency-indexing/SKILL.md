---
name: population-variant-frequency-indexing
description: Use when running miniSNV's candidate-finder on nanopore BAM alignments
  and you need the prebuilt known-variant index (from a 1000 Genomes population VCF)
  to classify pileup candidate loci as high- or low-quality before genotyping.
when_to_use_negative:
- The organism or population has no suitable population VCF available, since the index
  (1000 Genomes by default) is human-focused and built from population variation data
- You need de novo variant discovery without population priors — without --useindex
  the classification degrades and the 0.3 R_alt shortcut for known variants cannot
  be applied
- Input alignments are short reads rather than ONT long reads; miniSNV's pileup-and-phasing
  design is optimized for ONT error profiles and long-read spanning
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: miniSNV
  role: Builds and consumes the known common-variant index during its read-pileup
    candidate-finder stage to classify loci into high- and low-quality sets
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: 1000 genome project
  role: Source population VCF (default input) from which allele frequencies and wid_pos_ref_alt
    UIDs for the known common-variant index are derived
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:ab9bde6cd036afa61cd26a87f06e1d519d18872559fe5263e6a7b056b8abbb18
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - 1000 genome project
  - KSW2
  - WhatsHap
  - abPOA
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# population-variant-frequency-indexing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds and applies miniSNV's prebuilt index of known common variants (1000 Genomes population VCF, allele frequency > 5%) so that candidate SNV loci found in ONT read pileups can be stratified into high- and low-quality loci using prior population allele-frequency knowledge.

## When to use

Use when running miniSNV's candidate-finder on nanopore BAM alignments and you need the prebuilt known-variant index (from a 1000 Genomes population VCF) to classify pileup candidate loci as high- or low-quality before genotyping.

## When NOT to use

- The organism or population has no suitable population VCF available, since the index (1000 Genomes by default) is human-focused and built from population variation data
- You need de novo variant discovery without population priors — without --useindex the classification degrades and the 0.3 R_alt shortcut for known variants cannot be applied
- Input alignments are short reads rather than ONT long reads; miniSNV's pileup-and-phasing design is optimized for ONT error profiles and long-read spanning

## Inputs

- Population VCF of known variants (1000 Genomes project by default) for building the index
- Sorted, samtools-indexed ONT BAM alignment (e.g. HG002 k12 data, MAPQ >= 20 records only)
- Samtools-indexed FASTA reference (e.g. GRCh38)

## Outputs

- miniSNV index folder containing five index files of known common-variant UIDs (wid_pos_ref_alt) with per-allele population frequencies
- Stratified candidate loci with high-quality/low-quality labels feeding candidates.bed and downstream genotyping (final output pipeline_output.vcf.gz)

## How to apply

Build the miniSNV index offline by parsing the population VCF (1000 Genomes by default, keeping alleles with frequency above 5%) into five index files whose records are keyed by UIDs of the form wid_pos_ref_alt, one per alternative allele at each genome position. During candidate finding, miniSNV computes the alternative-allele ratio R_alt from the MAPQ>=20, baseQ>=13, read_Ratio 0.98-filtered pileup and looks up each candidate's (pos, ref, alt) UID in the index: a locus found in the index with R_alt > 0.3 is classified high-quality, while an unknown locus qualifies as high-quality only if R_alt > 0.7 and no other candidate lies within 10 bp. Any candidate in a low-complexity region (GIAB v3.3 stratifications) is demoted to low-quality, which routes it to the consensus-based haplotype-aware genotyping path instead of the binomial-model genotyper. Rationale: prior knowledge of common population variants accelerates and stabilizes identification of high-frequency variants, allowing the 0.3–0.7 R_alt threshold combination (selected via in-house experiments, Supplementary Table S1) to exploit population priors rather than a single uniform cutoff. At runtime the index is optional and must be activated explicitly with --useindex plus -i pointing at the five-file index folder.

## Related tools

- **miniSNV** (Builds and consumes the known common-variant index during its read-pileup candidate-finder stage to classify loci into high- and low-quality sets) — https://github.com/CuiMiao-HIT/miniSNV
- **1000 genome project** (Source population VCF (default input) from which allele frequencies and wid_pos_ref_alt UIDs for the known common-variant index are derived)

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG002_k12.bam --workDir WORKDIR --useindex -i miniSNV_index --human
```

## Evaluation signals

- Index folder contains exactly five files, as required by --useindex/-i in the README
- Every locus recorded in the index carries a UID of the form wid_pos_ref_alt corresponding to one specific alternative allele
- Candidate loci found in the index are promoted to high-quality only when R_alt > 0.3; unknown loci require R_alt > 0.7 plus no other candidate within 10 bp
- All candidate sites in low-complexity regions (GIAB v3.3 stratifications) are labeled low-quality regardless of index status
- Per the paper, the downstream callset on HG002 sup-datasets should reach F1-score and recall greater than 99.7% against the GIAB truth set

## Limitations

- Index accuracy depends on the population VCF: variants absent from 1000 Genomes fall back to the stricter unknown-locus thresholds (R_alt > 0.7, isolation within 10 bp)
- The 0.3–0.7 R_alt threshold combination was chosen from in-house experiments (Supplementary Table S1) and may need re-tuning for non-human or non-ONT data
- No discussion/changelog section is available in the source document, so operational caveats beyond methods/results are unreported

## Evidence

- [methods] High-quality classification thresholds: [evidence span withheld — anchor status: not_found]
