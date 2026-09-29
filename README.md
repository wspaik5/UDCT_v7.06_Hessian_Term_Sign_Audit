# UDCT v7.06 Hessian-term sign audit — reproducibility code

This folder accompanies

> **A sign audit of a Hessian term absent from the standard MOND equation:
> action-consistent virial work and conditional Jeans tests in three
> ultra-faint dwarfs** (29 September 2026),

which consolidates UDCT v7.01–v7.06. The script is an independently
executable extraction of the local-field numerical work used for v7.05–v7.06.
It reproduces the **conditional model calculations**. It is not an
observational detection, not a rerun of the historical O8d/D8b proxy, and not
a result for AQUAL or QUMOND.

Script file: `UDCT_v7.06_Hessian_Term_Sign_Audit_reproduce.py`

On GitHub, copy this file to `README.md` so the repository front page renders it.

## Install and run

Use Python 3.10+ with NumPy and SciPy:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install "numpy>=1.24" "scipy>=1.10"

# Fast first check (recommended). Baseline V/V_N still matches the printed table.
python UDCT_v7.06_Hessian_Term_Sign_Audit_reproduce.py --smoke --mode baseline > baseline.json

python UDCT_v7.06_Hessian_Term_Sign_Audit_reproduce.py --smoke --mode switch > switch.json
python UDCT_v7.06_Hessian_Term_Sign_Audit_reproduce.py --smoke --mode alpha > alpha.json
python UDCT_v7.06_Hessian_Term_Sign_Audit_reproduce.py --smoke --mode xs-crossing > xs_crossing.json
python UDCT_v7.06_Hessian_Term_Sign_Audit_reproduce.py --smoke --mode high-scale > high_scale.json
```

Run `python UDCT_v7.06_Hessian_Term_Sign_Audit_reproduce.py --help` for all
switches. The paper defaults are 360 radial points, 24 polar × 36 azimuthal
directions, 160 action nodes and a 120 × 64 × 12 disk quadrature. That default
grid is heavy; start with `--smoke` or one object at a time
(`--object 'Pegasus III'`). The output is JSON on stdout. The script itself
does not write files.

`--smoke` uses 180 × 16 × 24 × 96 action nodes and a 40 × 24 × 8 disk
quadrature. On that grid the printed baseline \(V/V_N\) values are recovered
to the paper’s rounding. `--mode all` on the default grid is much slower;
do not start there.

## Smoke-test expected values

Rows are Leo IV, Pegasus III, Hydrus I; each entry is Plummer / tapered cusp.
These figures were recovered with `--smoke --mode baseline` (and `--mode alpha`
for the crossings). They are checks, not hard-coded returns.

| Diagnostic | Expected smoke / paper value |
| --- | --- |
| Baseline \(V/V_N\) | −683.84 / −969.06 ; −810.90 / −1489.62 ; −415.42 / −299.67 |
| Joint \(\alpha\) crossing of \(E\) and \(H\) | 0.01903 / 0.06425 ; 0.02426 / 0.10556 ; 0.00320 / 0.00520 |
| Visible baryon mass in the disk+bulge quadrature | \(6.621 \times 10^{10}\,M_\odot\) |
| Pegasus III Plummer local \(g_r/g_*\) at \(r_h\) | \(+0.88\) |
| Hydrus I Plummer local \(g_r/g_*\) at \(r_h\) | \(-483.44\) |

A third party can confirm a working checkout with:

```bash
python UDCT_v7.06_Hessian_Term_Sign_Audit_reproduce.py --smoke --mode baseline --object "Hydrus I"
```

and looking for Plummer `"virial":` near `-415.42`.

## Inputs and conventions

- The three frozen table inputs (luminosity, projected half-light radius,
  heliocentric sky position and historical velocity reference), numerical
  constants, four visible McMillan17 disk components, and the
  \(8.62\times 10^9\) solar-mass point bulge are all declared near the top of
  the script. Solar Galactocentric radius is 8.2 kpc; no dark halo or LMC is
  included. Sky coordinates and luminosities follow Pace, Local Volume
  Database (2025) and the archived UDCT v5.61 table. The reported catalog
  Galactocentric distance is descriptive; the integral evaluates the Galaxy
  at the position calculated from sky coordinates.
- The galaxy force **and** Hessian are integrated from the same visible mass
  density. This script implements exactly the **local second-order Taylor
  environment** of the cited analysis; it does not solve the global,
  nonaxisymmetric boundary-value problem.
- The stellar mass is 2 times the tabulated luminosity in solar units. The
  Plummer scale equals the observed projected half-light radius. The
  alternative is a Hernquist-like cusp with an \(\exp[-(r/(10 R_e))^4]\)
  taper; its scale is calibrated numerically so half its projected mass lies
  inside \(R_e\).
- The action uses \(a_0=1.082401\times 10^{-10}\,\mathrm{m\,s^{-2}}\),
  \(p=4\), and \(X_s=4\times 10^9\) at baseline. The angular average includes
  both the positive multiplier \(\nu\) and the Hessian tensor \(P\) derived
  from the same action. In the `switch` and `xs-crossing` modes, **both**
  terms change together.
- The normalized work \(V/V_N\) is evaluated by radial integration by parts,
  including its finite-radius boundary term. The JSON key `direct` separately
  integrates the numerically differentiated local force; compare it with
  `virial` as a numerical cross-check. Ratios use a positive stellar Newtonian
  denominator and an inward-positive acceleration convention. The JSON keys
  `leading` and `derivative` split that ratio: at baseline, `leading` stays
  positive while `derivative` is a larger negative term. That is why the
  negative virial work is not produced by \(\nu\) becoming negative.
- The `high-scale` calculation additionally solves a **conditional** spherical,
  isotropic Jeans equation with zero outer pressure. Its projected LOS rms is
  a formal aperture integral at \(R_e\), not a matched-aperture fit to the
  archived observations. Negative \(\sigma_r^2\) signals incompatibility of
  these assumptions with that calculated force; it is not a physical negative
  velocity variance.

## Paper comparison

These figures are convenient checks, not hard-coded results returned by the
program. Rows follow Leo IV, Pegasus III, Hydrus I; each entry is
Plummer / tapered cusp.

| Diagnostic | Paper result |
| --- | --- |
| Baseline \(V/V_N\) | −683.84 / −969.06 ; −810.90 / −1489.62 ; −415.42 / −299.67 |
| Jointly scaled external \(E\) and \(H\): first \(\alpha\) crossing | 0.01903 / 0.06425 ; 0.02426 / 0.10556 ; 0.00320 / 0.00520 |
| High-\(X_s\) work zero crossing, in \(10^{12}\) units | 2.374 / 2.382 ; 3.891 / 4.034 ; 2.263 / 1.756 |
| \(V/V_N\) at \(X_s=4\times 10^{13}\) | 12.318 / 12.208 ; 15.982 / 15.758 ; 3.525 / 3.210 |
| \(V/V_N\) at \(X_s=4\times 10^{17}\) | 14.013 / 13.895 ; 19.370 / 19.214 ; 3.994 / 3.557 |

The default (paper) grid reproduces the virial numbers to their printed
precision. The \(\alpha\) and transition-scale crossings agree to the shown
rounding even on the smoke grid. The `switch` command prints all
\(p=1,2,3,4,6,8\) rather than just the \(p=1,4,8\) columns in the paper.

**Jeans precision.** The default grid gives projected aperture rms values
within roughly 0.001 km/s of the paper’s Table 6. For the Pegasus III tapered
cusp at \(X_s=4\times 10^{13}\), the script’s 1800 × 32 × 48 × 256 refinement
gives a minimum of −0.006229 (km/s)\(^2\) in 4–9 \(r_h\), agreeing with the
printed Table 7. The pointwise `force_at_6rh_over_newton` is about −0.862 on
that grid, versus −0.894 printed in the PDF; this pointwise derivative remains
a visible unresolved difference, despite agreement of the integrated work,
pressure minimum, and aperture rms. Do not present every printed pointwise
digit as independently reproduced. The `direct` versus `virial` comparison
isolates integration error in the work integral but is not a proof of a fully
self-consistent stellar distribution function.

## Scope of the released code

The historical preregistered O8d/D8b result was **0/22**, with a pass gate of
11/22. This code does **not** recalculate those 22 observed-system proxy
predictions: the input table and proxy implementation are not part of this
stand-alone v7.05–v7.06 numerical extraction. The later smooth action and the
three-dwarf diagnostics were proposed **after** that historical result. Do
not label any output here as a new preregistered prediction.

This release does not calculate AQUAL or QUMOND on the same three objects,
nor does it establish a general MOND sign theorem. The positive interpolation
factor is kept positive. The large negative virial work comes from the
action-derived Hessian contribution in the stipulated dwarf and Galactic-field
models. Hydrus I lacks an LMC field; stellar anisotropy, time dependence,
boundary stresses and a full 3D Jeans or distribution-function solution are
not included.

For provenance and exact narrative assumptions, cite the companion PDF and
the earlier UDCT v7.01–v7.06 notes. Relevant inputs: P. J. McMillan (2017),
MNRAS 465, 76, doi:10.1093/mnras/stw2759; A. B. Pace, Local Volume Database
(2025), doi:10.33232/001c.144859. The toy alternative stellar profile is
inspired by L. Hernquist (1990), ApJ 356, 359, doi:10.1086/168845.
