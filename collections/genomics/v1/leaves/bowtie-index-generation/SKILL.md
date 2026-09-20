---
name: bowtie-index-generation
description: Use when building a bowtie1 index of a reference genome augmented with
  tRNA-derived artificial chromosomes so that small RNA reads mapping exclusively
  to tRNA loci can be classified as tncRNAs (tRF-5, tRF-3, tRF-1, tRNA halves).
when_to_use_negative:
- The reference is a transcriptome or already-annotated mature tRNA set without genomic
  context — no masking of tRNA regions plus flanks can be performed, so the artificial-genome
  logic does not apply
- Genome FASTA headers do not start with 'chr[Num]' (e.g. >chr1, >chrMt, >chrPt),
  preventing automated nuclear/organellar chromosome separation by the toolkit scripts
- The analysis targets human datasets with tools trained on human data (e.g. tDRmapper,
  MINTmap, tRF2Cancer) rather than plant tncRNA profiling
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0659
tools:
- name: tRNAscan-SE
  role: Predicts nuclear and organellar tRNA genes from the genome FASTA; predictions
    are filtered to remove pseudogenes and low-scoring tRNAs (score < 50) before index
    construction
- name: bedtools
  role: Uses the 'maskfasta' script (v2.29) to mask genuine tRNA regions plus 50 bp
    5'/3' flanks in the reference genome
- name: bowtie1
  role: bowtie-build (v1.3) creates the index of the artificial genome placed in lib/indexes/<species>;
    later used with '-v2 --best' and '--norc -v 0 -m 50' for read alignment
- name: tncRNA Toolkit
  role: Orchestrates the whole index-build step via 'python3 tncRNAs.py -g <genome
    fasta> -s <species name>'
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:1a97d4cabe95e34dae4e3282974fdaba01e3ace05db573c83697fc96296a5b03
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
  - bowtie1
  - tRNAscan-SE
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bowtie-index-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a bowtie1 index over an 'artificial genome' in which genuine tRNA gene regions (predicted by tRNAscan-SE, score >= 50, pseudogenes removed) plus 50 nt leader/trailer flanks are appended as extra chromosomes onto a masked reference genome, enabling exclusive-to-tRNA read mapping for tncRNA detection.

## When to use

Use when building a bowtie1 index of a reference genome augmented with tRNA-derived artificial chromosomes so that small RNA reads mapping exclusively to tRNA loci can be classified as tncRNAs (tRF-5, tRF-3, tRF-1, tRNA halves).

## When NOT to use

- The reference is a transcriptome or already-annotated mature tRNA set without genomic context — no masking of tRNA regions plus flanks can be performed, so the artificial-genome logic does not apply
- Genome FASTA headers do not start with 'chr[Num]' (e.g. >chr1, >chrMt, >chrPt), preventing automated nuclear/organellar chromosome separation by the toolkit scripts
- The analysis targets human datasets with tools trained on human data (e.g. tDRmapper, MINTmap, tRF2Cancer) rather than plant tncRNA profiling

## Inputs

- Genome FASTA with headers starting '>chr[Num]' (e.g. >chr1, >chr2; organellar as >chrMt, >chrPt), e.g. TAIR10.1 for Arabidopsis thaliana
- tRNAscan-SE (v2.0.6) prediction output (eukaryote and organellar '-O' models)
- Genome annotation of tRNA coordinates for bedtools maskfasta masking

## Outputs

- Bowtie1 index directory at lib/indexes/<species name> for the artificial genome
- Masked genome FASTA with genuine tRNA regions plus 50 bp flanks masked
- Three appended tRNA FASTA files as artificial chromosomes: mature tRNA (CCA-added), leader tRNA (50 bp 5' flank), trailer tRNA (50 bp 3' flank)

## How to apply

Run tRNAscan-SE (eukaryote model; '-O' for organellar tRNAs) on the genome FASTA, then filter predictions by eliminating pseudogenes and any tRNA with a tRNAscan-SE score below 50. Mask the retained tRNA regions plus 50 bp 5' and 3' flanks in the reference genome using the bedtools (v2.29) 'maskfasta' script. Prepare three FASTA files — mature tRNA (introns removed, CCA added at the 3' terminus), leader tRNA (50 bp 5' genomic flank), and trailer tRNA (50 bp 3' genomic flank) — and append them to the masked genome as 'additional chromosomes', with headers starting with 'chr' (e.g. >chr1, >chrMt, >chrPt) to enable automated nuclear/organellar separation. Build the bowtie index with bowtie-build (bowtie v1.3) into lib/indexes/<species>; in the tncRNA Toolkit this happens automatically via 'python3 tncRNAs.py -g <genome fasta> -s <species name>'. The rationale for masking is to force reads to map exclusively to the artificial tRNA chromosomes, avoiding ambiguous assignments from non-tRNA regions.

## Related tools

- **tRNAscan-SE** (Predicts nuclear and organellar tRNA genes from the genome FASTA; predictions are filtered to remove pseudogenes and low-scoring tRNAs (score < 50) before index construction)
- **bedtools** (Uses the 'maskfasta' script (v2.29) to mask genuine tRNA regions plus 50 bp 5'/3' flanks in the reference genome)
- **bowtie1** (bowtie-build (v1.3) creates the index of the artificial genome placed in lib/indexes/<species>; later used with '-v2 --best' and '--norc -v 0 -m 50' for read alignment)
- **tncRNA Toolkit** (Orchestrates the whole index-build step via 'python3 tncRNAs.py -g <genome fasta> -s <species name>') — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python3 tncRNAs.py -g genome.fa -s arabidopsis
```

## Evaluation signals

- Only non-pseudogene tRNAs with tRNAscan-SE score equal to or above 50 are retained in the annotation
- The bowtie index for the artificial genome is successfully built and located in lib/indexes/<species>
- Appended tRNA FASTA headers start with 'chr' so nuclear and organellar chromosomes can be separated automatically
- Genome FASTA headers follow the '>chr[Num]' convention (nuclear >chr1, >chr2...; organellar >chrMt, >chrPt) so downstream automation works
- Masked genome contains no unmasked tRNA regions, and the artificial chromosomes include exactly 50 nt upstream and downstream flanks

## Limitations

- Genome FASTA headers must start with '>chr[Num]'; mitochondrial and plastid headers must be >chrMt and >chrPt — non-conforming headers break automated nuclear/organellar separation
- tRNAscan-SE, samtools, bedtools must be globally installed or in PATH; bowtie1 and HAMR are provided in the tar package but must be built (bowtie via 'make' in util/), and HAMR-1.2 additionally requires python2
- Masking tRNA regions with flanks means only reads mapping exclusively to the artificial chromosomes are classified as tncRNAs; ambiguous reads from non-tRNA regions are excluded by design

## Evidence

- [methods] The reference construction starts from tRNA gene copies predicted by tRNAscanSE: "The tRNA pseudogenes and tRNAs with a score less than 50 were eliminated"
- [readme] README: dependencies and index location: [evidence span withheld — anchor status: not_found]
