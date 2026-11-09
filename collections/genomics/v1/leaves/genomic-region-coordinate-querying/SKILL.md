---
name: genomic-region-coordinate-querying
description: Use when the user names a specific chromosomal interval (e.g. 12:11396601-11436500)
  and asks for its copy number genotype or region-level RD/BAF statistics from an
  existing .pytor archive.
when_to_use_negative:
- The task is de novo CNV discovery across the whole genome — that requires the -rd/-his/-partition/-call
  pipeline, not region genotyping
- No .pytor archive exists yet; the region genotyper cannot parse raw BAM files directly
- The question concerns SNP-level variant calling rather than copy number of an interval
  — VCF import via -snp is a different workflow step
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: CNVpytor
  role: Provides the '-genotype' command that computes copy number for a supplied
    region from binned RD in the .pytor archive, with '-a' extended statistics and
    '-rd_use_mask' P-filtered RD
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor whose RD binning and mean-shift core engine underlies the RD signal
    used for region genotyping in CNVpytor
- name: Python
  role: Implementation language of CNVpytor; the .pytor archive is HDF5 and can also
    be accessed programmatically from Python
  repo: https://github.com/abyzovlab/CNVpytor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:1b1594ab93f3e96fb71446e5063a865637f07a88cd417b97418b890df22df0e4
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
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# genomic-region-coordinate-querying

## Summary

Genotypes a user-supplied genomic region with CNVpytor by computing copy number from read depth: mean RD within the region divided by mean autosomal RD and scaled by 2, optionally enriched with BAF/e-value statistics via the '-a' genotype mode.

## When to use

Use when the user names a specific chromosomal interval (e.g. 12:11396601-11436500) and asks for its copy number genotype or region-level RD/BAF statistics from an existing .pytor archive.

## When NOT to use

- The task is de novo CNV discovery across the whole genome — that requires the -rd/-his/-partition/-call pipeline, not region genotyping
- No .pytor archive exists yet; the region genotyper cannot parse raw BAM files directly
- The question concerns SNP-level variant calling rather than copy number of an interval — VCF import via -snp is a different workflow step

## Inputs

- Existing CNVpytor .pytor (HDF5) archive with RD signal computed at bin sizes 10000 and 100000
- User-supplied genomic region string in chromosome:start-end form (e.g. 12:11396601-11436500)
- Optional P-filtered (1000 Genome Project strict mask) annotation for -rd_use_mask mode

## Outputs

- Region-level genotype record containing the region coordinate and copy-number estimate (e.g. 1.933261)
- Extended statistics with '-a': e-value, q0, pN, maximum-likelihood BAF level, homozygous/heterozygous counts, and BAF P-value

## How to apply

Load the pre-built .pytor HDF5 archive (already containing binned RD signal, e.g. at bin sizes 10000 and 100000) and run the '-genotype' command with those bin sizes on the requested region string. Enable '-a' extended mode so the record additionally contains the binned-RD Gaussian tail e-value, q0, pN, maximum-likelihood BAF level, homozygous/heterozygous variant counts, and BAF-based P-value. Optionally add '-rd_use_mask' to recompute with RD signal filtered by the 1000 Genome Project strict P-mask, which improves RD accuracy in non-P regions. The copy number itself is calculated as mean RD within the region divided by mean autosomal RD scaled by 2, using first/last-bin overlap weighting, and the region-level statistics are written to a genotype record for downstream use.

## Related tools

- **CNVpytor** (Provides the '-genotype' command that computes copy number for a supplied region from binned RD in the .pytor archive, with '-a' extended statistics and '-rd_use_mask' P-filtered RD) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor whose RD binning and mean-shift core engine underlies the RD signal used for region genotyping in CNVpytor)
- **Python** (Implementation language of CNVpytor; the .pytor archive is HDF5 and can also be accessed programmatically from Python) — https://github.com/abyzovlab/CNVpytor

## Examples

```
cnvpytor -root file.pytor -genotype 10000 100000 -a
```

## Evaluation signals

- Output record contains the exact region coordinate string supplied by the user (e.g. 12:11396601-11436500)
- Copy-number estimate is plausible (near 2) for a known diploid region; README examples show 1.933261 and 2.0152
- With '-a', the record includes all extended fields: e-value, q0, pN, BAF level, homozygous/heterozygous counts, and BAF P-value
- Running with and without '-rd_use_mask' changes the estimate only modestly (e.g. 1.949186 vs 1.957068 for 22:20999401-21300400) and mask mode flags non-P content via q0/pN
- Two genotypes are reported per region (one per requested bin size, 10000 and 100000)

## Limitations

- Requires a pre-existing .pytor archive with the relevant bin sizes; it cannot genotype directly from BAM
- Somatic CNA calling remains a prototype; genotyping accuracy for cancer/mosaic copy-aberrant samples is limited and a more accurate approach is under development
- The P-mask option excludes ~22% of bases/variants (only ~74% of 1000 Genomes mask bases are marked passed), so mask-filtered RD can differ from unfiltered RD

## Evidence

- [discussion] Copy number computed from mean RD normalized to autosomal RD: "The copy number of a provided genomic region is calculated as a mean RD within the region divided by mean autosomal RD scaled by 2."
- [methods] P-filtered RD genotyping option: "Genotyping using P filtered (1000 Genome Project strict mask) RD signal:"
- [readme] Concrete genotype command from README: "> cnvpytor -root file.pytor -genotype 10000 100000 -a -rd_use_mask"
