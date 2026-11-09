---
name: degenerate-motif-disambiguation
description: Use when Snapper's final_motifs_[strand]_[contig_name].fasta outputs
  contain degenerate-base motifs that must be reconciled with REBASE reference motifs
  or resolved to concrete k-mers before declaring a run validated.
when_to_use_negative:
- Motifs are already concrete (no IUPAC ambiguity codes) and match REBASE references
  exactly — direct table comparison suffices
- The goal is de novo motif discovery itself rather than post-hoc reconciliation of
  Snapper output against reference specificities
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0080
tools:
- name: Snapper
  role: produces the de novo methylation motifs (final_motifs FASTA files) that are
    subsequently reconciled with REBASE references
  repo: DNKonanov/Snapper
- name: snapper-ont
  role: pip distribution of Snapper used to generate per-strain motif FASTA outputs
    with default parameters
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:26bcfbfa1bc9697bdf0d52016aeee5e17047b6813d6d77c361482c848bd57019
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DNKonanov/Snapper
  tools:
  - Python
  - Snapper
  - Tombo
  - pip
  - snapper-ont
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# degenerate-motif-disambiguation

## Summary

Resolves degenerate IUPAC-coded methylation motifs reported by Snapper (e.g., the REBASE reference CACNNNNNRTAAA) into the concrete sequence instances actually detected in a strain (e.g., CACNNNNNATAAA), so Snapper output can be judged against REBASE methyltransferase specificities.

## When to use

Use when Snapper's final_motifs_[strand]_[contig_name].fasta outputs contain degenerate-base motifs that must be reconciled with REBASE reference motifs or resolved to concrete k-mers before declaring a run validated.

## When NOT to use

- Motifs are already concrete (no IUPAC ambiguity codes) and match REBASE references exactly — direct table comparison suffices
- The goal is de novo motif discovery itself rather than post-hoc reconciliation of Snapper output against reference specificities

## Inputs

- Snapper final_motifs_[strand]_[contig_name].fasta files per strain
- REBASE reference methyltransferase specificity motifs for T. phaeum, N. gonorrhoeae, M. hungatei, C. perfringens

## Outputs

- Per-strain motif table mapping detected motifs to REBASE references
- Resolved concrete motif instances (e.g., CACNNNNNATAAA for CACNNNNNRTAAA)
- List of novel motifs not present in REBASE (CCAG/TGGCCA, WGGCCW)

## How to apply

After running snapper with default parameters (ks_t = 3, coverage = 40, threads = 8, k_size = 15, long_k_size = 29, min_conf = 100), collect the final_motifs_[strand]_[contig_name].fasta files per strain into per-strain motif tables. Compare each detected motif against the REBASE reference methyltransferase specificity for the strain; where the reference is degenerate (e.g., CACNNNNNRTAAA for T. phaeum), resolve it to the concrete inferred instance by checking which fixed base satisfies the ambiguous position (R -> A giving CACNNNNNATAAA). Distinguish recovered reference motifs from genuinely novel detections such as the CCAG/TGGCCA pair in T. phaeum DSM 12270 and WGGCCW in C. perfringens ATCC 13124. Accept the run when the motif table recovers all expected REBASE motifs plus, where applicable, these novel sites.

## Related tools

- **Snapper** (produces the de novo methylation motifs (final_motifs FASTA files) that are subsequently reconciled with REBASE references) — DNKonanov/Snapper
- **snapper-ont** (pip distribution of Snapper used to generate per-strain motif FASTA outputs with default parameters)

## Examples

```
snapper -sample_fast5dir ../Tphaeum/fast5/native_multi/ -control_fast5dir ../Tphaeum/fast5/wga_multi/ -reference ../Tphaeum/genome/ref.fasta; grep '>' outdir/final_motifs_*.fasta
```

## Evaluation signals

- Per-strain motif table recovers all expected REBASE reference motifs for the four SRP219538 strains
- The degenerate reference CACNNNNNRTAAA is resolved consistently to the concrete instance CACNNNNNATAAA (R position fixed to A)
- Novel motifs CCAG/TGGCCA (T. phaeum DSM 12270) and WGGCCW (C. perfringens ATCC 13124) are correctly flagged as absent from REBASE rather than mismapped to a reference
- Every detected motif in the table is either matched to a REBASE specificity or explicitly labeled novel — no unclassified motifs remain

## Limitations

- A degenerate REBASE reference may correspond to a single concrete instance in the strain, so naive string comparison against the IUPAC motif can either over- or under-report matches
- Snapper defaults (k_size = 15, long_k_size = 29, min_conf = 100) may not capture all motif variants; deviations like R->A substitution must be judged manually against REBASE

## Evidence

- [results] with default parameters, Snapper recovered motifs almost identical to the REBASE references for the four SRP219538 strains, the only deviation being the long motif CACNNNNNRTAAA inferred as CACNNNNNATAAA: [evidence span withheld — anchor status: not_found]
- [results] it additionally detected the novel CCAG/TGGCCA motifs in T. phaeum DSM 12270 and WGGCCW in C. perfringens ATCC 13124: [evidence span withheld — anchor status: not_found]
- [methods] Collect the final_motifs_[strand]_[contig_name].fasta files per strain into per-strain motif tables and compare each detected motif against REBASE reference methyltransferase specificity: [evidence span withheld — anchor status: not_found]
- [readme] collect the final_motifs files and validate against REBASE: "A more detailed usage guideline and few usercases are available in [Snapper's documentation](https://snapper-tutorial.readthedocs.io/en/latest/index.html)"
