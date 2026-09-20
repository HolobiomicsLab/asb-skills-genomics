---
name: splice-junction-annotation-matching
description: Use when you need to quantify splice-junction prediction accuracy of
  Iso-seq aligners against a curated annotation (e.g., Ensembl GTF) on PacBio full-length
  transcript FASTA data.
when_to_use_negative:
- Input data is short-read RNA-seq rather than PacBio Iso-seq full-length transcripts;
  TAGET's alignment and junction prediction components are designed for Iso-seq data
- The goal is detecting truly novel junctions rather than benchmarking against a fixed
  annotation; matching to Ensembl by definition excludes novel-junction discovery
- No reference genome or GTF annotation is available for the organism, since gtf_db_make.py
  and gmap_build both require hg38.fa-style reference FASTA and Ensembl GTF inputs
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: TAGET
  role: Annotates full-length Iso-seq transcripts via TransAnnot.py and predicts splice
    junctions with a CNN filter, producing the {sample_id}.annot.junction file
  repo: https://github.com/XiDsLab/TAGET
- name: Minimap2
  role: Baseline aligner of the same full-length transcript reads against indexed
    hg38 for comparison junction predictions
- name: GMAP
  role: Baseline splice-aware aligner (index built with gmap_build) used to generate
    comparison junction predictions
- name: samtools
  role: Required environmental dependency for handling alignment outputs during the
    annotation workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:438c6e8e2de4fb0c15a0b1207c123a7af29fc414914301c96bc48be547c820b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiDsLab/TAGET
  tools:
  - GMAP
  - Minimap2
  - TAGET
  - samtools
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# splice-junction-annotation-matching

## Summary

Benchmark the precision of splice-junction prediction from long-read Iso-seq aligners (TAGET with its CNN splice-junction filter, minimap2, and GMAP) by computing the fraction of predicted junctions that match junctions in the Ensembl hg38 annotation.

## When to use

Use when you need to quantify splice-junction prediction accuracy of Iso-seq aligners against a curated annotation (e.g., Ensembl GTF) on PacBio full-length transcript FASTA data.

## When NOT to use

- Input data is short-read RNA-seq rather than PacBio Iso-seq full-length transcripts; TAGET's alignment and junction prediction components are designed for Iso-seq data
- The goal is detecting truly novel junctions rather than benchmarking against a fixed annotation; matching to Ensembl by definition excludes novel-junction discovery
- No reference genome or GTF annotation is available for the organism, since gtf_db_make.py and gmap_build both require hg38.fa-style reference FASTA and Ensembl GTF inputs

## Inputs

- Full-length transcript FASTA from PacBio Iso-seq (e.g., COLO829 dataset)
- hg38 reference genome FASTA (hg38.fa)
- Ensembl gene annotation in GTF format (hg38.ensembl.gtf)
- TAGET gtf_db_make.py pickle database of the annotation GTF
- GMAP index built with gmap_build; HISAT2/minimap2 indices of the reference genome

## Outputs

- TAGET splice junction information file {sample_id}.annot.junction
- Per-tool junction match rates (e.g., junction_match_rates.csv with ~97.4% TAGET, ~95.2% minimap2, ~94.3% GMAP)
- TAGET annotation outputs ({sample_id}.annot.stat, {sample_id}.annot.bed) usable for transcript classification cross-checks

## How to apply

Download the hg38 reference genome (hg38.fa) and Ensembl annotation (hg38.ensembl.gtf), build the GMAP index with gmap_build, and convert the GTF into TAGET's pickle database with gtf_db_make.py. Run TAGET via TransAnnot.py (e.g., python TransAnnot.py -f [fasta] -g hg38.fa -o [outdir] -a hg38.ensembl.gtf -p 8) with USE_MINIMAP2 and/or USE_GMAP enabled in TransAnnot.Config and the CNN splice-junction filter enabled. Independently align the same full-length transcript reads with minimap2 and GMAP against the indexed hg38 reference, then extract predicted splice junctions from each tool's output, using TAGET's {sample_id}.annot.junction file for the TAGET predictions. For each tool, compute the fraction of predicted junctions that match a junction in hg38.ensembl.gtf; correct application reproduces rates of approximately 97.4% for TAGET versus 95.2% for minimap2 and 94.3% for GMAP on the COLO829 Iso-seq data. This comparison matters because over 40% of novel junctions predicted by current alignment tools are non-canonical, indicating many are false positives, which TAGET's CNN filter reduces.

## Related tools

- **TAGET** (Annotates full-length Iso-seq transcripts via TransAnnot.py and predicts splice junctions with a CNN filter, producing the {sample_id}.annot.junction file) — https://github.com/XiDsLab/TAGET
- **Minimap2** (Baseline aligner of the same full-length transcript reads against indexed hg38 for comparison junction predictions)
- **GMAP** (Baseline splice-aware aligner (index built with gmap_build) used to generate comparison junction predictions)
- **samtools** (Required environmental dependency for handling alignment outputs during the annotation workflow)

## Examples

```
python TransAnnot.py -f COLO829_full_len.fa -g hg38.fa -o outdir -a hg38.ensembl.gtf -p 8 --use_minimap2 1
```

## Evaluation signals

- Computed junction-match rates reproduce the reported values: ~97.4% for TAGET, ~95.2% for minimap2, ~94.3% for GMAP on the COLO829 Iso-seq dataset
- TAGET run completes in under 1 hour on 8 cores on a Linux server, per the README runtime note
- The {sample_id}.annot.junction file exists and contains junction records for every predicted splice junction
- gtf_db_make.py successfully produces hg38.ensembl.v20200306.1.pickle from hg38.ensembl.gtf before running TransAnnot.py

## Limitations

- Over 40% of predicted novel junctions by current alignment tools are non-canonical, indicating many novel junctions are false positives; matching against Ensembl will not reveal these errors
- TAGET's CNN performance claims (e.g., ~98% junction accuracy) are deferred to supplementary figures not present in the main sections
- The workflow depends on environmental setup (HISAT2/minimap2/GMAP indices, samtools, rpy2/pandas/numpy, R >= 3.3 with stringr/optparse/DEGseq) and a Linux CentOS environment

## Evidence

- [other] On the Iso-seq data, 97.4% of splice junctions predicted by TAGET matched Ensembl database junctions, versus 95.2% for minimap2 and 94.3% for GMAP: [evidence span withheld — anchor status: not_found]
- [readme] README fast-run command: "python TransAnnot.py -f [fasta] -g [genome fasta] -o [output directory] -a[annot gtf] -p [process]"
- [readme] README junction output file description: "[{sample_id}.annot.junction]()  the information of splice junction file"
- [intro] Novel junction false-positive motivation: "We observe that over 40% of the predicted novel junctions by the current alignment tools are non-canonical, indicating that many novel junctions are false positives."
- [methods] GTF pickle generation: "python gtf_db_make.py hg38.ensembl.gtf hg38.ensembl.v20200306.1.pickle"
