---
name: rd-normalization-to-autosomal-baseline
description: Use when you need to assign a copy-number genotype to a specific genomic
  region (e.g. 12:11396601-11436500) from an existing CNVpytor .pytor file, rather
  than calling CNVs genome-wide.
when_to_use_negative:
- You want genome-wide CNV discovery/calling rather than genotyping a pre-specified
  region — use the -rd/-his/-partition/-call workflow instead.
- No .pytor archive with RD signal exists yet; RD must first be parsed from the BAM
  with cnvpytor -root file.pytor -rd file.bam.
- The region is on a non-autosomal contig — normalization divides by mean autosomal
  RD, so sex-chromosome or mitochondrial regions need a different baseline.
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: CNVpytor
  role: Runs the -genotype command (optionally with -a and -rd_use_mask) that computes
    region copy number as mean regional RD / mean autosomal RD scaled by 2, plus BAF-based
    extended statistics
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor whose RD binning/mean-shift/partitioning core engine CNVpytor
    inherits; context for the RD signal being genotyped
  repo: https://github.com/abyzovlab/CNVpytor
- name: Python
  role: Implementation language of CNVpytor; can drive genotype calculations programmatically
    via the CNVpytor library
  repo: https://github.com/abyzovlab/CNVpytor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:ba37c8d51b6985e7766fa982555d131f489fa3994e158c9c630c9d2b59320896
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Python
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# rd-normalization-to-autosomal-baseline

## Summary

Genotypes the copy number of a user-supplied genomic region in CNVpytor by computing the mean read depth within the region, dividing it by the mean autosomal read depth, and scaling by 2, with optional extended statistics (e-value, q0, pN, BAF level, homozygous/heterozygous counts, BAF P-value) and P-mask-filtered RD signal.

## When to use

Use when you need to assign a copy-number genotype to a specific genomic region (e.g. 12:11396601-11436500) from an existing CNVpytor .pytor file, rather than calling CNVs genome-wide.

## When NOT to use

- You want genome-wide CNV discovery/calling rather than genotyping a pre-specified region — use the -rd/-his/-partition/-call workflow instead.
- No .pytor archive with RD signal exists yet; RD must first be parsed from the BAM with cnvpytor -root file.pytor -rd file.bam.
- The region is on a non-autosomal contig — normalization divides by mean autosomal RD, so sex-chromosome or mitochondrial regions need a different baseline.

## Inputs

- Existing CNVpytor .pytor (HDF5) archive with imported RD signal (from BAM) and optionally SNP/BAF data from VCF
- Target genomic region coordinate in chr:start-end format (e.g. 12:11396601-11436500)
- Bin sizes for RD calculation (e.g. 10000 and 100000)

## Outputs

- Region-level genotype record containing the region coordinate and copy-number estimate (two values at the two bin sizes, e.g. 1.933261 and 1.937531)
- With -a: extended fields — RD e-value, q0, pN, maximum-likelihood BAF level, homozygous SNP count, heterozygous SNP count, and BAF P-value
- With -rd_use_mask: genotype recomputed from P-filtered (1000 Genome Project strict mask) RD signal

## How to apply

Load the read-depth data from the existing file.pytor archive, then run the CNVpytor genotype command on the requested region using bin sizes 10000 and 100000. Add the -a flag to extend each record with the binned-RD Gaussian-tail e-value, q0, pN, maximum-likelihood BAF level, homozygous and heterozygous variant counts, and the BAF-based P-value. Optionally re-run with -rd_use_mask so the calculation uses the P-filtered (1000 Genome Project strict mask) RD signal, which removes non-P bases that harbor false-positive heterozygous SNPs. The copy number is computed as the mean RD within the region divided by the mean autosomal RD scaled by 2, using first/last-bin overlap weighting. Write the region-level genotype statistics to a genotype_record and validate the schema before reuse.

## Related tools

- **CNVpytor** (Runs the -genotype command (optionally with -a and -rd_use_mask) that computes region copy number as mean regional RD / mean autosomal RD scaled by 2, plus BAF-based extended statistics) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor whose RD binning/mean-shift/partitioning core engine CNVpytor inherits; context for the RD signal being genotyped) — https://github.com/abyzovlab/CNVpytor
- **Python** (Implementation language of CNVpytor; can drive genotype calculations programmatically via the CNVpytor library) — https://github.com/abyzovlab/CNVpytor

## Examples

```
cnvpytor -root file.pytor -genotype 10000 100000 -a 12:11396601-11436500
```

## Evaluation signals

- Output record contains the queried region coordinate and a copy-number estimate at both requested bin sizes (e.g. two values like 1.933261 and 1.937531 for 12:11396601-11436500).
- Copy-number estimate is plausibly near 2 for an expected diploid region.
- With -a, the record includes all extended fields: e-value, q0, pN, maximum-likelihood BAF level, homozygous and heterozygous counts, and BAF P-value, matching the columns CNVpytor prints.
- With -rd_use_mask, the genotype is recomputed from the P-filtered (1000 Genome Project strict mask) RD signal and reported separately (e.g. 1:800000-900000 → 2.3012).
- The two bin-size genotypes (10000 vs 100000) are consistent in copy-number state (same approximate level, e.g. both ~1.9).

## Limitations

- Genotyping requires pre-computed RD in the .pytor file; RD must have been imported from BAM beforehand.
- The P-mask variant of genotyping relies on the 1000 Genomes Project strict mask, where only ~74% of bases are marked passed (P).
- Genotype is derived from RD ratio only; BAF provides supporting evidence (max-likelihood BAF level and P-value) but the copy number itself is the autosomal-normalized mean RD scaled by 2.

## Evidence

- [discussion] Copy number formula: "The copy number of a provided genomic region is calculated as a mean RD within the region divided by mean autosomal RD scaled by 2."
- [readme] Genotype command output: "> cnvpytor -root file.pytor -genotype 10000 100000
12:11396601-11436500
12:11396601-11436500	1.933261	1.937531"
- [readme] Extended genotype mode: "> cnvpytor -root file.pytor -genotype 10000 100000 -a
12:11396601-11436500
12:11396601-11436500    2.0152  1.629621e+04    9.670589e+08    0.0000  0.0000  4156900 1.0000  50      4       0.0000"
- [methods] P-mask filtered genotyping: "Genotyping using P filtered (1000 Genome Project strict mask) RD signal:
> cnvpytor -root file.pytor -genotype 10000 100000 -a -rd_use_mask"
