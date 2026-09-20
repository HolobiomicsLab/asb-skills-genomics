---
name: trna-gene-annotation
description: 'Use when building a tRNA reference for plant small RNA-seq analysis:
  a genome FASTA is available and tRNA-derived fragments (tRFs, tRNA halves) must
  be detected and classified against a curated tRNA gene pool.'
when_to_use_negative:
- Input is already a small RNA FASTQ to be classified — this skill only builds the
  tRNA reference/index; use the downstream tncRNA prediction step instead.
- The genome FASTA headers do not follow the '>chr[Num]' convention and cannot be
  renamed, since the Toolkit's scripts rely on 'chr' prefixes for automated nuclear/organellar
  separation.
- Working with human or animal tncRNA detection, where tools like tDRmapper, MINTmap,
  or tRF2Cancer (trained on human datasets) may be more appropriate.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: tRNAscan-SE
  role: Predicts nuclear tRNA genes (eukaryotes model) and organellar tRNAs ('-O'
    option); its score is used to filter pseudogenes and low-quality tRNAs (cutoff
    50)
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: bedtools
  role: Its 'maskfasta' script masks filtered tRNA regions plus 50 bp flanks in the
    reference genome (bedtools v2.29)
- name: bowtie1
  role: bowtie-build constructs the bowtie index of the artificial genome (bowtie
    v1.3) used for downstream tncRNA read alignment
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tncRNA Toolkit
  role: 'Orchestrates the reference construction: index-building mode (''tncRNAs.py
    -g <genome fasta> -s <species>'') creates the bowtie index and required files
    in lib/indexes/<species>'
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:015aa75509f9d0e4e6e61adeb2ac5bd060539e92db50b6e57a96a6f53b7fec91
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
  - bedtools
  - biopython
  - bowtie1
  - pandas
  - tRNAscan-SE
  - tRNAscanSE
  - tncRNA Toolkit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# trna-gene-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts nuclear and organellar tRNA genes with tRNAscan-SE, filters out pseudogenes and low-scoring tRNAs (score < 50), and constructs a masked artificial genome with mature/leader/trailer tRNA FASTA appendices for bowtie indexing in the tncRNA Toolkit.

## When to use

Use when building a tRNA reference for plant small RNA-seq analysis: a genome FASTA is available and tRNA-derived fragments (tRFs, tRNA halves) must be detected and classified against a curated tRNA gene pool.

## When NOT to use

- Input is already a small RNA FASTQ to be classified — this skill only builds the tRNA reference/index; use the downstream tncRNA prediction step instead.
- The genome FASTA headers do not follow the '>chr[Num]' convention and cannot be renamed, since the Toolkit's scripts rely on 'chr' prefixes for automated nuclear/organellar separation.
- Working with human or animal tncRNA detection, where tools like tDRmapper, MINTmap, or tRF2Cancer (trained on human datasets) may be more appropriate.

## Inputs

- Genome FASTA file (headers must start with '>chr[Num]', e.g. >chr1, >chr2; organellar as >chrMt and >chrPt)
- tRNAscan-SE prediction output for nuclear (eukaryotes model) and organellar ('-O') tRNA genes
- Species name used for the bowtie index directory lib/indexes/<species>

## Outputs

- Filtered tRNA annotation (non-pseudogene tRNAs with tRNAscan-SE score >= 50)
- Masked genome FASTA with tRNA regions plus 50 bp 5'/3' flanks masked
- Three tRNA FASTA files: mature tRNA (CCA-added, introns removed), leader tRNA, trailer tRNA
- Artificial genome FASTA and bowtie1 index stored in lib/indexes/<species name>

## How to apply

Run tRNAscan-SE with the eukaryotes model on the nuclear genome FASTA and use its '-O' option for organellar tRNA detection. Filter the tRNA gene pool by eliminating pseudogenes and any tRNA with a tRNAscan-SE score below 50, keeping only high-quality tRNAs. Mask the filtered tRNA regions together with a 50 bp flank on both the 5' and 3' sides in the reference genome using the bedtools (v2.29) 'maskfasta' script. Prepare three FASTA files — mature tRNA (introns removed, CCA added at the 3' terminus), leader tRNA (50 bp 5' genomic flank), and trailer tRNA (50 bp 3' genomic flank) — and append them to the masked genome as additional chromosomes with headers starting with 'chr' (e.g. '>chr1', '>chrMt', '>chrPt') to enable automated nuclear/organellar separation. Build the bowtie index of this artificial genome with bowtie-build (bowtie v1.3) via the Toolkit's index-building mode, placing it in lib/indexes/<species>; the rationale is that only reads mapping exclusively to the artificial tRNA chromosomes are later classified as tncRNAs, avoiding ambiguous reads from non-tRNA regions.

## Related tools

- **tRNAscan-SE** (Predicts nuclear tRNA genes (eukaryotes model) and organellar tRNAs ('-O' option); its score is used to filter pseudogenes and low-quality tRNAs (cutoff 50)) — https://github.com/skbinfo/tncRNA-Toolkit
- **bedtools** (Its 'maskfasta' script masks filtered tRNA regions plus 50 bp flanks in the reference genome (bedtools v2.29))
- **bowtie1** (bowtie-build constructs the bowtie index of the artificial genome (bowtie v1.3) used for downstream tncRNA read alignment) — https://github.com/skbinfo/tncRNA-Toolkit
- **tncRNA Toolkit** (Orchestrates the reference construction: index-building mode ('tncRNAs.py -g <genome fasta> -s <species>') creates the bowtie index and required files in lib/indexes/<species>) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python3 tncRNAs.py -g <genome fasta> -s <species name>
```

## Evaluation signals

- Only non-pseudogene tRNAs with a tRNAscan-SE score equal to or above 50 are retained in the annotation.
- Genome FASTA headers all start with '>chr[Num]' (and '>chrMt'/'>chrPt' for mitochondria/plastids) as required for automated nuclear/organellar separation.
- The bowtie index for the artificial genome is successfully built and placed in lib/indexes/<species name>.
- The masked genome contains tRNA regions plus 50 bp flanks on both 5' and 3' sides, and the three appended tRNA FASTA files (mature, leader, trailer) are present as additional chromosomes.

## Limitations

- Allowing a single mismatch during read mapping (to overcome sequencing errors) can cause base misidentification and inflate false negatives — relevant to how the reference is later used.
- tRNAscan-SE, samtools, and bedtools must be globally installed or on the PATH; bowtie1 and HAMR must be compiled/extracted in the 'util' directory (HAMR 1.2 requires python2).
- Genome FASTA headers must start with '>chr[Num]'; non-conforming headers break the Toolkit's automated nuclear/organellar separation.

## Evidence
