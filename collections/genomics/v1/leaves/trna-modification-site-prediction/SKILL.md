---
name: trna-modification-site-prediction
description: Use when you have a sorted BAM of small RNA reads aligned to an artificial
  tRNA-masked genome and a masked reference FASTA, and you need to infer which modified
  nucleosides occur at covered positions on mature tRNAs and their 50 bp leader/trailer
  flanks.
when_to_use_negative:
- When the input reads were mapped with permissive mismatch settings and base misidentification
  from sequencing errors would inflate false modification calls — the article notes
  that allowing a single mismatch 'may lead to base misidentification, and hence causes
  the inflation of the false negatives'
- When the goal is tncRNA discovery/quantification rather than modification inference
  — use the tncRNA Toolkit prediction mode instead of the HAMR step
- When reads are not restricted to the artificial chromosomes (tRNA plus 50 nt leader/trailer),
  since ambiguous reads from non-tRNA regions must be excluded
edam_operation: http://edamontology.org/operation_0487
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0659
tools:
- name: HAMR
  role: Infers modified nucleoside types at each covered nucleotide position using
    the eukaryotic tRNA modification model; bundled with tncRNA-Toolkit (HAMR v1.2,
    requires python2)
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: bowtie
  role: Aligns processed reads to the artificial genome index with '-v2 --best' for
    error-tolerant, uniquely-mapping alignments feeding HAMR
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: samtools
  role: Prepares/manages the sorted BAM alignment used as HAMR input
- name: pandas
  role: Aggregates modification types and positions per tncRNA into a summary table
- name: tncRNA Toolkit
  role: Pipeline that builds the bowtie index, generates the artificial genome, and
    ships HAMR 1.2 for modification prediction
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:685b29dbded4d2be1ec3ebe76a68591d0c6e29bd3df077f1ecec01184fa9ce82
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
  - bowtie
  - pandas
  - samtools
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0487
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# trna-modification-site-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict modified nucleoside sites on tRNA-derived non-coding RNAs (tncRNAs) using HAMR v1.2 with the eukaryotic tRNA modification model, applied to small RNA-seq alignments against a masked artificial genome, and aggregate modification calls across species to assess conservation.

## When to use

Use when you have a sorted BAM of small RNA reads aligned to an artificial tRNA-masked genome and a masked reference FASTA, and you need to infer which modified nucleosides occur at covered positions on mature tRNAs and their 50 bp leader/trailer flanks.

## When NOT to use

- When the input reads were mapped with permissive mismatch settings and base misidentification from sequencing errors would inflate false modification calls — the article notes that allowing a single mismatch 'may lead to base misidentification, and hence causes the inflation of the false negatives'
- When the goal is tncRNA discovery/quantification rather than modification inference — use the tncRNA Toolkit prediction mode instead of the HAMR step
- When reads are not restricted to the artificial chromosomes (tRNA plus 50 nt leader/trailer), since ambiguous reads from non-tRNA regions must be excluded

## Inputs

- Sorted BAM alignment of processed small RNA reads mapped to the artificial (tRNA-masked) genome index with bowtie '-v2 --best'
- Masked reference FASTA (tRNA gene regions plus 50 nt upstream/downstream as artificial chromosomes)
- HAMR v1.2 eukaryotic tRNA modification model file (models/euk_trna_mods.Rdata)
- Annotation of mature tRNA and 50 bp leader/trailer coordinates in artificial chromosome space

## Outputs

- HAMR modification calls per covered nucleotide position (inferred modification type per site)
- Filtered table of modification sites restricted to mature tRNA and 50 bp leader/trailer regions
- Per-tncRNA summary table of modification types and positions (pandas)
- Cross-species comparison of conserved modifications across six angiosperms

## How to apply

Load the sorted BAM (produced by bowtie with '-v2 --best' on the artificial genome index) and the masked reference FASTA as inputs to HAMR v1.2. Run HAMR with the eukaryotic tRNA modification model ('models/euk_trna_mods.Rdata') and the parameter string '30 10 0.05 H4 0.01 0.05 0.05' so modification types are inferred at each covered nucleotide position with embedded FDR-related thresholds of 0.01/0.05. Restrict HAMR calls to sites falling on mature tRNA and 50 bp leader/trailer regions recorded from the artificial chromosome coordinates (the artificial genome masks genuine tRNA gene regions with 50 nt upstream and downstream). Aggregate modification types and positions per tncRNA into a summary table with pandas, then compare detected modifications across the six angiosperm genomes (e.g., adenosine replaced by m1A|m1I|ms2i6A in 20 isoacceptors, guanosine modified to m1G at T loop position 58 in TyrGTA) to assess cross-species conservation. Validate that modification calls pass HAMR's model-based significance thresholds and that only mature tRNA plus leader/trailer sites are retained.

## Related tools

- **HAMR** (Infers modified nucleoside types at each covered nucleotide position using the eukaryotic tRNA modification model; bundled with tncRNA-Toolkit (HAMR v1.2, requires python2)) — https://github.com/skbinfo/tncRNA-Toolkit
- **bowtie** (Aligns processed reads to the artificial genome index with '-v2 --best' for error-tolerant, uniquely-mapping alignments feeding HAMR) — https://github.com/skbinfo/tncRNA-Toolkit
- **samtools** (Prepares/manages the sorted BAM alignment used as HAMR input)
- **pandas** (Aggregates modification types and positions per tncRNA into a summary table)
- **tncRNA Toolkit** (Pipeline that builds the bowtie index, generates the artificial genome, and ships HAMR 1.2 for modification prediction) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python3 tncRNAs.py -s <species name> -i <processed small RNA reads>.fastq -o <output dir>
```

## Evaluation signals

- All retained modification calls pass HAMR's model-based significance thresholds (FDR-related parameters 0.01/0.05 embedded in the HAMR command)
- Output table contains only modification sites on mature tRNA and 50 bp leader/trailer regions in artificial chromosome coordinates
- Known conserved modifications are recovered as positive controls (e.g., adenosine replaced by m1A|m1I|ms2i6A in 20 isoacceptors; m1G at T loop position 58 in TyrGTA)
- Each called modification maps to a covered nucleotide position in the sorted BAM with adequate read support (HAMR minimum coverage parameter 30)
- Cross-species comparison yields modifications detected in all six angiosperm species for conserved sites

## Limitations

- Conventional RNA-seq suffers from tRNA modification biases; tRNA-seq would be advantageous for identifying tncRNAs and modifications
- Allowing even a single mismatch during alignment to overcome sequencing errors may cause base misidentification and inflate false negatives in modification detection
- HAMR v1.2 requires python2, which may complicate modern environments
- The detailed mechanisms showing how tRNA modifications affect the regulation of tRNA cleavage remain unclear

## Evidence

- [methods] Run HAMR (v1.2) with the eukaryotic tRNA modification model ('models/euk_trna_mods.Rdata') using the parameters '30 10 0.05 H4 0.01 0.05 0.05': [evidence span withheld — anchor status: not_found]
