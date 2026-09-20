---
name: nucleotide-resolution-breakpoint-refinement
description: Use when nanopore fusion-gene candidates have been flagged from a first-pass
  SV VCF and exact, nucleotide-level breakpoint coordinates, breakpoint sequence,
  and breakpoint-spanning primers must be reported.
when_to_use_negative:
- Input is not long-read (nanopore) sequencing data — the pipeline is built on NanoSV's
  long-read SV calling and LAST remapping of noisy ONT reads
- Sample DNA is fragmented (e.g., FFPE material), which produces short reads that
  fail the FUDGE/NanoFG workflow
- Only one fusion-spanning read supports a candidate — such calls fall below the two-read
  cut-off and require manual breakpoint PCR validation rather than automated reporting
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Initial alignment of nanopore FASTQ reads to GRCh37 to produce the BAM used
    for first-pass SV screening
  repo: https://github.com/lh3/minimap2
- name: NanoSV
  role: 'Two-pass structural-variant caller: first screening pass with default parameters
    (min_mapq=12, depth_support=False, mapq_flag=48), second refinement pass with
    cluster_count=2 on LAST-remapped reads'
  repo: https://github.com/mroosmalen/nanosv
- name: Sniffles
  role: Alternative first-pass SV caller (v.1.0.9, defaults -s 2 -n -1 --genotype)
    whose VCF can seed the refinement stage
- name: LAST
  role: Remaps extracted fusion-spanning reads (v.921) for increased alignment accuracy
    prior to the second NanoSV pass
- name: Samtools
  role: Indexes and sorts the minimap2 BAM (v.1.7) and supports extraction of supporting
    reads
  repo: http://samtools.sourceforge.net/
- name: NanoFG
  role: Integrating pipeline that orchestrates screening, ENSEMBL gene annotation,
    fusion flagging, read extraction, LAST remapping, and breakpoint refinement; provides
    NanoFG.sh entry point
  repo: https://github.com/SdeBlank/NanoFG
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:560b9432140d2d60b2a9635eba7fcf13ebdc3a88e6616f8559a617b4ce4532c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - LAST
  - NanoFG
  - NanoSV
  - Samtools
  - Sniffles
  - minimap2
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nucleotide-resolution-breakpoint-refinement

## Summary

A two-stage SV-refinement strategy within the NanoFG pipeline: after an initial NanoSV (or Sniffles) structural-variant screen on minimap2-aligned nanopore reads, fusion-spanning reads are re-extracted, remapped with LAST for higher alignment accuracy, and re-screened with a second NanoSV pass (cluster_count=2) to refine fusion breakpoints at nucleotide resolution.

## When to use

Use when nanopore fusion-gene candidates have been flagged from a first-pass SV VCF and exact, nucleotide-level breakpoint coordinates, breakpoint sequence, and breakpoint-spanning primers must be reported.

## When NOT to use

- Input is not long-read (nanopore) sequencing data — the pipeline is built on NanoSV's long-read SV calling and LAST remapping of noisy ONT reads
- Sample DNA is fragmented (e.g., FFPE material), which produces short reads that fail the FUDGE/NanoFG workflow
- Only one fusion-spanning read supports a candidate — such calls fall below the two-read cut-off and require manual breakpoint PCR validation rather than automated reporting

## Inputs

- Sorted/indexed BAM of minimap2-aligned nanopore reads (GRCh37 reference)
- Initial NanoSV or Sniffles VCF of candidate structural variants (insertions removed)
- Extracted FASTQ/BAM of fusion-spanning supporting reads per candidate fusion

## Outputs

- Refined VCF with fusion breakpoints at nucleotide resolution
- Per-fusion breakpoint-location, breakpoint-sequence, and breakpoint-spanning primer information
- Fusion calls each supported by at least two fusion-spanning reads

## How to apply

After minimap2 (v.2.6) alignment to GRCh37 and indexing/sorting with Samtools (v.1.7), run NanoSV (v.1.2.4; min_mapq=12, depth_support=False, mapq_flag=48) or Sniffles (v.1.0.9; -s 2 -n -1 --genotype) to call candidate SVs, dropping SVs that fail built-in caller filters and all insertions from the VCF. Annotate both SV ends with ENSEMBL genes and flag SVs with ends in different genes as candidate fusions, retaining same-strand continuous-transcript candidates supported by at least two fusion-spanning reads. Extract the supporting reads for each candidate fusion from the BAM, then remap them with LAST (v.921), which yields more accurate alignments around the breakpoint than the original minimap2 mapping. Finally run a second NanoSV pass on the LAST-remapped reads with cluster_count=2 to pin the exact fusion breakpoints at nucleotide resolution in the output VCF.

## Related tools

- **minimap2** (Initial alignment of nanopore FASTQ reads to GRCh37 to produce the BAM used for first-pass SV screening) — https://github.com/lh3/minimap2
- **NanoSV** (Two-pass structural-variant caller: first screening pass with default parameters (min_mapq=12, depth_support=False, mapq_flag=48), second refinement pass with cluster_count=2 on LAST-remapped reads) — https://github.com/mroosmalen/nanosv
- **Sniffles** (Alternative first-pass SV caller (v.1.0.9, defaults -s 2 -n -1 --genotype) whose VCF can seed the refinement stage)
- **LAST** (Remaps extracted fusion-spanning reads (v.921) for increased alignment accuracy prior to the second NanoSV pass)
- **Samtools** (Indexes and sorts the minimap2 BAM (v.1.7) and supports extraction of supporting reads) — http://samtools.sourceforge.net/
- **NanoFG** (Integrating pipeline that orchestrates screening, ENSEMBL gene annotation, fusion flagging, read extraction, LAST remapping, and breakpoint refinement; provides NanoFG.sh entry point) — https://github.com/SdeBlank/NanoFG

## Examples

```
git clone https://github.com/SdeBlank/NanoFG.git NanoFG && bash NanoFG.sh -f /path/to/fastqdir -n SAMPLE_NAME
```

## Evaluation signals

- The refined output VCF reports the expected test-set fusion breakpoints for the NanoFG repository test data (~25000 reads, ~20 min single-threaded)
- Every reported fusion is supported by at least two fusion-spanning reads
- All insertions and SVs failing NanoSV/Sniffles built-in filters are absent from the intermediate VCF
- Refined breakpoints are at nucleotide resolution, with each SV end annotated to an ENSEMBL gene and candidate fusions having ends in different genes on the same strand
- The second NanoSV pass ran with cluster_count=2 on LAST-remapped reads, not on the original minimap2 alignments

## Limitations

- Two fusions in the study were detected with only one supporting read each and required manual breakpoint PCR validation
- Results depend on non-fragmented, high-quality DNA; FFPE-derived material with short read lengths will most likely fail to comprehensively identify fusion genes
- Runs the whole NanoFG module in roughly 20 min single-threaded for ~25000 reads; much larger datasets may need more compute

## Evidence

- [discussion] Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6): "Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6)"
- [discussion] NanoSV defaults: "The user can choose either NanoSV (v. 1.2.4)10 with default parameters: ‘min_mapq=12, depth_support=False, mapq_flag=48’"
- [discussion] Sniffles defaults: "or Sniffles (v.1.0.9)28 with default parameters: ‘-s 2 -n -1 --genotype’ to detect SVs."
- [discussion] Filter steps: "By default, all SVs that do not pass the built-in NanoSV or Sniffles filters are removed. In addition, all insertions are also removed from the VCF."
- [discussion] Read cut-off: "We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation."
- [discussion] Runtime: "The run time of NanoFG on +−25000 nanopore reads is approximately 20 min using a single thread."
- [readme] Clone command: "git clone https://github.com/SdeBlank/NanoFG.git NanoFG"
- [readme] Run command: "bash NanoFG.sh -f </path/to/fastqdir>  [-n SAMPLE_NAME ] [-s SELECTION] [-cf] [-cc] [-df] [-dc]"
