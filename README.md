# ROCm 7.2.1 Windows Support for RDNA2 (gfx1031 / RX 6700 XT)

> [!NOTE]
> **About This Project**: This repository is a community-driven effort to enable high-performance GPGPU and AI workloads on the **AMD Radeon RX 6700 XT (gfx1031)** using the ROCm 7.2.1 stack on Windows. By surgically patching the toolchain and applying linker-level overrides, we provide a path for RDNA2 users to run MIOpen, rocBLAS, and other core libraries that are currently "unofficial" in the standard Windows distribution.

## 🚀 Status: Successfully Built Components (v7.2.1-gfx1031)

The following components have been successfully compiled for `gfx1031` and verified to generate valid ISA. This distribution uses a **Standard Unified Layout** mirroring official ROCm installs.

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
| **rocWMMA** | Matrix Multi-Add | ✅ Optimized for RDNA2 |
| **hipCUB / rocThrust** | Parallel Algorithms | ✅ Verified |
| **hipBLAS / hipSOLVER** | HIP Wrappers | ✅ Verified |

---

## 📦 Binary Releases (Modular Install)

To comply with GitHub's asset limits and provide flexibility, the distribution is split into modular archives. **To create a full install, download all parts and extract them into the same root folder (e.g., `C:\ROCM\7.2.1`).**

1.  **[ROCM_gfx1031_Runtime_v7.2.1.zip]**: Essential DLLs, `amdgcn` device bitcode, and CMake modules. (Required for all users).
2.  **[ROCM_gfx1031_Compiler_v7.2.1_Part1.zip]** & **Part2.zip**: The full patched LLVM/Clang toolchain. (Required for developers).
3.  **[ROCM_gfx1031_SDK_v7.2.1.zip]**: Header files and `.lib` import libraries. (Required for building projects).

### Installation Steps:
1. Create a directory: `C:\ROCM\7.2.1`.
2. Extract all 4 ZIP files into this directory.
3. Add `C:\ROCM\7.2.1\bin` to your system `PATH`.
4. Set `HIP_PATH=C:\ROCM\7.2.1` and `ROCM_PATH=C:\ROCM\7.2.1`.

---

## 🛠️ Replication Instructions (TheRock Superbuild)

If you wish to compile from source using our patches:

### 1. Environment Setup
- Install **Visual Studio 2022** (with C++ and Windows SDK).
- Install **Python 3.10+**, **CMake**, and **Ninja**.
- Clone the [ROCm/TheRock](https://github.com/ROCm/TheRock) super-project.

### 2. Apply Patches
Overlay the `patches/` directory from this repository onto your ROCm source tree. Key patches include:
- `AMDGPU.td` & `GCNProcessors.td` (LLVM) -> Explicitly adds `gfx1031` support.
- `ROCMCheckTargetIds.cmake` -> Allows `gfx1031` as a valid build target.
- `host_alloc.cpp` (hipBLAS) -> Fixes Windows `/proc/meminfo` crashes.

### 3. Build Command
```powershell
python build_tools/build.py --amdgpu-targets gfx1031 --build-type Release MIOpen rocFFT rocSPARSE
```

---

## ⚠️ Limitations & Notes
- **Media Support**: `rocdecode` and `rocjpeg` are currently unsupported on Windows in the ROCm stack.
- **Linker Flags**: These builds use `/FORCE:UNRESOLVED` for specific test utilities to bypass missing `OpenBLAS` symbols. This does NOT affect the performance or correctness of the core AI stack.

---
*Created with ❤️ for the RDNA2 Community.*
