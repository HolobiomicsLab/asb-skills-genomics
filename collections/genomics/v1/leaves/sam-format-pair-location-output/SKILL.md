---
name: sam-format-pair-location-output
description: Use when a scaffolding-graph pipeline must convert alignment-free read-to-contig
  placements into standard SAM pair records that short-read scaffolders can consume,
  or when validating Fast-SG's -p output.
when_to_use_negative:
- You need full base-level alignments with mapping quality and CIGAR strings — Fast-SG
  reports only pair locations, not alignments.
- Linking information comes from long-range technologies with unknown read-pair orientation
  and mixed insert sizes (e.g. 10X Genomics, DOVETAIL), where gap-size estimation
  from these SAM pairs is documented as challenging.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Fast-SG
  role: Produces the SAM-format pair-location file via FAST-SG.pl with the -p output
    prefix
  repo: https://github.com/adigenova/fast-sg
- name: ntHash
  role: Rolling hash over read k-mers used to query the unique-k-mer index before
    pair locations are emitted
  repo: https://github.com/bcgsc/ntHash
- name: KMC
  role: Counts k-mers over contigs to build the unique k-mer index backing the reported
    pair locations
  repo: https://github.com/refresh-bio/KMC
- name: kseqcpp
  role: Parses the read files listed in the read configuration for scoring and pair-location
    output
  repo: https://github.com/ctSkennerton/kseqcpp
- name: BBHash
  role: Minimal perfect hash over unique k-mers supporting the k-mer look-up behind
    each reported location
  repo: https://github.com/rizkg/BBHash
- name: quasi_dictionary
  role: Probabilistic membership structure backing the k-mer index used to place reads
  repo: https://github.com/pierrepeterlongo/quasi_dictionary
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:b73ff07f4d8abf7b6efae3954a266e6be405f3b553d951771843718051645f42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - BBHash
  - Fast-SG
  - KMC
  - g++
  - kseqcpp
  - ntHash
  - quasi_dictionary
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# sam-format-pair-location-output

## Summary

Emits Fast-SG's alignment-free read placement results as a SAM-format file of pair locations under the -p output prefix, using SAM flags to distinguish single (1) versus paired (2) read configurations so downstream scaffolders can build the scaffolding graph.

## When to use

Use when a scaffolding-graph pipeline must convert alignment-free read-to-contig placements into standard SAM pair records that short-read scaffolders can consume, or when validating Fast-SG's -p output.

## When NOT to use

- You need full base-level alignments with mapping quality and CIGAR strings — Fast-SG reports only pair locations, not alignments.
- Linking information comes from long-range technologies with unknown read-pair orientation and mixed insert sizes (e.g. 10X Genomics, DOVETAIL), where gap-size estimation from these SAM pairs is documented as challenging.

## Inputs

- FASTA/FASTQ read files listed in a read configuration file (e.g. examples/ecoli-reads.txt)
- Contig sequences (e.g. example/ecoli-illumina.fa.gz) indexed via KMC3 + BBHash + quasi_dictionary
- k-mer size (-k, supported range 12-256)
- Output prefix given via the -p argument

## Outputs

- SAM-format file of pair locations named under the -p output prefix
- SAM records carrying flags that encode single (1) vs paired (2) read configuration

## How to apply

After building the unique k-mer index (KMC3 counts over the contigs, BBHash minimal perfect hash plus quasi_dictionary membership), stream reads with kseqcpp and compute ntHash rolling hashes over each read's k-mers to fill hit vectors against the indexed contigs. Apply windowed scoring over the hit vectors to localize reads, enforcing the Illumina thresholds (forward read score > 3, each read score > 5, rescue score 4) so only confident or jointly consistent mate pairs are retained; attempt pair rescue when individual placement is weak. Serialize the surviving pair locations in SAM format to the file named by the -p prefix argument, setting SAM flags per the read configuration declared in the library file (ecoli-reads.txt), where 1 denotes single and 2 denotes paired reads. Validate by compiling with g++ 5.3 (Linux) or clang 4.2 (Mac OSX) and running 'make test', then confirm the SAM file exists, parses as SAM, and contains pair records for the paired libraries in the read configuration.

## Related tools

- **Fast-SG** (Produces the SAM-format pair-location file via FAST-SG.pl with the -p output prefix) — https://github.com/adigenova/fast-sg
- **ntHash** (Rolling hash over read k-mers used to query the unique-k-mer index before pair locations are emitted) — https://github.com/bcgsc/ntHash
- **KMC** (Counts k-mers over contigs to build the unique k-mer index backing the reported pair locations) — https://github.com/refresh-bio/KMC
- **kseqcpp** (Parses the read files listed in the read configuration for scoring and pair-location output) — https://github.com/ctSkennerton/kseqcpp
- **BBHash** (Minimal perfect hash over unique k-mers supporting the k-mer look-up behind each reported location) — https://github.com/rizkg/BBHash
- **quasi_dictionary** (Probabilistic membership structure backing the k-mer index used to place reads) — https://github.com/pierrepeterlongo/quasi_dictionary

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- The SAM file named by the -p prefix exists and parses as valid SAM after 'make test'.
- The SAM file contains pair records for every paired library declared in the read configuration (ecoli-reads.txt).
- SAM flags conform to the declared read configuration convention: 1 for single reads, 2 for paired reads.
- Retained pairs satisfy the documented Illumina thresholds (forward read > 3, each read > 5, rescue score 4); weak individual placements only survive if the pair-rescue step deems the joint placement consistent.
- Running over a k-mer range (e.g. -k 15-25:5) yields SAM outputs consistent in record structure across k values.

## Limitations

- Synthetic library insert size is restricted to the actual long-read size.
- Fast-SG reports pair locations only, not base-level alignments; downstream gap-size estimation with long-range mate-pair technologies (10X Genomics, DOVETAIL) is challenging because insert sizes vary and read-pair orientation may be unknown.
- The quasi_dictionary introduces false positives at a rate below 1/2^fingerprint_size, so reported locations rely on probabilistic k-mer membership.

## Evidence

- [other] For Illumina mate-pair reads, Fast-SG performs rolling-hash k-mer look-ups in its unique-k-mer index, fills hit vectors, and reports pair locations only when the forward read scores above 3 and each read scores above 5, otherwise attempting pair rescue; pair locations are output in SAM format.: [evidence span withheld — anchor status: not_found]
- [methods] Fast-SG usage example with output prefix: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [readme] README project description: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
- [methods] Compilation and testing instructions: "Compilation was tested with g++ version 5.3 (Linux) and clang version 4.2 (Mac OSX)"
