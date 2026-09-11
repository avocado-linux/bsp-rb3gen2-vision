---
language: yaml
targets:
  - rb3gen2
topics:
  - bsp
  - camera
  - device-tree
---

# RB3 Gen 2 Vision Mezzanine

Camera support for the mezzanine of the Qualcomm Robotics RB3 Gen 2 Vision Kit:
the SoC's CAMSS ISP, its clock controller, the V4L2 core, and the Sony IMX577
on CCI1.

## Why this is a separate extension

The RB3 Gen 2 is a core kit plus one of several mezzanines, and the mezzanine is
what varies between products. Keeping it separate is the same split the
EXMP-Q911 uses, where the module is the MACHINE and the EXCC carrier is an
extension: anything that is a property of the removable board belongs in an
extension, or every product combination needs its own machine.

So the division is:

| | |
|---|---|
| `avocado-bsp-rb3gen2` | core kit — QPS615 Ethernet, WCN6750 wifi/BT, LT9611UXC |
| `avocado-bsp-rb3gen2-vision` | this — CAMSS + IMX577 |
| `avocado-bsp-rb3gen2-industrial` | the other mezzanine, not yet written |

This extension `depends_on` the core-kit one rather than repeating it.

## The sensor is an IMX577, and its driver is called imx412

There is no `imx577` module. The in-tree `imx412` driver claims both parts:

```c
/* drivers/media/i2c/imx412.c */
{ .compatible = "sony,imx412", .data = "imx412" },
{ .compatible = "sony,imx577", .data = "imx577" },
```

The overlay declares `compatible = "sony,imx577"`, so `kernel-module-imx412` is
the right package. Checked against this build's kernel source, not inferred from
the package name.

## This does not work yet, and the reason is not in this extension

Verified on hardware, on a board running this BSP:

```
# ls -d /proc/device-tree/soc@0/camss*   ->  no such node
# ls /dev/video* /dev/media*             ->  no such file
```

The base `qcs6490-rb3gen2.dtb` describes no CAMSS and no sensor. What enables
them is `overlays/qcs6490-rb3gen2-vision-mezzanine.dtso` (upstream meta-qcom's,
carried verbatim), which sets `&camss` and `&cci1` to `okay` and declares the
IMX577 on `cci1_i2c1`.

That overlay is **shipped here but not applied**, for two reasons, both outside
this extension:

1. Declaring `device_tree_overlays:` pulls in `avocado-dtc-overlay-deliver`,
   which this feed does not carry for this target — `avocado install` fails with
   `No match for argument: avocado-dtc-overlay-deliver`.
2. Even with delivery, the Qualcomm flow flashes ONE dtb to `dtb_a` and has no
   overlay-application step.

The same gap blocks the EXCC-Q911 carrier overlay, so it is a flow-level
problem, not a board-level one.

The merge itself is already proven against this build's real base DTB:

```sh
fdtoverlay -i qcs6490-rb3gen2.dtb -o merged.dtb \
           qcs6490-rb3gen2-vision-mezzanine.dtbo    # applies, +765 bytes
```

So closing this needs a flow that performs that merge and flashes the result —
`fdtoverlay` at build time, or teaching the deploy recipe to emit a FIT. Until
then, the packages here install and the modules load, with nothing to bind to.

## What is NOT attempted

The full camera pipeline on an upstream kernel needs CamX, which is the harder
half of Vision Kit support. Upstream ships a separate
`qcs6490-rb3gen2-vision-mezzanine-camx.dtso` for that path. This extension
targets the mainline CAMSS route only: enough to enumerate the sensor and prove
it streams with `media-ctl` and `v4l2-ctl`, not a tuned ISP.
