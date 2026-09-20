---
name: read-configuration-file-authoring
description: 'Use when preparing to run FAST-SG.pl on raw or Lordec-corrected ONT/PacBio
  libraries: you must first write the read configuration file in the space-separated
  format (#type libID Path(long read) Insert-sizes SAM) that the -l argument expects.'
when_to_use_negative:
- The downstream scaffolding graph already exists and you only need to run a scaffolder
  like Links on the extracted synthetic libraries
- The input is short-read Illumina data only — the configuration file is for long-read
  libraries extracted against an Illumina assembly, not for the assembly input itself
tools:
- name: Fast-SG
  role: Consumes the authored read configuration file via the -l flag of FAST-SG.pl
    to build the alignment-free scaffolding graph and extract synthetic mate-pair
    libraries
  repo: https://github.com/adigenova/fast-sg
- name: PERL
  role: Runtime for the FAST-SG.pl driver script that parses the read configuration
    file
- name: KMC
  role: k-mer counter bundled inside the Fast-SG directory and used during graph construction
    on the libraries declared in the configuration file
  repo: https://github.com/refresh-bio/KMC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:1ef1dc8729a8a727f2c80730d64d33b9fdcbb03444737b46e74b3d98ee394c04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Fast-SG
  - KMC
  - PERL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# read-configuration-file-authoring

## Summary

Authors the space-separated long-read configuration file that Fast-SG consumes via the -l flag, declaring each ONT/PacBio library's type, ID, path, insert size, and SAM orientation so synthetic mate-pair libraries can be extracted alignment-free.

## When to use

Use when preparing to run FAST-SG.pl on raw or Lordec-corrected ONT/PacBio libraries: you must first write the read configuration file in the space-separated format (#type libID Path(long read) Insert-sizes SAM) that the -l argument expects.

## When NOT to use

- The downstream scaffolding graph already exists and you only need to run a scaffolder like Links on the extracted synthetic libraries
- The input is short-read Illumina data only — the configuration file is for long-read libraries extracted against an Illumina assembly, not for the assembly input itself

## Inputs

- Raw ONT R9.2 long-read library files (E. coli K12)
- Raw PacBio Sequel long-read library files (E. coli K12)
- Lordec hybrid error-corrected versions of the same ONT and PacBio libraries
- Insert-size specifications spanning 0.5-8 kb per library

## Outputs

- Space-separated read configuration file (one '#type libID Path(long read) Insert-sizes SAM' line per library), e.g. ecoli-reads.txt format
- File consumable by FAST-SG.pl via the -l argument

## How to apply

Write one line per long-read library using the space-separated format '#type libID Path(long read) Insert-sizes SAM', where #type declares the read kind, libID names the library, Path points to the ONT or PacBio long-read file, an insert size is given, and the SAM field carries the pair orientation. For the E. coli K12 benchmark, list each ONT R9.2 and PacBio Sequel library with insert sizes spanning 0.5-8 kb, including both raw and Lordec hybrid error-corrected versions as separate entries so the four technology/correction-state combinations are all represented. Model the file on the shipped example examples/ecoli-reads.txt rather than inventing the schema. Pass the finished file to FAST-SG.pl with -l alongside -k 15 (or a k-mer range such as 15-25:5), -r for the Illumina E. coli K12 assembly contigs in FASTA, and -p for the output prefix. Keep declared insert sizes consistent with the reads: the insert size is restricted to the actual long-read length, so a declared size cannot exceed the physical read length.

## Related tools

- **Fast-SG** (Consumes the authored read configuration file via the -l flag of FAST-SG.pl to build the alignment-free scaffolding graph and extract synthetic mate-pair libraries) — https://github.com/adigenova/fast-sg
- **PERL** (Runtime for the FAST-SG.pl driver script that parses the read configuration file)
- **KMC** (k-mer counter bundled inside the Fast-SG directory and used during graph construction on the libraries declared in the configuration file) — https://github.com/refresh-bio/KMC

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- FAST-SG.pl accepts the file with -l and runs to completion without configuration-format errors
- Every line contains all five space-separated fields: #type, libID, path to the long-read file, insert size, and SAM orientation tag
- The file enumerates all four technology/correction-state combinations (raw/corrected ONT R9.2 and raw/corrected PacBio Sequel)
- Declared insert sizes fall within the 0.5-8 kb span used in the benchmark and do not exceed the actual long-read lengths
- Downstream recall-metrics table contains one parseable row (values in [0, 100]) per declared library combination

## Limitations

- The insert size is restricted to the actual long-read size, so declared insert sizes cannot exceed physical read lengths
- Gap size estimation with existing long-range mate-pair technologies (10X genomics and DOVETAIL genomics) is challenging because linking information comes from a range of insert sizes and read-pair orientation may be unknown
- No version number is stated for Fast-SG itself (only RRID SCR_015934), so behavior may drift from the documented configuration format

## Evidence

- [methods] Example read configuration file location: "Example of a read configuration file can be found in examples/ecoli-reads.txt"
- [methods] FAST-SG.pl invocation consuming the configuration file: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [discussion] Insert size constrained by read length: "The insert size is restricted to the actual long-read size."
- [abstract] Fast-SG extracts synthetic libraries from raw or corrected long reads: "can be used to extract linking information from either raw or error-corrected long reads"
