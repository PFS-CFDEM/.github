# 🧪 CFDEM® Installation Guide (HSU – Hsuper Cluster, October 2025)

This installation guide was **tested on the Hsuper cluster at Helmut-Schmidt-University (HSU)** in **October 2025**.  
It installs **CFDEM®coupling** with **OpenFOAM-6**, using:

- **GCC 8.5.0** (default compiler on Hsuper as of October 2025)  
- **OpenMPI 3.1.6** (recommended — newer versions may compile but can cause runtime issues)  
- **LIGGGHTS-PUBLIC** and **LPP** from the PFS-CFDEM GitHub organization  

👉 **Reference manual (highly recommended):**  
[CFDEM Official Installation Guide](https://www.cfdem.com/media/CFDEM/docu/CFDEMcoupling_Manual.html#installation)

---

## 🪜 1. Create a working directory

```bash
cd "$HOME"
mkdir CFDEM-PFS
cd CFDEM-PFS
```

---

## 🧰 2. GCC and OpenMPI

### GCC

GCC 8.5.0 is currently the standard system compiler on Hsuper.

```bash
gcc --version
```

Expected:
```
gcc (GCC) 8.5.0
```

If another GCC is active in your `.bashrc`, comment it out.  
If GCC 8.5.0 is not standard anymore, install it via USER-SPACK or build manually as:


> ⚠️ **Optional:** If GCC 8.5.0 is not available on your system, you can build it manually as follows:

> ```bash
> cd $HOME/CFDEM-PFS/
> mkdir gcc && cd gcc
>
> wget https://ftp.fu-berlin.de/unix/languages/gcc/releases/gcc-8.5.0/gcc-8.5.0.tar.gz
> tar xvzf gcc-8.5.0.tar.gz
>
> ## download prerequisites
> cd gcc-8.5.0
> ./contrib/download_prerequisites
>
> ## configure
> ./configure --disable-multilib --prefix=$HOME/CFDEM-PFS/opt/gcc/8.5.0 --enable-languages=c,c++,fortran
>
> ## make & install
> make -j 12
> make install
> 
> After installation, update your environment to compile openmpi with this gcc version
> 
> export PATH="$HOME/CFDEM-PFS/opt/gcc/8.5.0/bin:$PATH"
> export LD_LIBRARY_PATH="$HOME/CFDEM-PFS/opt/gcc/8.5.0/lib64:$LD_LIBRARY_PATH"
>```

### OpenMPI 3.1.6

Although CFDEM may compile with newer OpenMPI versions, some of them lead to runtime problems.  
Recommended: **OpenMPI 3.1.6**

```bash
cd "$HOME/CFDEM-PFS"
mkdir openmpi
cd openmpi

wget https://download.open-mpi.org/release/open-mpi/v3.1/openmpi-3.1.6.tar.bz2
tar xvjf openmpi-3.1.6.tar.bz2
cd openmpi-3.1.6

./configure --prefix="$HOME/CFDEM-PFS/opt/openmpi/3.1.6"
make -j 12 all install
```

---

## 📦 3. Clone the repositories

```bash
cd "$HOME/CFDEM-PFS"
mkdir CFDEM
cd CFDEM
git clone git@github.com:PFS-CFDEM/CFDEMcoupling-PUBLIC-6.git

cd "$HOME/CFDEM-PFS"
mkdir LIGGGHTS
cd LIGGGHTS
git clone git@github.com:PFS-CFDEM/LIGGGHTS-PUBLIC.git
git clone git@github.com:PFS-CFDEM/LPP.git lpp

cd "$HOME/CFDEM-PFS"
mkdir OpenFOAM
cd OpenFOAM
git clone git@github.com:PFS-CFDEM/OpenFOAM-6.git
git clone git@github.com:PFS-CFDEM/ThirdParty-6.git
```

---

## 🌱 4. Environment setup 

```bash
emacs "$HOME/CFDEM-PFS/cfdem.env"
```

Paste the following content:

```bash
# === CFDEM environment file  ===

# --- Determine CFDEM root from *this file* location ---
# Works on Linux; resolves symlinks if readlink -f exists.
if command -v readlink >/dev/null 2>&1; then
  _SELF="$(readlink -f "${BASH_SOURCE[0]}")"
else
  _SELF="${BASH_SOURCE[0]}"
fi
export CFDEM_ROOT="$(cd "$(dirname "$_SELF")" && pwd)"
export LOCDIR="$(basename "$CFDEM_ROOT")"     # optional, if you still want the name

# --- MPI used to build/run CFDEM ---
export PATH="$CFDEM_ROOT/opt/openmpi/3.1.6/bin:$PATH"
export LD_LIBRARY_PATH="$CFDEM_ROOT/opt/openmpi/3.1.6/lib:$LD_LIBRARY_PATH"

# --- if GCC 8.5.0 was manually compiled ---
#export PATH="$CFDEM_ROOT/opt/gcc/8.5.0/bin:$PATH"
#export LD_LIBRARY_PATH="$CFDEM_ROOT/opt/gcc/8.5.0/lib64:$LD_LIBRARY_PATH"

# --- OpenFOAM settings ---
export WM_NCOMPPROCS=12
export WM_MPLIB=SYSTEMMPI

# --- OpenFOAM (quiet normal output; errors still visible) ---
# (Assumes your OpenFOAM tree lives under CFDEM_ROOT/OpenFOAM)
source "$CFDEM_ROOT/OpenFOAM/OpenFOAM-6/etc/bashrc" >/dev/null

# --- CFDEM paths (all relative to CFDEM_ROOT) ---
export CFDEM_VERSION="PUBLIC"
export CFDEM_PROJECT_DIR="$CFDEM_ROOT/CFDEM/CFDEMcoupling-PUBLIC-$WM_PROJECT_VERSION"
export CFDEM_PROJECT_USER_DIR="$CFDEM_ROOT/CFDEM/$LOGNAME-$CFDEM_VERSION-$WM_PROJECT_VERSION"

export CFDEM_bashrc="$CFDEM_PROJECT_DIR/src/lagrangian/cfdemParticle/etc/bashrc"
export CFDEM_LIGGGHTS_SRC_DIR="$CFDEM_ROOT/LIGGGHTS/LIGGGHTS-PUBLIC/src"
export CFDEM_LIGGGHTS_MAKEFILE_NAME="auto"
export CFDEM_LPP_DIR="$CFDEM_ROOT/LIGGGHTS/lpp/src"

# --- CFDEM init  ---
. "$CFDEM_bashrc"

```

Activate the environment:
```bash
source "$HOME/CFDEM-PFS/cfdem.env"
```
you will get a question regarding creating a user directory for CFDEM, answer with yes (y).

Test:
```bash
mpirun --version
```

Expected:
```
mpirun (Open MPI) 3.1.6
```

Optional alias in `~/.bashrc`:
```bash
alias cfdemenv='source $HOME/CFDEM-PFS/cfdem.env'
```

---

## 🧱 5. Compilation

Activate the cfdem environment. Either:
```bash
source "$HOME/CFDEM-PFS/cfdem.env"
```
Or if you implemented the alias

```bash
cfdemenv
```
Compile OpenFOAM-6
```bash
cd $WM_PROJECT_DIR
./Allwmake
```
Then:

```bash
cd $CFDEM_LIGGGHTS_SRC_DIR/MAKE
cfdemCompLIG
```
You will get an error like "Could not determine suitable appendix of VTK library with VTK_INC..".
To solve this issue:

```bash
emacs Makefile.user
```
and set  USE_VTK = "OFF" instead of "ON".

Then

```bash
cd $CFDEM_PROJECT_DIR
cfdemCompCFDEMall
```

cfdemCompCFDEMuti
This compiles:
- LIGGGHTS® executable & shared library (cfdemCompLIG)  
- CFDEM® coupling libraries (cfdemCompCFDEMsrc) 
- CFDEM® solvers (cfdemCompCFDEMsol)
- CFDEM® utilities (cfdemCompCFDEMuti)

Logs:
```
$CFDEM_PROJECT_DIR/src/lagrangian/cfdemParticle/etc/log
```

---

## 🧪 6. Running cases

### PFS deagglomeration test cases (by Ali Khalifa):
```bash
cd "$HOME/CFDEM-PFS"
git clone git@github.com:PFS-CFDEM/PFS_testcases.git
```

Further explanations in the Readme file: 
https://github.com/PFS-CFDEM/PFS_testcases/tree/main



Note: for pure LIGGGHTS® simulations (only DEM):
```bash
cfdemLiggghts inputScriptName
cfdemLiggghtsPar inputScriptName nProcs
```

---

## 🧭 7. SLURM job script



DO not forget to source the cfdem environment file in your SLURM job submission file:
```bash


# Activate CFDEM environment (quiet)
source "$HOME/CFDEM-PFS/cfdem.env" 

```

## 🧪 8. Post-Processing Environment Setup

For post-processing simulation results, we use:

- 📊 **ParaView** — for visualization  
- 🧮 **Fortran scripts** — for data extraction and pre-processing  
- 🐍 **Python (Conda)** — for analysis and automation

To set up **ParaView** and **Miniforge (Conda)** for Python post-processing, please follow the detailed installation guide here:

👉 [Processing Software Setup Instructions](https://github.com/PFS-CFDEM/Processing_software/blob/main/README.md)

These instructions include:
- Local installation of ParaView (no root privileges required)  
- Clean Miniforge/Conda environment setup  
- Creating and activating the `demcfd` Conda environment  
- Using the environment inside SLURM job scripts

Once the environment is set up, you can run your Python post-processing scripts directly on the cluster or locally.


---

## 🔸 What Makes the **PFS Version** of CFDEM Special?

The **PFS-CFDEM** fork builds upon the original [CFDEMcoupling](https://www.cfdem.com/) framework with several targeted improvements and new capabilities, primarily aimed at enhancing **immersed boundary (IB)** simulations and **turbulent flow–particle interaction studies**.

---

### 🚀 I. Customized Immersed Boundary (IB) Solver for HIT
- **New solver:** `cfdemSolverIB_HIT`  
  📂 `CFDEM/CFDEMcoupling-PUBLIC-6/applications/solvers/cfdemSolverIB_HIT/`  
- **Reference:** M. Bassenne (2016) – method for generating *homogeneous isotropic turbulence (HIT)* via a source term.  
- **Key feature:** Extends the standard IB solver (`cfdemSolverIB`) with additional forcing capabilities to maintain statistically stationary HIT conditions, making it ideal for particle–turbulence interaction studies.

---

### 🧩 II. Major Improvements to the IB Cloud Routine
📂 `CFDEM/CFDEMcoupling-PUBLIC-6/src/lagrangian/cfdemParticle/derived/cfdemCloudIB/`  

- ✅ **Gaussian Interface field option**  
  - Added the possibility to compute the Interface field based on a *Gaussian Kernel*, improving control of local mesh refinement around particles. This is especially important if the particle is smaller than the grid spacing of the initial mesh before any Dynamic mesh refinement, since the standard method for calculating the Interface field might fail to find the particle.
  -  **Activation:** This feature can be turned on in the `couplingProperties` file located in the `constant/` directory of the CFD case.
- 🐞 **Periodic image detection bug fix**  
  - Corrected a long-standing issue preventing proper activation of periodic boundary detection for particles.
- 🧮 **Particle velocity enforcement fix**  
  - Resolved a bug related to imposing particle velocities on cells shared between multiple particles, improving physical consistency of the IB forcing.

---


### 🛠️ III. Fixes & Enhancements to the Standard IB Void Fraction Model
📂 `CFDEM/CFDEMcoupling-PUBLIC-6/src/lagrangian/cfdemParticle/subModels/voidFractionModel/IBVoidFraction/`  

- Corrected inconsistencies and improved numerical stability of the default IB void fraction implementation.
- Ensures more accurate and stable fluid–particle coupling, particularly in dense particle suspensions.

---

### 🧪 IV. New Optional Monte-Carlo Void Fraction Model
📂 `CFDEM/CFDEMcoupling-PUBLIC-6/src/lagrangian/cfdemParticle/subModels/voidFractionModel/IBMCVoidFraction/`  

- Introduced a **Monte Carlo–based void fraction model** that provides a more flexible and statistically robust estimation of the fluid–particle interaction volume fraction.
-  **Activation:** This feature can be turned on in the `couplingProperties` file located in the `constant/` directory of the CFD case. 
- Useful for highly resolved IB simulations and cases involving complex particle geometries or dense packings.
- Might be more time-consuming (more testing is required)


---

### ⚙️ V. New Contact Force Model Including van der Waals Attraction
📂 `CFDEM-PFS/LIGGGHTS/LIGGGHTS-PUBLIC/src/normal_model_hertz_fvdw.h`  

- Introduced a **Hertzian contact model** augmented with **attractive van der Waals forces** based on the **Hamaker formulation**.  
- Captures **cohesive interactions** between particles and between particles and walls.  
- Implemented directly in the **LIGGGHTS DEM solver**, ensuring full compatibility with CFDEM coupling for cohesive particle simulations.


---

✅ **Summary of Advantages of PFS-CFDEM:**
- Enhanced IB solver with built-in HIT forcing.  
- More accurate and flexible interface field and void fraction computation.  
- Improved robustness in periodic and multi-particle IB interactions.  
- Backward-compatible with the original CFDEM workflow.
- Support for **cohesive particle dynamics** via van der Waals force modeling. 


> 💡 *These developments are part of the PFS research effort to improve fully resolved simulations of particle-laden flows, enabling more accurate and efficient large-scale simulations.*


---

## 📝 Additional notes

- ✅ Always `source cfdem.env` before compiling or running CFDEM.  
- ❌ Don’t mix compiler or MPI versions between build and runtime.  
- 🧹 Clean rebuild:
```bash
cfdemCleanCFDEM
cfdemCompCFDEMall
```
- 🧭 If you see `libmpi` or `GLIBCXX` errors, double-check:
  - Correct environment file
  - Matching GCC and MPI versions

---

## 📚 References

- [Official CFDEM® Manual](https://www.cfdem.com/media/CFDEM/docu/CFDEMcoupling_Manual.html)  
- [OpenFOAM 6](https://openfoam.org/)  
- [LIGGGHTS®](https://www.cfdem.com/)
