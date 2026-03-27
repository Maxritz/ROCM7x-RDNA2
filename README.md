# ROCm 7.2.1 Windows Support for RDNA2 (gfx1031 / RX 6700 XT)

> [!NOTE]
> **About This Project**: This repository is a community-driven effort to enable high-performance GPGPU and AI workloads on the **AMD Radeon RX 6700 XT (gfx1031)** using the ROCm 7.2.1 stack on Windows. By surgically patching the toolchain and applying linker-level overrides, we provide a path for RDNA2 users to run MIOpen, rocBLAS, and other core libraries that are currently "unofficial" in the standard Windows distribution.

## 🚀 Status: Successfully Built Components (v7.2.1-gfx1031)

This distribution provides a **Standard Unified Layout** mirroring official ROCm installs. The following components are fully functional and optimized for `gfx1031`:

| Component | Category | Status |
|-----------|----------|--------|
| **LLVM / Clang++** | Patcher Compiler (ISA v10.3.1) | ✅ Patched & Verified |
| **MIOpen** | Deep Learning Framework / Operators | ✅ Fully Functional (with CK) |
| **Composable Kernel** | Templated ML Kernels (Backends) | ✅ Optimized for GFX1031 |
| **rocBLAS** | Dense Linear Algebra (Tensile Assembly) | ✅ Optimized Assembly Kernels |
| **hipBLASLt** | Lightweight Gemm / Sparsity Support | ✅ Verified |
| **rocFFT** | Fast Fourier Transforms | ✅ Verified |
| **rocSPARSE** | Sparse Linear Algebra | ✅ Verified |
| **rocRAND** | PRNG / QRNG Generators | ✅ Verified |
| **rocSOLVER** | LAPACK-style Dense Solvers | ✅ Verified |
| **rocWMMA** | Matrix Multiply-Add (RDNA2 optimized) | ✅ Verified |
| **hipCUB / rocThrust**| Parallel Algorithms / Primitives | ✅ Verified |
| **hipBLAS/hipSOLVER**| High-level HIP API Wrappers | ✅ Verified |

---

## 📦 Binary Distributions (What's Published)

To comply with GitHub's asset limits and provide a clear installation path, the release is split into **4 modular ZIP files**. Extract all 4 into the same directory to create a complete ROCm environment.

### 1. [ROCM_gfx1031_Runtime_v7.2.1.zip] (~282MB)
*   **Purpose**: Essential runtime components for users and developers.
*   **Key Contents**: 
    - `bin/*.dll`: Core runtimes (`amdhip64.dll`, `rocblas.dll`, `MIOpen.dll`, `rocfft.dll`, `rocsparse.dll`, `rocwmma.dll`, `rocrand.dll`, etc.).
    - `amdgcn/bitcode/`: Device-side bitcode libraries (`hip.bc`, `ockl.bc`, `ocml.bc`) required for RDNA2 kernel execution.
    - `share/` & `cmake/`: Root configuration modules for project discovery.

### 2. [ROCM_gfx1031_SDK_v7.2.1.zip] (~37MB)
*   **Purpose**: Development headers and linking artifacts.
*   **Key Contents**:
    - `include/`: Full C++ header stack for the math and AI libraries.
    - `lib/*.lib`: MSVC-compatible import libraries for all ROCm components.

### 3. [ROCM_gfx1031_Compiler_v7.2.1_Part1.zip] (~533MB)
*   **Purpose**: Primary compiler toolchain (Part A).
*   **Key Contents**: First half of the patched `bin/` executables, including `clang.exe`, `clang++.exe`, and `lld.exe`.

### 4. [ROCM_gfx1031_Compiler_v7.2.1_Part2.zip] (~410MB)
*   **Purpose**: Primary compiler toolchain (Part B).
*   **Key Contents**:
    - Remaining `bin/` utilities like `hipcc.exe`, `hipconfig.exe`, and `amdgpu-arch`.
    - `lib/clang/`: Critical LLVM internal headers and resource files.

---

## 🛠️ How to Use the Binaries (Installation)

1.  **Extract**: Create a folder (e.g., `C:\ROCM\7.2.1`) and extract **all 4 ZIP files** into it. 
    > [!IMPORTANT]
    > Ensure the `bin/`, `include/`, `lib/`, and `amdgcn/` folders are merged directly under the `C:\ROCM\7.2.1` root.

2.  **Path Setup**: Add `C:\ROCM\7.2.1\bin` to your System **PATH**.

3.  **Environment Variables**:
    Set the following to ensure the toolchain finds the correct paths:
    - `HIP_PATH` = `C:\ROCM\7.2.1`
    - `ROCM_PATH` = `C:\ROCM\7.2.1`
    - `DEVICE_LIB_PATH` = `C:\ROCM\7.2.1\amdgcn\bitcode`
    - `HIP_DEVICE_LIB_PATH` = `C:\ROCM\7.2.1\amdgcn\bitcode`

4.  **Verify**: Open PowerShell and run:
    ```powershell
    hipcc --version
    # Target should be: x86_64-pc-windows-msvc
    ```

---

## 🏗️ How to Use the Source (Reproduction)

If you wish to build from source using the [ROCm/TheRock](https://github.com/ROCm/TheRock) superbuild:

1.  **Clone the superbuild**:
    `git clone https://github.com/ROCm/TheRock.git`
2.  **Apply Patches**:
    - Copy `patches/AMDGPU.td` and `GCNProcessors.td` to `llvm-project/llvm/lib/Target/AMDGPU/`.
    - Copy `patches/ROCMCheckTargetIds.cmake` to `rocm-cmake/share/rocm-cmake/modules/`.
    - Replace `rocm-libraries/projects/hipblas/clients/common/host_alloc.cpp` with our version.
3.  **Initiate Build**:
    ```powershell
    # Build for GFX1031 target precisely
    python build_tools/build.py --amdgpu-targets gfx1031 --build-type Release MIOpen rocFFT rocSPARSE
    ```

---

## ⚠️ Limitations
- **Video/Media**: `rocdecode` and `rocjpeg` remain as Linux dependencies.
- **Verification Tools**: Client tests for `rocBLAS`/`hipBLASLt` are linked with `/FORCE:UNRESOLVED` to bypass `OpenBLAS` on Windows; this does not affect library performance.

---
*Maintained for the GFX1031 / RDNA2 Community.*
