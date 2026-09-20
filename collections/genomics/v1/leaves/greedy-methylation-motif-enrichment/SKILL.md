---
name: greedy-methylation-motif-enrichment
description: Use when you have raw ONT fast5 reads from a native (methylated) sample
  plus a whole-genome-amplified control and a reference genome FASTA, and need a deterministic,
  sensitive de novo methylation motif catalog that MEME/Tombo/Nanodisco may miss.
when_to_use_negative:
- When reads are not ONT native signal data (e.g. PacBio or Illumina) — Snapper is
  built on ONT raw signal statistics from Tombo resquiggled fast5 files.
- When pre-existing motif annotations are sufficient — this skill performs de novo
  enrichment, not validation of a known methyltransferase specificity.
- When a python 3.7 environment with Tombo resquiggling cannot be built; the README
  warns 'python 3.7 (later versions might be incompatible because of inner biopython
  dependencies)'.
tools:
- name: Tombo
  role: Resquiggles raw fast5 signal to the reference so per-k-mer signal statistics
    can be computed
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files to the multi-read fast5 format
    required by snapper
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Snapper / snapper-ont
  role: Runs the greedy position-specific motif enrichment against sample and control
    multi-fast5 dirs and emits motif FASTAs
  repo: https://github.com/DNKonanov/Snapper
- name: Python
  role: Runtime environment (python 3.7 conda env) hosting snapper-ont and its dependencies
    (h5py, biopython, scipy, seaborn)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:d2e98bbaa4539f4f3f11e1e1f402222b4dc35f33d665d146ed98a706ba96e912
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - Python
  - Snapper
  - Snapper / snapper-ont
  - Tombo
  - ont_fast5_api
  - snapper-ont
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# greedy-methylation-motif-enrichment

## Summary

Detect methylated DNA motifs de novo from Oxford Nanopore sequencing data using Snapper's high-sensitive greedy algorithm for position-specific motif enrichment, comparing native sample reads against WGA control reads.

## When to use

Use when you have raw ONT fast5 reads from a native (methylated) sample plus a whole-genome-amplified control and a reference genome FASTA, and need a deterministic, sensitive de novo methylation motif catalog that MEME/Tombo/Nanodisco may miss.

## When NOT to use

- When reads are not ONT native signal data (e.g. PacBio or Illumina) — Snapper is built on ONT raw signal statistics from Tombo resquiggled fast5 files.
- When pre-existing motif annotations are sufficient — this skill performs de novo enrichment, not validation of a known methyltransferase specificity.
- When a python 3.7 environment with Tombo resquiggling cannot be built; the README warns 'python 3.7 (later versions might be incompatible because of inner biopython dependencies)'.

## Inputs

- Raw ONT fast5 files from the native (methylated) sample
- Raw ONT fast5 files from a WGA (whole-genome-amplified) control sample
- Reference genome in FASTA format
- Resquiggled multi-read fast5 directories for sample and control

## Outputs

- passed_motifs_[strand]_[contig_name].fasta files listing k-mers most likely carrying a modified base, with per-k-mer confidence values
- Final enriched methylation motif set (final_motifs FASTA) for the sample genome

## How to apply

Resquiggle the raw fast5 files against the reference genome with Tombo so raw signal is aligned to basecalls, then convert the resquiggled files to the multi-read fast5 format using ont_fast5_api. In a python 3.7 conda environment (later versions may break biopython dependencies), install ont-fast5-api and ont-tombo from bioconda and snapper-ont via pip. Run snapper with -sample_fast5dir (native multi-fast5), -control_fast5dir (WGA multi-fast5), and -reference (genome FASTA), optionally raising the coverage threshold (e.g. 100x instead of the default 40) while keeping defaults for ks_t=3, k_size=15, long_k_size=29, min_conf=100. Snapper compares native versus control signal statistics per k-mer and greedily enriches position-specific modified motifs without manual control, emitting passed_motifs_[strand]_[contig].fasta files containing all k-mers most likely carrying a modified base with per-k-mer confidence values.

## Related tools

- **Tombo** (Resquiggles raw fast5 signal to the reference so per-k-mer signal statistics can be computed) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files to the multi-read fast5 format required by snapper) — https://github.com/nanoporetech/ont_fast5_api
- **Snapper / snapper-ont** (Runs the greedy position-specific motif enrichment against sample and control multi-fast5 dirs and emits motif FASTAs) — https://github.com/DNKonanov/Snapper
- **Python** (Runtime environment (python 3.7 conda env) hosting snapper-ont and its dependencies (h5py, biopython, scipy, seaborn))

## Examples

```
snapper -sample_fast5dir J99_multi/ -control_fast5dir J99_wga_multi/ -reference J99.fasta -coverage 100 -outdir J99_snapper_out
```

## Evaluation signals

- passed_motifs_[strand]_[contig].fasta and final_motifs FASTA files are present for every contig of the sample reference
- The final motif set reproduces independently characterized methylation motifs (e.g. the H.pylori J99 PacBio motif set, apart from motifs judged non-motifs such as TCNNGA in J99)
- Resquiggling completed without errors and multi-fast5 conversion produced non-empty J99_multi and J99_wga_multi directories
- The chosen coverage threshold was actually met (e.g. -coverage 100 on the sample genome)
- Results are deterministic across runs — Snapper is designed to 'return deterministic results' unlike MEME-based approaches

## Limitations

- Python 3.7 is required; later versions might be incompatible because of inner biopython dependencies
- Tombo is deprecated and unsupported by Oxford Nanopore, who recommend Remora instead
- Motifs recovered can differ from prior characterizations: in H.pylori J99 the TCNNGA motif was judged not to be an individual methylation motif
- Snapper compares native signal against a WGA control, so a properly amplified control dataset is required

## Evidence

- [abstract] abstract quote on greedy algorithm: "We present Snapper, a new highly sensitive approach, to extract methylation motif sequences based on a greedy motif selection algorithm."
- [methods] methods resquiggle step: "Firstly, fast5 files should be resquiggled using Tombo software."
- [methods] methods multi-fast5 conversion: "After resquiggling, fast5 files should be converted to the multi-fast5 format using ont_fast5_api."
- [methods] methods snapper invocation: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [readme] README python version caveat: "python 3.7 (later versions might be incompatible because of inner biopython dependencies)"
- [abstract] abstract MEME sensitivity limitation: "the MEME suit does not extract all Helicobacter pylori methylation sites de novo even using the iterative approach implemented in the most up-to-date methylation analysis tool Nanodisco"
