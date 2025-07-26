Parallel PSwarm Version 1.6
===========================

Copyright (C) 2009
A. Ismael F. Vaz and L. N. Vicente

Website: http://www.norg.uminho.pt/aivaz/pswarm

This library is free software; you can redistribute it and/or modify
it under the terms of the GNU Lesser General Public License as
published by the Free Software Foundation; either version 2.1 of the
License, or (at your option) any later version.

See the accompanying lgpl.txt for details.


Introduction
------------

**Parallel PSwarm (PPSwarm)** implements advanced global optimization
algorithms for bound- and linearly-constrained problems, using a hybrid
Particle Swarm and Pattern Search approach.

Key features:
- Serial and parallel (MPI) versions
- Support for bound-only or bound+linear constraints
- Optional AMPL, Python (3.x/NumPy), and MATLAB/MEX interfaces
- Fully cross-platform (Linux and Windows)
- Fast, robust, and customizable


Reference Publications
----------------------

- Vaz, A.I.F., Vicente, L.N., "A particle swarm pattern search method
  for bound-constrained global optimization," Journal of Global
  Optimization, 39 (2007), pp. 197-219.
- Vaz, A.I.F., Vicente, L.N., "PSwarm: A hybrid solver for linearly
  constrained global derivative-free optimization."


Change Log
----------

- v1.6: Modern CMake build system, cross-platform static/shared/optional interfaces, Python 3 and latest MATLAB supported
- v1.5: Improved memory allocation for larger problems
- v1.4: Vectorized objective function calls; Python and R interfaces
- v1.3: Iteration print callback, new options exported to interfaces
- v1.2: Bug fixes in Python interface and Makefile
- v1.1: Linear constraints, R and Python interfaces added
- v0.1: Initial release with serial/parallel and AMPL support


System Requirements
-------------------

- C compiler (GCC, Clang, or MSVC/Visual Studio)
- [Optional] MPI for parallel version (OpenMPI or MS MPI)
- BLAS and LAPACK libraries for linear constraints (often system-installed)
- [Optional] AMPL interface: AMPL solver library (`amplsolver.a`)
- [Optional] Python 3.x with NumPy (`pip install numpy`)
- [Optional] MATLAB for MEX interface

**Note:** All dependencies (except AMPL) are auto-detected by CMake.


Directory Structure
-------------------

- README.txt             : This file
- lgpl.txt               : GNU LGPL license
- pattern.c, pattern.h   : Pattern search algorithm
- pswarm.c, pswarm.h     : Particle swarm optimization
- pswarm_main.c, .h      : Main program and interface logic
- user.c                 : User-supplied objective/problem definition
- cache.c, cache.h       : (Optional) Caching code (disabled by default)
- mve_presolve.c         : Interior point code for ellipsoid
- mve_solver.c           : Maximum Volume Ellipsoid computation
- pswarm_py.c, .h        : Python interface
- pswarm_r.c, .h         : R/MATLAB/MEX interface
- RunPswarm.py, RunPswarm.r: Example scripts
- hs024.py, hs024.r      : Hock-Schittkowski example (Python/R)
- nl/                    : AMPL models (not distributed)
- include/               : Additional interface headers
- libs/                  : Additional libraries (AMPL, BLAS, LAPACK)


Building and Installation (with CMake)
--------------------------------------

**Prerequisites:**
- CMake 3.15 or newer
- Required compilers/libraries as above

**Quickstart (Linux/Mac):**
1. Open a terminal in the project root directory.
2. Create a build directory:
      mkdir build && cd build
3. Configure for a static build (default):
      cmake ..
4. Build the software:
      cmake --build .
5. All executables and libraries will be in `build/bin/` and `build/lib/`.

**Enabling Optional Interfaces:**
- **Python interface (requires Python 3 and NumPy):**
      cmake .. -DBUILD_PYTHON=ON
- **MATLAB MEX interface (requires MATLAB):**
      cmake .. -DBUILD_MATLAB=ON
- **AMPL interface (requires AMPL solver library):**
      cmake .. -DUSE_AMPL_SOLVER=ON [-DAMPL_SOLVER=/path/to/amplsolver.a]
- **Build with shared libraries:**
      cmake .. -DBUILD_SHARED_LIBS=ON

You may combine options, e.g.:
      cmake .. -DBUILD_PYTHON=ON -DBUILD_MATLAB=ON -DUSE_AMPL_SOLVER=ON

**Build types:**
- **Release (optimized, default):**
      cmake .. -DCMAKE_BUILD_TYPE=Release
- **Debug (with debug info):**
      cmake .. -DCMAKE_BUILD_TYPE=Debug

**Clean build artifacts:**
      cmake --build . --target distclean

**Quickstart (Windows, with Visual Studio):**
1. Install [CMake](https://cmake.org/download/) and Visual Studio.
2. Open "x64 Native Tools Command Prompt for VS".
3. As above, create and enter a build directory.
4. Configure:
      cmake .. -G "Visual Studio 17 2022" -A x64
5. Build:
      cmake --build . --config Release
6. Binaries will be in `build/bin/Release` and libraries in `build/lib/Release`.


Typical Usage
-------------

- **Standalone (serial) version:**  
  Edit `user.c` to define your objective/problem.  
  Build and run `pswarm_serial` (or `pswarm_serial_linear` for linear constraints).

- **Parallel version:**  
  Use `mpirun -np <N> ./pswarm_parallel`  
  (See `job.pbs` for cluster usage examples.)

- **AMPL interface:**  
  Build with `USE_AMPL_SOLVER=ON`.  
  Run with an AMPL `.nl` model: `./pswarm_ampl model.nl`

- **Python interface:**  
  Build with `BUILD_PYTHON=ON`.  
  In Python, use:  
      import RunPswarm

- **MATLAB/MEX interface:**  
  Build with `BUILD_MATLAB=ON`.  
  Use the resulting `.mex*` file from MATLAB or as an R shared object.


Credits and License
-------------------

PPSwarm was developed by A. Ismael F. Vaz and L. N. Vicente.
Portions of the code (MVE routines) are based on work by Y. Zhang and L. Gao.

This software is distributed under the GNU Lesser General Public License (LGPL v2.1 or later).
See lgpl.txt for full details.

Contact:
- A. Ismael F. Vaz — http://www.norg.uminho.pt/aivaz
- L. N. Vicente    — http://www.mat.uc.pt/~lnv

Project page: http://www.norg.uminho.pt/aivaz/pswarm
