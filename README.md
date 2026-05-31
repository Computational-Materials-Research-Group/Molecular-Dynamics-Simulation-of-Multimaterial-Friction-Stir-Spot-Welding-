# Molecular Dynamics Simulation of Multimaterial Friction Stir Spot Welding of Aluminum and Copper Lap Joints Using EAM Potential

<p align="center">
  <img src="https://img.shields.io/badge/LAMMPS-MD%20Simulation-blue?style=for-the-badge&logo=gnu&logoColor=white"/>
  <img src="https://img.shields.io/badge/FSSW-Al--Cu%20Lap%20Joint-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/EAM-CuAlW%20Potential-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Tool-Tungsten%20(W)-yellow?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/OVITO-Visualization-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Ubuntu-WSL-red?style=for-the-badge&logo=ubuntu&logoColor=white"/>
  <img src="https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey?style=for-the-badge"/>
</p>

<p align="center">
  A fully atomistic <b>molecular dynamics simulation of Friction Stir Spot Welding (FSSW)
  on a dissimilar Al–Cu lap joint</b> using LAMMPS. A rigid Tungsten tool (shoulder + pin)
  physically plunges, dwells, and retracts through a Copper top plate into an Aluminum
  bottom plate, driving frictional heating, plastic deformation, and atomic intermixing
  across the Al–Cu faying surface — all resolved at the angstrom scale.
  The simulation uses the <i>CuAlW EAM/alloy potential</i> with all three cross-element
  interactions (Al–Cu, Al–W, Cu–W) fully active, NVT integration with per-material
  thermostats, and full trajectory output for OVITO visualization.
</p>

<img width="1280" height="720" alt="multimat" src="https://github.com/user-attachments/assets/e5bf9f84-bd29-42ed-b01f-9dd0f41ebf02" />

---

## Simulation Overview

| Property | Value |
|----------|-------|
| Bottom plate | Aluminum — FCC, a₀ = 4.05 Å |
| Top plate | Copper — FCC, a₀ = 3.615 Å |
| Tool | Tungsten — BCC, a₀ = 3.16 Å |
| Potential | CuAlW EAM/alloy (Al–Cu–W cross-terms active) |
| Box size | 200 × 200 × 130 Å |
| Total atoms | ~308,000 (after overlap deletion) |
| Timestep | 1 fs (0.001 ps) |
| Ensemble | NVT (separate thermostats for Al and Cu) |
| Total simulation time | ~370 ps |

---

## Tool Geometry

```
z = 130 Å  ┌─────────────────────────────┐  ← box top
            │                             │
z = 120 Å  │    ████████████████████     │  ← Shoulder top
z = 110 Å  │    ████████████████████     │  ← Shoulder bottom (start)
            │           ████              │
            │           ████              │  ← Pin (r = 40 Å)
z = 100 Å  ├───────────────────────────  │  ← Cu top surface
            │  Cu top plate (type 2)      │
z =  50 Å  ├─────────────────────────────┤  ← Al–Cu faying surface
            │  Al bottom plate (type 1)   │
z =  45 Å  │           ████              │  ← Pin tip (start position)
            │                             │
z =   0 Å  └─────────────────────────────┘  ← Clamped base (anvil)
```

| Component | Radius | Z start | Z after plunge |
|-----------|--------|---------|----------------|
| Shoulder | 120 Å | 110–120 Å | 100–110 Å |
| Pin | 40 Å | 45–110 Å | 35–100 Å |
| Pin tip penetration | — | z = 45 Å | z = 35 Å (into Al) |

---

## Simulation Phases

```
Minimization (CG)
      ↓
Equilibration Phase 1 — NVT ramp 50 K → 300 K  (20 ps, Tdamp = 0.5 ps)
      ↓
Equilibration Phase 2 — NVT hold 300 K          (20 ps, Tdamp = 0.1 ps)
      ↓
Plunge Phase — rotation (ω = −0.03 rad/ps) + downward (−0.2 Å/ps)  (50 ps)
      ↓
Dwell Phase  — rotation only, Al–Cu mixing at faying surface        (150 ps)
      ↓
Retract Phase — rotation + upward (+0.2 Å/ps), system cools         (50 ps)
      ↓
Cooling Phase — NVT 300 K, weld nugget solidifies                   (100 ps)
```

---

## Why Multimaterial

Standard FSSW simulations use identical plates (Al–Al or Cu–Cu). This simulation models
the **industrially relevant dissimilar Al–Cu lap joint** used in battery tabs, heat exchangers,
and lightweight automotive structures. Key physics that emerge:

- **Differential heating** — Cu (heavier, higher melting point) heats faster on tool contact;
  Al heats more slowly via conduction across the faying surface
- **Atomic intermixing** — Al and Cu atoms exchange across z = 50 Å during dwell,
  observable as intermetallic-like clusters in OVITO
- **Thermal mismatch** — different lattice constants (4.05 vs 3.615 Å) create residual stress
  at the interface visible in per-atom stress computes
- **True 3-element potential** — all Al–Cu, Al–W, Cu–W pair interactions are active
  simultaneously via the CuAlW EAM/alloy file

---

## Atom Types and Color Coding

| Type | Element | OVITO Color | Role |
|------|---------|-------------|------|
| 1 | Al | 🔴 Red | Bottom plate |
| 2 | Cu | 🔵 Blue | Top plate |
| 3 | W | 🟡 Yellow | Rigid tool |

In OVITO: **Color Coding → Color by Type** reproduces the standard red / blue / yellow scheme shown in the snapshot above.

---

## Repository Structure

```
AtomicWeld/
│
├── in.fssw_AlCu.lammps          # Main LAMMPS input script
├── CuAlW.txt                    # EAM/alloy potential file (required)
├── README.md                    # This file
│
└── output/                      # Generated on run
    ├── diagnostic.lammpstrj          # Equilibration trajectory (100-step frames)
    ├── fssw_AlCu_production.lammpstrj # Plunge + dwell + retract (500-step frames)
    ├── step0_with_tool.xyz            # Initial geometry with tool above surface
    ├── step1_overlap_removed.xyz      # After Al–Cu interface overlap deletion
    ├── step2_minimized.xyz            # After CG energy minimization
    ├── step3_equilibrated.xyz         # Al and Cu both at 300 K
    ├── step4_plunge_done.xyz          # After plunge — pin inside Al plate
    ├── step5_dwell_done.xyz           # After dwell — Al–Cu mixed at interface
    ├── step6_retract_done.xyz         # After retract — weld nugget visible
    ├── step7_final.xyz                # Final cooled structure
    ├── fssw_AlCu_final.restart        # Binary restart file
    └── fssw_AlCu_final.data           # Final LAMMPS data file
```

---

## Requirements

- LAMMPS (7 Feb 2024 or newer, with MANYBODY package): https://www.lammps.org
- EAM potential file `CuAlW.txt` — place in the same directory as the input script
- OVITO for visualization: https://www.ovito.org

---

## Installation

```bash
# Ubuntu / WSL
sudo apt update && sudo apt install -y lammps
```

---

## Running the Simulation

```bash
# Single-core run
lmp -in in.fssw_AlCu.lammps

# Parallel run (recommended for ~308,000 atoms)
mpirun -np 8 lmp -in in.fssw_AlCu.lammps
```

---

## Simulation Parameters

### Thermostat

| Phase | Al thermostat | Cu thermostat |
|-------|--------------|--------------|
| Equil ramp | 50 → 300 K | 50 → 300 K |
| Equil hold | 300 K | 300 K |
| Plunge | 300 → 700 K | 300 → 900 K |
| Dwell | 700 K | 900 K |
| Retract | 700 → 300 K | 900 → 300 K |
| Cooling | 300 K | 300 K |

Cu is allowed to reach higher temperatures because it contacts the tool first and has a higher melting point (1358 K vs Al 933 K).

### Tool Motion

| Phase | Rotation ω (rad/ps) | Axial velocity (Å/ps) |
|-------|--------------------|-----------------------|
| Plunge | −0.03 (CW) | −0.2 (down) |
| Dwell | −0.03 (CW) | 0.0 |
| Retract | −0.03 (CW) | +0.2 (up) |

### Neighbor List

| Parameter | Value |
|-----------|-------|
| Skin distance | 2.5 Å |
| Max neighbors/atom | 10,000 |
| Update | every step, check yes |

---

## Key Script Features

| Feature | Implementation |
|---------|---------------|
| No double-integration | Single `nvt` fix per group — no `nve + temp/rescale` combo |
| Correct overlap removal | Threshold 2.0 Å (Cu nn = 2.556 Å) |
| Gentle equilibration | Start at 50 K, ramp to 300 K, Tdamp = 0.5 ps |
| Per-material thermostats | Separate `thermo_Al` and `thermo_Cu` groups |
| Combined tool motion | Single `fix move variable` for rotation + plunge (LAMMPS disallows two `fix move` on same group) |
| Energy sanity check | `run 0` after minimization — verifies pe/atom is negative before velocity init |
| Safe lost-atom handling | `thermo_modify lost warn` during equilibration, `lost error` in production |

---

## Visualization in OVITO

### Access trajectory from Windows (WSL users)

```
\\wsl$\Ubuntu\home\<username>\output\fssw_AlCu_production.lammpstrj
```

### Recommended modifiers

1. **Color Coding → Color by Type** — Al red, Cu blue, W yellow
2. **Common Neighbor Analysis (CNA)** — FCC (green), surface/disordered (red/white) atoms reveal mixing zone
3. **Polyhedral Template Matching (PTM)** — identify intermetallic-like disordered regions at faying surface
4. **Slice at z = 50 Å** — cross-section view of Al–Cu mixing at faying surface
5. **Per-atom stress (von Mises)** — thermal mismatch stress field around weld nugget

---

## What to Expect

### After Plunge (step4)
The W pin has crossed the Al–Cu faying surface. Cu atoms near the pin are significantly
displaced and heated. Al atoms just below z = 50 Å show early displacement.

### After Dwell (step5)
This is the most physically rich snapshot. The faying surface region (z = 45–55 Å) shows
Al and Cu atoms intermixed — analogous to the nugget zone in real FSSW experiments.
Temperature in the nugget region peaks near 700–900 K.

### After Cooling (step7)
The weld nugget has solidified. A disordered intermetallic-like zone is visible at the
original faying surface. Residual stress due to differential thermal contraction of Al and Cu
is present in the per-atom stress field.

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `integrated more than once` | Overlapping fix groups | Use single `nvt` per group — never `nve + rescale` |
| `Lost atoms` | Overlap at faying surface not removed | Increase `delete_atoms overlap` threshold to 2.0–2.5 Å |
| Temperature explosion | Velocity shock at init | Initialize at 50 K, ramp slowly with large Tdamp |
| `Unknown group keyword` | `group A B` copy syntax invalid | Use `group A type 2` or `group A union B` |
| `region subtract` error | `subtract` is not a valid LAMMPS region style | Use `union` of component regions instead |
| `unfix rot` crash | Fix name never defined | Remove — tool rotation is handled by `fix move variable` |

---

## Extending the Simulation

| Extension | What to Change |
|-----------|---------------|
| Different dissimilar pair | Change top plate lattice + mass + element in `pair_coeff` |
| Deeper pin penetration | Extend pin region lower, increase plunge distance |
| Higher RPM | Increase `omega` variable (e.g. −0.05 rad/ps) |
| Larger workpiece | Increase box from 200×200 to 300×300 Å |
| Add heat input zone | Use `fix heat` on faying group to mimic frictional heat source |
| Compute mixing index | Use `compute rdf` on faying_Al + faying_Cu to quantify intermixing |
| Apply clamping pressure | Add `fix pressurebeam` or `fix wall/reflect` on top surface |
| Stress–strain analysis | Add `compute stress/atom` + `compute reduce` for virial stress |

---

## Citation

If you use this simulation in your research, please cite:

```bibtex
@software{mishra_2026_fssw,
  author    = {Mishra, Akshansh},
  title     = {Molecular Dynamics Simulation of Multimaterial
               Friction Stir Spot Welding of Aluminum and Copper Lap Joints Using EAM Potential},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.20477826},
  url       = {https://doi.org/10.5281/zenodo.20477826}
}
```

Plain text citation:

> Mishra, A. (2026). *Molecular Dynamics Simulation of Multimaterial Friction Stir Spot Welding of Aluminum and Copper Lap Joints Using EAM Potential*. Zenodo. https://doi.org/10.5281/zenodo.20477826

---

## Author

**Akshansh Mishra**
GitHub: https://github.com/akshansh11

---

## License

<p>
<a rel="license" href="http://creativecommons.org/licenses/by/4.0/">
<img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png"/>
</a>
<br/>
This work is licensed under a
<a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.
</p>

You are free to:

- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material for any purpose, including commercially

Under the following terms:

- **Attribution** — You must give appropriate credit to Akshansh Mishra and provide a link to this repository

Copyright 2026 Akshansh Mishra.
