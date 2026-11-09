---
name: sum-of-pairs-alignment-scoring
description: Use when an MSA output FASTA from FMAlign2 or its backends must be quality-checked
  or compared across methods (e.g., Y-chromosome truncation or mtDB 20x/50x/100x benchmarks)
  via an SP score.
when_to_use_negative:
- The alignment file is not a valid FASTA MSA (e.g., raw unaligned input.fasta) —
  sp.py expects aligned sequences of equal length.
- A different scoring scheme is required by the study design; the documented scheme
  (match 0, mismatch 1, gap1 2, gap2 2) must be kept fixed for cross-method comparability.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: sp.py
  role: Computes the Sum-of-Pairs score of an MSA output FASTA using the fixed scoring
    scheme (match 0, mismatch 1, gap1 2, gap2 2) to evaluate alignment quality.
  repo: https://github.com/malabz/FMAlign2
- name: FMAlign2
  role: Produces the aligned output FASTA (with -p mafft|clustalo|halign3|halign4
    backends) that sp.py then scores in benchmark comparisons.
  repo: https://github.com/malabz/FMAlign2
provenance:
  source_task_ids:
  - task_004
  - task_008
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:23bc879ea2125fbdacae68acedc9cc2b9cf0d9f43e0549c1fe7ad731e546b6f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/malabz/FMAlign2
  tools:
  - Clustal Omega
  - FMAlign
  - FMAlign2
  - HAlign3
  - HAlign4
  - MAFFT
  - sp.py
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# sum-of-pairs-alignment-scoring

## Summary

Score multiple nucleotide sequence alignments produced by FMAlign2 (or its MAFFT/Clustal Omega/HAlign3/HAlign4 backends) using the sp.py script, which computes the Sum-of-Pairs (SP) score under a match 0 / mismatch 1 / gap1 2 / gap2 2 scheme to verify alignment quality in benchmark comparisons.

## When to use

Use when an MSA output FASTA from FMAlign2 or its backends must be quality-checked or compared across methods (e.g., Y-chromosome truncation or mtDB 20x/50x/100x benchmarks) via an SP score.

## When NOT to use

- The alignment file is not a valid FASTA MSA (e.g., raw unaligned input.fasta) — sp.py expects aligned sequences of equal length.
- A different scoring scheme is required by the study design; the documented scheme (match 0, mismatch 1, gap1 2, gap2 2) must be kept fixed for cross-method comparability.

## Inputs

- Aligned multiple sequence alignment in FASTA format (output.fasta from FMAlign2 or a standalone backend)
- scoring scheme parameters (--match 0 --mismatch 1 --gap1 2 --gap2 2)

## Outputs

- Sum-of-Pairs (SP) score printed by sp.py for the input alignment
- per-method SP scores compiled into a benchmark table alongside runtimes and success/failure flags

## How to apply

After FMAlign2 produces an aligned output FASTA (via FMAlign2 -i input.fasta -o output.fasta with a chosen -p backend), run the sp.py script with the fixed scoring scheme --match 0 --mismatch 1 --gap2 2 --gap2 2 (i.e., --gap1 2 --gap2 2) to compute the Sum-of-Pairs score. Record the SP score for every completed alignment — for each truncation length (10,000 bp to 15 Mbp human Y chromosome) or duplication level (mtDB 20x/50x/100x) and each method — so accuracy is checked alongside wall-clock runtime. Validation logic: the run is accepted if output.fasta exists as a valid FASTA alignment and sp.py successfully computes and prints its SP score; relative SP scores across methods confirm that FMAlign2's speedup does not sacrifice alignment accuracy. Keep scoring parameters fixed across all methods so that SP scores are directly comparable in the benchmark table.

## Related tools

- **sp.py** (Computes the Sum-of-Pairs score of an MSA output FASTA using the fixed scoring scheme (match 0, mismatch 1, gap1 2, gap2 2) to evaluate alignment quality.) — https://github.com/malabz/FMAlign2
- **FMAlign2** (Produces the aligned output FASTA (with -p mafft|clustalo|halign3|halign4 backends) that sp.py then scores in benchmark comparisons.) — https://github.com/malabz/FMAlign2

## Examples

```
python sp.py --input output.fasta --match 0 --mismatch 1 --gap1 2 --gap2 2
```

## Evaluation signals

- sp.py runs without error and prints an SP score for output.fasta, confirming the file is a valid FASTA alignment.
- The same scoring scheme (--match 0 --mismatch 1 --gap1 2 --gap2 2) is used for every method and dataset so SP scores are comparable.
- FMAlign2-produced alignments maintain SP scores comparable to standalone MAFFT/Clustal Omega/HAlign3/HAlign4 baselines, confirming accuracy is maintained despite speedup.
- For the ultralong benchmark, SP scores exist for every truncation length where FMAlign2 completed (up to ≥10 Mbp), whereas HAlign2-centered methods' failures leave no score past 100,000 bp.

## Limitations

- sp.py requires a Python environment, which must be available in the runtime.
- The SP score alone does not capture runtime; benchmark conclusions require pairing it with wall-clock time and success/failure records per method.
- If a run stalls with default MEM parameters (-l 30, -f accurate), the retry with -f fast may alter results, so record the mode used for each scored alignment.

## Evidence

- [methods] sp.py invocation with fixed scoring scheme: "you can run the `sp.py` script (requires a Python environment)"
- [readme] README scoring command and interpretation: "python sp.py --input output.fasta --match 0 --mismatch 1 --gap1 2 --gap2 2"
- [readme] README SP score rationale: "By running this command, you will obtain the SP score, which provides an evaluation of the alignment quality."
- [abstract] Abstract accuracy claim: "Compared to FMAlign, FMAlign2 markedly augments the segmentation of sequences and significantly reduces the time while maintaining accuracy, especially on ultralong datasets."
