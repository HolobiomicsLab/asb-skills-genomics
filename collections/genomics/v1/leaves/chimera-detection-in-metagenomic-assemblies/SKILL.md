---
name: chimera-detection-in-metagenomic-assemblies
description: Use when evaluating whether an assembly or contig-merging step created
  or removed chimeras, and per-reference reference genomes are available for Nucmer
  contig alignment and chimera flagging.
when_to_use_negative:
- No reference genomes are available — chimera flagging here depends on Nucmer alignment
  to references and cannot be performed from reads alone
- The question is total assembly size or N50/N90, which the authors describe as 'not
  very useful for comparing metagenome assembly performance' and which are additionally
  affected by removal of small contigs
- Input contigs are <2 kb — the MeGAMerge workflow removes these before merging, so
  they would not be part of the merged contig set being evaluated
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Nucmer
  role: Aligns contigs to reference genome sequences to obtain reference coverage
    and detect non-contiguous (chimeric) alignments
- name: MeGAMerge
  role: Merging pipeline whose outputs are evaluated for chimera introduction and
    removal; final contigs written with -o (default MergedContigs.fasta)
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Minimus2
  role: Performs the OLC-based merge whose chimeric joins are being quantified
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:92808dbb5437da0bd4ca183304639e2b0b0d7817f8c441ebe6705d671a0dec39
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/LANL-Bioinformatics/MeGAMerge
  repo_url: https://github.com/LANL-Bioinformatics/MeGAMerge
  tools:
  - AMOS toolbox
  - IDBA
  - MeGAMerge
  - MetaSim
  - Minimus2
  - Nucmer
  - Ray
  - SOAPdenovo
  - Velvet
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# chimera-detection-in-metagenomic-assemblies

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies chimeric contigs in metagenomic assemblies by aligning contigs to reference genomes with Nucmer and flagging any contig that does not contiguously align to a single reference genome, enabling quantification of chimeras introduced or removed by post-assembly merging (e.g. MeGAMerge).

## When to use

Use when evaluating whether an assembly or contig-merging step created or removed chimeras, and per-reference reference genomes are available for Nucmer contig alignment and chimera flagging.

## When NOT to use

- No reference genomes are available — chimera flagging here depends on Nucmer alignment to references and cannot be performed from reads alone
- The question is total assembly size or N50/N90, which the authors describe as 'not very useful for comparing metagenome assembly performance' and which are additionally affected by removal of small contigs
- Input contigs are <2 kb — the MeGAMerge workflow removes these before merging, so they would not be part of the merged contig set being evaluated

## Inputs

- Multi-FASTA contig files from individual assemblers, the dereplicated pool, and the merged output (e.g. MergedContigs.fasta)
- Reference genome sequences for the community (e.g. the 1,000 simulated reference genomes)
- Nucmer alignment coordinates/delta output between contigs and references

## Outputs

- Set of contigs flagged as chimeric (not contiguously aligned to a single reference genome) with chimera locations identified
- Chimera counts before merging, after merging, and carried over from dereplicated inputs
- Per-reference coverage rows usable in validation_summary.csv

## How to apply

Align all assembled contigs (input assemblies, dereplicated pools, and merged outputs such as MeGAMerge's MergedContigs.fasta) to the reference genome set using Nucmer. Flag contigs that did not contiguously align to a single reference genome as chimeric and identify the locations of the chimeras. Tally chimeras at three checkpoints: before merging (in the dereplicated inputs), after merging (in the final contig set), and chimeras carried over from the inputs, so that new chimeras introduced by the merging process can be separated from those inherited. On the simulated 1,000-genome dataset this distinguished the 10,087 chimeras input from 9,225 after merging, with 6,940 carried over, showing merging introduces new chimeras yet reduces the total input count.

## Related tools

- **Nucmer** (Aligns contigs to reference genome sequences to obtain reference coverage and detect non-contiguous (chimeric) alignments)
- **MeGAMerge** (Merging pipeline whose outputs are evaluated for chimera introduction and removal; final contigs written with -o (default MergedContigs.fasta)) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Minimus2** (Performs the OLC-based merge whose chimeric joins are being quantified)

## Examples

```
nucmer -prefix refalign references.fasta MergedContigs.fasta && delta-filter -r -q refalign.delta > refalign.filter.delta && show-coords -r refalign.filter.delta > refalign.coords
```

## Evaluation signals

- Chimera count after merging (e.g. 9,225) is lower than chimeras input before merging (e.g. 10,087), with carried-over chimeras quantified separately (e.g. 6,940)
- Every flagged chimera corresponds to a contig whose Nucmer alignment is not contiguous to a single reference genome
- Reference coverage after merging is maintained versus the dereplicated input (80.25% vs 80.26% on the 1,000-genome simulation)
- validation_summary.csv contains one per-genome coverage row for each of the 1,000 reference genomes
- Unmapped contig count drops after merging (318 dereplicated inputs vs 114 merged, parsed from BWA/SAMtools BAM)

## Limitations

- The merging process does introduce new chimeras, even though MeGAMerge reduces the total number of chimeras input
- Chimera detection is reference-based and requires reference genomes, limiting applicability to simulations or well-characterized communities
- Overlap-based consensus tools' run-times increase exponentially with input dataset size, constraining how large a contig pool can be chimera-profiled in the pipeline

## Evidence
