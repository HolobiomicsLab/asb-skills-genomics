---
name: separate-detection-and-subtraction-sv-filtering
description: Use when evaluating a joint tumor/control somatic SV caller against a
  naive baseline, or when only single-sample germline SV callers (Sniffles2, cuteSV,
  Delly, SVIM) are available for paired long-read data.
when_to_use_negative:
- When a joint tumor/control caller such as nanomonsv is available — the paper shows
  the subtraction approach yields more false positives because germline SVs can pass
  the threshold in the tumor and narrowly miss it in the matched control
- When calling somatic SVs at low tumor purity or low sequence yield — the subtraction
  baseline loses precision/recall relative to nanomonsv at 10–20x coverage and 20%
  tumor purity in NanoSim simulations
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sniffles2
  role: Single-sample long-read SV caller run on tumor and control BAMs with --minsupport
    1 --non-germline before subtraction
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: cuteSV
  role: Alternative single-sample SV caller (--min_support 1, max_cluster_bias_INS/DEL
    100, diff_ratio_merging 0.3) in the subtraction baseline
- name: Delly
  role: Long-read 'lr' mode used as another subtraction-baseline caller on tumor and
    control separately
- name: SVIM
  role: SV caller run in 'alignment' mode with --skip_genotyping on tumor and control
    separately
- name: CAMPHORsomatic
  role: Paired-sample somatic SV caller that jointly compares candidates between cancer
    and matched normal, used as an alternative to naive subtraction
  repo: https://github.com/afujimoto/CAMPHORsomatic
- name: minimap2
  role: Aligns simulated or experimental long-read FASTQ to GRCh38 before SV calling
  repo: https://github.com/lh3/minimap2
- name: nanomonsv
  role: Joint tumor/control somatic SV caller used as the method this baseline is
    compared against
  repo: https://github.com/friend1ws/nanomonsv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:dd8e35b3652bfb85c45783750428fafe7d0f3f6b821574e3157521601b17fc06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/Sniffles
  tools:
  - CAMPHORsomatic
  - Delly
  - NanoSim
  - SURVIVOR
  - SVIM
  - Sniffles
  - Sniffles2
  - cuteSV
  - minimap2
  - nanomonsv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# separate-detection-and-subtraction-sv-filtering

## Summary

A baseline somatic SV-calling strategy in which a long-read SV caller is run independently on the tumor and matched-control BAMs and tumor calls overlapping control calls (with a margin, e.g. 200 bp) are subtracted. It is used by the nanomonsv paper as a comparison baseline that nanomonsv outperforms.

## When to use

Use when evaluating a joint tumor/control somatic SV caller against a naive baseline, or when only single-sample germline SV callers (Sniffles2, cuteSV, Delly, SVIM) are available for paired long-read data.

## When NOT to use

- When a joint tumor/control caller such as nanomonsv is available — the paper shows the subtraction approach yields more false positives because germline SVs can pass the threshold in the tumor and narrowly miss it in the matched control
- When calling somatic SVs at low tumor purity or low sequence yield — the subtraction baseline loses precision/recall relative to nanomonsv at 10–20x coverage and 20% tumor purity in NanoSim simulations

## Inputs

- Tumor long-read BAM aligned to GRCh38 (minimap2)
- Matched-control long-read BAM aligned to GRCh38 (minimap2)
- Per-caller settings (e.g. Sniffles2 --minsupport 1 --non-germline; cuteSV --min_support 1, max_cluster_bias_INS/DEL 100, diff_ratio_merging 0.3)

## Outputs

- Per-sample germline SV VCFs from tumor and control
- Subtracted somatic SV VCF (tumor calls minus control-overlapping calls within 200 bp margin)
- Precision/recall values against a simulated somatic SV truth set for benchmarking

## How to apply

Align long-read FASTQ (ONT or PacBio) to GRCh38 with minimap2, then run the same single-sample SV caller separately on the tumor and matched control BAMs: Sniffles2 with '--minsupport 1 --non-germline', cuteSV with '--min_support = 1' (max_cluster_bias_INS/DEL 100, diff_ratio_merging 0.3), Delly 'lr' mode, or SVIM 'alignment' with '--skip_genotyping'. Next, remove tumor calls that overlap control calls within a 200 bp margin to approximate somatic filtering. The paper notes this subtraction approach 'can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control', and shows via NanoSim simulations (10–50x, 0–100% tumor purity) that nanomonsv achieves higher precision and recall. CAMPHORsomatic, which instead compares SV candidates between cancer and matched-normal samples jointly, is an alternative to naive subtraction.

## Related tools

- **Sniffles2** (Single-sample long-read SV caller run on tumor and control BAMs with --minsupport 1 --non-germline before subtraction) — https://github.com/fritzsedlazeck/Sniffles
- **cuteSV** (Alternative single-sample SV caller (--min_support 1, max_cluster_bias_INS/DEL 100, diff_ratio_merging 0.3) in the subtraction baseline)
- **Delly** (Long-read 'lr' mode used as another subtraction-baseline caller on tumor and control separately)
- **SVIM** (SV caller run in 'alignment' mode with --skip_genotyping on tumor and control separately)
- **CAMPHORsomatic** (Paired-sample somatic SV caller that jointly compares candidates between cancer and matched normal, used as an alternative to naive subtraction) — https://github.com/afujimoto/CAMPHORsomatic
- **minimap2** (Aligns simulated or experimental long-read FASTQ to GRCh38 before SV calling) — https://github.com/lh3/minimap2
- **nanomonsv** (Joint tumor/control somatic SV caller used as the method this baseline is compared against) — https://github.com/friend1ws/nanomonsv

## Examples

```
sniffles -i tumor.bam -v tumor_sniffles.vcf --minsupport 1 --non-germline
```

## Evaluation signals

- Compute precision and recall of the subtracted somatic SV calls against a simulated somatic SV truth set (e.g. the SURVIVOR simSV-injected GRCh38 diploid/tumor pairs)
- Verify the 200 bp overlap-margin subtraction was applied uniformly between tumor and control call sets
- Confirm per-caller minimum-support thresholds (Sniffles2 --minsupport 1; cuteSV --min_support 1) were used so low-support germline events are not missed in control
- Expect nanomonsv to outperform this baseline: subtraction calls should show more false positives from germline SVs narrowly missing threshold in control
- Precision should degrade at low tumor purity (20%) and low coverage (10–20x) relative to high purity/high coverage points

## Limitations

- Germline SVs that pass the caller threshold in the tumor but narrowly miss it in the matched control become false-positive somatic calls after subtraction
- Subtraction quality depends on caller sensitivity: calls detected in tumor but undetected in control (even if germline) are retained as somatic
- In the nanomonsv NanoSim benchmark, precision and recall of the subtraction approaches were inferior to nanomonsv across tumor purities and sequencing depths

## Evidence

- [intro] existing separate-detection-and-subtraction approaches generate false positives: "this approach can generate many false positives, such as germline SVs that pass the threshold in the tumor and narrowly miss it in the matched control"
- [readme] CAMPHORsomatic joint comparison: [evidence span withheld — anchor status: external_doc]
- [readme] Sniffles2 long-read SV calling scope: [evidence span withheld — anchor status: external_doc]
- [full_text] nanomonsv superiority over subtraction baseline: [evidence span withheld — anchor status: not_found]
