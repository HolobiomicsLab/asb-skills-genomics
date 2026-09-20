---
name: long-read-consensus-polishing
description: Use when a DBG2OLC (or similar) hybrid assembly backbone has been produced
  and base-level errors from high-error-rate third-generation reads (up to ~40% ONT
  MinION error) must be corrected before final assembly evaluation.
when_to_use_negative:
- The backbone N50 from backbone_raw.fasta is unsatisfactory — the README says to
  keep tuning DBG2OLC parameters (AdaptiveTh, KmerCovTh, MinOverlap) and not proceed
  to consensus.
- The input is only short-read Illumina data with no long reads; this polishing step
  requires raw 3GS reads aligned to the backbone.
- The DBG contigs were produced with scaffolding or gap-closing (non-raw contigs),
  which the README warns introduces errors and degrades the final assembly.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: DBG2OLC
  role: Produces the backbone_raw.fasta and DBG2OLC_Consensus_info.txt that define
    which reads support each backbone position during polishing
  repo: https://github.com/yechengxi/DBG2OLC
- name: BLASR
  role: Aligns all raw long reads to the assembly backbone (option -m 5, -bestn 1,
    -minMatch 19) as input to the consensus module
- name: Sparc
  role: Sparsity-based consensus algorithm that corrects base-level errors in the
    backbone using the BLASR m5 mappings; run via split_and_run_sparc.sh with coverage
    threshold 2
  repo: https://github.com/yechengxi/Sparc
- name: AssemblyUtility
  role: Provides AssemblyStatistics to evaluate backbone N50 before polishing and
    the polished assembly's contiguity/error rate afterwards
  repo: https://github.com/yechengxi/AssemblyUtility
- name: SparseAssembler
  role: Supplies the accurate short-read DBG contigs (Contigs.txt) that are concatenated
    with the long reads and boost consensus accuracy in the hybrid Sparc mode
  repo: https://github.com/yechengxi/SparseAssembler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:a9a23192168b4c6e94b0b8eae6c04cf4fb09d0f273e36e2ca0d0822dfa399260
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/DBG2OLC
  tools:
  - AssemblyUtility
  - BLASR
  - DBG2OLC
  - Sparc
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# long-read-consensus-polishing

## Summary

A consensus/polishing step for DBG2OLC hybrid assemblies in which BLASR aligns all raw long reads (e.g. Oxford Nanopore MinION or PacBio) to the backbone_raw.fasta and the Sparc sparsity-based consensus module corrects the backbone, reducing error rates from tens of percent down to ~0.23% on the polished E. coli MinION assembly.

## When to use

Use when a DBG2OLC (or similar) hybrid assembly backbone has been produced and base-level errors from high-error-rate third-generation reads (up to ~40% ONT MinION error) must be corrected before final assembly evaluation.

## When NOT to use

- The backbone N50 from backbone_raw.fasta is unsatisfactory — the README says to keep tuning DBG2OLC parameters (AdaptiveTh, KmerCovTh, MinOverlap) and not proceed to consensus.
- The input is only short-read Illumina data with no long reads; this polishing step requires raw 3GS reads aligned to the backbone.
- The DBG contigs were produced with scaffolding or gap-closing (non-raw contigs), which the README warns introduces errors and degrades the final assembly.

## Inputs

- backbone_raw.fasta (DBG2OLC overlap-layout backbone)
- DBG2OLC_Consensus_info.txt (DBG2OLC consensus layout info)
- Contigs.txt (raw de Bruijn graph contigs from SparseAssembler, FASTA)
- Long reads FASTA (e.g. Oxford Nanopore MinION pb_reads.fasta or PacBio reads)
- BLASR installation (>=5.1 requires double-dash options) and Sparc consensus module

## Outputs

- DBG2OLC_Consensus.fasta (polished consensus assembly)
- consensus_dir/ intermediate mapped-read files from split_and_run_sparc.sh
- cns_log.txt (consensus run log)
- AssemblyStatistics/QUAST metrics of the polished assembly (contig count, N50, error rate)

## How to apply

After Step 2 (Overlap and layout) produces backbone_raw.fasta and DBG2OLC_Consensus_info.txt, check the backbone N50 with AssemblyStatistics and only proceed if satisfied. Concatenate the DBG contigs and long reads (cat Contigs.txt pb_reads.fasta > ctg_pb.fasta), raise the open-file limit with 'ulimit -n unlimited', and run split_and_run_sparc.sh, which calls BLASR to align all raw reads to the assembly backbone (use --nproc/--bestn/--minMatch 19 double-dash options for BLASR >= 5.1) and Sparc to compute a sparsity-based consensus (suggested k 1-2, coverage threshold c ~2, adaptive threshold t ~0.1). Rationale: ~90% of DBG2OLC computation time is spent in this consensus step, and skipping base-level errors during layout while correcting them at the end is a core design principle of the method.

## Related tools

- **DBG2OLC** (Produces the backbone_raw.fasta and DBG2OLC_Consensus_info.txt that define which reads support each backbone position during polishing) — https://github.com/yechengxi/DBG2OLC
- **BLASR** (Aligns all raw long reads to the assembly backbone (option -m 5, -bestn 1, -minMatch 19) as input to the consensus module)
- **Sparc** (Sparsity-based consensus algorithm that corrects base-level errors in the backbone using the BLASR m5 mappings; run via split_and_run_sparc.sh with coverage threshold 2) — https://github.com/yechengxi/Sparc
- **AssemblyUtility** (Provides AssemblyStatistics to evaluate backbone N50 before polishing and the polished assembly's contiguity/error rate afterwards) — https://github.com/yechengxi/AssemblyUtility
- **SparseAssembler** (Supplies the accurate short-read DBG contigs (Contigs.txt) that are concatenated with the long reads and boost consensus accuracy in the hybrid Sparc mode) — https://github.com/yechengxi/SparseAssembler

## Examples

```
cat Contigs.txt pb_reads.fasta > ctg_pb.fasta; ulimit -n unlimited; sh ./split_and_run_sparc.sh backbone_raw.fasta DBG2OLC_Consensus_info.txt ctg_pb.fasta ./consensus_dir 2 >cns_log.txt
```

## Evaluation signals

- The polished DBG2OLC_Consensus.fasta resolves the E. coli MinION dataset to a single contig when evaluated with AssemblyStatistics or QUAST against the E. coli reference.
- The polished error rate falls near the reported ~0.23% despite the ~40% raw MinION error rate.
- backbone_raw.fasta N50 is checked with AssemblyStatistics before running consensus; proceed only if satisfied.
- cns_log.txt and consensus_dir contain complete BLASR m5 mappings (option -m 5) with no missing alignment files.
- ~90% of pipeline compute time is expected in this consensus step; unusually fast completion may indicate BLASR alignments failed.

## Limitations

- For BLASR >= 5.1, the split_and_run_sparc.sh script must be edited to use double-dash options (e.g. --nproc, --bestn, --minMatch 19) or the alignment will fail.
- 'ulimit -n unlimited' is required because the consensus step distributes the concatenated ctg_pb.fasta into many smaller files.
- Consensus is the dominant cost (~90% of runtime; ~2000 CPU hours for the human assembly), so budget resources accordingly.
- The README warns that substituting other error-correction tools may produce worse results because 'many existing assembly/error correction tools introduce errors'.

## Evidence

- [results] Consensus step aligning all raw reads to the assembly backbone with BLASR: "the consensus step, in which BLASR28 is called to align all raw reads to the assembly backbone"
- [results] ~90% of computation spent on consensus: "most of the computation time (~90%) is spent on the consensus step, in which BLASR28 is called to align all raw reads"
- [readme] Sparc consensus invocation: [evidence span withheld — anchor status: external_doc]
- [readme] Consensus shell command and inputs: "sh ./split_and_run_sparc.sh backbone_raw.fasta DBG2OLC_Consensus_info.txt ctg_pb.fasta ./consensus_dir 2 >cns_log.txt"
- [readme] Check backbone N50 before polishing: "You can check the N50 of (1) to see if you are satisfied, otherwise keep tuning and don"
- [results] E. coli single-contig polished result: "DBG2OLC still successfully assembled the E. coli into one single contig"
