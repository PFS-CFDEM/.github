# HSU-PFS-Lab / CFDEMcoupling Setup

This repository hosts projects related to CFDEM®coupling, a coupled CFD-DEM framework combining OpenFOAM® and LIGGGHTS®.  

Below is the **Installation** section as per the CFDEMcoupling 3.8.1 manual with the initial part specific for the HSUper of the Helmut-Schmidt University Hamburg. 

---

## Installation

This section describes how to download the CFDEM®project repositories and compile LIGGGHTS® and CFDEM®coupling.  

### Procedure (Short Summary)
1. Install and load required modules on HSUper (gcc@4.9.1, openmpi@3.1.6, flex)
2. Install git  
3. Download CFDEM®project software  
4. Setup prerequisites  
5. Setup and compile OpenFOAM®  
6. Set environment variables and paths  
7. Compile LIGGGHTS® and CFDEM®coupling  
8. Additional information  
9. Run your own cases  

---

### Detailed Installation Guide

#### Prerequisites: Install Required Modules

The following module versions have been verified to work with CFDEM®. Note that newer versions of GCC and OpenMPI may cause compatibility issues.

```bash
# Load Spack module manager
module load USER-SPACK/0.22.1

# Install required packages
spack install openmpi@3.1.6 %gcc@8.5.0
spack install gcc@4.9.1 %gcc@8.5.0
```

Load Required Modules:

```bash
​module load flex
module load USER-SPACK/0.22.1
spack load openmpi@3.1.6
spack load gcc@4.9.1
```

> **Compatibility Note:** The specified versions (gcc@4.9.1, openmpi@3.1.6) have been thoroughly tested and confirmed to work with CFDEM®. Other version combinations may result in compilation errors.

---

#### Install git

Git is preinstalled on HSUper. However, for completeness, this step is  kept here. Git allows you to update the source easily via `git pull`.  
On Debian-based systems:

```bash
sudo apt-get install git-core
```

On other systems you may use:

```bash
sudo zypper install git-core
sudo yum install git
```

> **Note:** If port 9418 is blocked, use `https://` instead of `git://` when cloning repositories.

---

#### Download CFDEM®project Software

Example commands (assumes `$HOME`):

```bash
cd $HOME
mkdir CFDEM
cd CFDEM
git clone git://github.com/HSU-PFS-Lab/CFDEMcoupling-PUBLIC.git

cd $HOME
mkdir LIGGGHTS
cd LIGGGHTS
git clone git://github.com/HSU-PFS-Lab/LIGGGHTS-PUBLIC.git
git clone git://github.com/HSU-PFS-Lab/LPP.git lpp


cd $HOME
mkdir OpenFOAM
cd OpenFOAM
git clone git://github.com/HSU-PFS-Lab/OpenFOAM/OpenFOAM-5.x
git clone git://github.com/HSU-PFS-Lab/OpenFOAM/ThirdParty-5.x
```

If you do not have git, you may download ZIP archives from GitHub and unzip them.  
GitHub often appends `-master` to folder names—rename them as:

```bash
cd $HOME/CFDEM
mv CFDEMcoupling-PUBLIC-master CFDEMcoupling-PUBLIC

cd $HOME/LIGGGHTS
mv LIGGGHTS-PUBLIC-master LIGGGHTS-PUBLIC
mv LPP-master lpp
```


#### Setup Prerequisites

On Ubuntu (16.04+), for example:

```bash
sudo apt-get install build-essential flex bison cmake \
     zlib1g-dev libboost-system-dev libboost-thread-dev \
     libopenmpi-dev openmpi-bin gnuplot libreadline-dev \
     libncurses-dev libxt-dev libscotch-dev libptscotch-dev
```

CFDEM also recommends using VTK (for output to VTK format) — minimum version 6.3, recommended 8.0.1:

```bash
sudo apt-get install libvtk7-dev
```

Also, for post-processing, install:

```bash
sudo apt-get install python-numpy
```

If you compile VTK yourself, ensure MPI support and X11 libraries are enabled.

---

#### Setup and Compile OpenFOAM®

Follow standard OpenFOAM compilation instructions, with a few CFDEM-specific settings:

- Ensure `WM_LABEL_SIZE=32` is used (the standard setting).  
- In your `~/.bashrc`, add:

  ```bash
  export WM_NCOMPPROCS=<NofProcs>
  source $HOME/OpenFOAM/OpenFOAM-5.x/etc/bashrc
  ```

- Then:

  ```bash
  source ~/.bashrc
  cd $WM_PROJECT_DIR
  foamSystemCheck
  ./Allwmake
  ```

Additional hints are available from the OpenFOAM project.

---

#### Set Environment Variables and Paths

It’s common to tag your CFDEMcoupling folder with the OpenFOAM version, e.g.:

```bash
cd $HOME/CFDEM
mv CFDEMcoupling-PUBLIC CFDEMcoupling-PUBLIC-$WM_PROJECT_VERSION
```

In your `~/.bashrc` (or `~/.cshrc`), include:

```bash
# ================================================================  
# — source CFDEM environment variables  
export CFDEM_VERSION=PUBLIC  
export CFDEM_PROJECT_DIR=$HOME/CFDEM/CFDEMcoupling-$CFDEM_VERSION-$WM_PROJECT_VERSION  
export CFDEM_PROJECT_USER_DIR=$HOME/CFDEM/$LOGNAME-$CFDEM_VERSION-$WM_PROJECT_VERSION  
export CFDEM_bashrc=$CFDEM_PROJECT_DIR/src/lagrangian/cfdemParticle/etc/bashrc  
export CFDEM_LIGGGHTS_SRC_DIR=$HOME/LIGGGHTS/LIGGGHTS-PUBLIC/src  
export CFDEM_LIGGGHTS_MAKEFILE_NAME=auto  
export CFDEM_LPP_DIR=$HOME/LIGGGHTS/lpp/src  
. $CFDEM_bashrc  
# ================================================================
```

You may insert an **EXTENDED** block before `. $CFDEM_bashrc` for further customization.  
Then load the environment:

```bash
source ~/.bashrc
cfdemSysTest
```

This will set up useful aliases and environment variables (e.g. `cfdemEtc`).

---

#### Compile LIGGGHTS® and CFDEM®coupling

In a fresh terminal, run:

```bash
cfdemCompCFDEMall
```

This command compiles:

- LIGGGHTS executable  
- LIGGGHTS as a shared library  
- CFDEM coupling libraries  
- CFDEM solvers & utilities  

If there are build errors, it will stop.  
Note: if you have previously compiled LIGGGHTS, it needs to have been built as a *shared library* via the `cfdemCompLIG` command.

You can compile subsets of the code with:

```bash
cfdemCompLIG
cfdemCompCFDEMsrc
cfdemCompCFDEMsol
cfdemCompCFDEMuti
```

Build logs are in:

```
$CFDEM_SRC_DIR/lagrangian/cfdemParticle/etc/log
```

---

#### Run Your Own Cases

Your runnable cases should go into:

```text
$CFDEM_PROJECT_USER_DIR/run
```

You can copy tutorial cases there and adapt them.  
Note: changes in the `tutorials/` directory may be lost on updates.

To run all tutorial cases:

```bash
cfdemTestTUT
```

Or run each tutorial manually via its `Allrun.sh`.

For pure LIGGGHTS cases, use:

```bash
cfdemLiggghts inputScriptName
cfdemLiggghtsPar inputScriptName nOfProcs
```

You may also link the LIGGGHTS executable into `/usr/local/bin`.

---

#### Backwards Compatibility & Version Notes

CFDEM®coupling supports a single OpenFOAM version at a time.  
Supported OpenFOAM and LIGGGHTS versions are listed in:

```
src/lagrangian/cfdemParticle/cfdTools/versionInfo.H
```

If you wish to try alternative versions, you can edit

```
src/lagrangian/cfdemParticle/etc/OFversion/OFversion.H
```

—but not all functionality may work for non-supported versions.

---

### Additional Notes

- **VTK compilation:** If you need to build VTK manually, version 8.0.1 is recommended. Enable MPI, X11, and other dependencies; use `ccmake` for configuration.  
- **Makefile auto in LIGGGHTS:** You can set `AUTOINSTALL_VTK=ON` to download & compile VTK automatically.  
- **MPI issues:** If compiling on a cluster or with custom MPI, define in your OpenFOAM bashrc:

  ```bash
  export WM_MPLIB=SYSTEMMPI
  ```

  You may also set:

  ```bash
  export MPI_ROOT=<path>
  export MPI_ARCH_PATH=$MPI_ROOT
  export MPI_ARCH_FLAGS="-DMPICH_SKIP_MPICXX"
  export MPI_ARCH_INC="-I$MPI_ARCH_PATH/include"
  export MPI_ARCH_LIBS='-L$(MPI_ARCH_PATH)/lib -lmpich -lmpichcxx -lmpl -lopa -lrt'
  ```

- **LIGGGHTS shared library linking:** CFDEM coupling uses a symbolic link from `CFDEM_LIB_DIR` to the compiled LIGGGHTS library. This link is created during the coupling compilation phase.  
- **Debug builds:** To build in debug mode, set `WM_COMPILE_OPTION=Debug` in your OpenFOAM bashrc and rebuild. Ensure LIGGGHTS is compiled with `-O0 -g` or `-O2 -g` if not using the auto Makefile.

---

## License & Contact

CFDEM®coupling is open-source under the **GNU Public License (GPL)**.  
If you run into issues or have feedback, you may contact the developers or consult the CFDEM forum at [www.cfdem.com](https://www.cfdem.com).  
