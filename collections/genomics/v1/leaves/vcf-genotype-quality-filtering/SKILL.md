---
name: vcf-genotype-quality-filtering
description: Use when a joint-genotyped SV VCF carries a GQ quality field and the
  goal is to trade call count for a higher concordant-call proportion, e.g. before
  pedigree or truth-set concordance comparison.
when_to_use_negative:
- The caller output has no FORMAT/GQ genotype-quality field (e.g. pbsv in this comparison
  was left unfiltered), so thresholding cannot be applied
- The goal is maximum sensitivity callset generation rather than a high-precision
  subset for concordance or truth-set benchmarking
- Variant calls are single-sample with no joint-genotyped pedigree structure to validate
  concordance against
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: sawfish
  role: Produces the joint-genotyped SV VCF whose FORMAT/GQ field is thresholded (GQ>=40)
    to raise pedigree concordance to 87.8-88%
  repo: https://github.com/PacificBiosciences/sawfish
- name: Sniffles2
  role: Comparator caller whose SV VCF is GQ-filtered at a lower threshold (GQ>=10,
    8348 SVs retained) for an apples-to-apples concordance comparison
- name: pbsv
  role: Comparator SV caller whose calls were left unfiltered in the concordance tabulation
  repo: https://github.com/PacificBiosciences/pbsv
- name: bcftools
  role: VCF filtering/view utility used in the paper's workflow for selecting VCF
    records
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:b42ec84822efb82aa407491085b88398ada92093cab91d9894584389e7a24a67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - Sniffles2
  - bcftools
  - pbmm2
  - pbsv
  - sawfish
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# vcf-genotype-quality-filtering

## Summary

Filters joint-genotyped structural variant VCFs by the FORMAT/GQ field to select high-confidence genotype calls, as used to elevate sawfish CEPH-1463 pedigree concordance from 81.2% to 87.8-88% while retaining thousands more concordant SVs than comparators.

## When to use

Use when a joint-genotyped SV VCF carries a GQ quality field and the goal is to trade call count for a higher concordant-call proportion, e.g. before pedigree or truth-set concordance comparison.

## When NOT to use

- The caller output has no FORMAT/GQ genotype-quality field (e.g. pbsv in this comparison was left unfiltered), so thresholding cannot be applied
- The goal is maximum sensitivity callset generation rather than a high-precision subset for concordance or truth-set benchmarking
- Variant calls are single-sample with no joint-genotyped pedigree structure to validate concordance against

## Inputs

- Multi-sample joint-genotyped SV VCF from sawfish joint-call (with FORMAT/GQ scores)
- SV VCFs from comparator callers (Sniffles2, pbsv) on the same mapped BAMs
- Known pedigree inheritance patterns (CEPH-1463 trio/family genotypes from Kronenberg et al. 2024)

## Outputs

- GQ-filtered high-quality SV call subset (e.g. sawfish at GQ>=40, Sniffles2 at GQ>=10)
- Summary table of per-caller total calls, pedigree-concordant counts, and concordance proportions (ceph1463_sv_genotype_concordance.tsv)

## How to apply

After sawfish joint-call produces the multi-sample VCF, apply a genotype-quality threshold on the FORMAT/GQ field to select the high-quality subset: the paper recomputed sawfish concordance at GQ>=40, yielding 87.8% (88%) concordant SVs, while Sniffles2 was filtered at the lower GQ>=10 (retaining 8348 SVs at 85.2% concordance) and pbsv was left unfiltered. The rationale is that sawfish's local-haplotype quality model scores each genotype, so thresholding recovers a purer callset; compare per-caller total calls, concordant counts, and concordance proportions side by side (Supplementary Table S5) and write the tabulation (e.g. ceph1463_sv_genotype_concordance.tsv). Calibrate thresholds per caller so call retention stays competitive — at GQ>=40 sawfish still called nearly 5000 more pedigree-concordant SVs than other callers.

## Related tools

- **sawfish** (Produces the joint-genotyped SV VCF whose FORMAT/GQ field is thresholded (GQ>=40) to raise pedigree concordance to 87.8-88%) — https://github.com/PacificBiosciences/sawfish
- **Sniffles2** (Comparator caller whose SV VCF is GQ-filtered at a lower threshold (GQ>=10, 8348 SVs retained) for an apples-to-apples concordance comparison)
- **pbsv** (Comparator SV caller whose calls were left unfiltered in the concordance tabulation) — https://github.com/PacificBiosciences/pbsv
- **bcftools** (VCF filtering/view utility used in the paper's workflow for selecting VCF records)

## Examples

```
bcftools view -i 'FORMAT/GQ>=40' -Oz -o sawfish.joint.gq40.vcf.gz sawfish.joint.vcf.gz
```

## Evaluation signals

- Sawfish calls at GQ>=40 should reach ~87.8% (reported as 88%) pedigree-concordant proportion on CEPH-1463, matching Supplementary Table S5
- The filtered sawfish callset should still retain nearly 5000 more pedigree-concordant SVs than unfiltered comparators (9160 more concordant calls unfiltered, 81.2% concordant)
- Sniffles2 at GQ>=10 should retain 8348 SVs at 85.2% concordance; pbsv stays unfiltered at 77.7%
- Summary table must contain, per caller, total calls, concordant-call counts, and concordance proportions with no missing samples
- Every retained record must carry a FORMAT/GQ value satisfying the applied threshold

## Limitations

- Thresholds are caller-specific: sawfish's GQ model is not calibrated against Sniffles2's, so the same cutoff is not portable across callers (sawfish GQ>=40 vs Sniffles2 GQ>=10 vs pbsv unfiltered)
- GQ filtering trades recall for precision — absolute concordant-call counts drop relative to the unfiltered 81.2% callset
- Pedigree concordance is only measurable where known inheritance patterns (e.g. CEPH-1463) exist; it is a proxy, not direct truth-set accuracy

## Evidence

- [abstract] Sawfish's quality model enables selection for an even higher proportion of concordant SVs (88%), while still calling nearly 5000 more pedigree-concordant SVs than other callers.: "Sawfish's quality model enables selection for an even higher proportion of concordant SVs (88%), while still calling nearly 5000 more pedigree-concordant SVs than other callers."
- [methods] Recompute sawfish high-quality concordance using the VCF GQ field at GQ>=40, and Sniffles2 concordance at GQ>=10, leaving pbsv unfiltered: [evidence span withheld — anchor status: not_found]
- [results] sawfish has over 9000 more pedigree-concordant calls than other state-of-the-art SV callers, with the highest proportion of concordant SVs (81%).: "sawfish has over 9000 more pedigree-concordant calls than other state-of-the-art SV callers, with the highest proportion of concordant SVs (81%)."
- [methods] The analysis uses pbmm2 1.13.1 for read mapping, and sawfish 0.12.9 for SV calling.: "The analysis uses pbmm2 1.13.1 for read mapping, and sawfish 0.12.9 for SV calling."
