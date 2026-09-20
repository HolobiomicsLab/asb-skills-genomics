---
name: novel-mirna-prediction
description: 'Use when the Annotation stage must identify novel miRNA precursors:
  non-conserved reads remain after miRBase annotation and a reference genome (e.g.
  TAIR10 .fa) is available for hairpin search with miRCat.'
when_to_use_negative:
- Input reads are conserved miRNAs already aligned to miRBase⁺ via miRProf — novel
  prediction uses only the non-conserved fraction.
- The goal is tasiRNA prediction — the non-conserved reads are also routed through
  the ta-si predictor (Step 3), which is a separate branch from miRCat.
- No reference genome is supplied; miRCat requires a user-specified genome to search
  for precursor hairpin loci.
edam_operation: http://edamontology.org/operation_0464
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: miRCat
  role: UEA sRNA workbench tool that searches the reference genome with non-conserved
    reads for putative miRNA hairpin precursor loci
  repo: http://srna-workbench.cmp.uea.ac.uk/tools/analysis-tools/mircat/
- name: miRPursuit
  role: Bash pipeline that orchestrates miRCat as Step 4, merges part-genome results,
    applies max_genome_hits filtering, and writes mircat_novel_mirnas.csv
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: Java 8
  role: Runtime for miRCat; miRPursuit works best with Oracle's Java v.8, and MEMORY
    (e.g. 10g, 2000m) bounds memory-intensive java steps
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:b923572263482212881a92d62928c74c0f2b354417242a96e17066ae5c91e8ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Java
  - Java 8
  - miRCat
  - miRPursuit
  edam_operation: http://edamontology.org/operation_0464
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# novel-mirna-prediction

## Summary

Predict novel plant miRNAs by running filtered non-conserved sRNA reads through miRCat (UEA sRNA workbench) against a reference genome to find putative hairpin precursor loci, merging part-based results and filtering multi-mapping predictions.

## When to use

Use when the Annotation stage must identify novel miRNA precursors: non-conserved reads remain after miRBase annotation and a reference genome (e.g. TAIR10 .fa) is available for hairpin search with miRCat.

## When NOT to use

- Input reads are conserved miRNAs already aligned to miRBase⁺ via miRProf — novel prediction uses only the non-conserved fraction.
- The goal is tasiRNA prediction — the non-conserved reads are also routed through the ta-si predictor (Step 3), which is a separate branch from miRCat.
- No reference genome is supplied; miRCat requires a user-specified genome to search for precursor hairpin loci.

## Inputs

- Filtered non-conserved sRNA library reads (FASTA, produced by Steps 1–3 of miRPursuit: adaptor trimming via fastx_clipper, Wbench Filter, Filter Genome & miRBase)
- Reference genome FASTA (e.g. Arabidopsis_thaliana.TAIR10.dna_rm.chromosome.4.fa from the testDataset), possibly split into part-*.fa files
- wbench_mircat.cfg configuration file
- workdirs.cfg settings (GENOME_MIRCAT, MEMORY, THREADS, workdir paths)

## Outputs

- mircat_novel_mirnas.csv containing novel miRNA precursors and genomic loci
- Run logging files recording all miRCat parameters used
- Read-count statistics (space-separated tsv) in [workdir]/counts

## How to apply

Load the wbench_mircat.cfg parameter set (broad defaults: extend=100.0, min_energy=-25.0, min_paired=17, max_gaps=3, max_genome_hits=16, min_length=18, max_length=26, min_gc=20, max_unpaired=60, pval=0.05, min_abundance=1; plant-mode variant: min_length=20, max_length=22, max_unpaired=50, Thread_Count=20). As Step 4 of the miRPursuit pipeline (after Step 3 Tasi), execute miRCat from the UEA sRNA workbench, aligning the filtered non-conserved library reads to the genome. If RAM is restricted, set GENOME_MIRCAT=${GENOME/.fa/part-1.fa} in workdirs.cfg to run against a split genome part file, then merge predictions from all genome parts into a single result set. Filter the merged predictions to remove miRNAs pairing with more genome sites than max_genome_hits=16. Write surviving novel miRNA precursors and loci to mircat_novel_mirnas.csv and log all used parameters in the [workdir]/logs files; read counts go to the counts directory (space-separated tsv with format %y%m%d:%h%m%s-type-lib[lib_first]-[lib_last].tsv).

## Related tools

- **miRCat** (UEA sRNA workbench tool that searches the reference genome with non-conserved reads for putative miRNA hairpin precursor loci) — http://srna-workbench.cmp.uea.ac.uk/tools/analysis-tools/mircat/
- **miRPursuit** (Bash pipeline that orchestrates miRCat as Step 4, merges part-genome results, applies max_genome_hits filtering, and writes mircat_novel_mirnas.csv) — https://github.com/forestbiotech-lab/miRPursuit
- **Java 8** (Runtime for miRCat; miRPursuit works best with Oracle's Java v.8, and MEMORY (e.g. 10g, 2000m) bounds memory-intensive java steps)

## Examples

```
bash pathToMiRPursuit/miRPursuit.sh -f 1 -l 2 --fasta test_dataset- --step 4
```

## Evaluation signals

- mircat_novel_mirnas.csv exists and contains precursor and locus records for each processed library
- No predicted miRNA in the output aligns to more than max_genome_hits=16 genome sites
- All wbench_mircat.cfg parameters used (e.g. min_energy=-25.0, pval=0.05, plant-mode min_length=20/max_length=22) are recorded in the [workdir]/logs files
- When the genome was split, predictions from all part-*.fa files were merged before filtering (no per-part-only results remain)
- Counts tsv files in [workdir]/counts follow the %y%m%d:%h%m%s-type-lib[lib_first]-[lib_last].tsv naming and are space-separated

## Limitations

- miRCat can be memory intensive; with short RAM the genome must be split into parts (GENOME_MIRCAT=${GENOME/.fa/part-1.fa}) and results merged, adding a merge/filter step that can fail if parts are not all processed
- Novel miRNA calls depend on upstream filtering (length, abundance, low-complexity, t/rRNA removal, 0-mismatch patman genome mapping); overly strict wbench filter settings can starve miRCat of reads
- miRPursuit's reporting stage is incomplete (various tables and report file generation still TODO), so downstream annotation of novel miRNAs may require manual handling

## Evidence

- [intro] four-stage workflow with reanalysis entry points: [evidence span withheld — anchor status: not_found]
- [readme] GENOME_MIRCAT supports split genome for RAM restrictions: [evidence span withheld — anchor status: external_doc]
