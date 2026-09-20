---
name: vcf-manipulation-for-sv-curation
description: Use when a merged consensus SV call set (VCF) must be filtered — e.g.
  by caller support, mapping-quality regions, or repeat/LTR overlap — prior to manual
  read-alignment curation.
when_to_use_negative:
- When the goal is maximum sensitivity rather than high specificity — the study deliberately
  prioritized specificity and states that not all SVs were detected
- When the input VCF has not been merged into a consensus set yet (single-caller output);
  apply SURVIVOR merge with the >=2-caller rule first to obtain the 315-call-style
  consensus set
- When individual variants rather than whole call sets are being re-examined — the
  study excluded translocations and inversions from genotyping re-examination because
  they were too challenging to genotype accurately
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SURVIVOR
  role: Provides the filter module (VCF filtering by size/regions), the merge module
    for consensus generation, and the bincov module that bins coverage into a bed
    file to filter SVs in low-MQ regions
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:8e987d5665a4117e6106719c76f1a3e52d173d84b7f923f2ad0ca0d31ec58b14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - SURVIVOR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# vcf-manipulation-for-sv-curation

## Summary

Rigorous filtering of a consensus structural-variant VCF using SURVIVOR's filter and bincov modules, as applied to the 315-call S. pombe consensus set before manual inspection. Removes low paired-end support calls (<=10 reads), breakpoints in MQ=0 regions, and calls overlapping annotated LTRs.

## When to use

Use when a merged consensus SV call set (VCF) must be filtered — e.g. by caller support, mapping-quality regions, or repeat/LTR overlap — prior to manual read-alignment curation.

## When NOT to use

- When the goal is maximum sensitivity rather than high specificity — the study deliberately prioritized specificity and states that not all SVs were detected
- When the input VCF has not been merged into a consensus set yet (single-caller output); apply SURVIVOR merge with the >=2-caller rule first to obtain the 315-call-style consensus set
- When individual variants rather than whole call sets are being re-examined — the study excluded translocations and inversions from genotyping re-examination because they were too challenging to genotype accurately

## Inputs

- Consensus SV call set in VCF format (the 315-call set produced by SURVIVOR merge of Delly/Lumpy/Pindel/cn.MOPs outputs, accepted by >=2 callers)
- Bed file of low mapping-quality (MQ=0) regions derived from the bincov coverage-vector binning step
- Annotation of previously characterized long terminal repeats (LTRs) / low complexity regions

## Outputs

- Filtered SV VCF (e.g. filtered_svs.vcf) containing no calls with paired-end support <=10, no breakpoint in MQ=0 regions, and no breakpoint overlapping an annotated LTR
- VCF ready for downstream manual inspection of read alignments in multiple strains

## How to apply

Load the consensus SV VCF produced by the SURVIVOR merge step — in this study the 315-call set accepted by at least two of the four callers (Delly, Lumpy, Pindel, cn.MOPs). Run the SURVIVOR filter module to drop SVs with paired-end support at or below 10 reads, since the study targeted high specificity over sensitivity. Use SURVIVOR's bincov step to bin the coverage vector into a bed file marking low mapping-quality (MQ=0) regions, and filter out any SV whose breakpoints fall in those regions. Intersect SV breakpoints with the annotation of previously characterized long terminal repeats (LTRs) and remove calls overlapping either an LTR or a low-complexity region. Write the surviving calls to a filtered VCF (e.g. filtered_svs.vcf) that becomes the input for downstream manual inspection of read alignments; translocations and inversions retained at this stage were later excluded from genotyping re-examination because they were harder to genotype accurately.

## Related tools

- **SURVIVOR** (Provides the filter module (VCF filtering by size/regions), the merge module for consensus generation, and the bincov module that bins coverage into a bed file to filter SVs in low-MQ regions) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
./SURVIVOR filter consensus_315_svs.vcf filtered_svs.vcf
```

## Evaluation signals

- The filtered VCF parses as valid VCF and every retained call had support from at least two callers (starting point was the 315-call consensus set)
- No call in the output has paired-end support <=10 reads
- No breakpoint falls in an MQ=0 region (verify against the bincov-generated bed file)
- No breakpoint overlaps a previously annotated LTR or low-complexity region
- The filtered set is a strict subset of the 315-call consensus set; subsequent manual vetting of read alignments in the article recovered 113 curated SVs

## Limitations

- Filtering prioritizes high specificity over high sensitivity, so real SVs absent from the filtered set are expected (the authors note they 'focused on high specificity rather than high sensitivity')
- Translocations and inversions that survive filtering are difficult to genotype accurately and were excluded from re-examination
- Failure to call SVs at this stage can overestimate SNP impact on traits and contribute to the missing-heritability problem in downstream GWAS

## Evidence

- [abstract] we further filtered the 315 variants by removing SV calls whose breakpoints overlapped with low complexity regions or any that corresponded to previously annotated long terminal repeats: "we further filtered the 315 variants by removing SV calls whose breakpoints overlapped with low complexity regions or any that corresponded to previously annotated long terminal repeats"
- [abstract] consensus set from >=2 callers: "We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls"
- [discussion] SURVIVOR consensus + rigorous filtering: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [readme] SURVIVOR filter command description: [evidence span withheld — anchor status: external_doc]
- [readme] SURVIVOR bincov command description: [evidence span withheld — anchor status: external_doc]
