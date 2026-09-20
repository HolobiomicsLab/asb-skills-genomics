---
name: conserved-microrna-identification
description: Use when processing genome-mapped plant sRNA reads (18–26 nt, min_abundance=5)
  and you need to distinguish reads matching miRBase (conserved miRNAs) from non-conserved
  reads destined for novel miRNA/tasiRNA prediction.
when_to_use_negative:
- Input is raw FASTQ that has not yet passed adaptor trimming (fastx_clipper) and
  the Wbench Filter / PatMaN genome-mapping stage — miRProf expects genome-mapped
  reads
- The goal is novel (non-conserved) miRNA discovery rather than conserved annotation
  — use miRCat on the non-conserved fraction instead
- No reference genome or miRBase database is available, as both the genome filter
  and the miRBase alignment are prerequisites of this step
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: miRProf
  role: Aligns genome-mapped reads of each sRNA library to the miRBase mature database
    with 0 mismatches and splits them into conserved and non-conserved fractions
  repo: http://srna-workbench.cmp.uea.ac.uk
- name: miRPursuit
  role: Pipeline (bash scripts) that orchestrates the Annotation stage, reads wbench_mirprof.cfg,
    and connects miRProf output to downstream miRCat / ta-si / PAREsnip steps
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: PatMaN
  role: Upstream pattern matcher that maps reads to the genome with 0 mismatches,
    producing the genome-mapped reads that miRProf consumes
  repo: https://bioinf.eva.mpg.de/patman/
- name: miRBase
  role: Reference database (mature.fa) of conserved miRNA sequences used as the alignment
    target in the Annotation stage
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:ca84609718a2b71bec25853ca4de2cb0a6cf57c9ad386d6f2a6a00d701fb83b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PatMaN
  - miRBase
  - miRProf
  - miRPursuit
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# conserved-microrna-identification

## Summary

Identify conserved miRNAs in plant sRNA libraries by aligning genome-mapped reads to the miRBase mature database with miRProf (UEA sRNA workbench), within the miRPursuit Annotation stage, and partition each library into conserved and non-conserved read fractions.

## When to use

Use when processing genome-mapped plant sRNA reads (18–26 nt, min_abundance=5) and you need to distinguish reads matching miRBase (conserved miRNAs) from non-conserved reads destined for novel miRNA/tasiRNA prediction.

## When NOT to use

- Input is raw FASTQ that has not yet passed adaptor trimming (fastx_clipper) and the Wbench Filter / PatMaN genome-mapping stage — miRProf expects genome-mapped reads
- The goal is novel (non-conserved) miRNA discovery rather than conserved annotation — use miRCat on the non-conserved fraction instead
- No reference genome or miRBase database is available, as both the genome filter and the miRBase alignment are prerequisites of this step

## Inputs

- genome-mapped sRNA library reads in FASTA format within the workdir (produced by PatMaN genome mapping with 0 mismatches)
- miRBase mature.fa reference database (downloaded from ftp://mirbase.org/pub/mirbase/CURRENT/)
- wbench_mirprof.cfg configuration file with miRProf parameters (min_length=18, max_length=26, min_abundance=5, grouping flags)

## Outputs

- per-library conserved reads file (reads matching miRBase with 0 mismatches) in the workdir
- per-library non-conserved reads file (non-matching reads) retained for miRCat and ta-si predictor
- miRBase hits reported in the pipeline's counts directory (space-separated count files, format %y%m%d:%h%m%s-type-lib[lib_first]-[lib_last].tsv)

## How to apply

After the Filtering stage has mapped reads to the genome with PatMaN using 0 mismatches, take the genome-mapped reads of each sRNA library in the workdir and align them to the miRBase mature reference (MIRBASE=${source_data}/mirbase/mature.fa) using miRProf with parameters read from wbench_mirprof.cfg. Enforce mismatches=0, overhangs=true, only_keep_best=true, min_length=18, max_length=26 and min_abundance=5, and set grouping options group_mismatches=true, group_organisms=true, group_variant=true with group_mature_and_star=false so hits are grouped by mismatch, organism and variant rather than merging mature/star strands. Split each library's reads into a conserved file (reads matching miRBase with zero mismatches) and a non-conserved file (non-matching reads) in the workdir; the non-conserved fraction remains the input for downstream novel miRNA prediction (miRCat) and tasiRNA prediction (ta-si predictor) and target validation (PAREsnip). Because miRPursuit is organized into Pre-preprocessing, Filtering, Annotation and Reporting stages, re-running with different miRProf parameters can start from the Annotation stage onward without redoing earlier steps.

## Related tools

- **miRProf** (Aligns genome-mapped reads of each sRNA library to the miRBase mature database with 0 mismatches and splits them into conserved and non-conserved fractions) — http://srna-workbench.cmp.uea.ac.uk
- **miRPursuit** (Pipeline (bash scripts) that orchestrates the Annotation stage, reads wbench_mirprof.cfg, and connects miRProf output to downstream miRCat / ta-si / PAREsnip steps) — https://github.com/forestbiotech-lab/miRPursuit
- **PatMaN** (Upstream pattern matcher that maps reads to the genome with 0 mismatches, producing the genome-mapped reads that miRProf consumes) — https://bioinf.eva.mpg.de/patman/
- **miRBase** (Reference database (mature.fa) of conserved miRNA sequences used as the alignment target in the Annotation stage)

## Examples

```
bash pathToMiRPursuit/miRPursuit.sh -f 1 -l 2 --fasta test_dataset- -s 2
```

## Evaluation signals

- Every read in the conserved fraction matches miRBase with exactly 0 mismatches
- All conserved reads fall within the 18–26 nt length window and meet min_abundance=5
- Hits are grouped by mismatch, organism and variant (group_mismatches=true, group_organisms=true, group_variant=true) with mature and star NOT merged (group_mature_and_star=false)
- The non-conserved fraction is non-empty and retained in the workdir as input for miRCat novel-miRNA and ta-si prediction
- Counts of miRBase hits per library appear in the counts directory as space-separated files matching the %y%m%d:%h%m%s-type-lib[lib_first]-[lib_last].tsv naming scheme

## Limitations

- miRPursuit performs no QC step yet (fastqc not implemented), so read-quality problems upstream propagate into the conserved/non-conserved split
- Detailed reporting tables and the final report file generation are not yet completed (TODO), so conserved-miRNA results must be inspected via counts/logs rather than a polished report
- PatMaN mapping uses 0 mismatches, so genome-mapping errors can drop reads before miRProf ever sees them; conserved annotation is also limited to sequences present in the miRBase release used
- Pipeline expects UNIX/Java environment (Oracle Java ~8 per article; README mentions Java ~1.7 and perl 5.8) — running miRProf standalone outside miRPursuit requires reproducing the config parameters manually

## Evidence

- [intro] The mapped reads are then aligned to the miRBase⁺ (ref) database using miRProf: "The mapped reads are then aligned to the miRBase⁺ (ref) database using miRProf"
- [intro] Four-stage workflow: "This is useful if you want to re-do the analysis of the pipeline with different parameters only from a specific stage onward."
- [methods] Filter config defaults: "#Broad range default values
      min_length=18
      max_length=26
      min_abundance=5"
- [methods] miRBase path config: "#Path to the mirbase database. Go to http://www.mirbase.org or download latest from: ftp://mirbase.org/pub/mirbase/CURRENT/"
- [intro] PatMaN genome mapping upstream: "The reads are further filtered by mapping them to the setup genome file with '0' mismatches using patman."
- [readme] README pipeline steps: "Step 2: Filter Genome & miRBase"
