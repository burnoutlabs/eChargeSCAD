# Shear pin and shear screw sizing

This note summarises the method that `calcs.scad` uses to size nylon shear
screws. The method and the example case follow the page "Shear Pin/Screw
Calculations" on feretich.com, which is no longer online; an archived copy is
at
<https://web.archive.org/web/20260417090259/http://feretich.com/Rocketry/Resources/shearPins.html>.
Read that page for the original explanation. The text below is a summary, and
the table was recomputed for this repository from the listed diameters.

## Method in brief

1. **Find the load the screws must hold.** In flight, air trapped in a sealed
   bay pushes the sections apart as outside pressure drops with altitude. The
   separating force is that pressure difference times the bay's end area, less
   whatever force the coupler friction already supplies.
2. **Get the material's shear strength.** Use the manufacturer's or
   distributor's figure. If only a tensile strength is published, about two
   thirds of it is a common estimate of shear strength. Treat the published
   value as a best case: moisture, UV exposure and unspecified generic
   "nylon" can all lower it, so a strength-rated part from an industrial
   supplier is preferable to unlabelled hardware-store screws.
3. **Bound each screw's breaking load.** A threaded screw does not have one
   obvious shear area. The area at the minor (root) diameter gives a lower
   bound, and the area at the pitch diameter gives an upper bound for a tight
   fit that forces the break straight across the threads. Multiply the minor
   area by the lowest quoted strength for the minimum breaking load, and the
   pitch area by the highest quoted strength for the maximum.
4. **Choose the count and the charge.** Pick enough screws that their combined
   *minimum* breaking load exceeds the flight load. Size the ejection charge
   to exceed their combined *maximum* breaking load, plus any pressure lost to
   leaks, so that the screws are sure to shear.

Space the screws evenly around the circumference, and never use only one,
since the coupler can cock and bind. Tap only the body tube, not the coupler
as well: two separately tapped threads rarely line up, which preloads the
screw and can shear it early.

## Screw data (Nylon 6/6, 9600 to 10500 psi)

Areas are π(d/2)² from the pitch and minor diameters. Breaking loads use
9600 psi on the minor area (minimum) and 10500 psi on the pitch area
(maximum). `calcs.scad` uses the same minor and pitch diameters in
`get_screw_diameters()`.

| Screw | Major dia. (in) | Pitch dia. (in) | Minor dia. (in) | Pitch area (in²) | Minor area (in²) | Min. load (lbf) | Max. load (lbf) |
|-------|-----------------|-----------------|-----------------|------------------|------------------|-----------------|-----------------|
| M2    | 0.07874 | 0.06851 | 0.05942 | 0.00369 | 0.00277 | 27 | 39  |
| 2-56  | 0.08600 | 0.07440 | 0.06410 | 0.00435 | 0.00323 | 31 | 46  |
| 4-40  | 0.11200 | 0.09576 | 0.08130 | 0.00720 | 0.00519 | 50 | 76  |
| M3    | 0.11811 | 0.10532 | 0.09396 | 0.00871 | 0.00693 | 67 | 91  |
| 6-32  | 0.13800 | 0.11770 | 0.09970 | 0.01088 | 0.00781 | 75 | 114 |

## Example case

The source page works through this case, and the hard-coded inputs in
`calcs.scad` start from it:

- A sealed bay climbing to 12,000 ft with no venting sees about 5.5 psi of
  pressure difference.
- On a 6 in diameter tube that is about 155 lbf (5.5 psi × π × 3² in²).
- Coupler friction is taken as 5 lbf, leaving about 150 lbf for the screws.
- With the minimum loads above, three 4-40 screws (3 × 50 = 150 lbf) just
  cover it.
- Shearing them takes about another 3 psi, so the charge is sized for the
  full 8.5 psi, plus margin for any leakage.

`calcs.scad` departs from this example in several ways (a 5.5 in tube
diameter, no friction, M3 screws, a factor of safety of 2, an extra screw and
a `seal_factor`); see the README.
