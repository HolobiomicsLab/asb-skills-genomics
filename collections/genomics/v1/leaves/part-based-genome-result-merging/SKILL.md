---
name: part-based-genome-result-merging
description: Use when running the miRPursuit novel-miRNA branch (Step 4, miRCat) on
  a large reference genome such as TAIR10 on a machine with limited RAM, requiring
  genome-part execution and result merging.
when_to_use_negative:
- When sufficient RAM is available to run miRCat against the whole genome in one pass
  — the README states to set GENOME_MIRCAT=${GENOME} if you don't need to run various
  parts
- When the goal is conserved miRNA quantification (miRProf against miRBase) rather
  than novel miRNA prediction from non-conserved reads
- When the analysis target is tasiRNA prediction (Step 3, ta-si predictor) rather
  than Step 4 miRCat
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: miRCat
  role: UEA sRNA workbench tool run as Step 4 of miRPursuit to search genome parts
    for hairpin precursor loci using the non-conserved reads
  repo: http://srna-workbench.cmp.uea.ac.uk/tools/analysis-tools/mircat/
- name: miRPursuit
  role: Bash pipeline that wires the split-genome variable GENOME_MIRCAT, merges part-based
    miRCat results, applies the max_genome_hits filter, and writes mircat_novel_mirnas.csv
    and logs
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: Java 8
  role: Runtime environment for the UEA sRNA workbench tools including miRCat
  repo: https://www.java.com
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:11aec5122788a0b3a64f67e946ebf94bed4eb9848582c67834447dd16bd52a24
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Java 8
  - miRCat
  - miRPursuit
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# part-based-genome-result-merging

## Summary

Handles RAM-limited novel miRNA prediction in miRPursuit by running miRCat against split genome part files (e.g. GENOME_MIRCAT=${GENOME/.fa/part-1.fa}) instead of the full genome, merging the part-based prediction sets, and filtering merged hits by the max_genome_hits threshold in wbench_mircat.cfg.

## When to use

Use when running the miRPursuit novel-miRNA branch (Step 4, miRCat) on a large reference genome such as TAIR10 on a machine with limited RAM, requiring genome-part execution and result merging.

## When NOT to use

- When sufficient RAM is available to run miRCat against the whole genome in one pass — the README states to set GENOME_MIRCAT=${GENOME} if you don't need to run various parts
- When the goal is conserved miRNA quantification (miRProf against miRBase) rather than novel miRNA prediction from non-conserved reads
- When the analysis target is tasiRNA prediction (Step 3, ta-si predictor) rather than Step 4 miRCat

## Inputs

- Filtered non-conserved sRNA library reads (FASTA) from the miRPursuit Filtering/Annotation stages
- Split reference genome part file (e.g. part-1.fa derived from the TAIR10 genome FASTA)
- wbench_mircat.cfg configuration file with miRCat parameters (extend=100.0, min_energy=-25.0, max_genome_hits=16, pval=0.05, etc.)

## Outputs

- Merged set of putative novel miRNA precursor loci across all genome parts
- mircat_novel_mirnas.csv containing surviving novel miRNA precursors and loci after max_genome_hits filtering
- Run log files recording all miRCat parameters used

## How to apply

Load the wbench_mircat.cfg parameter set (broad defaults: min_energy=-25.0, min_paired=17, max_gaps=3, max_genome_hits=16, min_length=18, max_length=26, pval=0.05, min_abundance=1; plant-mode variant: min_length=20, max_length=22, max_unpaired=50). Point GENOME_MIRCAT at a genome part file (e.g. ${GENOME/.fa/part-1.fa}) and run miRCat from the UEA sRNA workbench as Step 4 of the pipeline, aligning the filtered non-conserved library reads to that part of the genome to find putative hairpin precursor loci. Repeat for all genome parts, then merge the part-based prediction sets into a single result set. Finally, filter the merged predictions to remove miRNAs that pair with more genome sites than max_genome_hits=16, and write the surviving novel miRNA precursors and loci to mircat_novel_mirnas.csv with all used parameters recorded in the run log files. The rationale is that miRCat's genome search is memory-intensive, so the README recommends the part-file variable when the user has a short amount of RAM.

## Related tools

- **miRCat** (UEA sRNA workbench tool run as Step 4 of miRPursuit to search genome parts for hairpin precursor loci using the non-conserved reads) — http://srna-workbench.cmp.uea.ac.uk/tools/analysis-tools/mircat/
- **miRPursuit** (Bash pipeline that wires the split-genome variable GENOME_MIRCAT, merges part-based miRCat results, applies the max_genome_hits filter, and writes mircat_novel_mirnas.csv and logs) — https://github.com/forestbiotech-lab/miRPursuit
- **Java 8** (Runtime environment for the UEA sRNA workbench tools including miRCat) — https://www.java.com

## Examples

```
bash pathToMiRPursuit/miRPursuit.sh -f 1 -l 2 --fasta test_dataset-
```

## Evaluation signals

- The run log files record all wbench_mircat.cfg parameters actually used (extend=100.0, min_energy=-25.0, max_genome_hits=16, pval=0.05, etc.)
- The merged result set contains predictions from every genome part file processed (no parts silently dropped)
- No surviving miRNA in the merged set pairs with more than max_genome_hits=16 genome sites
- mircat_novel_mirnas.csv exists in the workdir and miRNA lengths fall within the configured min_length/max_length bounds (18–26 broad defaults, or 20–22 in plant mode)
- Step-4 count/log files in [workdir]/logs and [workdir]/counts use the %y%m%d:%h%m%s-type naming and *.log.ok completion markers

## Limitations

- Splitting the genome is only necessary under short-RAM conditions; if misconfigured (GENOME_MIRCAT not pointing to all parts), part predictions may be incomplete before merging
- max_genome_hits filtering (default 16) discards multi-mapping miRNAs, which may remove real repeats/paralogous loci
- miRCat predictions depend on RNAfold, which in the README setup depends on /lib/ld-linux.so.2 (libc6-i386 on Ubuntu); missing 32-bit support breaks precursor folding

## Evidence

- [readme] GENOME_MIRCAT  _The path to the genome to be used by mircat. Set to ${GENOME} if you don't need to run various parts.: [evidence span withheld — anchor status: external_doc]
