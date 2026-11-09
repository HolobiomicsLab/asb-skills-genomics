---
name: nucleoside-modification-annotation-on-tncrnas
description: Use when small RNA-seq reads have been mapped to an artificial tRNA-masked
  genome and you need to infer which nucleoside modifications occur on tncRNAs and
  which are conserved across species.
when_to_use_negative:
- Input reads have not been mapped against the artificial tRNA-masked genome, so modification
  calls would be confounded by ambiguous non-tRNA mappings
- The goal is tncRNA discovery/quantification rather than modification annotation
  — use the tncRNA Toolkit prediction steps instead
- Conventional sRNA-seq data where modification-induced misincorporation cannot be
  interpreted; the article notes switching to tRNA-seq is needed to overcome modification
  biases
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0080
tools:
- name: HAMR
  role: Runs the model-based inference of modified nucleosides (v1.2, euk_trna_mods
    model, parameters 30 10 0.05 H4 0.01 0.05 0.05) on the sorted BAM; requires python2
- name: bowtie1
  role: Aligns processed reads to the artificial genome index with '-v2 --best' for
    error-tolerant, uniquely mapping alignments that feed HAMR
- name: tncRNA-Toolkit
  role: Packages the HAMR and bowtie1 executables and builds the artificial genome
    bowtie index from the genome FASTA
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: pandas
  role: Aggregates HAMR modification calls into per-tncRNA summary tables
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:dcf94d7ee11717706e9b9534bfccaf5206797fab0077188d51127bbe0bd0dc6d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - HAMR
  - bowtie1
  - pandas
  - samtools
  - tncRNA-Toolkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nucleoside-modification-annotation-on-tncrnas

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts modified nucleosides (e.g., m1A, m1G) on plant tRNA-derived non-coding RNAs by running HAMR v1.2 with the eukaryotic tRNA modification model on alignments restricted to mature tRNA plus 50 nt leader/trailer regions, enabling cross-species comparison of conserved modification sites across six angiosperms.

## When to use

Use when small RNA-seq reads have been mapped to an artificial tRNA-masked genome and you need to infer which nucleoside modifications occur on tncRNAs and which are conserved across species.

## When NOT to use

- Input reads have not been mapped against the artificial tRNA-masked genome, so modification calls would be confounded by ambiguous non-tRNA mappings
- The goal is tncRNA discovery/quantification rather than modification annotation — use the tncRNA Toolkit prediction steps instead
- Conventional sRNA-seq data where modification-induced misincorporation cannot be interpreted; the article notes switching to tRNA-seq is needed to overcome modification biases

## Inputs

- Sorted BAM alignment of small RNA reads produced by bowtie1 '-v2 --best' against the artificial genome index
- Masked reference FASTA (artificial chromosomes: tRNA gene regions plus 50 nt upstream/downstream)
- Eukaryotic tRNA modification model file ('models/euk_trna_mods.Rdata' shipped with HAMR v1.2)
- Coordinate records of mature tRNA and 50 bp leader/trailer regions on artificial chromosomes

## Outputs

- HAMR-inferred modification calls (type and position) per covered nucleotide on tncRNAs
- pandas summary table of modification types and positions aggregated per tncRNA
- Cross-species comparison of conserved modification sites (e.g., m1A|m1I|ms2i6A on adenosine in 20 isoacceptors; m1G at position 58 in TyrGTA)

## How to apply

Align quality/adapter-trimmed (Trim Galore v0.6.6) small RNA reads to the artificial genome index (masked genuine tRNA genes plus 50 nt flanks) using bowtie with '-v2 --best' so only uniquely, error-tolerantly mapped reads are retained. Feed the sorted BAM and the masked reference FASTA into HAMR v1.2 with the eukaryotic tRNA modification model ('models/euk_trna_mods.Rdata') and parameters '30 10 0.05 H4 0.01 0.05 0.05' (coverage, per-position read count, identity ratio, base quality, mapping quality, and FDR-related cutoffs) to classify each covered nucleotide as a specific modification type. Restrict output sites to those on mature tRNA bodies and 50 bp leader/trailer flanks as recorded on the artificial chromosome coordinates. Aggregate modification types and positions per tncRNA into a summary table with pandas, then compare calls across the six angiosperms (Arabidopsis, Medicago, tomato, chickpea, rice, maize) to flag conserved modifications such as m1G at T-loop position 58 in TyrGTA.

## Related tools

- **HAMR** (Runs the model-based inference of modified nucleosides (v1.2, euk_trna_mods model, parameters 30 10 0.05 H4 0.01 0.05 0.05) on the sorted BAM; requires python2)
- **bowtie1** (Aligns processed reads to the artificial genome index with '-v2 --best' for error-tolerant, uniquely mapping alignments that feed HAMR)
- **tncRNA-Toolkit** (Packages the HAMR and bowtie1 executables and builds the artificial genome bowtie index from the genome FASTA) — https://github.com/skbinfo/tncRNA-Toolkit
- **pandas** (Aggregates HAMR modification calls into per-tncRNA summary tables)

## Examples

```
python HAMR.py annotate.py aln_sorted.bam ref_masked.fa sample models/euk_trna_mods.Rdata 30 10 0.05 H4 0.01 0.05 0.05
```

## Evaluation signals

- All retained modification sites pass HAMR's model-based significance thresholds (FDR-related parameters 0.01/0.05 embedded in the command)
- Every reported site falls within mature tRNA or 50 bp leader/trailer coordinates on the artificial chromosomes
- Conserved calls replicate across all six angiosperms (e.g., adenosine→m1A|m1I|ms2i6A in 20 isoacceptors; m1G at T-loop position 58 of TyrGTA)
- Coverage at called sites meets the minimum of 30 reads per position and 10 modifying reads per the HAMR parameters
- Underlying alignments are unique and error-tolerant (bowtie '-v2 --best'), not multi-mapped (suppressed at >50 alignments)

## Limitations

- HAMR v1.2 requires python2 to run
- Allowing a single mismatch during alignment may lead to base misidentification and inflation of false negatives for modification calling
- Conventional RNA-seq suffers from tRNA modification biases; tRNA-seq is needed to fully overcome them
- Mechanisms by which the detected tRNA modifications affect cleavage regulation remain unclear
- Experimentally, predicted modification sites are in-silico and require validation

## Evidence
