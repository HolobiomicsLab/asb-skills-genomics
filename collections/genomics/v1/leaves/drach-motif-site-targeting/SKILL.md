---
name: drach-motif-site-targeting
description: Use when resquiggle-corrected single-read FAST5 files, per-read alignment
  TSVs, and basecalling-error TSVs are available and per-site read-level features
  must be extracted for each DRACH/RRACH adenosine before RedNano prediction or training.
when_to_use_negative:
- The features have already been extracted and a combined feature table with chrom/pos/k_mer/k_signals_rect/qual/mis/ins/dele
  columns already exists — proceed directly to aggregation and prediction.
- Sequencing reads have not been basecalled and resquiggled (no RawGenomeCorrected_001
  group in the FAST5 files) — preprocessing (multi_to_single_fast5, guppy_basecaller,
  tombo resquiggle, minimap2 mapping) must run first.
- The goal is site-level modified probabilities from a feature table — use the predict.py
  step instead, since this skill only produces per-read per-site features.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3524
tools:
- name: RedNano
  role: Provides scripts/extract_features.py that one-hot encodes the 5-mer sequence
    and expands embeddings L=65 to build the raw-signal and basecalling-error features
    per targeted DRACH/RRACH adenosine
  repo: https://github.com/Derryxu/RedNano
- name: ont-tombo
  role: Resquiggles raw signals against the reference transcriptome fasta, producing
    the RawGenomeCorrected_001 corrected group consumed by feature extraction
- name: h5py
  role: Python library used to read the resquiggle-corrected single-read FAST5 files
    during feature extraction
- name: jvarkit
  role: sam2tsv.jar converts the minimap2/samtools BAM alignment into the test.tsv
    alignment file split per read to derive basecalling errors
  repo: https://github.com/lindenb/jvarkit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:6f8773be5304459e68ad0b519f507edaf579f9ca3293852a38450bbb986f7ae8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - Python
  - RedNano
  - h5py
  - jvarkit
  - ont-tombo
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# drach-motif-site-targeting

## Summary

Encodes each targeted DRACH/RRACH adenosine in a Nanopore DRS read as a paired raw-signal (one-hot 5-mer sequence context expanded to 65 normalized signal positions, 5x325) and basecalling-error (8x5) feature matrix for downstream m6A deep-learning prediction with RedNano.

## When to use

Use when resquiggle-corrected single-read FAST5 files, per-read alignment TSVs, and basecalling-error TSVs are available and per-site read-level features must be extracted for each DRACH/RRACH adenosine before RedNano prediction or training.

## When NOT to use

- The features have already been extracted and a combined feature table with chrom/pos/k_mer/k_signals_rect/qual/mis/ins/dele columns already exists — proceed directly to aggregation and prediction.
- Sequencing reads have not been basecalled and resquiggled (no RawGenomeCorrected_001 group in the FAST5 files) — preprocessing (multi_to_single_fast5, guppy_basecaller, tombo resquiggle, minimap2 mapping) must run first.
- The goal is site-level modified probabilities from a feature table — use the predict.py step instead, since this skill only produces per-read per-site features.

## Inputs

- resquiggle-corrected single-read FAST5 files (tombo corrected group RawGenomeCorrected_001) from the guppy workspace
- per-read basecalling-error TSV files split into a tmp/ folder (from sam2tsv on test.bam)
- reference transcriptome length BED file with a header (ID and length columns, e.g. test/referance.transcript.bed)
- k-mer length k=5 and raw signal length s=65 parameters

## Outputs

- combined per-read feature table (one sample per row) with columns: chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele
- site-level aggregated feature table (test.features.sort.aggre.tsv)
- coverage-filtered table (test.features.sort.aggre.c20.tsv, coverage >= 20)

## How to apply

Load resquiggle-corrected single-read FAST5 files from the guppy basecalling workspace (corrected group RawGenomeCorrected_001) with RedNano's scripts/extract_features.py (Python 3.7, h5py 3.4.0). Supply the per-read basecalling-error TSVs split into a tmp/ folder via --errors_dir and the reference transcriptome length BED with header via -b (e.g. test/referance.transcript.bed). Run with k-mer length k=5, raw signal length s=65, --w_is_dir 1, and 30 workers (-n 30); RedNano one-hot encodes each 5-mer's nucleotides (embedding size 4) and expands each base's embedding L=65 times to match the 65 normalized signals, forming the 5x325 raw-signal feature, paired with an 8x5 basecalling-error matrix (match/mismatch/deletion/insertion status plus base quality per base) from the sam2tsv alignment. Aggregate read-level features to site level afterwards with scripts/aggre_features_to_site_level.py and apply a coverage filter of $4>=20 before prediction with the multi-instance-learning model.

## Related tools

- **RedNano** (Provides scripts/extract_features.py that one-hot encodes the 5-mer sequence and expands embeddings L=65 to build the raw-signal and basecalling-error features per targeted DRACH/RRACH adenosine) — https://github.com/Derryxu/RedNano
- **ont-tombo** (Resquiggles raw signals against the reference transcriptome fasta, producing the RawGenomeCorrected_001 corrected group consumed by feature extraction)
- **h5py** (Python library used to read the resquiggle-corrected single-read FAST5 files during feature extraction)
- **jvarkit** (sam2tsv.jar converts the minimap2/samtools BAM alignment into the test.tsv alignment file split per read to derive basecalling errors) — https://github.com/lindenb/jvarkit

## Examples

```
python scripts/extract_features.py -i /path/to/fast5_guppy/workspace/ -o test/features/ --errors_dir test/tmp/ --corrected_group RawGenomeCorrected_001 -b test/referance.transcript.bed --w_is_dir 1 -k 5 -s 65 -n 30
```

## Evaluation signals

- Every output feature row carries all required columns: chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele.
- Each raw-signal feature spans exactly 65 normalized signals per site (5x325 expanded one-hot matrix) consistent with -s 65.
- The basecalling-error matrix is 8x5, encoding match/mismatch/deletion/insertion status plus base quality for each base of the 5-mer.
- Validation on synthesized demo data: 5-mer signal matrices align to targeted DRACH/RRACH adenosines at expected transcript positions.
- After aggregation, the awk coverage filter ($4>=20) retains only sites with at least 20 read observations before prediction.

## Limitations

- Feature extraction depends on tombo resquiggle against the reference transcript fasta (not a genome file), so a transcript-level reference must be prepared; the resquiggle must use --corrected-group RawGenomeCorrected_001.
- The test.tsv alignment file must be split per read into a tmp/ folder to allow parallelised feature extraction; supplying unsplit TSVs will break the --errors_dir workflow.
- Multi-instance-learning site-level prediction requires aggregation plus a minimum coverage of 20 reads per site; low-coverage sites are filtered out.
- Pre-trained models target RNA DRACH motifs (mil_allspecies_model_states.pt trained on synthetic, human, and Arabidopsis data); non-DRACH contexts are outside the demonstrated scope.

## Evidence

- [other] RedNano one-hot encodes the nucleotide sequence of each 5-mer and expands the embedding vector of each base L times (L = 65) to match the sequence of the normalized signals, forming the raw-signal feature.: [evidence span withheld — anchor status: not_found]
- [readme] python scripts/extract_features.py -i /path/to/fast5_guppy/workspace/ -o test/features/ --errors_dir test/tmp/ --corrected_group RawGenomeCorrected_001 -b test/referance.transcript.bed --w_is_dir 1 -k 5 -s 65 -n 30: "python scripts/extract_features.py -i /path/to/fast5_guppy/workspace/ -o test/features/ --errors_dir test/tmp/ --corrected_group RawGenomeCorrected_001 -b test/referance.transcript.bed --w_is_dir 1"
- [methods] awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv: "awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv"
- [intro] RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads: "RedNano is a deep-learning method that utilizes both raw signals and basecalling errors to detect m6A from Nanopore DRS reads"
- [readme] To allow downstream parallelisation of the feature extraction, the generated test.tsv is split into the different reads in a tmp folder.: "To allow downstream parallelisation of the feature extraction, the generated test.tsv is split into the different reads in a tmp folder."
