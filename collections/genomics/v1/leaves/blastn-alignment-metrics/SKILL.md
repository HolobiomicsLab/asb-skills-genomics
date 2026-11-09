---
name: blastn-alignment-metrics
description: Use when you need to quantify contiguous alignment proportion or sequence
  accuracy of long-read 16S consensus reads (e.g., INC-Seq or chopSeq output) against
  known reference sequences, especially to validate read-correction or size-selection
  steps.
when_to_use_negative:
- The analysis is de novo with no reference sequences available — the article notes
  the de novo consensus approach (nanoClust) exists precisely to avoid reference dependence;
  blastn metrics require references.
- Input is raw noisy nanopore reads before consensus generation, where high error
  rates make reference-based mapping unreliable.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: chopSeq
  role: Produces the corrected consensus reads (fragmentation, orientation correction,
    optional 1,300–1,450 bp size selection) whose aligned proportion and accuracy
    are measured by blastn
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:69c43bd9f1b91288b98b474231a777c4d217a1bdb89be3a83d92c6a604009568
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - chopSeq
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# blastn-alignment-metrics

## Summary

Use blastn comparisons against reference 16S sequences to quantify what proportion of INC-Seq/chopSeq consensus nanopore reads aligns contiguously and to compute consensus sequence accuracy, serving as the validation metric for the NanoAmpli-Seq/chopSeq read-correction workflow.

## When to use

Use when you need to quantify contiguous alignment proportion or sequence accuracy of long-read 16S consensus reads (e.g., INC-Seq or chopSeq output) against known reference sequences, especially to validate read-correction or size-selection steps.

## When NOT to use

- The analysis is de novo with no reference sequences available — the article notes the de novo consensus approach (nanoClust) exists precisely to avoid reference dependence; blastn metrics require references.
- Input is raw noisy nanopore reads before consensus generation, where high error rates make reference-based mapping unreliable.

## Inputs

- INC-Seq consensus reads (FASTA/FASTQ) from raw nanopore mock-community amplicon reads
- chopSeq-processed consensus reads (with and without 1,300–1,450 bp size selection)
- Reference 16S rRNA gene sequences for the mock community

## Outputs

- Aligned proportion (%) of reads aligning contiguously to reference 16S sequences at each workflow stage
- Consensus sequence accuracy (%) from blastn comparison against references
- metrics.csv tabulating aligned proportions and accuracy values

## How to apply

Run blastn with INC-Seq or chopSeq consensus reads as query against reference 16S sequences and compute the fraction of each read aligning contiguously to a reference. Report the aligned proportion at each stage of the NanoAmpli-Seq workflow: raw INC-Seq consensus (~73±14.6%), after chopSeq fragmentation/orientation correction (~96±2.3%), and after chopSeq with 1,300–1,450 bp size selection (~98.4±0.7%). For sequence accuracy, compare the chopSeq_SS consensus sequences to the references via blastn and report percent identity, expecting an average of 98±0.23%. The rising aligned proportion across stages demonstrates that chopSeq removes fragmented and wrongly oriented regions of the consensus reads; size selection additionally removes partial/indel-riddled fragments. Collect all aligned proportions and the accuracy value into a metrics table (metrics.csv) for validation against the reported error ranges.

## Related tools

- **chopSeq** (Produces the corrected consensus reads (fragmentation, orientation correction, optional 1,300–1,450 bp size selection) whose aligned proportion and accuracy are measured by blastn) — github.com/umerijaz/nanopore

## Examples

```
blastn -query chopSeq_SS_consensus.fa -subject reference_16S.fa -outfmt 6; python collect_metrics.py --stage INC-Seq,chopSeq,chopSeq_SS --out metrics.csv
```

## Evaluation signals

- blastn aligned proportion after INC-Seq only falls within ~73±14.6%
- Aligned proportion after chopSeq without size selection falls within ~96±2.3%
- Aligned proportion after chopSeq_SS (1,300–1,450 bp size selection) falls within ~98.4±0.7%
- Consensus sequence accuracy after chopSeq_SS falls within 98±0.23%
- All aligned proportions and the accuracy value are collected into metrics.csv

## Limitations

- Reference-based mapping of raw noisy nanopore reads is unreliable due to high error rates, which is why metrics are computed on consensus reads rather than raw reads
- Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions, capping achievable accuracy (~98% after chopSeq_SS; 99.5% for 2D/1D2 with NanoAmpli-Seq) unless homopolymer-aware base calling or error correction is used

## Evidence

- [results] Following chopSeq read correction and size selection, overall sequence accuracy increased to an average of 98 ± 0.23%, with 98.4 ± 0.7% of the read aligning to the reference.: [evidence span withheld — anchor status: not_found]
- [abstract] chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [intro] incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads: "incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore.: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
