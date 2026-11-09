---
name: multi-caller-performance-tabulation
description: Use when several SV callers must be compared on the same alignment inputs
  against a GIAB benchmark truth set, and per-caller precision/recall/F1 tables restricted
  to deletion and insertion classes are needed.
when_to_use_negative:
- The goal is comparing short-read paired-end callers — the paper evaluates those
  against manta, delly, and lumpy rather than the long-read caller set (SVIM, NanoVar,
  Sniffles)
- Input VCFs are already merged or evaluated outputs rather than raw caller VCFs on
  the same alignment files — re-running callers is required for a fair comparison
- Only one caller exists and no truth set is available — svbench requires a reference
  VCF to score against
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: dysgu
  role: Primary SV caller run in --mode pacbio and --mode nanopore on HiFi/ONT BAMs;
    calls filtered by maximum model probability thresholds
  repo: https://github.com/kcleal/dysgu
- name: svbench
  role: Benchmarking tool scoring each caller's VCF against the GIAB truth set to
    produce TP/FP/FN, precision, recall, and F1
  repo: https://github.com/kcleal/svbench
- name: SVIM
  role: Comparison long-read SV caller run on the same HG002 PacBio HiFi and ONT alignments
- name: nanovar
  role: Comparison long-read SV caller (neural network classifier) run on the same
    alignment files
- name: Sniffles
  role: Comparison long-read SV caller run on the same alignment files
  repo: https://github.com/fritzsedlazeck/Sniffles
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:3ef2b4bc314cb8ec7a2b894dd21342303d2c12547b78f856edf8035393e7fa35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - SVIM
  - Sniffles
  - dysgu
  - nanovar
  - svbench
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multi-caller-performance-tabulation

## Summary

Benchmarks multiple structural variant callers (dysgu in PacBio/ONT modes, SVIM, NanoVar, Sniffles) on identical long-read HG002 alignments against a GIAB truth set using svbench, and tabulates precision, recall, and F1 per caller per read type.

## When to use

Use when several SV callers must be compared on the same alignment inputs against a GIAB benchmark truth set, and per-caller precision/recall/F1 tables restricted to deletion and insertion classes are needed.

## When NOT to use

- The goal is comparing short-read paired-end callers — the paper evaluates those against manta, delly, and lumpy rather than the long-read caller set (SVIM, NanoVar, Sniffles)
- Input VCFs are already merged or evaluated outputs rather than raw caller VCFs on the same alignment files — re-running callers is required for a fair comparison
- Only one caller exists and no truth set is available — svbench requires a reference VCF to score against

## Inputs

- HG002 PacBio HiFi BAM alignments (SRA SRR10188368, SRR10188369)
- HG002 ONT BAM alignments (GIAB AshkenazimTrio FTP directory)
- GIAB v0.6 HG002 SV benchmark truth VCF
- Tier 1 high-confidence regions BED file
- Sorted and indexed reference genome FASTA

## Outputs

- Per-caller SV VCFs (dysgu PacBio, dysgu ONT, SVIM, NanoVar, Sniffles)
- Per-caller, per-read-type precision, recall, and F1 metrics for deletion and insertion classes (reproduces Table 5)
- lr_sv_comparison_table.csv

## How to apply

Download HG002 PacBio HiFi alignments (SRA accessions SRR10188368 and SRR10188369) and HG002 ONT alignments from the GIAB AshkenazimTrio FTP directory, then run dysgu on the PacBio HiFi BAM with --mode pacbio and on the ONT BAM with --mode nanopore. Apply the paper's probability filters to dysgu calls: maximum model probability >=0.5 for PacBio and >=0.35 for ONT combinations. Run SVIM, NanoVar, and Sniffles on the same alignment files so all five call sets are directly comparable. Evaluate each VCF against the GIAB v0.6 HG002 SV benchmark using svbench (pip install svbench; usage 'svbench [OPTIONS] REFERENCE_VCF QUERY_VCFS...'), using --include to restrict to Tier 1 regions and restricting to deletion and insertion event classes. Tabulate F1, precision, and recall per caller per read type; the expected result pattern is dysgu leading (deletions F1 0.945, insertions 0.934) with SVIM close behind (0.935, 0.928).

## Related tools

- **dysgu** (Primary SV caller run in --mode pacbio and --mode nanopore on HiFi/ONT BAMs; calls filtered by maximum model probability thresholds) — https://github.com/kcleal/dysgu
- **svbench** (Benchmarking tool scoring each caller's VCF against the GIAB truth set to produce TP/FP/FN, precision, recall, and F1) — https://github.com/kcleal/svbench
- **SVIM** (Comparison long-read SV caller run on the same HG002 PacBio HiFi and ONT alignments)
- **nanovar** (Comparison long-read SV caller (neural network classifier) run on the same alignment files)
- **Sniffles** (Comparison long-read SV caller run on the same alignment files) — https://github.com/fritzsedlazeck/Sniffles

## Examples

```
svbench --include tier1.bed --pass-only --min-size-ref 50 GIAB_v0.6_HG002_SV.vcf dysgu_pacbio.vcf svim.vcf nanovar.vcf sniffles.vcf > lr_sv_comparison.txt
```

## Evaluation signals

- All five VCFs were generated from the identical alignment input files, differing only by caller and mode
- dysgu PacBio calls are filtered at maximum model probability >=0.5 and ONT calls at >=0.35 before evaluation
- svbench comparison is restricted to deletion and insertion event classes and Tier 1 regions
- Reproduced F1 values fall near the published ranges: dysgu deletions 0.945 / insertions 0.934; svim 0.935 / 0.928 (Table 5)
- Output table contains F1, precision, and recall for each caller per read type (PacBio vs ONT)

## Limitations

- No software versions are stated for any caller in the paper, so exact reproducibility of the numbers may drift with tool updates
- Newer dysgu releases changed --mode presets (e.g. pacbio-sequel2 | pacbio-revio, nanopore-r9 | nanopore-r10), so the paper's --mode pacbio/--mode nanopore flags may not map directly to the installed version
- For lower-accuracy long reads, the default divergence 0.02 can be too stringent and reduce sensitivity; the README recommends '--divergence auto' when read accuracy is uncertain

## Evidence

- [results] filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations (Table 7): "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations (Table 7)"
- [supplementary] svbench release: "Analysis scripts used to reproduce results found in this paper can be found under https://github.com/kcleal/svbench."
- [supplementary] PacBio data accessions: "Two lanes of PacBio data were downloaded from SRA (https://www.ncbi.nlm.nih.gov/sra) under accessions SRR10188368 and SRR10188369."
- [results] Table 5 finding: [evidence span withheld — anchor status: not_found]
- [readme] svbench README: [evidence span withheld — anchor status: external_doc]
