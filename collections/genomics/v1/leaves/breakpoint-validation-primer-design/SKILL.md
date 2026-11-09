---
name: breakpoint-validation-primer-design
description: Use when a candidate fusion gene with a nucleotide-resolution breakpoint
  (e.g. from NanoFG/NanoSV output) needs spanning PCR primers for Sanger or PCR validation.
when_to_use_negative:
- No candidate fusion has been called yet — primer design requires a nucleotide-resolution
  breakpoint sequence, not raw FASTQ/BAM input
- Input DNA is fragmented (e.g. FFPE material), where long-amplicon breakpoint validation
  is expected to fail
- Fusion supported by only one fusion-spanning read, which the article says requires
  manual breakpoint PCR validation rather than automated primer-based reporting alone
edam_operation: http://edamontology.org/operation_0308
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3175
tools:
- name: Primer3
  role: Designs breakpoint-spanning PCR validation primers from the fusion junction
    sequence in the final NanoFG pipeline step
  repo: https://github.com/primer3-org/primer3
- name: NanoFG
  role: Upstream pipeline that produces the filtered fusion calls, breakpoint sequences
    and fusion-spanning reads that serve as primer-design input
  repo: https://github.com/SdeBlank/NanoFG
- name: NanoSV
  role: Calls the structural variants (v1.2.4, min_mapq=12, depth_support=False, mapq_flag=48)
    whose filtered breakpoints define the primer targets
  repo: https://github.com/mroosmalen/nanosv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:17334ea5773a8e213937cbb535786c4ce05fa75fe2fa1bf4e8d54901da94455f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/primer3-org/primer3
  tools:
  - FUDGE
  - LAST
  - Minimap2
  - NanoFG
  - NanoSV
  - Primer3
  - Samtools
  - Wtdbg2
  edam_operation: http://edamontology.org/operation_0308
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# breakpoint-validation-primer-design

## Summary

Final step of the NanoFG pipeline: design PCR primers spanning the exact fusion gene breakpoint with Primer3, so each reported fusion from CRISPR-Cas9-enriched nanopore reads can be independently validated by breakpoint PCR.

## When to use

Use when a candidate fusion gene with a nucleotide-resolution breakpoint (e.g. from NanoFG/NanoSV output) needs spanning PCR primers for Sanger or PCR validation.

## When NOT to use

- No candidate fusion has been called yet — primer design requires a nucleotide-resolution breakpoint sequence, not raw FASTQ/BAM input
- Input DNA is fragmented (e.g. FFPE material), where long-amplicon breakpoint validation is expected to fail
- Fusion supported by only one fusion-spanning read, which the article says requires manual breakpoint PCR validation rather than automated primer-based reporting alone

## Inputs

- Filtered NanoFG fusion call list: fusion partners, exact breakpoint locations, breakpoint sequences (from minimap2/GRCh37-aligned nanopore reads passing the ≥2 fusion-spanning-read cutoff)
- Fusion-spanning read sequences flanking the breakpoint junction

## Outputs

- Breakpoint-spanning validation primer pairs per reported fusion (Primer3 output)
- Final NanoFG fusion report listing fusion gene configuration, breakpoint-location, breakpoint-sequence and fusion-spanning primer sequences

## How to apply

After NanoFG filters the NanoSV VCF (removing SVs failing built-in filters and all insertions), annotates both SV ends with ENSEMBL genes, and retains same-strand inter-gene fusions supported by at least two fusion-spanning reads, extract the breakpoint sequence from the fusion-spanning reads for each surviving candidate. Feed each breakpoint-flanking sequence into Primer3 (installed from github.com/primer3-org/primer3 and configured via NanoFG's paths.ini) to design primers that span the junction. Record the primer sequences together with the fusion partners, exact breakpoint locations and breakpoint sequences in the final fusion report. Confirm success by checking that each reported fusion comes with breakpoint-spanning validation primers and that, experimentally, a junction-spanning PCR amplicon can be produced; fusions detected with only one supporting read must be validated manually by breakpoint PCR.

## Related tools

- **Primer3** (Designs breakpoint-spanning PCR validation primers from the fusion junction sequence in the final NanoFG pipeline step) — https://github.com/primer3-org/primer3
- **NanoFG** (Upstream pipeline that produces the filtered fusion calls, breakpoint sequences and fusion-spanning reads that serve as primer-design input) — https://github.com/SdeBlank/NanoFG
- **NanoSV** (Calls the structural variants (v1.2.4, min_mapq=12, depth_support=False, mapq_flag=48) whose filtered breakpoints define the primer targets) — https://github.com/mroosmalen/nanosv

## Examples

```
git clone https://github.com/primer3-org/primer3.git primer3; cd primer3/src && make && ./primer3_core ../example
```

## Evaluation signals

- Each fusion reported in the final NanoFG output has an associated primer pair spanning the breakpoint junction
- The fusion report includes, per event: fusion gene configuration, exact breakpoint-location, breakpoint-sequence and fusion-spanning primer sequences
- Every automatically reported fusion is supported by at least two fusion-spanning reads (the article's reliability cutoff)
- Running NanoFG on the repository's bundled test FASTQ set reproduces a final fusion report with matching partner genes, breakpoints and primers
- Primer3 binary path is correctly set in paths.ini so the pipeline completes without tool-not-found errors

## Limitations

- Fusions detected with only one fusion-spanning read fall below the ≥2-read cutoff and require manual breakpoint PCR validation
- Intratumoral heterogeneity and tumor purity likely influence the assay's lower detection limits, affecting which breakpoints can be validated
- Primer-based validation presumes non-fragmented DNA; FFPE-derived material will most likely fail

## Evidence

- [results] Primer3 designs breakpoint-spanning validation primers as the final NanoFG step: [evidence span withheld — anchor status: not_found]
- [discussion] Two-read cutoff before automated reporting: "We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation."
- [discussion] Fusions with single-read support need manual PCR validation: "two fusions were only detected with one fusion-spanning read each, requiring the manual validation of the fusion gene by breakpoint PCR."
- [readme] Primer3 installation instructions from README: [evidence span withheld — anchor status: external_doc]
- [discussion] NanoFG report contents: [evidence span withheld — anchor status: not_found]
