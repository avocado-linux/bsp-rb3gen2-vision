# Changelog

All notable changes to avocado-bsp-rb3gen2-vision are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0]

### Fixed
- Republished so the payload actually carries
  `overlays/qcs6490-rb3gen2-vision-mezzanine.dtso`. Earlier releases declared
  the overlay but shipped only `avocado.yaml`: device-tree overlay sources were
  not among the files `avocado ext package` collected. Fixed in avocado-cli
  (#254), which also resolves a package-sourced extension's `src` from its own
  include dir rather than the project tree.
- Version bump rather than a same-version republish on purpose: the payload
  changes without `avocado.yaml` changing, and the stamp for a package-sourced
  extension folds its resolved version, so republishing 0.3.0 would have
  shipped different bytes under a version nothing would treat as new.

## [0.3.0]

### Added
- `sdk.compile.rb3gen2-vision-dt` pulling `kernel-devsrc`, which puts the
  kernel's `dt-bindings` in the target-dev sysroot. The overlay `#include`s
  them, and they resolve against the project's `kernel.version` -- so the
  headers match the kernel the user chose rather than whatever a tool was
  built against.

## [0.2.0]

### Added
- `device_tree_overlays` declaration for the mezzanine's overlay. This is what
  makes the camera actually reach the board: avocado-cli compiles it, the BSP's
  `device-tree-overlay-deliver` hook claims it, and the UKI rebuild merges it
  into the embedded device tree. Previously the overlay shipped and nothing
  applied it.
- Composability follows from the same change: a core kit does not install this
  extension and gets a tree with no camss and no phantom IMX577.

## [0.1.0]

### Added
- Camera support for the RB3 Gen 2 Vision Mezzanine: the CAMSS ISP and its
  clock controller, the V4L2 core, and the Sony IMX577 sensor.
- `media-ctl` and `v4l-utils` for inspecting and proving the pipeline.
- The mezzanine's device tree as `overlays/qcs6490-rb3gen2-vision-mezzanine.dtso`.

### Notes
- CI targets the 2026 feed only; this board does not ship on 2024.
- The sensor's driver is `imx412`: that in-tree driver claims both
  `sony,imx412` and `sony,imx577`, and there is no `imx577` module.
- `depends_on: avocado-bsp-rb3gen2` — the core kit carries everything that is
  not the mezzanine.
- The overlay is shipped but **not** declared in `device_tree_overlays`, so the
  camera does not yet work. See README.
