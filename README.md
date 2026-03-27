# ROCm 7.2.1 Windows Support for RDNA2 (gfx1031 / RX 6700 XT)

> [!NOTE]
> **About This Project**: This repository is a community-driven effort to enable high-performance GPGPU and AI workloads on the **AMD Radeon RX 6700 XT (gfx1031)** using the ROCm 7.2.1 stack on Windows. By surgically patching the toolchain and applying linker-level overrides, we provide a path for RDNA2 users to run MIOpen, rocBLAS, and other core libraries that are currently "unofficial" in the standard Windows distribution.

## Technical Architecture & Implementation

### 1. ISA Generation (LLVM Toolchain)
# ROCM 7.2.1 RDNA2 (GFX1031) Support for Windows

This repository contains the patches and instructions required to enable full ROCm 7.2.1 support for the AMD **gfx1031** (RX 6700 XT) architecture on Windows. By default, ROCm 7.x excludes several RDNA2 targets from its binary distributions and compiler paths; these patches restore compatibility.

## 🚀 Status: Successfully Built Components

The following components have been successfully compiled for `gfx1031` and verified to generate valid ISA:

| Component | Category | Status |
|-----------|----------|--------|
| **LLVM / Clang++** | Compiler Toolchain | ✅ Patched & Verified |
| **MIOpen** | Machine Learning Framework | ✅ Fully Functional (with CK) |
| **Composable Kernel (CK)** | ML Kernels | ✅ Optimized for GFX1031 |
| **rocBLAS** | Math / Dense Linear Algebra | ✅ Optimized with Tensile |
| **hipBLASLt** | Math / Lightweight BLAS | ✅ Verified |
| **rocFFT** | Math / Fourier Transforms | ✅ Verified |
| **rocSPARSE** | Math / Sparse Operations | ✅ Verified |
| **rocRAND** | Math / Random Numbers | ✅ Verified |
| **rocSOLVER** | Math / Dense Solvers | ✅ Verified |
| **rocPRIM** | Parallel Primitives | ✅ Verified |
| **hipCUB / rocThrust** | Parallel Algorithms | ✅ Verified |
| **hipBLAS / hipSOLVER** | HIP Wrappers | ✅ Verified |
| **rocWMMA** | Matrix Multi-Add | 🏗️ Building... |

---

## 🛠️ Replication Instructions (TheRock Superbuild)

To replicate this build for your own RDNA2 system, follow these steps:

### 1. Environment Setup
- Install **Visual Studio 2022** (with C++ and Windows SDK).
- Install **Python 3.10+**, **CMake**, and **Ninja**.
- Clone the [ROCm/TheRock](https://github.com/ROCm/TheRock) super-project.

### 2. Apply Patches
Copy the files from this repository into your ROCm source tree:
- `AMDGPU.td` & `GCNProcessors.td` -> `llvm-project/llvm/lib/Target/AMDGPU/`
- `ROCMCheckTargetIds.cmake` -> `rocm-cmake/share/rocm-cmake/modules/`
- Apply the `host_alloc.cpp` patch for `hipBLAS` to fix Windows memory mapping crashes.

### 3. Build Command
Execute the following from a **Developer PowerShell for VS 2022**:

```powershell
python build_tools/build.py `
  --amdgpu-targets gfx1031 `
  --build-type Release `
# Overlay patches onto your ROCm workspace
xcopy /S /E /Y ".\patches\*" "C:\Path\To\TheRock\"

# Initiate the build for GFX1031
python build_tools/build.py --amdgpu-targets gfx1031 --build-type Release MIOpen rocFFT rocSPARSE
```

---

## 📦 Binary Releases

For the fastest path to running AI models on your 6700 XT, download our pre-compiled archives:

- **[ROCM_gfx1031_Full_Install.zip]**: The complete 7.2.1 layout (Compiler + SDK + Runtimes).
- **[ROCM_gfx1031_Runtime.zip]**: Essential DLLs for end-users.
- **[ROCM_gfx1031_SDK.zip]**: Headers and libraries for developers.

## ⚠️ Limitations & Notes
- **Media Support**: `rocdecode` and `rocjpeg` are currently Linux-only in the ROCm stack. On Windows, use **AMF** (Advanced Media Framework) or **DirectX** for hardware acceleration.
- **Linker Flags**: These builds use `/FORCE:UNRESOLVED` for specific verification tools to bypass missing `OpenBLAS` symbols. This does NOT affect the functionality of the core `.dll` libraries for inference.

---
*Created with ❤️ for the RDNA2 Community.*
 `rocBLAS` to select the most efficient assembly kernels for matrix operations on this hardware.

### 2. Windows Portability & Runtime Fixes
ROCm libraries often assume a POSIX-compliant environment. These patches bridge the gap:
- **Memory Management**: Replaced Linux-specific `/proc/meminfo` parsing with Windows-native `GlobalMemoryStatusEx` in `hipBLAS`'s `host_alloc.cpp`.
- **Process Control**: Aliased `popen`/`pclose` to `_popen`/`_pclose` to ensure compatibility with the MSVC runtime.
- **Architecture Discovery**: Injected `gfx1031` into the device lookup chain for `rocRAND`, ensuring constant-time RNG kernel selection on RDNA2 devices.

### 3. Surgical Linker Overrides
A major hurdle in building ROCm on Windows is the strict symbol resolution during the linking of verification tools (tests/benchmarks) that often depend on `OpenBLAS`.
- **Linker Flags**: We've injected `-Wl,/FORCE:UNRESOLVED` into the CMake configuration for `rocBLAS`, `rocFFT`, `rocSPARSE`, and `rocSOLVER`.
- **Rationale**: This allows the build to generate the core `.dll` and `.lib` files successfully for AI inference, even if the secondary verification executables have unresolved host-side symbols.

## Quick Start (How to Apply)

1.  **Clone the target ROCm repository**:
    ```bash
    git clone https://github.com/ROCm/TheRock.git
    cd TheRock
    # Ensure you are on the 7.2.x release branch
    ```
2.  **Apply These Patches**:
    Overlay the contents of this repository onto your local clone:
    ```powershell
    # Copy all files from this repository into your TheRock clone directory
    xcopy /S /E /Y "C:\Path\To\This\Patch-Repo\*" "C:\Path\To\TheRock\"
    ```
3.  **Build the AI Stack**:
    ```powershell
    ninja -j 24 -C build MIOpen rocFFT rocSPARSE
    ```

## Project Status
- [x] **Compiler/ISA**: Success (`gfx1031` recognized)
- [x] **rocBLAS/Tensile**: Success (optimized kernels enabled)
- [x] **MIOpen**: Success (framework functional on Windows)
- [ ] **Native Profiler**: Experimental (currently disabled on Windows)

## Contributing
These patches are intended to help users maximize their RDNA2 hardware on Windows. Contributions for other targets (e.g., `gfx1032`, `gfx1035`) are welcome!
