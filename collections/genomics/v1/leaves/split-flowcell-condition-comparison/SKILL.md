---
name: split-flowcell-condition-comparison
description: Use when a live read-until run splits a flow cell by channel into RISER-deplete
  and control halves and you must quantify per-transcript percent change in read/nt
  counts.
when_to_use_negative:
- Comparing two fully separate sequencing runs rather than channel-split conditions
  of the same flow cell, where run-to-run technical variation confounds the effect
- Reads whose mapping has not been filtered with samtools (-F 2324), since secondary/supplementary
  and unmapped alignments would inflate per-transcript counts
- Bioinformatic discarding of globin reads post-sequencing, which the paper reports
  is insufficient to counteract limited coverage of non-globin RNAs
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: RISER
  role: Performs live channel-split depletion and emits decisions.csv classifying
    accepted vs rejected reads per condition
  repo: https://github.com/comprna/riser
- name: Guppy (v4.0.14)
  role: Basecalls raw fast5 signal into sequence for per-transcript counting
- name: minimap2 (v2.17)
  role: Maps reads to the GENCODE reference transcriptome for transcript-level assignment
- name: samtools (v1.10)
  role: Removes unmapped/low-confidence alignments with -F 2324 before counting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:d16d284666adb06c72b374cfe78a9f7700067973adced8952e52e29522ee61af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - Guppy (v4.0.14)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Split-flowcell condition comparison

## Summary

Compares per-transcript read and nucleotide counts between RISER-deplete and control conditions on a single split flow cell (even channels = RISER, remaining channels = control), quantifying percent changes for targeted vs off-target transcripts.

## When to use

Use when a live read-until run splits a flow cell by channel into RISER-deplete and control halves and you must quantify per-transcript percent change in read/nt counts.

## When NOT to use

- Comparing two fully separate sequencing runs rather than channel-split conditions of the same flow cell, where run-to-run technical variation confounds the effect
- Reads whose mapping has not been filtered with samtools (-F 2324), since secondary/supplementary and unmapped alignments would inflate per-transcript counts
- Bioinformatic discarding of globin reads post-sequencing, which the paper reports is insufficient to counteract limited coverage of non-globin RNAs

## Inputs

- BAM of direct RNA reads basecalled with Guppy v4.0.14 and mapped with minimap2 v2.17 to GENCODE release 43 (GRCh38.p13), filtered with samtools v1.10 (-F 2324)
- decisions.csv output by RISER for the globin-deplete and control split-flow-cell conditions
- Independent non-RISER run BAM for the randomly split null comparison

## Outputs

- percent_change_results.csv with per-transcript read and nt count percent changes (RISER vs control)
- Average percent change for globin transcripts (HBA1, HBA2, HBB) and for non-globin transcripts
- Wilcoxon signed rank p-value from the randomly split non-RISER control comparison

## How to apply

Load the basecalled (Guppy v4.0.14), minimap2 v2.17-aligned (GENCODE GRCh38.p13) BAM filtered with samtools v1.10 (-F 2324) plus the RISER decisions.csv for each split-flow-cell condition. Compute per-transcript read and nucleotide counts per condition, splitting reads by channel (even channels = RISER deplete; remaining = control). Restrict to transcripts capturing ≥95% of control reads with ≥30 counts in both conditions, excluding lncRNAs overlapping protein-coding exons. Use the RISER decisions file to separate accepted from rejected reads, compute per-transcript percent change (RISER vs control), and average separately for globin (HBA1, HBA2, HBB) and non-globin transcripts. As a null control, recompute percent changes between randomly split halves of an independent non-RISER run and test with a paired Wilcoxon signed rank test.

## Related tools

- **RISER** (Performs live channel-split depletion and emits decisions.csv classifying accepted vs rejected reads per condition) — https://github.com/comprna/riser
- **Guppy (v4.0.14)** (Basecalls raw fast5 signal into sequence for per-transcript counting)
- **minimap2 (v2.17)** (Maps reads to the GENCODE reference transcriptome for transcript-level assignment)
- **samtools (v1.10)** (Removes unmapped/low-confidence alignments with -F 2324 before counting)

## Examples

```
python3 riser.py --target globin --mode deplete --duration 6 --kit RNA002  # then compare decisions.csv split by channel (even=deplete) against control
```

## Evaluation signals

- Globin transcript read-count decrease exceeds the reported >90% depletion threshold
- Non-globin transcript read increase matches the reported ~16% average increase
- Percent changes in the randomly split non-RISER control halves are not significant by paired Wilcoxon signed rank test
- Only transcripts with ≥30 read counts in both conditions and ≤95% control-condition read share (i.e., transcripts taking up at least 95% of control reads retained by the filter) are in the result table
- LncRNAs whose exons overlap protein-coding exons are excluded from the summary

## Limitations

- The lag between the read-until API receiving a reject command and requesting voltage reversal, and the reversal efficiency, are beyond RISER's control and vary across platforms, adding noise to per-channel depletion magnitude
- RNA secondary structures on the trans side of the pore can impede or inhibit reverse translocation by voltage reversal, reducing observed depletion in the deplete channels
- Low-count transcripts below the 30-read threshold in either condition cannot be reliably quantified for percent change

## Evidence

- [methods] Split by channel assignment: [evidence span withheld — anchor status: not_found]
- [methods] Filter thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] Reported effect sizes: "achieving a decrease in globin reads by more than 90% as well as an increase in non-globin reads by 16% on average"
- [readme] Read-until mechanism: [evidence span withheld — anchor status: not_found]
- [discussion] Null-control rationale: "bioinformatically discarding globin reads post-sequencing is insufficient to counteract the problem of limited coverage of non-globin RNAs"
