---
name: microscopy-workflow-results-extraction
description: Use when a paper reports an autonomous imaging demonstration with per-phase
  metrics (slide counts, pass durations, per-slide rates, object-detection ranges,
  re-imaging yields) scattered across supplemental methods text that must be compiled
  into a verifiable table.
when_to_use_negative:
- 'The article reports no quantitative demonstration metrics (e.g., the OpenHiCAMM
  discussion section is qualitative only: ''robust, flexible, easily adapted'') —
  there is nothing to extract.'
- The metrics you need are raw instrument logs or image files from your own run rather
  than reported figures in published text.
- The reported workflow is a single-pass acquisition with no phase decomposition,
  so per-phase duration/object-yield tabulation does not apply.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: OpenHiCAMM
  role: 'Subject of the extracted metrics: its two-pass (5x detection, 20x re-imaging)
    autonomous run supplies the slide counts, durations, and object yields being tabulated'
  repo: github.com/bdgp/OpenHiCAMM
- name: Fiji
  role: Package in which OpenHiCAMM runs; the extracted Phase 1/Phase 2 figures characterize
    this Fiji-based workflow's throughput
  repo: http://fiji.sc
- name: μManager
  role: Hardware-control layer whose slide-loader-driven acquisition the reported
    8 min/slide and 39 min/slide figures quantify
  repo: http://micromanager.org
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:69e6a63dfba2a938443cc59aa81b27301901b86f131dc55dabe2f235fb11879f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fiji
  - OpenHiCAMM
  - μManager
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# microscopy-workflow-results-extraction

## Summary

Extracts quantified performance metrics from a demonstrated autonomous microscopy/HCS run (e.g., OpenHiCAMM's two-pass 95-slide Drosophila embryo screen) out of the paper's Transparent Methods supplemental text and assembles them into a structured tabular (CSV) record for reuse or verification.

## When to use

Use when a paper reports an autonomous imaging demonstration with per-phase metrics (slide counts, pass durations, per-slide rates, object-detection ranges, re-imaging yields) scattered across supplemental methods text that must be compiled into a verifiable table.

## When NOT to use

- The article reports no quantitative demonstration metrics (e.g., the OpenHiCAMM discussion section is qualitative only: 'robust, flexible, easily adapted') — there is nothing to extract.
- The metrics you need are raw instrument logs or image files from your own run rather than reported figures in published text.
- The reported workflow is a single-pass acquisition with no phase decomposition, so per-phase duration/object-yield tabulation does not apply.

## Inputs

- Transparent Methods supplemental file (SI text) of the OpenHiCAMM article
- Article main text (abstract/results) describing the two-pass embryo-imaging demonstration
- Source-card finding strings quoting per-phase metric values

## Outputs

- CSV table with columns: workflow phase, slide count, metric description, reported value
- Verified set of key figures: 95 slides, ~12 hr Phase 1 at 8 min/slide, 26-751 objects/slide, 39 min Phase 2 time for 61 objects yielding 119 high-resolution images

## How to apply

Load and read the Transparent Methods supplemental file accompanying the article, since the demonstration run details (here, OpenHiCAMM's two-pass workflow) are documented there rather than in the main text. Locate Phase 1 (5x low-resolution detection pass) metrics: slide count (95), total pass duration (~12 hr), per-slide rate (8 min/slide), and the per-slide object-detection range (26–751 objects, where an object is a continuous area containing one or multiple embryos). Locate Phase 2 (20x high-resolution re-imaging after manual objective change) metrics: per-slide imaging time (39 min for a slide with 61 detected objects yielding 119 high-resolution images) and the reported maximum per-slide imaging time. Assemble the values into a table with columns for workflow phase, slide count, metric description, and reported value, and write it to CSV with pandas. Validate each extracted value by direct comparison against the source passage, flagging any figure that cannot be matched verbatim.

## Related tools

- **OpenHiCAMM** (Subject of the extracted metrics: its two-pass (5x detection, 20x re-imaging) autonomous run supplies the slide counts, durations, and object yields being tabulated) — github.com/bdgp/OpenHiCAMM
- **Fiji** (Package in which OpenHiCAMM runs; the extracted Phase 1/Phase 2 figures characterize this Fiji-based workflow's throughput) — http://fiji.sc
- **μManager** (Hardware-control layer whose slide-loader-driven acquisition the reported 8 min/slide and 39 min/slide figures quantify) — http://micromanager.org

## Examples

```
import pandas as pd; pd.DataFrame([{'phase':1,'slides':95,'metric':'duration_hr','value':12},{'phase':1,'metric':'objects_per_slide','value':'26-751'},{'phase':2,'metric':'minutes_per_slide_61obj_119img','value':39}]).to_csv('hcs_phase_metrics.csv', index=False)
```

## Evaluation signals

- Slide count for the autonomous run equals exactly 95, matching the reported figure.
- Phase 1 (low-resolution) pass total duration is ~12 hr at a rate of 8 min/slide (12 hr × 60 / 8 ≈ 90-95 slides, consistent).
- Per-slide Phase 1 object-detection counts fall within the reported 26-751 range; no extracted value lies outside it.
- Phase 2 example reconciles internally: 39 min to re-image one slide with 61 detected objects produced 119 high-resolution images.
- Each tabulated value passes direct string/numeric comparison against the corresponding sentence in the Transparent Methods supplemental text.

## Limitations

- Metrics describe a single demonstration platform (Prior PL-200 slide loader, Zeiss Axioplan 2, Nikon DSLR D5100, Prior motorized stage) and Drosophila embryos; they are not general benchmarks for other samples or hardware.
- The run included a manual intervention point: the microscope was manually adjusted to the 20x objective between Phase 1 and Phase 2, so the 'autonomous' run is two-entry-point, not fully hands-off.
- Per-slide times vary with object density (e.g., 39 min for a 61-object slide; a maximum per-slide time is also reported), so single values should not be treated as fixed throughputs.

## Evidence

- [supplementary] the low-resolution (Phase 1) pass was completed in about 12 hr at a rate of 8 min/slide and yielded 26–751 objects: [evidence span withheld — anchor status: not_found]
- [results] After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2).: "After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2)."
- [abstract] we imaged Drosophila embryos, detecting the embryos at low resolution, followed by re-imaging the detected embryos at high resolution: "we imaged Drosophila embryos, detecting the embryos at low resolution, followed by re-imaging the detected embryos at high resolution"
- [readme] More detailed usage instructions can be found in the `docs/` folder.: "More detailed usage instructions can be found in the `docs/` folder."
