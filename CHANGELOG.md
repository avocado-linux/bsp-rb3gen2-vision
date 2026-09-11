# Changelog

All notable changes to avocado-bsp-rb3gen2-vision are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0]

### Added
- Camera support for the RB3 Gen 2 Vision Mezzanine: the CAMSS ISP and its
  clock controller, the V4L2 core, and the Sony IMX577 sensor.
- `media-ctl` and `v4l-utils` for inspecting and proving the pipeline.
- The mezzanine's device tree as `overlays/qcs6490-rb3gen2-vision-mezzanine.dtso`.

### Notes
- The sensor's driver is `imx412`: that in-tree driver claims both
  `sony,imx412` and `sony,imx577`, and there is no `imx577` module.
- `depends_on: avocado-bsp-rb3gen2` — the core kit carries everything that is
  not the mezzanine.
- The overlay is shipped but **not** declared in `device_tree_overlays`, so the
  camera does not yet work. See README.
