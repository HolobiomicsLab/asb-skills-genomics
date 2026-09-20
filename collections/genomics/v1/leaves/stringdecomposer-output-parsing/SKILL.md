---
name: stringdecomposer-output-parsing
description: Use when nanomonsv single breakend SVs are classified as 'High Repeat
  single breakend SVs' and RepeatMasker annotates them 'Satellite/centr', and you
  must quantify how consistently each contig matches centromeric HORs.
when_to_use_negative:
- SVs where both breakpoints lie in poorly characterized repetitive regions — the
  nanomonsv Single breakend SV module cannot detect these, so no contig is available
  to score
- Long repeat expansion events (VNTR/microsatellite) where the reference genome is
  unreliable and a reference-genome-based decomposition would be inappropriate
- Contigs already decomposed against monomer sets from a different chromosome than
  the one whose HOR definitions (cen*_hors.tsv) are used for consistency checking
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: nanomonsv
  role: Provides the single breakend SV result file from which high-repeat centromeric
    contigs are extracted (Contig column)
  repo: https://github.com/friend1ws/nanomonsv
- name: StringDecomposer
  role: Decomposes each centromeric contig into monomers against HORmon cen*_monomers.fa,
    emitting final_decomposition.tsv (use version 1.1.2)
- name: HORmon
  role: Supplies the curated monomer FASTA files (cen*_monomers.fa) and HOR definition
    tables (cen*_hors.tsv) used to judge monomer pre-/post-relationship consistency
- name: RepeatMasker
  role: Annotates the single breakend SV contigs as 'Satellite/centr', which is the
    filter trigger for this scoring workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:347401e95312168f6415d59654c48d5f77e3360d67132db6878977a17e9dc518
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# stringdecomposer-output-parsing

## Summary

Computes a per-contig HOR match score (0–1) for centromeric single breakend contigs by parsing StringDecomposer (v1.1.2) final_decomposition.tsv output line by line against HORmon-curated HOR definitions, then normalizing summed monomer concordance by contig length.

## When to use

Use when nanomonsv single breakend SVs are classified as 'High Repeat single breakend SVs' and RepeatMasker annotates them 'Satellite/centr', and you must quantify how consistently each contig matches centromeric HORs.

## When NOT to use

- SVs where both breakpoints lie in poorly characterized repetitive regions — the nanomonsv Single breakend SV module cannot detect these, so no contig is available to score
- Long repeat expansion events (VNTR/microsatellite) where the reference genome is unreliable and a reference-genome-based decomposition would be inappropriate
- Contigs already decomposed against monomer sets from a different chromosome than the one whose HOR definitions (cen*_hors.tsv) are used for consistency checking

## Inputs

- nanomonsv single breakend result file (Contig column) filtered to 'High Repeat single breakend SVs' with 'Satellite/centr' RepeatMasker annotation
- HORmon final monomer FASTA files per chromosome (cen*_monomers.fa under the monomersFinal directory)
- HORmon-curated HOR definition tables (cen*_hors.tsv under the HORmonHORs directory)
- StringDecomposer v1.1.2 per-contig decomposition output (final_decomposition.tsv)

## Outputs

- hor_match_scores.csv with columns: contig ID, chromosome, HOR match score (0–1)

## How to apply

First extract the contig sequences for single breakend SVs flagged as 'High Repeat single breakend SVs' from the nanomonsv single breakend result file (Contig column), which are mostly annotated 'Satellite/centr' by RepeatMasker. Run StringDecomposer 1.1.2 on each contig against the chromosome-specific monomer FASTA files produced by HORmon (cen*_monomers.fa under monomersFinal), yielding one final_decomposition.tsv per chromosome monomer file. Parse each final_decomposition.tsv one line at a time; for each adjacent monomer pair whose pre-/post-relationship matches the curated HOR definitions in cen*_hors.tsv (from the HORmonHORs directory), add (<end-pos> - <start-pos>) * <identity> / 100 to a running sum. Divide the total by the contig length to obtain the HOR match score, then write per-contig scores to hor_match_scores.csv with contig ID, chromosome, and score columns.

## Related tools

- **nanomonsv** (Provides the single breakend SV result file from which high-repeat centromeric contigs are extracted (Contig column)) — https://github.com/friend1ws/nanomonsv
- **StringDecomposer** (Decomposes each centromeric contig into monomers against HORmon cen*_monomers.fa, emitting final_decomposition.tsv (use version 1.1.2))
- **HORmon** (Supplies the curated monomer FASTA files (cen*_monomers.fa) and HOR definition tables (cen*_hors.tsv) used to judge monomer pre-/post-relationship consistency)
- **RepeatMasker** (Annotates the single breakend SV contigs as 'Satellite/centr', which is the filter trigger for this scoring workflow)

## Examples

```
score=0.0; for line in open('final_decomposition.tsv'): cols=line.split('\t'); if hor_rel(cols): score+=(int(cols['end-pos'])-int(cols['start-pos']))*float(cols['identity'])/100; print(score/contig_len)
```

## Evaluation signals

- Every HOR match score must fall in the valid range 0–1, since summed (end-pos - start-pos) * identity / 100 cannot exceed the contig length
- Per-chromosome final_decomposition.tsv files must exist for every (contig, cen*_monomers.fa) pair processed with StringDecomposer 1.1.2
- Scores should reproduce the paper's qualitative result: most contigs match canonical, chromosome-specific HORs, with a minority of non-canonical HORs (e.g. the 17-mer ABCDEFGHIJKLHIJKL on chromosome X)
- hor_match_scores.csv must contain exactly the three columns contig ID, chromosome, and HOR match score, with one row per extracted contig

## Limitations

- The score quantifies concordance with curated HOR monomer relationships; it does not by itself resolve breakpoint precision in noisy long-read data (a gap the paper notes for noisy long-read SV calling)
- Depends on HORmon-curated HOR definitions; non-canonical HORs are identified only qualitatively by mismatch to cen*_hors.tsv relationships
- Only applicable to contigs from single breakend SVs annotated as highly repetitive centromeric sequence; LINE1- or virus-mediated single breakend SVs are outside this scoring path

## Evidence

- [abstract] Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences"
- [intro] Single breakend SV module of nanomonsv can identify single breakend SVs where only one breakpoint is identified: "Single breakend SV module of nanomonsv can identify single breakend SVs where only one breakpoint is identified"
- [intro] the landscape of somatic SVs remains elusive in human cancer because repeat sequences hinder short-read alignment: "it is plausible to assume that the landscape of SVs remains elusive in human cancer"
