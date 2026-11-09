---
name: centromeric-hor-monomer-decomposition
description: Use when nanomonsv single breakend SVs are classified as 'High Repeat
  single breakend SVs' (mostly RepeatMasker 'Satellite/centr') and you need to test
  whether their contig sequences follow canonical, chromosome-specific centromeric
  HORs.
when_to_use_negative:
- Both breakpoints of the SV lie in poorly characterized repetitive regions where
  the reference genome is not well-characterized — the nanomonsv single breakend module
  cannot detect such SVs, so no contig is available to score.
- The rearrangement is not centromeric/satellite-associated (e.g. LINE1-mediated rearrangements
  or viral integrations), where centromeric HOR monomer sets are the wrong reference.
- The region of interest is a long VNTR/microsatellite repeat expansion, since the
  reference-genome-based framework is not reliable there.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: nanomonsv
  role: Source of single breakend SV contigs classified as 'High Repeat single breakend
    SVs' from the single breakend result file
  repo: https://github.com/friend1ws/nanomonsv
- name: StringDecomposer
  role: Decomposes each centromeric contig into monomers (version 1.1.2) producing
    final_decomposition.tsv used for HOR scoring
- name: HORmon
  role: Provides the chromosome-specific monomer FASTAs (cen*_monomers.fa) and curated
    HOR definitions (cen*_hors.tsv) used as the decomposition reference and concordance
    standard
- name: RepeatMasker
  role: Annotates candidate contigs as 'Satellite/centr' to confirm they are centromeric
    before scoring
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:c2f033a89999f69e59f0a29bb7354a8e958ba015220581bbd8e9f6cc1975c979
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - HORmon
  - Python
  - RepeatMasker
  - StringDecomposer
  - nanomonsv
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# centromeric-hor-monomer-decomposition

## Summary

Quantifies how well single breakend SV contigs from nanomonsv match the high-order repeat (HOR) structure of centromeres, by decomposing each contig into centromeric monomers with StringDecomposer and scoring concordance against curated HOR definitions from HORmon.

## When to use

Use when nanomonsv single breakend SVs are classified as 'High Repeat single breakend SVs' (mostly RepeatMasker 'Satellite/centr') and you need to test whether their contig sequences follow canonical, chromosome-specific centromeric HORs.

## When NOT to use

- Both breakpoints of the SV lie in poorly characterized repetitive regions where the reference genome is not well-characterized — the nanomonsv single breakend module cannot detect such SVs, so no contig is available to score.
- The rearrangement is not centromeric/satellite-associated (e.g. LINE1-mediated rearrangements or viral integrations), where centromeric HOR monomer sets are the wrong reference.
- The region of interest is a long VNTR/microsatellite repeat expansion, since the reference-genome-based framework is not reliable there.

## Inputs

- nanomonsv single breakend result file (Contig column, 'High Repeat single breakend SVs' class)
- RepeatMasker annotations identifying contigs as 'Satellite/centr'
- Chromosome-specific monomer FASTA files from HORmon (cen*_monomers.fa, monomersFinal directory)
- Curated HOR definition files from HORmon (cen*_hors.tsv, HORmonHORs directory)
- StringDecomposer 1.1.2 decomposition output (final_decomposition.tsv) per contig

## Outputs

- hor_match_scores.csv with columns for contig ID, chromosome, and HOR match score (0-1)
- Per-contig final_decomposition.tsv monomer decomposition files from StringDecomposer

## How to apply

First extract contig sequences of single breakend SVs labeled 'High Repeat single breakend SVs' from the nanomonsv single breakend result file (Contig column), retaining those predominantly annotated 'Satellite/centr' by RepeatMasker. Run StringDecomposer version 1.1.2 on each contig against the chromosome-specific final monomer FASTA files produced by HORmon (cen*_monomers.fa under the monomersFinal directory), yielding a final_decomposition.tsv per contig. Then read each final_decomposition.tsv line by line: whenever the pre-/post-relationship of adjacent monomers is consistent with the curated HOR definitions (cen*_hors.tsv from the HORmonHORs directory, e.g. the 17-mer ABCDEFGHIJKLHIJKL on chromosome X), add (<end-pos> - <start-pos>) * <identity> / 100 to a running sum. Divide the summed concordance by the contig length to obtain the HOR match score, and write per-contig results to hor_match_scores.csv with contig ID, chromosome, and score columns. The rationale is that contigs inserted at centromeres should be largely composed of the same chromosome's HOR array, so the normalized identity-weighted monomer span measures HOR consistency.

## Related tools

- **nanomonsv** (Source of single breakend SV contigs classified as 'High Repeat single breakend SVs' from the single breakend result file) — https://github.com/friend1ws/nanomonsv
- **StringDecomposer** (Decomposes each centromeric contig into monomers (version 1.1.2) producing final_decomposition.tsv used for HOR scoring)
- **HORmon** (Provides the chromosome-specific monomer FASTAs (cen*_monomers.fa) and curated HOR definitions (cen*_hors.tsv) used as the decomposition reference and concordance standard)
- **RepeatMasker** (Annotates candidate contigs as 'Satellite/centr' to confirm they are centromeric before scoring)

## Evaluation signals

- Each HOR match score must fall in the valid range 0-1, because the summed (end-pos - start-pos) * identity / 100 cannot exceed the contig length.
- Most scored contigs should match canonical, chromosome-specific HORs, with only a minority showing non-canonical HORs (e.g. the 17-mer ABCDEFGHIJKLHIJKL on chromosome X), reproducing the paper's qualitative result.
- Every contig in hor_match_scores.csv must originate from a 'High Repeat single breakend SV' entry in the nanomonsv result file and be RepeatMasker-annotated 'Satellite/centr'.
- Each final_decomposition.tsv must be produced against the matching chromosome's monomer file (contig on chr X scored against cenX_monomers.fa), not an arbitrary monomer set.

## Limitations

- The nanomonsv single breakend module cannot detect SVs where both breakpoints are located in highly repetitive regions where the reference genome is not well-characterized, so such centromeric events yield no contig to score.
- Interpretation of detailed structure and properties of complex SVs is not fully automated and much of the work is done manually; visualization methods for complex SV interpretation remain to be developed.
- The framework is reference-genome-based and may not appropriately capture long repeat expansion events where the reference is unreliable.
- Scores depend on the quality of the HORmon-curated HOR definitions; monomers or HOR units absent from the curated cen*_hors.tsv files are not credited as consistent.

## Evidence

- [abstract] Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences, and LINE1- and virus-mediated"
- [discussion] we have developed a workflow for detecting and classifying single breakend SVs: "we have developed a workflow for detecting and classifying single breakend SVs (Single breakend SV module)"
- [other] Run StringDecomposer version 1.1.2 on each contig against the final monomer FASTA files generated by HORmon (cen*_monomers.fa under the monomersFinal directory), obtaining a final_decomposition.tsv per chromosome monomer file.: [evidence span withheld — anchor status: not_found]
