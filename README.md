# CHESS Software Stacks

[![Build Base System](https://github.com/MTCam/chess-software-stack/actions/workflows/ci-build-base-system.yml/badge.svg)](https://github.com/MTCam/chess-software-stack/actions/workflows/ci-build-base-system.yml)
![Platforms](https://img.shields.io/badge/platforms-Linux%20%7C%20macOS-green.svg)
![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)
![CHESS](https://img.shields.io/badge/CHESS-Software%20Suite-orange.svg)

---

## Purpose

This repository provides **reusable GitHub Actions workflows and utilities** for building, testing, and distributing the **scientific computing software stacks** that support CHESS simulation capabilities.

It serves as a **central integration point** for building and maintaining the key dependencies required by CHESS codes (e.g., *Hegel*, *Plato*, *CHyPS*, *JOTS*, *Prandtl*).  
Each CHESS software package can use these pre-tested workflows directly, ensuring consistent versions, reproducibility, and less CI duplication.

---

## Supported Components

| Package | Description | Source / Build Type |
|----------|--------------|---------------------|
| **PETSc** | Parallel linear algebra and solvers | System (`petsc-dev`) |
| **HDF5** | Parallel I/O and data format | System (OpenMPI-enabled) |
| **CGNS** | CFD data format | Built from source |
| **SUNDIALS** | Time integration and nonlinear solvers | Built from source |
| **ADIOS2** | I/O and multi-physics coupling backend | Built from source |
| **Mutation++** | Thermochemical and kinetics library (C++/Python) | Built from source |
| **preCICE** | Multi-physics coupling library | Built from source |
| **METIS 5** | Graph partitioning | Built from source (explicit version) |
| **MFEM** | Finite-element framework with MPI, HYPRE, METIS, ADIOS2 | Built from source |

Each library is toggleable through feature flags within the reusable workflow, allowing tailored stacks per solver or CI configuration.

---

## Repository Layout

.github/
├── workflows/
│ ├── ci-build-base-system.yml ← Entry workflow: defines build matrix
│ └── build-base-system.yml ← Reusable workflow: performs actual builds
install/ ← Installation prefixes (per job)
build/ ← Intermediate build directories



- **`ci-build-base-system.yml`** — runs matrix builds for Linux/macOS configurations.  
- **`build-base-system.yml`** — defines reusable steps to build and install each software component.

---

## Usage in Other CHESS Repositories

CHESS codes can call this reusable workflow directly in their own CI:

```yaml
jobs:
  setup-hpc-stack:
    uses: MTCam/chess-software-stack/.github/workflows/build-base-system.yml@main
    with:
      runner: ubuntu-latest
      build_label: ubuntu-mpp-mfem-coupling
      enable_mutationpp: true
      enable_precice: true
      enable_sundials: true
      enable_adios2: true
      enable_mfem: true
```

After completion, a compressed artifact named
deps-ubuntu-mpp-mfem-coupling.tar.gz
will be available for download and reuse.

### Example Build Matrix

The table below lists representative build configurations and their enabled components.  
Each configuration corresponds to a CHESS software context or platform target.

| Build Label | Enabled Components |
|--------------|-------------------|
| **ubuntu-mpp-mfem-coupling** | PETSc, HDF5, CGNS, Mutation++, MFEM, preCICE, SUNDIALS, ADIOS2 |
| **ubuntu-mpp** | PETSc, HDF5, CGNS, Mutation++ |
| **ubuntu-mfem** | PETSc, HDF5, CGNS, MFEM (with ADIOS2 and SUNDIALS) |
| **macos-core** | PETSc, HDF5 (core stack only) |

Each entry defines a reusable workflow target that other CHESS repositories can reference.  
Additional matrix entries may be added as new combinations of software or platforms are introduced.

### Artifacts

Each workflow run produces a compressed tarball artifact named using the build label, for example:

`deps-ubuntu-mpp-mfem-coupling.tar.gz`

This archive contains the installed directories for all components built during that workflow run.  
Typical contents include:

- adios2  
- cgns  
- hdf5  
- metis  
- mfem  
- mpp  
- mpp-py  
- precice  
- sundials  

These artifacts can be downloaded and reused directly by other CHESS repositories or CI workflows.  
Downstream build systems (for example, CMake) can locate libraries by setting environment variables such as `CMAKE_PREFIX_PATH` or `*_DIR` to the extracted directories.

Artifacts serve as portable, version-controlled dependency bundles that unify the software environment across CHESS projects.

### 🧰 Example Local Reproduction

To replicate a workflow build locally for testing or development:

1. **Clone the repository**

   `git clone https://github.com/chess-uiuc/chess-hpc-stacks.git`  
   `cd chess-hpc-stacks`

2. **Run the workflow steps manually**

   Follow the order defined in `.github/workflows/build-base-system.yml` to build each dependency.  
   For example, you can invoke the build scripts for PETSc, HDF5, CGNS, MFEM, and others individually to confirm local reproducibility.

3. **Inspect the results**

   The resulting installations will be placed under `install/` in your workspace, organized by component.  
   A portable tarball `deps-<build_label>.tar.gz` will be created containing all installed prefixes.

4. **Use the local stack**

   Downstream CHESS software can reference the local stack by setting:

   `export CMAKE_PREFIX_PATH=$PWD/install:$CMAKE_PREFIX_PATH`

   or directly passing component paths to CMake using  
   `-D<component>_DIR=<path-to-component>`.

This process mirrors the automated CI build and allows local verification or customization before integrating changes upstream.
### Related CHESS Software

The following software projects rely on, or are intended to integrate with, the CHESS build stacks:

- [**CHESS Toolchains**](https://github.com/chess-uiuc) — Repositories for CHESS Software
- [**Prandtl**](https://github.com/chess-uiuc/prandtl) — MFEM-based CNS flowsolver
- [**Hegel**](https://github.com/chess-uiuc/hegel) — Multiphysics simulation platform for magneto-gas dynamics and plasma flows.
- [**Plato**](https://github.com/chess-uiuc/hegel) — Plama thermochemistry and thermodynamic properties for NLTE gases.
- [**CHyPS**](https://github.com/chess-uiuc/chyps) - Coupled Hypersonic Protection System Simulator
- [**PlasFlowSolver**](https://github.com/chess-uiuc/PlasFlowSolver) — 1D PlasmatronX flow simulation and experimental envelope analysis utilities.


These projects collectively represent the CHESS software ecosystem maintained by the  
**Center for Hypersonic Entry Systems Studies (CHESS)** at the **University of Illinois Urbana–Champaign**.
