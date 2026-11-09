---
name: globin-depletion-percent-change-analysis
description: Use when comparing read/nt counts of globin (HBA1, HBA2, HBB) versus
  non-globin mRNAs between even-channel (RISER deplete) and odd-channel (control)
  halves of a split-flow-cell live whole-blood DRS run with RISER decisions.csv files.
when_to_use_negative:
- Post-sequencing bioinformatic removal of globin reads from a completed run — the
  article notes bioinformatically discarding globin reads post-sequencing is insufficient
  to counteract limited coverage of non-globin RNAs
- Runs without a split-flow-cell RISER/control design or missing RISER decisions.csv
  files, since the even/odd channel split and accepted/rejected read separation depend
  on them
- Transcript abundance profiles where the ≥95% control-dominance and ≥30-read-count-in-both-conditions
  filters cannot be met
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RISER
  role: Perform the live globin mRNA depletion via read-until and emit the decisions.csv
    files used to separate accepted from rejected reads
  repo: https://github.com/comprna/riser
- name: Guppy (v4.0.14)
  role: Basecalling of fast5 files (–flowcell FLO-MIN106 –kit SQK-RNA002) prior to
    mapping
- name: minimap2 (v2.17)
  role: Map reads to the GENCODE reference transcriptome with -ax map-ont --secondary=no
- name: samtools (v1.10)
  role: Filter alignments with -F 2324 to remove unmapped/low-confidence mappings
    before counting
- name: nanoporetech/read_until_api
  role: ReadUntil API used by RISER to send reject commands and eject unwanted RNAs
    during live sequencing
  repo: https://github.com/nanoporetech/read_until_api
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:e04a6616d27870c9a10070ee046844c9799ee5f795738a50f377b1be62be3bb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - Guppy (v4.0.14)
  - RISER
  - minimap2 (v2.17)
  - nanoporetech/read_until_api
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# globin-depletion-percent-change-analysis

## Summary

Quantifies per-transcript percent change in read and nucleotide counts between a RISER globin-depletion condition and a no-RISER control during live whole-blood direct RNA sequencing, to verify that RISER depletes globin mRNA (>90% decrease) while boosting non-globin transcript coverage (~16% increase).

## When to use

Use when comparing read/nt counts of globin (HBA1, HBA2, HBB) versus non-globin mRNAs between even-channel (RISER deplete) and odd-channel (control) halves of a split-flow-cell live whole-blood DRS run with RISER decisions.csv files.

## When NOT to use

- Post-sequencing bioinformatic removal of globin reads from a completed run — the article notes bioinformatically discarding globin reads post-sequencing is insufficient to counteract limited coverage of non-globin RNAs
- Runs without a split-flow-cell RISER/control design or missing RISER decisions.csv files, since the even/odd channel split and accepted/rejected read separation depend on them
- Transcript abundance profiles where the ≥95% control-dominance and ≥30-read-count-in-both-conditions filters cannot be met

## Inputs

- Live whole-blood DRS reads basecalled with Guppy (v4.0.14), mapped with minimap2 v2.17 to GENCODE release 43 (GRCh38.p13) and filtered with samtools v1.10 (-F 2324)
- decisions.csv files output by RISER for the globin-deplete and control split-flow-cell conditions
- Channel-to-condition assignment (even channels = RISER deplete, remaining = control)
- GENCODE transcript annotations to classify globin genes (HBA1, HBA2, HBB) and exclude lncRNAs overlapping protein-coding exons
- An independent non-RISER whole-blood run for the random-split significance control

## Outputs

- percent_change_results.csv with per-transcript read-count and nt-count percent changes (RISER vs control)
- Average percent change for globin transcripts (n = 4) and non-globin transcripts (n = 100)
- Paired Wilcoxon signed rank test result comparing RISER effects against the random-split null

## How to apply

Load reads basecalled with Guppy (v4.0.14), mapped to the GENCODE release 43 transcriptome (GRCh38.p13) with minimap2 (-ax map-ont --secondary=no) and filtered with samtools (v1.10, -F 2324), together with the decisions.csv files RISER wrote for the globin-deplete and control conditions. Split reads by channel (even channels = RISER deplete, remaining channels = control) and compute per-transcript read and nucleotide counts per condition, separating accepted from rejected reads via the RISER decisions files. Restrict to transcripts taking ≥95% of control-condition reads with ≥30 read counts in both conditions, excluding lncRNAs whose exons overlap protein-coding exons. Compute per-transcript percent change (RISER vs control), then average separately for globin and non-globin transcripts; as a significance control, recompute percent changes between two randomly split halves of an independent non-RISER run and test with a paired Wilcoxon signed rank test. Write all values to percent_change_results.csv.

## Related tools

- **RISER** (Perform the live globin mRNA depletion via read-until and emit the decisions.csv files used to separate accepted from rejected reads) — https://github.com/comprna/riser
- **Guppy (v4.0.14)** (Basecalling of fast5 files (–flowcell FLO-MIN106 –kit SQK-RNA002) prior to mapping)
- **minimap2 (v2.17)** (Map reads to the GENCODE reference transcriptome with -ax map-ont --secondary=no)
- **samtools (v1.10)** (Filter alignments with -F 2324 to remove unmapped/low-confidence mappings before counting)
- **nanoporetech/read_until_api** (ReadUntil API used by RISER to send reject commands and eject unwanted RNAs during live sequencing) — https://github.com/nanoporetech/read_until_api

## Examples

```
minimap2 -ax map-ont --secondary=no gencode.v43.transcripts.fa reads.fastq | samtools view -F 2324 -b > mapped.bam
```

## Evaluation signals

- Globin transcript read counts drop by >90% on average (reported: 93% average decrease, n = 4) in RISER vs control
- Non-globin transcript read counts increase by ~16% on average (n = 100) in RISER vs control
- Paired Wilcoxon signed rank test on random-split halves of a non-RISER run shows no equivalent globin depletion signal (significance control)
- All counted transcripts pass the ≥95% control-condition read dominance, ≥30 reads in both conditions, and lncRNA-overlap-exclusion filters
- percent_change_results.csv is produced with per-transcript read and nt percent changes

## Limitations

- The lag between the read-until API receiving a reject command from RISER and requesting voltage reversal, as well as the efficiency of the voltage reversal, is beyond RISER's control and varies across platforms
- RNA secondary structures forming on the trans side of the pore could impede or completely inhibit reverse translocation by voltage reversal
- Bioinformatic discarding of globin reads post-sequencing is insufficient to counteract limited coverage of non-globin RNAs, so live depletion (not post-hoc filtering) is required for the coverage gain

## Evidence

- [abstract] Globin mRNA depletion in whole blood decreases globin reads by more than 90% and increases non-globin reads by 16% on average: "achieving a decrease in globin reads by more than 90% as well as an increase in non-globin reads by 16% on average"
- [methods] GENCODE release 43 mapping reference used for counting: "basecalled, mapped and filtered using the GENCODE reference transcriptome (release 43, assembly GRCh38.p13)"
- [methods] samtools alignment filter parameter: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [readme] RISER supports a globin mRNA target class: "Globin mRNA (globin)"
- [discussion] Bioinformatic post-hoc globin removal is insufficient, motivating live depletion analysis: "bioinformatically discarding globin reads post-sequencing is insufficient to counteract the problem of limited coverage of non-globin RNAs"
