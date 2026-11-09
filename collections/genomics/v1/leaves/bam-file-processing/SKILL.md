---
name: bam-file-processing
description: Use when adapter-trimmed plant sRNA-seq reads must be aligned to a masked/artificial
  tRNA genome and the resulting sorted BAM fed to HAMR or tncRNA classification.
when_to_use_negative:
- Input is already a modification-call table or tncRNA count matrix — no BAM alignment
  step is needed
- Aligning to the full (unmasked) reference genome instead of the masked artificial
  genome — reads from non-tRNA regions would be ambiguously misclassified as tncRNAs
- Data are not high-throughput small RNA/tRNA-derived sequencing; HAMR's mismatch-signature
  model is tuned for tRNA modifications and conventional RNA-seq suffers from tRNA
  modification biases
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: bowtie1
  role: Aligns processed sRNA reads to the artificial genome index with '-v2 --best';
    discards multi-mapped reads (>50 alignments) with '--norc -v 0 -m 50'
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: samtools
  role: Converts and sorts bowtie alignments into the sorted BAM loaded by HAMR (v1.10
    in the toolkit dependencies)
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: HAMR
  role: Loads the sorted BAM and masked reference FASTA to infer modified nucleosides
    at each covered position using the euk_trna_mods.Rdata model (v1.2)
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Trim Galore
  role: Produces the quality- and adapter-trimmed single-end small RNA reads (v0.6.6)
    that are the upstream input to alignment
  repo: https://github.com/FelixKrueger/TrimGalore
- name: tncRNA-Toolkit
  role: Builds the artificial genome bowtie index and bundles/executes bowtie1 and
    HAMR from the 'util' directory
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:d87098cb113e2e23ed879b06c0f3315a720a173b10584150c6041c49ce739dfc
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
  - Trim Galore
  - bowtie1
  - pandas
  - samtools
  - tncRNA-Toolkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# bam-file-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Processing of sorted BAM alignments from small RNA-seq reads mapped to the masked artificial tRNA genome (tRNA genes ±50 nt flanks as artificial chromosomes), used as the direct input to HAMR v1.2 for nucleoside modification calling on tncRNAs.

## When to use

Use when adapter-trimmed plant sRNA-seq reads must be aligned to a masked/artificial tRNA genome and the resulting sorted BAM fed to HAMR or tncRNA classification.

## When NOT to use

- Input is already a modification-call table or tncRNA count matrix — no BAM alignment step is needed
- Aligning to the full (unmasked) reference genome instead of the masked artificial genome — reads from non-tRNA regions would be ambiguously misclassified as tncRNAs
- Data are not high-throughput small RNA/tRNA-derived sequencing; HAMR's mismatch-signature model is tuned for tRNA modifications and conventional RNA-seq suffers from tRNA modification biases

## Inputs

- Quality-filtered, adapter-trimmed single-end small RNA-seq FASTQ file
- Bowtie1 index of the artificial genome (masked tRNA gene regions with 50 nt upstream/downstream flanks added as artificial chromosomes)
- Masked reference FASTA for HAMR
- Sorted BAM alignment (produced by bowtie '-v2 --best' and samtools)
- HAMR eukaryotic tRNA modification model file (models/euk_trna_mods.Rdata)

## Outputs

- Sorted BAM of reads mapping exclusively to the artificial chromosomes (candidate tncRNAs)
- HAMR modification calls per covered nucleotide position with inferred modification identities (e.g., m1A|m1I|ms2i6A, m1G)
- Pandas summary table of modification types and positions per tncRNA, restricted to mature tRNA plus 50 bp leader/trailer regions

## How to apply

Take quality-filtered, adapter-trimmed single-end reads (Trim Galore v0.6.6) and align them to the artificial genome index built by the tncRNA Toolkit using bowtie1 with '-v2 --best' to permit error-tolerant yet uniquely mapped alignments; discard multi-mapped reads aligned more than 50 times using bowtie1 with '--norc -v 0 -m 50'. Convert and sort the alignments with samtools into a sorted BAM, which is loaded together with the masked reference FASTA into HAMR (v1.2). Run HAMR with the eukaryotic tRNA modification model ('models/euk_trna_mods.Rdata') and the parameter string '30 10 0.05 H4 0.01 0.05 0.05' (minimum coverage 30, base-quality/map-quality and identity/mismatch-ratio thresholds, FDR-related 0.01/0.05 cutoffs) to infer modification types (e.g., m1A|m1I|ms2i6A on adenosine, m1G on guanosine) at each covered nucleotide. Restrict results to modification sites on mature tRNA and the 50 bp leader/trailer flanks recorded from the artificial chromosome coordinates, then aggregate modification types and positions per tncRNA into a summary table with pandas and compare across the six angiosperms. The rationale is that reads mapping exclusively to artificial chromosomes avoids ambiguous assignments from non-tRNA genomic regions, and HAMR's mismatch-signature model converts mismatch patterns in the BAM into putative modified nucleosides.

## Related tools

- **bowtie1** (Aligns processed sRNA reads to the artificial genome index with '-v2 --best'; discards multi-mapped reads (>50 alignments) with '--norc -v 0 -m 50') — https://github.com/skbinfo/tncRNA-Toolkit
- **samtools** (Converts and sorts bowtie alignments into the sorted BAM loaded by HAMR (v1.10 in the toolkit dependencies)) — https://github.com/skbinfo/tncRNA-Toolkit
- **HAMR** (Loads the sorted BAM and masked reference FASTA to infer modified nucleosides at each covered position using the euk_trna_mods.Rdata model (v1.2)) — https://github.com/skbinfo/tncRNA-Toolkit
- **Trim Galore** (Produces the quality- and adapter-trimmed single-end small RNA reads (v0.6.6) that are the upstream input to alignment) — https://github.com/FelixKrueger/TrimGalore
- **tncRNA-Toolkit** (Builds the artificial genome bowtie index and bundles/executes bowtie1 and HAMR from the 'util' directory) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python HAMR.py -i aln.sorted.bam -g masked_ref.fa -o mods -m models/euk_trna_mods.Rdata 30 10 0.05 H4 0.01 0.05 0.05
```

## Evaluation signals

- Reads are classified as tncRNAs only if they map exclusively to the artificial chromosomes; any ambiguous alignment to non-tRNA genomic regions must be absent
- Multi-mapped reads aligned more than 50 times are discarded (bowtie '-m 50' filter respected)
- Modification calls pass HAMR's model-based significance thresholds — FDR-related parameters 0.01/0.05 embedded in the HAMR command and minimum coverage 30
- Only modification sites on mature tRNA and the 50 bp leader/trailer flank regions are retained in the output summary table
- Cross-species check: conserved modification calls should reproduce expected patterns such as adenosine replaced by m1A|m1I|ms2i6A in 20 isoacceptors and m1G at T-loop position 58 in TyrGTA

## Limitations

- Allowing a single mismatch to overcome sequencing errors may lead to base misidentification and inflation of false negatives, which directly affects mismatch-based modification calling
- Conventional RNA-seq suffers from tRNA modification biases; switching to tRNA-seq would be advantageous for modification detection
- HAMR-1.2 requires python2 to run, and bowtie/HAMR must be executed from the 'util' directory of the tncRNA-Toolkit tar package
- The detailed mechanisms of how these tRNA modifications affect cleavage regulation remain unclear, so modification calls are correlative rather than mechanistic

## Evidence

- [methods] processed reads were aligned to the artificial genome index using bowtie [60] with '-v2 --best' options: "To permit error-tolerant and uniquely mapping reads alignments, processed reads were aligned to the artificial genome index using bowtie [60] with ‘-v2 --best’ options."
- [methods] multi-mapped reads discarded if alignment occurred greater than 50 times: "multi-mapped reads were discarded if alignment occurred greater than 50 times, using bowtie1 with “--norc -v 0 -m 50” arguments."
- [results] artificial genome with 50 nt flanks as artificial chromosomes: "we created an artificial genome by masking the genuine tRNA gene region with 50 nt upstream and downstream in the genome (masked genome) and adding them as artificial chromosomes"
- [results] HAMR predicts modified nucleosides in high-throughput sequencing datasets: "we utilized HAMR [48], a high utility tool for predicting modified nucleosides in high throughput sequencing datasets"
- [readme] HAMR v1.2 provided in toolkit tar package: "[Note: bowtie1, and HAMR are already provided in tar package.]"
- [readme] HAMR-1.2 requires python2: "[Note: 1. Bowtie and HAMR will be excuted from 'util' directory. You still need if you have already globaly installed. 2. HAMR-1.2 required python2 for running.]"
