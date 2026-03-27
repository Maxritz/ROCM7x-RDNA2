# ROCm 7.2.1 Windows Support for RDNA2 (gfx1031 / RX 6700 XT)

> [!NOTE]
> **About This Project**: This repository is a community-driven effort to enable high-performance GPGPU and AI workloads on the **AMD Radeon RX 6700 XT (gfx1031)** using the ROCm 7.2.1 stack on Windows. By surgically patching the toolchain and applying linker-level overrides, we provide a path for RDNA2 users to run MIOpen, rocBLAS, and other core libraries that are currently "unofficial" in the standard Windows distribution.

## Technical Architecture & Implementation

### 1. ISA Generation (LLVM Toolchain)
ROCm's internal compiler requires explicit processor definitions in the AMDGPU backend to generate valid ISA.
- **`amd-llvm` Patch**: We've modified `GCNProcessors.td` to include the `gfx1031` target, mapping it to the appropriate feature sets (Navi21/RDNA2).
- **Tensile Logic**: We provide 40+ optimized Tensile logic files (`yaml`) specifically for `gfx1031`. These files serve as the "recipe" for `rocBLAS` to select the most efficient assembly kernels for matrix operations on this hardware.

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
