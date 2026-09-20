---
name: long-read-splice-junction-extraction
description: Use when evaluating how many splice junctions predicted from long-read
  Iso-seq transcript alignments match a reference annotation (e.g., Ensembl hg38 GTF),
  especially comparing aligners like TAGET, minimap2, and GMAP.
when_to_use_negative:
- Input reads are short RNA-seq reads rather than full-length Iso-seq transcripts;
  TAGET is designed for Iso-seq full-length data
- The goal is differential expression/DIU analysis rather than junction accuracy benchmarking
- No reference GTF annotation exists, making an Ensembl match-rate computation impossible
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TAGET
  role: Primary toolkit; TransAnnot.py aligns and annotates full-length transcripts
    and outputs {sample_id}.annot.junction with predicted splice junctions (CNN filter
    enabled)
  repo: https://github.com/XiDsLab/TAGET
- name: Minimap2
  role: Independent baseline aligner of the same Iso-seq reads against indexed hg38
    for comparison junction sets (USE_MINIMAP2 config flag)
- name: GMAP
  role: Second baseline aligner; gmap_build creates the hg38 index, and its alignments
    provide the comparison junction set
- name: samtools
  role: Environment dependency used in the TAGET alignment/annotation workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:59d971eba1436f5381adf66c2cb69fb84c8daaa3df6b6f83af55ee3f61dc6124
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiDsLab/TAGET
  tools:
  - GMAP
  - Minimap2
  - TAGET
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# long-read-splice-junction-extraction

## Summary

Extract and benchmark splice junctions predicted from PacBio Iso-seq full-length transcript alignments (TAGET with CNN filtering vs minimap2 and GMAP) against an Ensembl GTF annotation to quantify junction accuracy.

## When to use

Use when evaluating how many splice junctions predicted from long-read Iso-seq transcript alignments match a reference annotation (e.g., Ensembl hg38 GTF), especially comparing aligners like TAGET, minimap2, and GMAP.

## When NOT to use

- Input reads are short RNA-seq reads rather than full-length Iso-seq transcripts; TAGET is designed for Iso-seq full-length data
- The goal is differential expression/DIU analysis rather than junction accuracy benchmarking
- No reference GTF annotation exists, making an Ensembl match-rate computation impossible

## Inputs

- Full-length transcript FASTA from PacBio Iso-seq (e.g., COLO829 dataset)
- Human reference genome hg38.fa
- Ensembl GTF annotation (hg38.ensembl.gtf) and its pickle database (hg38.ensembl.v20200306.1.pickle)
- GMAP index built with gmap_build (and optionally HISAT2 index)

## Outputs

- {sample_id}.annot.junction — TAGET splice junction information file
- Equivalent junction records parsed from minimap2 and GMAP alignment output
- junction_match_rates.csv — per-tool fraction of junctions matching Ensembl annotation

## How to apply

Build the required references first: download hg38.fa and hg38.ensembl.gtf, build a GMAP index with gmap_build, and convert the GTF into the TAGET pickle database with gtf_db_make.py. Run TAGET's TransAnnot.py on the COLO829 full-length transcript FASTA with the CNN splice-junction filter enabled (USE_MINIMAP2/USE_GMAP set per TransAnnot.Config), then independently align the same reads with minimap2 and GMAP against the indexed hg38 reference. Extract predicted junctions from each tool's output — TAGET's junctions are reported in the {sample_id}.annot.junction file. For each tool, compute the fraction of predicted junctions that match a junction in hg38.ensembl.gtf and write the rates to junction_match_rates.csv. The rationale is that current aligners produce many non-canonical novel junctions (over 40% in this study), so an annotation-match rate distinguishes precise junction prediction; expected values on COLO829 Iso-seq are ~97.4% (TAGET), ~95.2% (minimap2), ~94.3% (GMAP).

## Related tools

- **TAGET** (Primary toolkit; TransAnnot.py aligns and annotates full-length transcripts and outputs {sample_id}.annot.junction with predicted splice junctions (CNN filter enabled)) — https://github.com/XiDsLab/TAGET
- **Minimap2** (Independent baseline aligner of the same Iso-seq reads against indexed hg38 for comparison junction sets (USE_MINIMAP2 config flag))
- **GMAP** (Second baseline aligner; gmap_build creates the hg38 index, and its alignments provide the comparison junction set)
- **samtools** (Environment dependency used in the TAGET alignment/annotation workflow)

## Examples

```
python TransAnnot.py -f COLO829.full_length.fa -g hg38.fa -o ./out -a hg38.ensembl.gtf -p 8 --use_minimap2 1
```

## Evaluation signals

- Computed junction match rates reproduce the reported values: ~97.4% for TAGET, ~95.2% for minimap2, ~94.3% for GMAP on COLO829 Iso-seq
- TAGET run produces the complete output set including {sample_id}.annot.junction, {sample_id}.annot.stat, and {sample_id}.annot.bed
- The pickle database was generated successfully by gtf_db_make.py from hg38.ensembl.gtf before running TransAnnot
- Each tool is run on the identical full-length transcript FASTA so the match rates are directly comparable
- Junctions extracted from each aligner are assessed against the same Ensembl hg38 GTF version

## Limitations

- Over 40% of novel junctions predicted by current alignment tools are non-canonical, indicating many novel junctions are false positives
- Performance claims (e.g., CNN 98% accuracy) are deferred to supplementary figures not present in the main section
- TAGET depends on HISAT2/MINIMAP2/GMAP (at least one), samtools, python3 with rpy2/pandas/numpy, and R >= 3.3, limiting portability
- Running time is roughly under 1 hour on 8 cores; multi-threading is controlled by the -p PROCESS parameter

## Evidence

- [methods] python TransAnnot.py -f [fasta] -g [genome fasta] -o [output directory] -a[annot gtf] -p [process]: "python TransAnnot.py -f [fasta] -g [genome fasta] -o [output directory] -a[annot gtf] -p [process]"
- [readme] annot.junction: "[{sample_id}.annot.junction]()  the information of splice junction file"
- [methods] Reference genome needed: "Here we used hg38 to annotate transcripts,human reference hg38.fa and Ensemble gtf format files were needed"
- [intro] Novel junction false positives: "over 40% of the predicted novel junctions by the current alignment tools are non-canonical, indicating that many novel junctions are false positives"
- [readme] GTF pickle generation: "python gtf_db_make.py hg38.ensembl.gtf hg38.ensembl.v20200306.1.pickle"
