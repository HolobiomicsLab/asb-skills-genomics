---
name: single-breakend-contig-repeat-annotation-filtering
description: Use when nanomonsv single breakend results contain contigs from highly
  repetitive centromeric sequences that need to be subset for high-order repeat (HOR)
  consistency analysis with HORmon monomers and StringDecomposer.
when_to_use_negative:
- Single breakend SVs whose breakpoints lie in poorly characterized repetitive regions
  where the reference genome is unreliable and RepeatMasker gives no informative annotation
  (e.g. both breakpoints in uncharacterized repeats)
- Canonical (two-breakend) somatic SVs from the nanomonsv Canonical SV module, which
  are assessed by short-read-compatible precision/recall rather than centromeric HOR
  consistency
- Non-centromeric repeats (e.g. LINE1-mediated insertions, viral integrations) where
  HORmon monomer decomposition is not the appropriate analysis
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3175
tools:
- name: nanomonsv
  role: Produces the single breakend SV result file whose Contig column and 'High
    Repeat single breakend SVs' class label supply the contigs to be filtered
  repo: https://github.com/friend1ws/nanomonsv
- name: RepeatMasker
  role: Annotates extracted single breakend contigs by repeat class; 'Satellite/centr'
    annotations identify centromeric contigs
- name: HORmon
  role: Supplies curated monomer FASTA files (cen*_monomers.fa) and HOR definitions
    (cen*_hors.tsv) used downstream on the filtered contigs
- name: StringDecomposer
  role: Decomposes the filtered centromeric contigs against chromosome-specific monomer
    FASTAs to yield final_decomposition.tsv files for HOR match scoring
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:6f6b2b537926a5b275f203f1a5c598125678679d09e19b3a2903174720ca0e44
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
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# single-breakend-contig-repeat-annotation-filtering

## Summary

Selects contigs from the nanomonsv Single breakend SV module results that derive from highly repetitive centromeric regions, by filtering on the single breakend result file's Contig column and RepeatMasker 'Satellite/centr' annotations, so that downstream HOR match scoring is applied to the right contig set.

## When to use

Use when nanomonsv single breakend results contain contigs from highly repetitive centromeric sequences that need to be subset for high-order repeat (HOR) consistency analysis with HORmon monomers and StringDecomposer.

## When NOT to use

- Single breakend SVs whose breakpoints lie in poorly characterized repetitive regions where the reference genome is unreliable and RepeatMasker gives no informative annotation (e.g. both breakpoints in uncharacterized repeats)
- Canonical (two-breakend) somatic SVs from the nanomonsv Canonical SV module, which are assessed by short-read-compatible precision/recall rather than centromeric HOR consistency
- Non-centromeric repeats (e.g. LINE1-mediated insertions, viral integrations) where HORmon monomer decomposition is not the appropriate analysis

## Inputs

- nanomonsv single breakend SV result file (with Contig column and SV class labels such as 'High Repeat single breakend SVs')
- RepeatMasker repeat annotations for the single breakend contigs (e.g. 'Satellite/centr' classes)
- Contig sequences extracted from the single breakend results

## Outputs

- Filtered set of centromeric single breakend contigs (mostly 'Satellite/centr' by RepeatMasker) ready for StringDecomposer/HOR match scoring
- Per-contig annotation table linking contig ID to SV class and repeat class

## How to apply

Parse the nanomonsv single breakend result file and extract contigs of single breakend SVs classified as 'High Repeat single breakend SVs', taking the contig sequences from the Contig column. Cross-reference each contig's repeat annotation, retaining those mostly annotated as 'Satellite/centr' by RepeatMasker, which marks sequences originating from centromeric high-order repeat arrays. Restricting to this satellite/centromeric subset ensures that subsequent StringDecomposer (version 1.1.2) decomposition against chromosome-specific HORmon monomer FASTA files (cen*_monomers.fa under the monomersFinal directory) is performed only where HOR-based scoring is meaningful. The rationale is that nanomonsv's Single breakend SV module detects SVs with one breakpoint in repetitive regions, and only a well-annotated fraction of these contigs correspond to canonical centromeric HOR structure suitable for monomer-concordance scoring.

## Related tools

- **nanomonsv** (Produces the single breakend SV result file whose Contig column and 'High Repeat single breakend SVs' class label supply the contigs to be filtered) — https://github.com/friend1ws/nanomonsv
- **RepeatMasker** (Annotates extracted single breakend contigs by repeat class; 'Satellite/centr' annotations identify centromeric contigs)
- **HORmon** (Supplies curated monomer FASTA files (cen*_monomers.fa) and HOR definitions (cen*_hors.tsv) used downstream on the filtered contigs)
- **StringDecomposer** (Decomposes the filtered centromeric contigs against chromosome-specific monomer FASTAs to yield final_decomposition.tsv files for HOR match scoring)

## Examples

```
python select_high_repeat_centromeric_contigs.py --sbe_result nanomonsv_sbe.tsv --repeatmasker contigs_repeatmasker.out --class 'High Repeat single breakend SVs' --repeat_class 'Satellite/centr' --out centromeric_contigs/
```

## Evaluation signals

- Every retained contig is labeled 'High Repeat single breakend SVs' in the nanomonsv single breakend result file
- RepeatMasker annotations show the retained contigs are predominantly 'Satellite/centr'
- Each retained contig maps to exactly one chromosome-specific monomer FASTA (cen*_monomers.fa) for StringDecomposer decomposition
- Downstream HOR match scores computed on the retained contigs all fall in the valid 0-1 range
- Filtered contigs reproduce the paper's qualitative result: most contigs match canonical, chromosome-specific HORs, with a minority non-canonical (e.g. the 17-mer ABCDEFGHIJKLHIJKL on chromosome X)

## Limitations

- nanomonsv's Single breakend SV module cannot detect SVs where both breakpoints are located in poorly characterized, highly repetitive regions
- RepeatMasker annotation is approximate ('mostly' Satellite/centr), so borderline contigs may be misclassified into or out of the centromeric subset
- Complex SV interpretation is not fully automated and much work is done manually, so the filtering threshold for 'High Repeat' classification may require manual review

## Evidence

- [results] single breakend contigs classified as high repeat: "Single breakend SV module enables the detection of complex SVs that can only be identified by long-reads, such as SVs involving highly-repetitive centromeric sequences"
- [discussion] scope limit of the module: "it cannot detect SVs where both of the breakpoints are located in areas where reference genomes are not well-characterized, such as highly repetitive regions"
- [discussion] workflow purpose: "we have developed a workflow for detecting and classifying single breakend SVs (Single breakend SV module)"
- [discussion] manual interpretation caveat: "the interpretation of the detailed structure and properties of complex SVs is not fully automated at present, and much of the work is done manually"
