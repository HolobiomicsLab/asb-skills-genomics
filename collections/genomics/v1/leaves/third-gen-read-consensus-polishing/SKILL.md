---
name: third-gen-read-consensus-polishing
description: Use when a DBG2OLC backbone_raw.fasta satisfies its N50 check and needs
  base-level error correction using the original PacBio reads plus DBG contigs before
  final QUAST/AssemblyStatistics evaluation.
when_to_use_negative:
- The backbone N50 is still unsatisfactory — the README says to keep tuning DBG2OLC
  parameters and 'don't proceed' to consensus
- The short-read contigs came from a pipeline that applied repeat-resolving heuristics
  such as gap closing or scaffolding, which introduces errors that degrade the final
  result
- The goal is a purely short-read (Illumina-only) assembly with no long-read data
  for consensus
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: BLASR
  role: Aligns all raw PacBio reads to the assembly backbone using -minMatch 19 to
    produce .m5 mappings for consensus
- name: Sparc
  role: Sparsity-based consensus caller that corrects backbone base-level errors from
    the BLASR mappings, optionally boosting high-quality contig reads (HQ_Prefix Contig,
    boost 5)
  repo: https://github.com/yechengxi/Sparc
- name: DBG2OLC
  role: Produces backbone_raw.fasta and DBG2OLC_Consensus_info.txt that define the
    layout to be polished
  repo: https://github.com/yechengxi/DBG2OLC
- name: SparseAssembler
  role: Provides the accurate DBG contigs (Contigs.txt) that serve as high-quality
    inputs to the hybrid consensus
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: AssemblyStatistics evaluates the polished assembly N50 and contig statistics
  repo: https://github.com/yechengxi/AssemblyUtility
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:8581c556a136096c405fea4f5a41115b8669cfba3837b9274efeb4187904b72a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/Sparc
  tools:
  - AssemblyUtility
  - BLASR
  - DBG2OLC
  - QUAST 3.0
  - Sparc
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# third-gen-read-consensus-polishing

## Summary

Polishes a structurally correct long-read assembly backbone (e.g. DBG2OLC's backbone_raw.fasta) by aligning all raw PacBio reads with BLASR and calling a sparsity-based consensus with Sparc, the step where ~90% of DBG2OLC pipeline runtime is spent.

## When to use

Use when a DBG2OLC backbone_raw.fasta satisfies its N50 check and needs base-level error correction using the original PacBio reads plus DBG contigs before final QUAST/AssemblyStatistics evaluation.

## When NOT to use

- The backbone N50 is still unsatisfactory — the README says to keep tuning DBG2OLC parameters and 'don't proceed' to consensus
- The short-read contigs came from a pipeline that applied repeat-resolving heuristics such as gap closing or scaffolding, which introduces errors that degrade the final result
- The goal is a purely short-read (Illumina-only) assembly with no long-read data for consensus

## Inputs

- backbone_raw.fasta produced by DBG2OLC (Step2 overlap and layout)
- DBG2OLC_Consensus_info.txt read-to-contig layout file
- Contigs.txt — raw DBG contigs from SparseAssembler (no gap-closing/scaffolding)
- Raw PacBio long reads in FASTA format (e.g. Pacbio_20x.fasta)
- ctg_pb.fasta — concatenation of Contigs.txt and PacBio reads

## Outputs

- Polished consensus assembly (consensus directory output of split_and_run_sparc.sh)
- backbone.mapped.m5 BLASR alignment file (-m 5 format, -minMatch 19)
- Assembly statistics (N50, contig counts) from AssemblyStatistics or QUAST 3.0 recorded in a metrics file

## How to apply

Only proceed to consensus after checking the backbone N50 of backbone_raw.fasta; if unsatisfactory, keep retuning DBG2OLC parameters (KmerCovTh 2-5, MinOverlap 10-30, AdaptiveTh 0.001-0.01 for 10x-20x PacBio data) rather than polishing a bad backbone. Concatenate the SparseAssembler DBG contigs (Contigs.txt) and raw PacBio reads into ctg_pb.fasta with cat, raise the file-descriptor limit with 'ulimit -n unlimited' because the input is split into many smaller files, then run split_and_run_sparc.sh with backbone_raw.fasta, DBG2OLC_Consensus_info.txt, ctg_pb.fasta and an output directory. Internally BLASR aligns raw reads to the backbone using -minMatch 19 (e.g. -bestn 1 -m 5 output) and Sparc computes the consensus from the .m5 mappings. This consensus step consumes ~90% of pipeline compute (roughly 2000 CPU hours for the human assembly), so budget accordingly. Evaluate the polished assembly with AssemblyStatistics or QUAST 3.0 and record the N50.

## Related tools

- **BLASR** (Aligns all raw PacBio reads to the assembly backbone using -minMatch 19 to produce .m5 mappings for consensus)
- **Sparc** (Sparsity-based consensus caller that corrects backbone base-level errors from the BLASR mappings, optionally boosting high-quality contig reads (HQ_Prefix Contig, boost 5)) — https://github.com/yechengxi/Sparc
- **DBG2OLC** (Produces backbone_raw.fasta and DBG2OLC_Consensus_info.txt that define the layout to be polished) — https://github.com/yechengxi/DBG2OLC
- **SparseAssembler** (Provides the accurate DBG contigs (Contigs.txt) that serve as high-quality inputs to the hybrid consensus) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (AssemblyStatistics evaluates the polished assembly N50 and contig statistics) — https://github.com/yechengxi/AssemblyUtility

## Examples

```
cat Contigs.txt Pacbio_20x.fasta > ctg_pb.fasta; ulimit -n unlimited; sh ./split_and_run_sparc.sh backbone_raw.fasta DBG2OLC_Consensus_info.txt ctg_pb.fasta ./consensus_dir 2 >cns_log.txt
```

## Evaluation signals

- BLASR .m5 file is non-empty and mappings cover the backbone (produced with -minMatch 19, -bestn 1, -m 5)
- Polished assembly N50 evaluated with AssemblyStatistics or QUAST 3.0 meets or improves on the backbone N50 (583 kbp for the w303 20x PacBio test)
- split_and_run_sparc.sh completes without file-descriptor errors after 'ulimit -n unlimited'
- Consensus log (cns_log.txt) shows all backbone records processed against ctg_pb.fasta
- Final hybrid N50 is >10x better than the Illumina-only assembly, as expected for 10x-20x PacBio input

## Limitations

- Consensus dominates runtime: ~90% of pipeline time is spent in the BLASR-based consensus step (roughly 2000 CPU hours for the 3 Gbp human assembly)
- Polishing cannot fix structural errors — backbone quality must be verified (dot plots / N50) before calling consensus
- Consensus quality depends on raw DBG contigs free of gap-closing/scaffolding heuristics; combining DBG2OLC with other error-correction tools may produce worse results

## Evidence

- [results] Consensus step aligning all raw reads to the assembly backbone with BLASR: "the consensus step, in which BLASR28 is called to align all raw reads to the assembly backbone"
- [results] ~90% of computation time is consensus: "most of the computation time (~90%) is spent on the consensus step, in which BLASR28 is called to align all raw reads"
- [readme] README consensus workflow commands: [evidence span withheld — anchor status: not_found]
- [readme] Check backbone N50 before proceeding: "You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don't proceed."
- [results] N50 >10x better than Illumina alone with 10x-20x PacBio: [evidence span withheld — anchor status: not_found]
