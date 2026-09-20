---
name: gap-filled-scaffold-file-handling
description: Use when a Basic Protocol 2 gap-filled ntLink scaffold file (e.g., celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa)
  carries raw long-read sequence in its filled gaps and must be consensus-polished
  and evaluated before downstream use.
when_to_use_negative:
- The assembly was scaffolded without gap-filling (e.g., a Basic Protocol 1 overlap-detection
  scaffold or a draft contig FASTA) and no raw reads were inserted into gaps — this
  Racon-polish step is documented for the gap-filled output.
- The sequencing data are not Oxford Nanopore reads — the minimap2 map-ont preset
  used here is ONT-specific.
- The goal is generating ntLink mappings for downstream Tigmint-long correction (Alternate
  Protocol 2) rather than polishing a gap-filled assembly.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntLink
  role: Produces the gap-filled scaffold FASTA (Basic Protocol 2, k32/w100/z1000 parameters)
    that this skill polishes
  repo: https://github.com/bcgsc/ntLink
- name: minimap2
  role: Aligns the ONT long reads to the gap-filled scaffolds with -a -t 5 -x map-ont
    to generate the SAM alignments Racon consumes
  repo: https://github.com/lh3/minimap2
- name: Racon
  role: Consensus polishing of the gap-filled scaffolds, run with -u -t 5 to write
    the racon-polished FASTA
  repo: https://github.com/lbcb-sci/racon
- name: sra-tools
  role: Downloads the ONT long-read FASTQ for accession SRR10028109
  repo: https://github.com/ncbi/sra-tools
- name: QUAST
  role: Computes contiguity and correctness statistics on the polished assembly for
    comparison with Table 4
  repo: https://github.com/ablab/quast
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:15128ce239977de11f2aa2862d02a0748a6f78987f4b895cca10ab2ae583deca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - QUAST
  - Racon
  - minimap2
  - ntLink
  - sra-tools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# gap-filled-scaffold-file-handling

## Summary

Polishes ntLink gap-filled scaffold FASTAs — because gap-filling inserts raw, uncorrected ONT read sequence into scaffold gaps — by realigning the long reads with minimap2 (-a -t 5 -x map-ont) and running Racon (-u -t 5), then checking contiguity/correctness with QUAST against Table 4.

## When to use

Use when a Basic Protocol 2 gap-filled ntLink scaffold file (e.g., celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa) carries raw long-read sequence in its filled gaps and must be consensus-polished and evaluated before downstream use.

## When NOT to use

- The assembly was scaffolded without gap-filling (e.g., a Basic Protocol 1 overlap-detection scaffold or a draft contig FASTA) and no raw reads were inserted into gaps — this Racon-polish step is documented for the gap-filled output.
- The sequencing data are not Oxford Nanopore reads — the minimap2 map-ont preset used here is ONT-specific.
- The goal is generating ntLink mappings for downstream Tigmint-long correction (Alternate Protocol 2) rather than polishing a gap-filled assembly.

## Inputs

- Gap-filled ntLink scaffold FASTA from Basic Protocol 2 (celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa)
- ONT long-read FASTQ for SRA accession SRR10028109 (C. elegans)

## Outputs

- Polished scaffold FASTA: celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.racon-polished.fa
- minimap2 SAM file of ONT read-to-scaffold alignments
- QUAST contiguity and correctness statistics report for the polished assembly (compared with Table 4)

## How to apply

First fetch the ONT long-read FASTQ for accession SRR10028109 using sra-tools. Align these reads to the gap-filled scaffolds celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa with minimap2 using the map-ont preset (-a -t 5) to produce a SAM file of read-to-scaffold alignments. Because ntLink gap-filling incorporates raw long-read sequence into the scaffold gaps, run Racon with -u -t 5 on the reads, the alignments, and the gap-filled scaffold FASTA to write celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.racon-polished.fa. Finally, run QUAST on the polished FASTA to compute contiguity and correctness statistics and compare them against Table 4 of the paper; the input assembly was produced by ntLink with k32/w100/z1000 parameters on a C. elegans Flye draft.

## Related tools

- **ntLink** (Produces the gap-filled scaffold FASTA (Basic Protocol 2, k32/w100/z1000 parameters) that this skill polishes) — https://github.com/bcgsc/ntLink
- **minimap2** (Aligns the ONT long reads to the gap-filled scaffolds with -a -t 5 -x map-ont to generate the SAM alignments Racon consumes) — https://github.com/lh3/minimap2
- **Racon** (Consensus polishing of the gap-filled scaffolds, run with -u -t 5 to write the racon-polished FASTA) — https://github.com/lbcb-sci/racon
- **sra-tools** (Downloads the ONT long-read FASTQ for accession SRR10028109) — https://github.com/ncbi/sra-tools
- **QUAST** (Computes contiguity and correctness statistics on the polished assembly for comparison with Table 4) — https://github.com/ablab/quast

## Examples

```
racon -u -t 5 SRR10028109.fastq aln.sam celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa > celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.racon-polished.fa
```

## Evaluation signals

- The output FASTA celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.racon-polished.fa exists and retains the scaffold set/naming of the gap-filled input.
- A non-empty SAM file of read-to-scaffold alignments was produced by minimap2 with the map-ont preset against the gap-filled scaffolds.
- Racon was invoked with -u -t 5 on the reads, alignments, and gap-filled scaffold FASTA, writing the racon-polished FASTA.
- QUAST statistics on the polished assembly are consistent with the contiguity and correctness values reported in Table 4 of the paper.

## Limitations

- The protocol documents a single Racon pass (-u -t 5); no iterative polishing scheme is specified.
- The workflow is demonstrated on C. elegans ONT data (SRR10028109, Flye assembly, ntLink k32/w100/z1000); other read chemistries, organisms, or assemblers would require adjusted minimap2/Racon parameters.
- Validation by comparison to Table 4 depends on reproducing the same input assembly and read set.

## Evidence

- [abstract] Gap-filling is a recent ntLink feature, motivating post-gap-fill polishing: "Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations"
- [methods] The polishing step relies on long-read mapping to the scaffold assembly: "2. Use minimizers to map the long reads to the target assembly contigs"
- [readme] Racon is designed as a consensus module for raw long-read assemblies, fitting the raw sequence inserted by gap-filling: [evidence span withheld — anchor status: external_doc]
