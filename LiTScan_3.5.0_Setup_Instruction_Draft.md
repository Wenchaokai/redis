# LiTScan 3.5.0 Setup Instruction

> **Draft for team review**  
> **Platform:** Windows  
> **Purpose:** Provide a reproducible setup procedure for LiTScan customer deployment, C++ development, and Python/analysis-tool development.

---

## 0. Before You Start

Use the section that matches your task:

- **Part I — Customer Installation**: install LiTScan on an instrument/customer PC.
- **Part II — LiTScan C++ Development Environment**: build and debug the LiTScan main application.
- **Part III — LiTScan Third-party / Analysis Environment**: develop and package AIBOT, Batch, Destrip, FFC, SR, Amira-related tools, etc.

Do not mix dependencies from different LiTScan releases unless explicitly validated.

---

# 1. Validated Version Matrix

The table below should be maintained as the single source of truth for each LiTScan release.

| Component | LiTScan 3.5.0 validated value | Notes |
|---|---|---|
| Visual Studio | 2019 | C++ workload required |
| CMake | Release-compatible version | Must support VS2019 generator |
| CUDA | 12.1 | CUDA-enabled build |
| Qt | 6.3.1 / `msvc2019_64` | Keep compiler ABI consistent |
| VTK | Internal repository, branch `litscan_vtk_3.3.2` | **Do not replace with standard upstream VTK** |
| LiTScan source | branch `litscan_int_3.5.0` | Main application |
| LiTScan Core | branch `litscan_int_3.5.0` | MMCore / hardware layer |
| Python main environment | 3.11.9 | Main development/test/package environment |
| Python compatibility | 3.7.9 | AIBOT compatibility |
| Python legacy compatibility | 3.6.9 | Amira compatibility |
| CUDA PyTorch | `torch 2.5.1+cu121` | Current validated Python 3.11 CUDA build |

> **Important:** Internal/customized dependencies must always be identified explicitly in this table.  
> For LiTScan 3.5.0, VTK is an internal customized dependency and should not be replaced with a standard upstream VTK package.

---

# Part I — Customer Installation

## 2. Pre-installation Checklist

### 2.1 BIOS — Power Management

Confirm the following values:

| Item | Setting |
|---|---|
| Runtime Power Management | OFF |
| Hardware P-States | OFF |
| Energy/Performance Bias Control | OS Control EPB |
| Idle Power Savings | Normal with Enhanced Halt State disabled |
| PCI Express Power Management | OFF |

### 2.2 BIOS — Performance

| Item | Setting |
|---|---|
| Turbo Mode | ON |
| Intel Hyper-Threading Technology | ON |
| Active CPU Cores Per Processor | All |
| Sub-NUMA Clustering | OFF |
| Isoc Mode | Disable |
| Performance Control | Performance Mode |

### 2.3 Windows

Disable hibernation according to the validated customer-machine configuration.

---

## 3. Software Installation

1. Install software in the order defined by the **release-specific SW Installation Checklist**.
2. Identify the installed stage model before installing the stage driver:
   - **SmarAct stage**: use the SmarAct installation items defined in the release checklist.
   - **PI stage**: use the PI installation item defined in the release checklist.
3. When installing **DCAM-API**, select:
   ```text
   Active Silicon FireBird
   ```
4. Install the corresponding LiTScan release package.

> Do not reuse a hardware-driver checklist from another LiTScan release unless it has been validated for the target machine.

---

## 4. Post-installation Configuration

Update the hardware configuration file with machine-specific values, including as applicable:

```text
stage limits
stage type
camera type
move angle
NI-DAQ device
COM ports
relay configuration
hardware serial numbers
```

Use the validated configuration template for the specific instrument.

### 4.1 Customer Installation Validation

Before handover:

```text
[ ] LiTScan starts
[ ] Correct hardware configuration file is loaded
[ ] Camera is detected
[ ] Stage is detected
[ ] Required COM / DAQ devices are detected
[ ] No license error
[ ] Basic acquisition/movement smoke test passes
```

---

# Part II — LiTScan C++ Development Environment

## 5. Recommended Workspace

Use one root directory for the release:

```text
D:\litscan_350\
├── litscan\
├── litscan-core\
├── vtk\
└── third-party SDK/dependency folders
```

Keeping dependencies under one release root makes environment variables and troubleshooting easier.

---

## 6. Required Development Tools

Install/prepare:

```text
Visual Studio 2019
  └── Desktop development with C++

CMake
CUDA Toolkit 12.1
Qt 6.3.1 (msvc2019_64)
Internal customized VTK
OpenCV
Boost
OpenSSL
OME dependencies
TIFF dependencies
other release-specific SDKs
```

### 6.1 Visual Studio

Verify that Visual Studio 2019 includes the C++ compiler and build tools.

A quick check should confirm that the VS2019 C++ toolchain is available before running CMake.

### 6.2 CUDA

Verify:

```bat
nvcc --version
```

The expected CUDA toolkit for this release is:

```text
CUDA 12.1
```

CUDA should be installed with Visual Studio integration available.

---

## 7. Source Repositories and Branches

### 7.1 LiTScan

Repository directory:

```text
D:\litscan_350\litscan
```

Expected branch:

```text
litscan_int_3.5.0
```

### 7.2 LiTScan Core

Repository directory:

```text
D:\litscan_350\litscan-core
```

Expected branch:

```text
litscan_int_3.5.0
```

### 7.3 VTK

Repository directory:

```text
D:\litscan_350\vtk
```

Expected internal branch:

```text
litscan_vtk_3.3.2
```

> **Required:** use the internal VTK repository/branch.  
> Standard upstream VTK can be API-incompatible with LiTScan.

---

## 8. Development Environment Variables

Use release-specific paths. Typical variables include:

```text
QTDIR
VTKVHOME
BOOSTPATH
MMCOREINCLUDE
OMEXMLHOME
CUDA_PATH
LITONE_ENV
LITONE_DLL
PATH
```

Example structure:

```text
QTDIR=<root>\6.3.1\msvc2019_64
VTKVHOME=<root>\vtk
BOOSTPATH=<root>\boost_1_82_0
MMCOREINCLUDE=<root>\litscan-core\micromanager\trunk\MMCore
OMEXMLHOME=<root>\ome-xml
CUDA_PATH=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.1
```

Do not copy another developer's absolute paths without updating them for the local workstation.

---

## 9. Configure LiTScan with CMake

From the LiTScan source directory:

```bat
cd /d D:\litscan_350\litscan
mkdir build
cd build
```

Configure using Visual Studio 2019 x64:

```bat
cmake .. -G "Visual Studio 16 2019" -A x64
```

### Expected result

CMake should finish without unresolved required dependencies.

If CMake reports a missing dependency, stop and correct the dependency/path before building.

---

## 10. Build LiTScan

From the build directory:

```bat
cmake --build . --config Release -j 8
```

Expected executable:

```text
D:\litscan_350\litscan\bin\Release\litscan.exe
```

### 10.1 Alternative Visual Studio Workflow

After CMake generates the Visual Studio solution, developers may also:

```text
Open generated .sln
→ select x64 / Release
→ Build Solution
```

This is equivalent to using the Visual Studio/MSBuild toolchain through `cmake --build`.

---

## 11. Runtime Resources

A successful C++ build does not automatically guarantee a complete runtime package.

Verify the runtime tree contains the release-specific resources required by LiTScan.

Typical required resources include:

```text
conf/
conf/key/litscan.lic
MMCore / device configuration
luts/
required DLLs
other runtime configuration/resources
```

For the validated 3.5.0 setup, the hardware configuration includes:

```text
MMConfig_demo-xl.cfg
```

and the LUT resources include:

```text
luts\lut_csv
```

Run the application from its intended runtime directory so relative paths resolve correctly.

---

## 12. LiTScan Runtime Validation

Before considering the developer setup complete:

```text
[ ] CMake configuration completed successfully
[ ] Release build completed successfully
[ ] litscan.exe exists
[ ] required DLLs are present
[ ] conf/key/litscan.lic exists
[ ] hardware/MMCore configuration exists
[ ] LUT resources exist
[ ] LiTScan starts without immediate runtime error
[ ] required edition can be opened
```

---

# Part III — LiTScan Third-party / Analysis Environment

## 13. Repository Overview

Expected repository:

```text
litscan-3rdparty/
├── analysis/
│   ├── aibot/
│   ├── amira/
│   ├── destrip/
│   ├── ffc/
│   ├── imaris/
│   └── sr/
├── batch/
├── common/
├── conf/
├── requirements/
├── test/
├── setup_env311.bat
├── setup_env37.bat
├── setup_env36.bat
├── build_env311.bat
├── build_env37.bat
├── build_env36.bat
└── build.bat
```

Python environments:

```text
Python 3.11 → main development / testing / packaging
Python 3.7  → AIBOT compatibility
Python 3.6  → legacy Amira compatibility
```

For a full `output/` package, all three environments are required.

---

## 14. Install and Initialize Conda

Install Miniconda or Anaconda.

Verify:

```bat
conda --version
```

Initialize the shell used for development.

CMD:

```bat
conda init cmd.exe
```

PowerShell:

```powershell
conda init powershell
```

Close and reopen the terminal after initialization.

---

## 15. Configure Conda Package Source

### Preferred

Use the company-maintained internal Conda mirror/channel when available.

### Current validated public-mirror fallback

The following channel combination was validated for the required Python versions:

```yaml
channel_priority: flexible
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
show_channel_urls: true
```

Avoid relying on obsolete channels that return HTTP errors or do not contain the required Python release.

Verify availability before creating an environment, for example:

```bat
conda search python=3.11.9 --override-channels -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
```

---

## 16. Proxy / Network Configuration

If the workstation requires a proxy, configure it before running pip-based setup scripts.

Example:

```bat
set "HTTP_PROXY=http://<proxy-host>:<port>"
set "HTTPS_PROXY=http://<proxy-host>:<port>"
set "PIP_PROXY=http://<proxy-host>:<port>"
```

Using environment variables is preferred over supplying `--proxy` only to the top-level pip command because pip build subprocesses may need the same network configuration.

If the company provides an internal Conda/PyPI mirror, use the internal source instead of a developer-specific proxy whenever possible.

---

## 17. Create Python 3.11 Environment

From the repository root:

```bat
setup_env311.bat
```

Expected environment:

```text
venv\LitscanAnalysisEnv311
```

Verify:

```bat
.\venv\LitscanAnalysisEnv311\python.exe --version
```

Expected:

```text
Python 3.11.9
```

Then activate it:

```bat
conda activate D:\litscan_350\litscan-3rdparty\venv\LitscanAnalysisEnv311
```

Validate core dependencies:

```bat
python -c "import PyQt6; import tifffile; import imagecodecs; print('ok')"
python -c "import torch; print(torch.__version__); print(torch.version.cuda); print(torch.cuda.is_available())"
python -m pip check
```

> Do not use the final `Setup completed successfully!` message as the only validation.  
> Always run the explicit checks above.

---

## 18. Create Python 3.7 Environment

Run:

```bat
setup_env37.bat
```

Expected environment:

```text
venv\LitscanAnalysisEnv37
```

Verify after activation:

```bat
conda activate D:\litscan_350\litscan-3rdparty\venv\LitscanAnalysisEnv37
python --version
python -c "import ssl; print(ssl.OPENSSL_VERSION)"
python -c "import napari; print(napari.__version__)"
python -m pip check
```

Expected Python:

```text
Python 3.7.9
```

Expected legacy napari version from the current requirements:

```text
0.3.8
```

### Important

For network/package operations in old Conda environments, activate the environment first so its runtime DLL paths are available.

---

## 19. Create Python 3.6 Environment

Run:

```bat
setup_env36.bat
```

Expected environment:

```text
venv\LitscanAnalysisEnv36
```

Verify:

```bat
conda activate D:\litscan_350\litscan-3rdparty\venv\LitscanAnalysisEnv36
python --version
python -m pip check
```

Expected:

```text
Python 3.6.9
No broken requirements found.
```

---

## 20. External Application Paths

Some analysis functions integrate with external applications.

Current configuration keys:

```ini
IMARIS_HOME=<Imaris root>
AMIRA_HOME=<Amira root>
IMAGEJ_PATH=<Fiji/ImageJ executable>
BRAINREG_PATH=<brainreg environment>
```

These are **feature-specific** dependencies.

Do not require every new developer to install all of them unless their development/testing scope needs those integrations.

If these tools are installed, update:

```text
paths.conf
```

and run the repository's environment-variable setup process as documented.

---

## 21. License Files

AIBOT expects:

```text
conf\key\aibot.lic
```

LiTScan main application uses its own LiTScan license file in its runtime configuration.

Obtain valid license files through the internal release/license process.

Do not rename or modify license contents.

---

## 22. Development Launch — AIBOT

From the repository root, activate Python 3.11:

```bat
conda activate D:\litscan_350\litscan-3rdparty\venv\LitscanAnalysisEnv311
```

For the current source layout, ensure the AIBOT source directory is importable:

```bat
set "PYTHONPATH=%CD%\analysis\aibot;%PYTHONPATH%"
```

Launch:

```bat
python -m analysis.aibot
```

Expected result:

```text
AIBOT Qt GUI opens
No immediate module-import error
No license error
```

---

## 23. Full Third-party Build

For a complete package, first confirm:

```text
[ ] Python 3.11 environment valid
[ ] Python 3.7 environment valid
[ ] Python 3.6 environment valid
[ ] required licenses/resources available
```

Then deactivate any manually active environment and run from repository root:

```bat
build.bat
```

The current build sequence is:

```text
build_env36.bat
      ↓
build_env37.bat
      ↓
build_env311.bat
      ↓
collect output/
```

---

## 24. Validate Full Build Output

Expected:

```text
output/
├── litscan_aibot_py311.exe
├── litscan_batch.exe
├── litscan_destrip.exe
├── litscan_ffc.exe
├── litscan_sr.exe
├── libs/
├── conf/
├── amira/
└── fiji_plugins/
```

Check:

```bat
dir /b output
```

Required executables:

```text
litscan_aibot_py311.exe
litscan_batch.exe
litscan_destrip.exe
litscan_ffc.exe
litscan_sr.exe
```

> The current batch scripts may print a success message even if an intermediate command failed.  
> Always validate the actual output files.

---

## 25. Runtime Smoke Test

From:

```text
D:\litscan_350\litscan-3rdparty\output
```

start:

```bat
litscan_aibot_py311.exe
```

Expected:

```text
GUI opens successfully
No missing DLL/module error
No immediate license error
```

Also validate the remaining packaged tools as required:

```text
litscan_batch.exe
litscan_destrip.exe
litscan_ffc.exe
litscan_sr.exe
```

### Known packaging note

The current full-build collection step may report:

```text
File not found - libs
```

for:

```text
batch\bin\BatchBot\libs
```

The current Batch build is packaged as a PyInstaller executable and may not generate a separate `BatchBot\libs` directory.

Treat this as a packaging-script warning only if:

```text
output\litscan_batch.exe
```

exists and passes its runtime validation.

---

# 26. Common Troubleshooting

## 26.1 Conda cannot find the requested Python version

Check the configured channels:

```bat
conda config --show-sources
```

Search the required version explicitly:

```bat
conda search python=<version> --override-channels -c <validated-channel>
```

Do not immediately change the required Python version.

---

## 26.2 pip reports `ProxyError`

Check whether the machine uses a system/company proxy.

If a proxy is required, configure:

```bat
set "HTTP_PROXY=http://<proxy-host>:<port>"
set "HTTPS_PROXY=http://<proxy-host>:<port>"
set "PIP_PROXY=http://<proxy-host>:<port>"
```

Then retry from the activated environment.

---

## 26.3 Python reports SSL module unavailable

Activate the Conda environment first:

```bat
conda activate <environment-path>
```

Then verify:

```bat
python -c "import ssl; print(ssl.OPENSSL_VERSION)"
```

Do not diagnose PyPI/package availability until SSL works.

---

## 26.4 CMake compiles against an incompatible VTK

Confirm the configured VTK is the internal validated repository/branch:

```text
litscan_vtk_3.3.2
```

Do not substitute standard upstream VTK for LiTScan 3.5.0.

---

## 26.5 Build script says success but output is incomplete

Do not rely on the final console message.

Validate:

```text
expected executable files
libs/
conf/
required runtime resources
runtime startup
```

A build is complete only after the smoke test passes.

---

# 27. Completion Checklist

## Customer installation

```text
[ ] BIOS/Windows settings verified
[ ] correct hardware drivers installed
[ ] LiTScan installed
[ ] machine-specific config applied
[ ] hardware detected
[ ] runtime smoke test passed
```

## C++ development

```text
[ ] VS2019 C++ toolchain available
[ ] CUDA 12.1 available
[ ] correct LiTScan branch
[ ] correct LiTScan Core branch
[ ] internal VTK branch used
[ ] CMake configure passed
[ ] Release build passed
[ ] runtime resources installed
[ ] LiTScan starts successfully
```

## Third-party / analysis

```text
[ ] Python 3.11.9 environment passed
[ ] Python 3.7.9 environment passed
[ ] Python 3.6.9 environment passed
[ ] pip check passed in required environments
[ ] AIBOT development launch passed
[ ] build.bat completed
[ ] five expected executables exist
[ ] output runtime resources exist
[ ] packaged AIBOT smoke test passed
```

---

# 28. Maintenance Rules for Future Releases

When updating this instruction:

1. Update the **Version Matrix first**.
2. Mark internal/customized dependencies explicitly.
3. Do not change required versions only to work around package-source problems.
4. Prefer internal mirrors / cached wheels / locked environments for reproducibility.
5. Add validation commands for every setup stage.
6. Treat build success and runtime success as separate checks.
7. Keep commands copy-pasteable from a clean workstation.
8. Remove obsolete package sources and dead links when a new release is validated.

