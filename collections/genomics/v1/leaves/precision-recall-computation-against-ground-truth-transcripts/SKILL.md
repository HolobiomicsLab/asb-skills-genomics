---
name: precision-recall-computation-against-ground-truth-transcripts
description: Use when evaluating transcript reconstruction on simulated targeted Iso-Seq
  data where the ground-truth isoform set is known, e.g. comparing IsoCon against
  ICE precision/recall across read depths for multigene families like TSPY, HSFY,
  DAZ.
when_to_use_negative:
- Experimental datasets with no ground-truth transcript set — precision/recall against
  truth cannot be computed
- Genome-wide (untargeted) transcriptome reconstruction — IsoCon assumes reads span
  the target end-to-end, as in targeted Iso-Seq of gene families
- Candidate transcripts that already exist and only need statistical filtering (use
  IsoCon's stat_filter instead of re-running the full benchmark)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: IsoCon
  role: Reconstructs predicted transcripts from simulated full-length reads; evaluated
    method
  repo: https://github.com/ksahlin/IsoCon
- name: ICE
  role: PacBio Iso-Seq clustering/correction baseline comparator run on identical
    simulated read sets
- name: IsoCon_Eval
  role: Snakemake workflow driving all simulations, method runs, and precision/recall
    evaluations
  repo: https://github.com/ksahlin/IsoCon_Eval
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:18f3fa02039f4455fe3e3cf80e8ee2c695afc562a47a8ce62a2152afeee6a904
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - Iso-Seq
  - IsoCon
  - IsoCon_Eval
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# precision-recall-computation-against-ground-truth-transcripts

## Summary

Benchmark de novo transcript reconstruction methods (IsoCon vs ICE) on simulated PacBio Iso-Seq read sets with known ground-truth isoforms by scoring nucleotide-level precision and recall per gene family, mutation rate, and read depth.

## When to use

Use when evaluating transcript reconstruction on simulated targeted Iso-Seq data where the ground-truth isoform set is known, e.g. comparing IsoCon against ICE precision/recall across read depths for multigene families like TSPY, HSFY, DAZ.

## When NOT to use

- Experimental datasets with no ground-truth transcript set — precision/recall against truth cannot be computed
- Genome-wide (untargeted) transcriptome reconstruction — IsoCon assumes reads span the target end-to-end, as in targeted Iso-Seq of gene families
- Candidate transcripts that already exist and only need statistical filtering (use IsoCon's stat_filter instead of re-running the full benchmark)

## Inputs

- Simulated PacBio Iso-Seq full-length read sets (FASTQ/FASTA) per family, mutation rate, and read depth
- Ground-truth simulated isoform sequence set (30 isoforms per family: TSPY, HSFY, DAZ)
- IsoCon predicted transcripts (final_candidates.fa)
- ICE predicted transcripts on the same read sets
- IsoCon_Eval config file simulated_experiments. (EXPERIMENTS, GENE_MEMBERS, READ_COUNTS, RUNS, ABUNDANCE, TOOLS fields)

## Outputs

- Nucleotide-level precision and recall values per gene family, mutation rate, and read depth for IsoCon and ICE
- Grouped violin plots of precision and recall as a function of read depth comparing IsoCon vs ICE

## How to apply

Generate simulated PacBio Iso-Seq read sets for gene families (TSPY, HSFY, DAZ) with 30 isoforms each, unequal ('constant' vs 'exponential') transcript abundances, three mutation rates, and varying full-length read counts (e.g. 20, 100, 500, 2500, 12500) as configured in IsoCon_Eval's simulated_experiments.. Reconstruct transcripts with 'IsoCon pipeline -fl_reads <reads> -outfolder <output>' (output in final_candidates.fa) and run ICE on identical read sets as the baseline. Score both methods' predicted transcripts against the simulated ground-truth isoform set, computing nucleotide-level precision and recall for each family, mutation rate, and depth. Render grouped violin plots of precision and recall as a function of read depth. The driver is the snakemake workflow in ksahlin/IsoCon_Eval; pin IsoCon to commit 79589f3 used for the published results.

## Related tools

- **IsoCon** (Reconstructs predicted transcripts from simulated full-length reads; evaluated method) — https://github.com/ksahlin/IsoCon
- **ICE** (PacBio Iso-Seq clustering/correction baseline comparator run on identical simulated read sets)
- **IsoCon_Eval** (Snakemake workflow driving all simulations, method runs, and precision/recall evaluations) — https://github.com/ksahlin/IsoCon_Eval

## Examples

```
IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>
```

## Evaluation signals

- ICE precision drops to near 0 at read depths of 500 or higher on the simulated data
- IsoCon precision remains above 80% at read depths >= 500
- IsoCon final_candidates.fa output matches the published reconstruction output for the same inputs
- Precision and recall are reported separately for each gene family (TSPY, HSFY, DAZ), mutation rate, and read count (20, 100, 500, 2500, 12500) and across the 10 configured RUNS

## Limitations

- IsoCon results in the paper were obtained with commit 79589f3; other versions may behave differently
- The benchmark relies on simulated data; very low-abundance real transcripts may be missed since deep PacBio sequencing can be cost-prohibitive
- IsoCon requires reads to span the region of interest end-to-end (targeted Iso-Seq or amplicon-style data)

## Evidence

- [discussion] The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub.: "The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub."
- [discussion] Scripts for all analyses are available at https://github.com/ksahlin/IsoCon_Eval. This repository also include a snakemake63 workflow: "Scripts for all analyses are available at https://github.com/ksahlin/IsoCon_Eval. This repository also include a snakemake63 workflow"
- [readme] predicted transcripts are found in file /path/to/output/final_candidates.fa: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**."
- [readme] You can remove tools or experiments by changing the fields in the config file simulated_experiments..: [evidence span withheld — anchor status: not_found]
- [intro] Using simulated data, we demonstrate that IsoCon has substantially higher precision and recall than ICE18 across a wide range of sequencing depths: "Using simulated data, we demonstrate that IsoCon has substantially higher precision and recall than ICE18 across a wide range of sequencing depths"
