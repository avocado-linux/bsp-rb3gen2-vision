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

## How the device tree reaches the board

The base tree describes no camera: the CAMSS node is present but disabled
(`isp@acb3000`, `compatible = "qcom,sc7280-camss"`) and there is no sensor. So
the packages above would load with nothing to bind to, and there would be no
`/dev/video*` or `/dev/media*`.

What enables them is `overlays/qcs6490-rb3gen2-vision-mezzanine.dtso` (upstream
meta-qcom's, carried verbatim), declared here as a `device_tree_overlays` entry.
The path from that declaration to the board:

1. avocado-cli collects the overlays of **every extension enabled on the
   runtime** and compiles each with `avocado-dtc-overlay`.
2. It calls the BSP's `device-tree-overlay-deliver` hook, which on this
   platform validates and claims them. Any overlay left unclaimed fails the
   build — a declaration cannot silently ship nothing.
3. `avocado-build-qcom` merges the claimed overlays into the device tree it
   embeds in the UKI, which sd-stub installs over the firmware's.

**This is why the mezzanine is composable.** The tree is a function of the
installed extension set: a core kit does not install this extension and gets no
camss and no phantom IMX577. Board-level overlays stay in the machine —
`kodiak-el2` and `qcs6490-rb3gen2-staging` are true of every RB3 Gen 2 whatever
is fitted.

It is also OTA-updatable. The UKI is an `os_artifact` that stone already A/Bs as
`file:efi:EFI/Linux/avocado-{a,b}+3.efi`, so installing this extension and
redeploying changes the device tree through the ordinary update path, with boot
counting and rollback.

## What is NOT attempted

The full camera pipeline on an upstream kernel needs CamX, which is the harder
half of Vision Kit support. Upstream ships a separate
`qcs6490-rb3gen2-vision-mezzanine-camx.dtso` for that path. This extension
targets the mainline CAMSS route only: enough to enumerate the sensor and prove
it streams with `media-ctl` and `v4l2-ctl`, not a tuned ISP.
