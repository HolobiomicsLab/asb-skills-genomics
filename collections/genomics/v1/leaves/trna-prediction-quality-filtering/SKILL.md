---
name: trna-prediction-quality-filtering
description: Use when constructing a tRNA reference set for tncRNA detection and tRNAscan-SE
  predictions must be filtered to eliminate pseudogenes and low-scoring tRNAs before
  genome masking and index building.
when_to_use_negative:
- The input is already a curated tRNA annotation or GtRNAdb gene set rather than raw
  tRNAscan-SE predictions needing pseudogene removal
- The goal is tncRNA read classification/quantification itself rather than reference
  curation — that downstream step uses bowtie '-v2 --best' alignment against the already-built
  artificial genome index
- The organism's genome headers do not follow the '>chr' naming convention required
  by tncRNA Toolkit automation, since nuclear/organellar separation depends on it
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: tRNAscan-SE
  role: Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNA genes
    from the genome FASTA and provides the scores used to filter pseudogenes and low-quality
    tRNAs
- name: tncRNA Toolkit
  role: Wraps the filtering step; consumes the filtered tRNA pool to mask tRNA regions
    plus 50 bp flanks, build mature/leader/trailer tRNA FASTAs, and create the bowtie
    index in lib/indexes/<species>
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: bedtools
  role: Uses the filtered tRNA annotation (maskBed) to mask genuine tRNA regions plus
    50 bp 5'/3' flanks in the reference genome via the 'maskfasta' script
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:c97ee7e6591fa172b413b40fe874264fd8911d759840e0d845c26c50712d66ba
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
  - bedtools
  - tRNAscan-SE
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_3226
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# trna-prediction-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-filtering step of the tncRNA Toolkit reference build that removes tRNA pseudogenes and low-scoring predictions from the tRNAscan-SE gene pool so only genuine, high-confidence tRNAs (score ≥ 50) feed the artificial genome index.

## When to use

Use when constructing a tRNA reference set for tncRNA detection and tRNAscan-SE predictions must be filtered to eliminate pseudogenes and low-scoring tRNAs before genome masking and index building.

## When NOT to use

- The input is already a curated tRNA annotation or GtRNAdb gene set rather than raw tRNAscan-SE predictions needing pseudogene removal
- The goal is tncRNA read classification/quantification itself rather than reference curation — that downstream step uses bowtie '-v2 --best' alignment against the already-built artificial genome index
- The organism's genome headers do not follow the '>chr' naming convention required by tncRNA Toolkit automation, since nuclear/organellar separation depends on it

## Inputs

- Genome FASTA (e.g. TAIR10.1 for Arabidopsis thaliana; SL4.0 nuclear / SL3.0 organellar for tomato) with headers starting with '>chr[Num]' and '>chrMt'/' >chrPt' for organellar contigs
- tRNAscan-SE prediction output (tRNA gene coordinates, isotypes, pseudogene flags, scores)

## Outputs

- Filtered tRNA annotation containing only non-pseudogene tRNAs with tRNAscan-SE score ≥ 50
- Filtered tRNA gene pool feeding artificial-genome construction (masked genome plus mature/leader/trailer tRNA FASTAs) and the bowtie index at lib/indexes/<species>

## How to apply

Run tRNAscan-SE (eukaryotes model) on the genome FASTA to predict nuclear tRNA genes, using its '-O' option for organellar tRNA detection. Filter the resulting gene pool by eliminating tRNA pseudogenes and any prediction with a tRNAscan-SE score below 50, so that only high-quality tRNAs with a score equal to or above 50 are selected for mapping. The retained tRNAs are then used downstream to mask genuine tRNA regions plus 50 bp 5'/3' flanks in the reference genome (bedtools maskfasta v2.29) and to build the mature/leader/trailer FASTA files appended as artificial chromosomes to the masked genome before bowtie-build (bowtie v1.3). Validation: confirm only non-pseudogene tRNAs with score ≥ 50 remain in the annotation and that the bowtie index for the artificial genome was successfully built in lib/indexes/<species>.

## Related tools

- **tRNAscan-SE** (Predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNA genes from the genome FASTA and provides the scores used to filter pseudogenes and low-quality tRNAs)
- **tncRNA Toolkit** (Wraps the filtering step; consumes the filtered tRNA pool to mask tRNA regions plus 50 bp flanks, build mature/leader/trailer tRNA FASTAs, and create the bowtie index in lib/indexes/<species>) — https://github.com/skbinfo/tncRNA-Toolkit
- **bedtools** (Uses the filtered tRNA annotation (maskBed) to mask genuine tRNA regions plus 50 bp 5'/3' flanks in the reference genome via the 'maskfasta' script)

## Examples

```
python3 tncRNAs.py -g <genome fasta> -s <species name>
```

## Evaluation signals

- Every retained tRNA in the filtered annotation is non-pseudogene and has a tRNAscan-SE score equal to or above 50
- All tRNAs with score < 50 and all pseudogenes have been eliminated from the gene pool
- The masked genome excludes filtered tRNA regions plus 50 bp flanks on both 5' and 3' sides
- The bowtie (v1.3) index of the artificial genome was successfully built and placed in lib/indexes/<species>
- Artificial chromosome FASTA headers start with 'chr' so nuclear/organellar separation is automated downstream

## Limitations

- The score ≥ 50 cut-off is a hard threshold; tRNAs with lower tRNAscan-SE scores are discarded even if biologically real
- Genome FASTA headers must start with '>chr[Num]' (e.g. >chr1, >chrMt, >chrPt) or the toolkit's automated nuclear/organellar separation breaks
- tRNAscan-SE (v2.0.6), samtools (v1.10), bedtools (v2.29.2), python3, and bowtie1 (v1.3.0) must be globally installed or in PATH; HAMR-1.2 additionally requires python2

## Evidence
