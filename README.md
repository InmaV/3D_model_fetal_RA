# 3D Computational Model of the Fetal Right Atrium

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![Ansys Fluent](https://img.shields.io/badge/Ansys%20Fluent-2025%20R1-orange.svg)](https://www.ansys.com/products/fluids/ansys-fluent)

This repository contains the geometries, transient boundary conditions, dynamic-mesh generation code, Ansys Fluent setup journals, particle-tracking post-processing code, and processed results developed to investigate blood-flow distribution in the fetal right atrium.

The framework was designed to study how local right-atrial anatomy—particularly the **Eustachian valve (EV)**—affects the preferential streaming of blood from the **ductus venosus (DV)**, **inferior vena cava (IVC)**, and **superior vena cava (SVC)** toward the **foramen ovale (FO)** and **tricuspid valve (TV)**.

> **Research use only.** This repository is intended for computational research and is not a medical device or a clinical decision-support tool.

## Repository contents

### Surface geometries

| File | Description |
|---|---|
| `reference_RA.stl` | Reference right-atrial surface geometry. |
| `reference_EV.stl` | Eustachian-valve surface for the reference configuration. |
| `smallerEV_RA.stl` | Right-atrial surface for the smaller-EV configuration. |
| `smallerEV_EV.stl` | Eustachian-valve surface for the smaller-EV configuration. |
| `less_elemets_RA.stl` | Reduced-element right-atrial surface used for mesh-sensitivity analyses. The original filename is retained. |

The geometries are simplified computational models and are not patient-specific reconstructions.

### Dynamic-mesh generation

| File | Description |
|---|---|
| `constructDeformation.ipynb` | Generates the prescribed right-atrial deformation using a Laplace–Beltrami displacement field and a target volume curve. |
| `RA_volume.npz` | Right-atrial volume curve for one cardiac cycle. It contains `voi` (time) and `data` (volume), with 430 samples at 0.001 s intervals. |

The deformation notebook:

1. Reads a labelled `reference.vtk` mesh.
2. Solves a constrained Laplace–Beltrami problem to define the spatial deformation field.
3. Scales the deformation at each time step to reproduce the target right-atrial volume.
4. Generates 2,151 configurations, corresponding to five cardiac cycles plus the final repeated point.
5. Exports deformed VTK/STL surfaces and dynamic-mesh coordinate files.

The labelled `reference.vtk` used by the notebook must contain the following point-data arrays:

```text
tricuspid_valve
wall
RA
IVC
SVC
FO
eustachian_valve
```

Typical generated files include:

```text
test_mesh.vtk
RA_data_bin.dat
dynamicmesh_reference.dat
Meshes/RA_deformed_reference/*.vtk
Meshes/RA_deformed_reference_stl/*.stl
```

These generated files are not included in the repository and must be created locally.

### Transient boundary-condition profiles

| File | Description |
|---|---|
| `mass_flow_dv_5.prof` | DV mass-flow inlet profile. |
| `mass_flow_ivc_5.prof` | IVC mass-flow inlet profile. |
| `mass_flow_svc_5.prof` | SVC mass-flow inlet profile. |
| `pressure_la_5.prof` | Left-atrial pressure profile applied at the FO outlet. |
| `pressure_rv_5.prof` | Right-ventricular pressure profile applied at the TV outlet. |

Each profile contains 2,151 samples from 0 to 2.15 s, corresponding to five cardiac cycles of 0.43 s with a time step of 0.001 s.

The profile values use the SI units expected by Ansys Fluent:

- Mass flow: kg/s.
- Pressure: Pa.

### Ansys Fluent setup journals

| File | Description |
|---|---|
| `pipeline_no_UDF.jou` | Configures the transient laminar-flow model, blood properties, boundary conditions, discrete particle tracking, report definitions, and automatic data exports. |
| `pipeline_UDF.jou` | Configures dynamic-mesh smoothing, remeshing, deforming zones, and user-defined moving boundaries. |

The journals were recorded using **Ansys Fluent 2025 R1** (`TUI version 25.1`).

The reference Fluent setup includes:

- Transient laminar flow.
- Blood density: `1050 kg/m³`.
- Dynamic viscosity: `0.00385 kg/(m·s)`.
- Massless, unsteady particle tracking.
- Separate particle injections at the DV, IVC, and SVC.
- Flow reports at the DV, IVC, SVC, FO, and TV.
- EnSight solution and particle-history exports.

> **Important:** the journal files contain scenario-specific table names, GUI selections, zone indices, and absolute Windows paths. They must be reviewed and adapted before execution.

In particular, `pipeline_no_UDF.jou` currently refers to transient tables with names such as:

```text
mass_flow_dv_1e-4
mass_flow_ivc_1e-4
mass_flow_svc_1e-4
pressure_la_1e-4
pressure_rv_1e-4
```

These names must be changed to match the supplied profile files or the profiles must be renamed accordingly.

`pipeline_UDF.jou` configures the dynamic-mesh model but does not include the source code of the Fluent UDF required to prescribe the wall motion.

### Particle-tracking post-processing

| File | Description |
|---|---|
| `Process_particle_tracking.ipynb` | Processes Fluent/EnSight particle histories and calculates the proportion of blood from each venous inlet reaching the FO or TV. |
| `particle_proportions_EV_summary.xlsx` | Summary of simulations performed with an EV. |
| `particle_proportions_noEV_summary.xlsx` | Summary of simulations performed without an EV. |
| `particle_proportions_summary_10cycles.xlsx` | Cycle-to-cycle results used to assess numerical convergence. |

The post-processing notebook:

1. Reads the time-dependent FO and TV surfaces from an EnSight case.
2. Calculates the FO and TV centroids at each time step.
3. Reads Fluent particle-history files for the DV, IVC, and SVC injections.
4. Identifies the final recorded position of each particle.
5. Assigns each particle to the nearest outlet.
6. Weights particle counts using the corresponding inlet-flow profile.
7. Calculates source-specific FO and TV percentages.
8. Exports Excel summaries and figures.

The processed Excel workbooks contain separate `FO` and `TV` sheets. The main columns are:

```text
simulation
DV
IVC
SVC
```

For each venous source and simulation, the FO and TV proportions should sum to approximately 100%.

## Software requirements

### Python

The notebooks were developed using Python 3.9 and require:

```text
numpy
scipy
matplotlib
pandas
openpyxl
tqdm
pyvista
vtk
jupyter
```


### CFD software

- Ansys Fluent 2025 R1 is recommended.
- A valid Ansys licence is required.
- A Fluent-compatible volume mesh or case must be prepared separately.
- The dynamic-mesh UDF source or compiled library must be available and loaded separately.


## Files not included

A complete reproduction of the Fluent simulations additionally requires files that are not included in this repository:

- The final Fluent volume mesh. 
- The dynamic-mesh UDF source code or compiled library.
- EnSight case and solution files.

The repository therefore provides the principal surface assets, boundary profiles, setup pipelines, processing methods, and processed results, but not a directly executable complete Fluent case. Details on how to create any additional required file are described in:

> Villanueva Baxarias, M. I. **Advancing the understanding of congenital heart diseases through computational modeling of the fetal cardiovascular system.** PhD thesis, Universitat Pompeu Fabra, 2026.

## Reproducibility notes

- The notebooks and journals contain hard-coded paths that must be edited.
- The Fluent journals are version- and mesh-order-dependent.
- The journal table names do not currently match the supplied `.prof` filenames.
- The post-processing notebook assumes a specific particle numbering and injection strategy.
- The STL files do not contain the point labels required by the deformation notebook.
- The supplied data contain computational geometries and processed simulation results; no identifiable clinical data are included.
- Ansys Fluent is proprietary software and is not distributed with this repository.

## Citation

When using this repository, please cite the associated work:

> Villanueva-Baxarias, I.; Fernandez-Cisneros, A.; Molla, M.; Leal, C.; Segarra-Queralt, M.; Barrouhou, M.; Albors, C.; Crispi, F.; Nakaki, A.; Garcia-Canadilla, P.; Camara, O.; Bijnens, B.; Bernardino, G. **A 3D computational model to investigate the determinants of blood flow distribution in the fetal right atrium: the importance of the Eustachian valve.** Manuscript in preparation.

The methodological framework is also described in:

> Villanueva Baxarias, M. I. **Advancing the understanding of congenital heart diseases through computational modeling of the fetal cardiovascular system.** PhD thesis, Universitat Pompeu Fabra, 2026.

## Licence

This repository is distributed under the **GNU General Public License v3.0**. See [`LICENSE`](LICENSE) for the complete licence text.

Third-party software, including Ansys Fluent, remains subject to its own licence terms.

## Contact

For questions, reproducibility issues, or suggested improvements, please open an issue in this GitHub repository.
