---
name: two-phase-imaging-workflow-interpretation
description: Use when reproducing or benchmarking a microscope acquisition workflow
  that detects objects at low magnification (e.g., 5× tiled images) and re-images
  detected objects at high magnification (e.g., 20×), especially with an automated
  slide loader.
when_to_use_negative:
- The sample requires single-pass imaging without a second visit (the two-phase strategy
  exists because single-pass imaging forces a compromise between resolution and field
  depth, but it doubles slide handling)
- No automated slide loader or motorized stage is available, since Phase 2 requires
  reloading the slide and resuming at a second entry point
- The goal is HCS on cells grown in culture in standard plates rather than slide-based
  samples needing low-resolution detection and targeted re-imaging
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: OpenHiCAMM
  role: 'Fiji/μManager module that orchestrates the two-phase workflow: 5× tiled acquisition,
    embryo detection, coordinate capture, and 20× re-imaging at the Phase 2 entry
    point'
  repo: github.com/bdgp/OpenHiCAMM
- name: μManager
  role: Provides microscope hardware control (Prior PL-200 slide loader, motorized
    stage, camera) across both imaging phases
- name: Fiji
  role: Hosts OpenHiCAMM and supplies image stitching to assemble composite tiled
    5× images
- name: SlideSurveyor
  role: Optional alternative acquisition path that uses the camera video feed for
    rapid slide imaging, object detection, and re-imaging with SlideImager
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:d59296fb4e54ad818d81fbbfd0f929d0b035e845927c9f5fbd08ce0c71483575
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fiji
  - OpenHiCAMM
  - SlideSurveyor
  - μManager
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# two-phase-imaging-workflow-interpretation

## Summary

Interprets and reproduces OpenHiCAMM's two-phase imaging strategy for Drosophila embryo slides: Phase 1 tiles coverslip areas at 5× and detects embryo coordinates, then Phase 2 reloads the slide and re-images detected embryos at 20×. Includes the paper's stage-calibration and speed benchmarks used to judge correct operation.

## When to use

Use when reproducing or benchmarking a microscope acquisition workflow that detects objects at low magnification (e.g., 5× tiled images) and re-images detected objects at high magnification (e.g., 20×), especially with an automated slide loader.

## When NOT to use

- The sample requires single-pass imaging without a second visit (the two-phase strategy exists because single-pass imaging forces a compromise between resolution and field depth, but it doubles slide handling)
- No automated slide loader or motorized stage is available, since Phase 2 requires reloading the slide and resuming at a second entry point
- The goal is HCS on cells grown in culture in standard plates rather than slide-based samples needing low-resolution detection and targeted re-imaging

## Inputs

- Microscope slides with Drosophila embryos under a coverslip (Prior PL-200 slide loader, Zeiss Axioplan 2, Nikon DSLR D5100, Prior motorized stage)
- 5× tiled images of coverslip areas (200–300 images per slide)
- Detected embryo coordinates saved from the Phase 1 image analysis module
- Manually created imaging objects for alignment calibration across slide reloads

## Outputs

- High-resolution 20× images of detected embryos suitable for computational analysis and screening
- Embryo coordinate list from the Phase 1 detection module
- Alignment-calibration measurements (stage displacement after slide reload, in μm and as % of a 20× image)
- Acquisition-speed measurements (s/image for 5× tiling and 20× detection, e.g., as calibration_speed_measurements.csv with condition/quantity/value/units columns)

## How to apply

Set up OpenHiCAMM as a Fiji/μManager module (set fiji.dir, add MMJ_.jar and MMCore.jar to the local Maven repo, run mvn install). Run Phase 1 by imaging manually selected coverslip areas as tiled 5× images (200–300 images per slide), stitching tiles with Fiji image stitching, and running an image analysis module to capture embryo coordinates. Manually switch to the 20× objective, reload the slide, and resume the workflow at the second entry point (Phase 2) to re-image detected embryos at high resolution; expect SlideSurveyor to use the camera video feed for rapid live-view detection where applicable. Validate alignment by re-loading the same slide and measuring displacement of manually created imaging objects (benchmark: 0.059 μm average, about 1.5% of the image at 20×, per Figure S4 alignment calibration), and validate speed against 2.3 s/image for 5× tiling and 12 s/image for detection of twenty-five 20× images including a focus step.

## Related tools

- **OpenHiCAMM** (Fiji/μManager module that orchestrates the two-phase workflow: 5× tiled acquisition, embryo detection, coordinate capture, and 20× re-imaging at the Phase 2 entry point) — github.com/bdgp/OpenHiCAMM
- **μManager** (Provides microscope hardware control (Prior PL-200 slide loader, motorized stage, camera) across both imaging phases)
- **Fiji** (Hosts OpenHiCAMM and supplies image stitching to assemble composite tiled 5× images)
- **SlideSurveyor** (Optional alternative acquisition path that uses the camera video feed for rapid slide imaging, object detection, and re-imaging with SlideImager)

## Examples

```
echo "fiji.dir=/Applications/Fiji.app" >./build.properties; ./scripts/add-mmj-jar.sh ~/src/micromanager; mvn install
```

## Evaluation signals

- Average displacement of manually created imaging objects after slide reload is ≈0.059 μm, about 1.5% of the 20× image field
- 5× tiling throughput is near 2.3 s/image with 200–300 tiled images per slide
- Detection of twenty-five 20× images (including a focus step) completes in about 12 s/image
- Phase 2 resumes from the Phase 1 coordinate list after slide reload, with detected embryos re-imaged at 20× with correct centering (compare against Figure S4 alignment calibration)
- The two-phase strategy completes roughly 10× faster than an equivalent single-pass high-resolution acquisition

## Limitations

- The workflow is not fully hands-off: the user must manually adjust the microscope and switch to the 20× objective between Phase 1 and Phase 2
- Slide reload introduces stage displacement, so alignment calibration (Figure S4) is required before trusting Phase 2 targeting
- The discussion offers only qualitative capability claims (robust, flexible, extendable); quantitative benchmarks appear only in the Transparent Methods/Supplemental material
- No version number or changelog is published for OpenHiCAMM, and only a GitHub source is given with no archival DOI
- Hardware-specific: benchmarks were obtained on a Prior PL-200 slide loader, Zeiss Axioplan 2 microscope, Nikon DSLR D5100 camera, and Prior motorized stage

## Evidence

- [abstract] we imaged Drosophila embryos, detecting the embryos at low resolution, followed by re-imaging the detected embryos at high resolution: "we imaged Drosophila embryos, detecting the embryos at low resolution, followed by re-imaging the detected embryos at high resolution"
- [results] After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2).: "After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2)."
- [results] The average displacement was 0.059 μm, about 1.5% of the image at 20× magnification: "The average displacement was 0.059 μm, about 1.5% of the image at 20× magnification"
- [results] multiple tiled images are acquired and post-processed with Fiji image stitching to assemble a composite image: "multiple tiled images are acquired and post-processed with Fiji image stitching to assemble a composite image"
- [supplementary] Figure S4. Alignment calibration for the same slide, loaded twice and imaged at different magnifications, Related to Figure 1.: "Figure S4. Alignment calibration for the same slide, loaded twice and imaged at different magnifications, Related to Figure 1."
