---
name: srna-library-file-partitioning
description: Use when sRNA sequencing reads have already been genome-mapped with PatMan
  (0 mismatches) and you must split each library into miRBase-conserved vs non-conserved
  read files before novel miRNA or ta-siRNA prediction.
when_to_use_negative:
- Input reads have not yet been filtered and mapped to the genome with PatMan (0 mismatches)
  — the Filtering stage must run first
- The goal is novel miRNA discovery itself rather than separating conserved from non-conserved
  reads — use miRCat on the non-conserved fraction instead
- Conserved miRNAs should be identified against a custom reference other than miRBase
  mature sequences
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: miRProf
  role: Aligns genome-mapped reads to the miRBase mature database and separates matching
    (conserved) from non-matching (non-conserved) reads per library
  repo: http://srna-workbench.cmp.uea.ac.uk
- name: miRPursuit
  role: Pipeline that drives the Annotation stage, supplies configuration via wbench_mirprof.cfg
    and routes the two read partitions to downstream tools
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: patman
  role: Produces the genome-mapped reads (0 mismatches) that are the input to the
    miRProf-based partitioning
  repo: http://bioinf.eva.mpg.de/patman
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:31fec5fb9a80dce11ec1d44c2750304c48acd37317e9b03ea986bbdbaa1fa2fd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - miRProf
  - miRPursuit
  - patman
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# srna-library-file-partitioning

## Summary

Partitions the genome-mapped reads of each sRNA library into a conserved fraction (reads matching the miRBase mature database) and a non-conserved fraction using miRProf in the miRPursuit Annotation stage, so downstream novel miRNA (miRCat) and ta-siRNA prediction operate only on non-conserved reads.

## When to use

Use when sRNA sequencing reads have already been genome-mapped with PatMan (0 mismatches) and you must split each library into miRBase-conserved vs non-conserved read files before novel miRNA or ta-siRNA prediction.

## When NOT to use

- Input reads have not yet been filtered and mapped to the genome with PatMan (0 mismatches) — the Filtering stage must run first
- The goal is novel miRNA discovery itself rather than separating conserved from non-conserved reads — use miRCat on the non-conserved fraction instead
- Conserved miRNAs should be identified against a custom reference other than miRBase mature sequences

## Inputs

- Genome-mapped sRNA library reads (FASTA, produced by PatMan mapping to the setup genome with 0 mismatches)
- miRBase mature miRNA reference database (mature.fa, e.g. downloaded from ftp://mirbase.org/pub/mirbase/CURRENT/)
- wbench_mirprof.cfg configuration file

## Outputs

- Per-library conserved reads file (reads matching miRBase) written to the workdir
- Per-library non-conserved reads file (non-matching reads) for downstream ta-siRNA and novel miRNA prediction
- miRBase hit counts stored in the [workdir]/counts directory

## How to apply

Configure miRProf through the wbench_mirprof.cfg file, pointing MIRBASE at ${source_data}/mirbase/mature.fa and reading the genome-mapped library reads from the workdir. Run miRProf (UEA sRNA workbench) with mismatches=0, overhangs=true, only_keep_best=true, min_length=18, max_length=26, and min_abundance=5, and grouping options group_mismatches=true, group_organisms=true, group_variant=true, group_mature_and_star=false so hits are grouped by mismatch, organism and variant. For each library, split the aligned reads into a conserved file (reads matching miRBase with 0 mismatches in the 18-26 nt window at min_abundance=5) and a non-conserved file of non-matching reads. The non-conserved fraction is then retained as input for the ta-si predictor and miRCat novel-miRNA prediction steps of the pipeline.

## Related tools

- **miRProf** (Aligns genome-mapped reads to the miRBase mature database and separates matching (conserved) from non-matching (non-conserved) reads per library) — http://srna-workbench.cmp.uea.ac.uk
- **miRPursuit** (Pipeline that drives the Annotation stage, supplies configuration via wbench_mirprof.cfg and routes the two read partitions to downstream tools) — https://github.com/forestbiotech-lab/miRPursuit
- **patman** (Produces the genome-mapped reads (0 mismatches) that are the input to the miRProf-based partitioning) — http://bioinf.eva.mpg.de/patman

## Examples

```
bash pathToMirPursuit/miRPursuit.sh -f 1 -l 2 --fasta test_dataset- -s 2
```

## Evaluation signals

- Every read in the conserved fraction matches miRBase with 0 mismatches (mismatches=0 setting)
- Conserved reads fall within the 18-26 nt length window and pass min_abundance=5
- Each library produces exactly two output files (conserved and non-conserved) and the read counts of the two fractions sum to the genome-mapped input count
- Hits are grouped by mismatch, organism and variant (group_mismatches=true, group_organisms=true, group_variant=true) with group_mature_and_star=false
- Downstream steps (miRCat, ta-si predictor) receive only the non-conserved fraction as input

## Limitations

- The partition depends on the miRBase release at MIRBASE path; conserved status changes with database updates
- Strict 0-mismatch alignment to miRBase may miss divergent conserved miRNAs in non-model species
- Non-conserved reads remain only if they survived the upstream Filtering stage (length 18-26, abundance >= 5, t/r RNA and low-complexity removal)

## Evidence
