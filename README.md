# eChargeSCAD

OpenSCAD scripts for estimating black-powder ejection charges and modelling a
charge canister. Read the [disclaimer](DISCLAIMER.md) before using anything
here.

> **The output is an estimate, not a charge size.** Ejection depends on things
> this script does not model, such as the mass of the nose and recovery
> system, how tightly the recovery system fits, piston versus direct
> ejection, and leaks. Too little powder means a failed ejection; too much can
> damage the rocket or the recovery system. **Always ground test before
> flying**, starting low and working up.

## What is here

| File | What it does |
| --- | --- |
| `calcs.scad` | A worked example of the charge calculation. Inputs are hard-coded variables in the script; results are printed to the OpenSCAD console with `echo()`. It produces no geometry. |
| `canister.scad` | Unfinished charge-canister model: two back-to-back `G1/2` male thread adapters built from `Adapters_Mod/`. Its dimensions are fixed and are not taken from `calcs.scad`, and the cap is commented out. |
| `Adapters_Mod/` | Thread-adapter generator modified from Makrokaba's generator; see [Licence](#licence). |
| `shear_pin_details.md` | Notes on sizing nylon shear screws, which `calcs.scad` follows. |

## Usage

1. Install [OpenSCAD](https://openscad.org/).
2. Open `calcs.scad` and edit the input variables in the section below the
   `// Example Usage` comment. Units are given in each variable's comment:
   altitudes in metres, the tube diameter and bay dimensions in millimetres,
   friction in newtons, material strengths in psi.
3. Preview or render, and read the results in the console.

There is no unit parsing and there are no presets: the helper functions
(`convert_to_meters`, `convert_to_pascals`, `convert_to_newtons`,
`convert_to_grams`) only convert the units the script itself passes them.

`canister.scad` and `Adapters_Mod/` need the
[threadlib](https://github.com/adrianschlatter/threadlib) OpenSCAD library
installed in your OpenSCAD library path (`use <threadlib/threadlib.scad>`). It
is not included in this repository, and no version has been pinned.

## How `calcs.scad` works

The steps, in the order the script runs them:

1. **Flight load.** The pressure difference between `start_altitude` and
   `end_altitude` comes from the standard-atmosphere pressure formula. Acting
   on the end of a tube of diameter `tube_diameter`, plus `friction_force`,
   and multiplied by the factor of safety `fos`, it gives the minimum force
   the shear screws must hold.
2. **Shear screws.** Each screw's minimum breaking load uses its minor
   diameter and `material_strength_min`; its maximum uses the pitch diameter
   and `material_strength_max` (see `shear_pin_details.md`). The screw count
   is the minimum force divided by the per-screw minimum load, rounded up,
   **plus one**.
3. **Ejection force.** The count times the per-screw maximum load, minus
   `seal_factor` times the minimum force.
4. **Ejection pressure** is that force over the area of a circle of diameter
   $D$ (`internal_dia`):

   $P = \frac{F}{\pi \left( \frac{D}{2} \right)^2}$

5. **Black-powder mass** for a chamber of volume $V$, a cylinder of diameter
   `internal_dia` and length `internal_len`:

   $m = \frac{PV}{R_c T g}$

   with $R_c = 12.1579$ (combustion gas constant for FFFFg black powder, as
   given in the FreeCAD reference), $T = 1739\,\mathrm{K}$ (combustion gas
   temperature) and $g = 9.807\,\mathrm{m/s^2}$. The script prints the mass
   in grams, and again multiplied by the backup-charge factor `bucf`.
6. **Powder volume and canister size.** The mass is converted to a volume
   using 235 grains per cubic inch (from <https://www.tbullock.com/bpsg.html>),
   increased by 10%, and turned into a cylinder with a height-to-diameter
   ratio of 2. Foam packing is not included.

The current inputs (0 to 12,000 ft, M3 Nylon 6/6 screws, `fos = 2`,
`bucf = 1.5`) start from the example in `shear_pin_details.md` but depart from
it in several places. Known inconsistencies, such as the `tube_diameter`
comment saying 6 in while the value is 5.5 in, and the two different
diameters used for the same bay, are tracked in issue #1. The output has not
yet been compared with an independent calculator.

### Open questions

- **Where do `seal_factor = 0.25` and the `+ 1` in the screw count come
  from?** Neither is explained in the code or the notes, and their origin is
  not recorded. Until it is, treat them as unexplained margins rather than
  derived values.

## References

1. [FreeCAD Rocket Workbench: Rocket Ejection Charge Calculator](https://wiki.freecad.org/Rocket_Ejection_Charge_Calculator), source of the pressure and black-powder formulas and constants
2. [VernK: Ejection Charge Sizing](http://vernk.com/EjectionChargeSizing.htm)
3. [Insane Rocketry: Black Powder](https://www.insanerocketry.com/blackpowder.html)
4. [Rocketry Calculator: BP Estimator](https://rocketrycalculator.com/rocketry-calculator/bp-estimator)
5. [Rockethead: Black Powder Calculator](http://www.rockethead.net/black_powder_calculator.htm)
6. [HARA: How to Size Ejection Charge](http://hararocketry.org/hara/resources/how-to-size-ejection-charge)
7. feretich.com, "Shear Pin/Screw Calculations", [archived copy](https://web.archive.org/web/20260417090259/http://feretich.com/Rocketry/Resources/shearPins.html) (see `shear_pin_details.md`)

## Licence

Copyright (c) 2024 Cameron Brooks. Licensed under the BSD 3-Clause licence;
see [`LICENSE`](LICENSE).

`Adapters_Mod/*.scad` are modified from Makrokaba's "Universal customizable
OPENSCAD thread and pipe joint generator"
(<https://www.thingiverse.com/thing:4323468>), Copyright (c) 2020 Makrokaba,
published under a BSD licence. Each file keeps the original notice and
conditions in its header, alongside the modification notice.
