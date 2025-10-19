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
> ./configure --disable-multilib --prefix=$HOME/CFDEM-PFS/gcc/opt --enable-languages=c,c++,fortran
>
> ## make & install
> make -j 12
> make install
> ```

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

./configure --prefix="$HOME/CFDEM-PFS/openmpi/opt"
make -j 12 all install
```

Test:
```bash
$HOME/CFDEM-PFS/openmpi/opt/bin/mpirun --version
```

Expected:
```
mpirun (Open MPI) 3.1.6
```

---

## 📦 3. Clone the repositories

```bash
cd "$HOME/CFDEM-PFS"
mkdir CFDEM
cd CFDEM
git clone git@github.com:PFS-CFDEM/CFDEMcoupling-PUBLIC.git

mkdir LIGGGHTS
cd LIGGGHTS
git clone git@github.com:PFS-CFDEM/LIGGGHTS-PUBLIC.git
git clone git@github.com:PFS-CFDEM/LPP.git lpp

cd "$HOME/CFDEM-PFS"
git clone git@github.com:PFS-CFDEM/OpenFOAM.git
```

---

## 🌱 4. Environment setup 

```bash
emacs "$HOME/CFDEM-PFS/cfdem.env"
```

Paste the following content (NOTE: it is assumed that the name of the main folder is CFDEM-PFS, see LOCDIR below):

```bash
# === CFDEM environment file (Option B: no ~/.bashrc changes) ===

# --- MPI used to build/run CFDEM ---
export PATH="$HOME/CFDEM-PFS/openmpi/opt/bin:$PATH"
export LD_LIBRARY_PATH="$HOME/CFDEM-PFS/openmpi/opt/lib:$LD_LIBRARY_PATH"

# --- if GCC 8.5.0 was manually compiled ---
#export PATH="$HOME/CFDEM-PFS/gcc/opt/bin:$PATH"
#export LD_LIBRARY_PATH="$HOME/CFDEM-PFS/gcc/opt/lib64:$LD_LIBRARY_PATH"

# --- OpenFOAM settings ---
export WM_NCOMPPROCS=12
export WM_MPLIB=SYSTEMMPI

# --- OpenFOAM (quiet normal output; errors still visible) ---
source "$HOME/CFDEM-PFS/OpenFOAM/OpenFOAM-6/etc/bashrc" >/dev/null

# --- CFDEM paths ---
export LOCDIR="CFDEM-PFS"
export CFDEM_VERSION="PUBLIC"
export CFDEM_PROJECT_DIR="$HOME/$LOCDIR/CFDEMcoupling-PUBLIC-$WM_PROJECT_VERSION"
export CFDEM_PROJECT_USER_DIR="$HOME/$LOCDIR/$LOGNAME-$CFDEM_VERSION-$WM_PROJECT_VERSION"

export CFDEM_bashrc="$CFDEM_PROJECT_DIR/src/lagrangian/cfdemParticle/etc/bashrc"
export CFDEM_LIGGGHTS_SRC_DIR="$HOME/$LOCDIR/LIGGGHTS-PUBLIC/src"
export CFDEM_LIGGGHTS_MAKEFILE_NAME="auto"
export CFDEM_LPP_DIR="$HOME/$LOCDIR/LIGGGHTS/lpp/src"

# --- CFDEM init (quiet banner; errors still visible) ---
. "$CFDEM_bashrc" >/dev/null
```

Activate the environment:
```bash
source "$HOME/CFDEM-PFS/cfdem.env"
```

Optional alias in `~/.bashrc`:
```bash
alias cfdemenv='source $HOME/CFDEM-PFS/cfdem.env'
```

---

## 🧱 5. Compilation

```bash
source "$HOME/CFDEM-PFS/cfdem.env"

cd $WM_PROJECT_DIR
./Allwmake

cd CFDEM_PROJECT_DIR
cfdemCompCFDEMall
```

This compiles:
- OpenFOAM 6
- LIGGGHTS® executable & shared library  
- CFDEM® coupling libraries  
- CFDEM® solvers & utilities

Logs:
```
$CFDEM_PROJECT_DIR/src/lagrangian/cfdemParticle/etc/log
```

---

## 🧪 6. Running cases

Run directory:
```
$CFDEM_PROJECT_USER_DIR/run
```

Run all tutorials:
```bash
cfdemTestTUT
```

Run single tutorial:
```bash
cd "$CFDEM_PROJECT_USER_DIR/run/<YourCase>"
./Allrun.sh
```

Pure LIGGGHTS®:
```bash
cfdemLiggghts inputScriptName
cfdemLiggghtsPar inputScriptName nProcs
```

---

## 🧭 7. SLURM job script example

```bash
nano cfdem_job.slurm
```

Paste:
```bash
#!/bin/bash
#SBATCH -J cfdem_job
#SBATCH -n 64
#SBATCH -t 12:00:00
#SBATCH -p compute
#SBATCH -o slurm-%j.out

set -euo pipefail

# Activate CFDEM environment (quiet)
source "$HOME/CFDEM-PFS/cfdem.env" >/dev/null

# Run solver
cd "$SLURM_SUBMIT_DIR"
mpirun -np 64 cfdemSolverIB_MT -case "$PWD"
```

Submit:
```bash
sbatch cfdem_job.slurm
```

---

## 📝 Additional notes

- ✅ Always `source cfdem.env` before compiling or running CFDEM.  
- ❌ Don’t mix compiler or MPI versions between build and runtime.  
- 🧹 Clean rebuild:
```bash
cfdemCleanAll
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
