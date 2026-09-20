---
name: repeat-library-conversion-for-graph-genomes
description: Use when preparing a repeat library for graph-genome-based TE detection
  (e.g. GraffiTE with the 'pangenie' graph method), starting from a Dfam release that
  must be converted to the tool's required format.
when_to_use_negative:
- The downstream TE-calling tool is RetroInspector's Snakemake pipeline, which aligns
  reads with minimap2 against GRCh38 and calls variants with cuteSV+Sniffles2 rather
  than consuming a graph-genome repeat library
- The analysis targets short-read TE detection with tools like MELT or xTea, which
  do not use the FamDB/Dfam graph-genome library path
- You only need repeat annotation (RepeatMasker-style) and no graph-genome TE caller,
  in which case curated consensus (cc) FamDB components can be used directly without
  a GraffiTE-specific conversion
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: FamDB
  role: Query/convert Dfam families by taxonomy into the format required for graph-genome
    TE detection with GraffiTE
  repo: https://github.com/Dfam-consortium/FamDB
- name: GraffiTE
  role: Graph-genome TE insertion caller that consumes the converted library; configured
    with 'pangenie' graph method, --longreads and 'ont' entries
- name: Dfam
  role: Source database of transposable element family sequence models (profile HMMs
    and consensus sequences) exported as FamDB files
  repo: https://www.dfam.org/
- name: h5py
  role: Python dependency for reading/writing the HDF5-based FamDB files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:cce6b9d5d0bd88fafbb3be9f9392e54fbba569a84db1daeda8afae5733e86355
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/Dfam-consortium/FamDB
  tools:
  - Dfam
  - FamDB
  - FamDB toolkit
  - GNU parallel
  - GraffiTE
  - PALMER
  - PanGenIE
  - RetroInspector
  - Sniffles2
  - cuteSV
  - h5py
  - minimap2
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9310344827586207
n_claims: 29
---

# repeat-library-conversion-for-graph-genomes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts a complete Dfam transposable-element release into the graph-genome repeat-library format consumed by GraffiTE using the FamDB toolkit, extracting species-specific (human and ancestor) repeat families for TE insertion detection on nanopore reads.

## When to use

Use when preparing a repeat library for graph-genome-based TE detection (e.g. GraffiTE with the 'pangenie' graph method), starting from a Dfam release that must be converted to the tool's required format.

## When NOT to use

- The downstream TE-calling tool is RetroInspector's Snakemake pipeline, which aligns reads with minimap2 against GRCh38 and calls variants with cuteSV+Sniffles2 rather than consuming a graph-genome repeat library
- The analysis targets short-read TE detection with tools like MELT or xTea, which do not use the FamDB/Dfam graph-genome library path
- You only need repeat annotation (RepeatMasker-style) and no graph-genome TE caller, in which case curated consensus (cc) FamDB components can be used directly without a GraffiTE-specific conversion

## Inputs

- Complete Dfam release FamDB HDF5 files (root <base>.0.h5 plus cc/ch/uc/uh component partitions, e.g. Dfam version 3.8)
- Taxon term for filtering repeat families (e.g. 'Homo sapiens' or NCBI taxonomy ID 9606)
- Reference genome FASTA (GRCh38) for graph construction
- Nanopore FASTQ files (one per sample)

## Outputs

- Converted repeat library in the format required by GraffiTE (human and ancestor repeat families)
- Graph-genome TE insertion calls per sample (evaluated against a truth set with SURVIVOR at 100 bp or 500 bp distance margins)

## How to apply

Obtain a complete Dfam release (the benchmark used version 3.8) as FamDB files: a root <base>.0.h5 file plus curated/uncurated consensus and HMM component partitions, all placed in the same directory because files from the same export must co-locate. Install the FamDB toolkit (version 1.0.2) with Python 3.6+ and h5py (`pip3 install --user h5py`), then use famdb.py to query the taxonomy (e.g. 'Homo sapiens') and export the human and ancestor repeats in the format GraffiTE requires. Select GRCh38 as the reference genome when configuring the graph build, choose 'pangenie' as the graph method, and supply the nanopore FASTQ paths with 'ont' entries under --longreads. The rationale is that GraffiTE relies on graph genomes built from repeat libraries, and FamDB provides a taxonomy-filtered, HDF5-backed subset of Dfam (profile HMMs and consensus sequences with metadata) suitable for that conversion.

## Related tools

- **FamDB** (Query/convert Dfam families by taxonomy into the format required for graph-genome TE detection with GraffiTE) — https://github.com/Dfam-consortium/FamDB
- **GraffiTE** (Graph-genome TE insertion caller that consumes the converted library; configured with 'pangenie' graph method, --longreads and 'ont' entries)
- **Dfam** (Source database of transposable element family sequence models (profile HMMs and consensus sequences) exported as FamDB files) — https://www.dfam.org/
- **h5py** (Python dependency for reading/writing the HDF5-based FamDB files)

## Examples

```
pip3 install --user h5py; famdb.py -i /data/famdb names 'Homo sapiens'
```

## Evaluation signals

- `famdb.py -i DB_DIR info` reports the expected Dfam release version (e.g. 3.8), creation format version, and family counts for every installed component partition
- `famdb.py -i DB_DIR check 'Homo sapiens'` confirms that all partition files required for the human/ancestor query are locally present before conversion
- All FamDB files come from a single export run in one directory (FamDB warns if files from different export runs are mixed)
- The converted library reproduces GraffiTE benchmark behavior: high recall (~96.18%) with very low precision (32.33%, F1 48.40%) against the 5,630-insertion HGSVC truth set using SURVIVOR matching

## Limitations

- Some recently described TE families are absent from Dfam (e.g. the newer SVA F1 family, which RetroInspector instead identifies via the MAST2 thumbprint check), so the converted library cannot label those insertions
- FamDB requires Python 3.6+ and h5py; all files from one export must reside in the same directory or queries will warn on mixed-export directories
- In the comparative benchmark, GraffiTE ran on 96 threads despite being assigned 32, so it was excluded from runtime comparisons

## Evidence

- [readme] FamDB is a modular HDF5-based export format and query tool developed for offline access to the [Dfam] database of transposable element and repetitive DNA families.: [evidence span withheld — anchor status: external_doc]
- [readme] All files from the same export must reside in the same directory.: [evidence span withheld — anchor status: external_doc]
