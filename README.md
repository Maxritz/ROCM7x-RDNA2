# ROCm 7.2.1 Windows Support for RDNA2 (gfx1031 / RX 6700 XT)

This repository provides consolidated patches and source modifications to enable full **gfx1031** (AMD Radeon RX 6700 XT) support in the ROCm 7.2.1 software stack on Windows. It bypasses current architecture locks and Windows-specific build hurdles.

## Key Enhancements

### 1. Toolchain & ISA Generation
- **`amd-llvm`**: Patched `GCNProcessors.td` to officially define the `gfx1031` architecture, enabling the compiler to generate Nave21-compatible ISA.
- **Navi21 Logic**: Included 40+ pre-generated `navi21` Tensile logic files for `rocBLAS` to ensure optimal matrix multiplication kernels on GFX1031.

### 2. Math Library Portability
- **`hipBLAS`**: Resolved Windows portability issues in `host_alloc.cpp`, including a native `GlobalMemoryStatusEx` replacement for Linux-specific memory tracking and `popen`/`pclose` aliases.
- **`rocRAND`**: Patched `config_types.hpp` for architecture-aware RNG kernel selection.
- **`hipBLASLt`**: Updated TensileLite generators and CMake logic to recognize GFX1031 and use consistent MSVC runtime properties.

### 3. Surgical Linker Overrides
To resolve `OpenBLAS` symbol mismatches in verification clients (tests/benchmarks) while preserving the core library functionality, we apply:
- `-Wl,/FORCE:UNRESOLVED` overrides in `rocBLAS`, `rocFFT`, `rocSPARSE`, and `rocSOLVER`.
- This ensures the core DLLs are built for AI inference even if the secondary test suites have unresolved host symbols.

## Quick Start (How to Apply)

1.  **Clone the target ROCm repository**:
    ```bash
    git clone https://github.com/ROCm/TheRock.git
    cd TheRock
    # Ensure you are on the 7.2.x release branch
    ```
2.  **Apply These Patches**:
    Copy the contents of this repository over your local clone:
    ```powershell
    # Root of this repository
    Copy-Item -Path ".\*" -Destination "C:\Path\To\TheRock\" -Recurse -Force
    ```
3.  **Initiate Build**:
    ```powershell
    ninja -j 24 -C build MIOpen rocFFT rocSPARSE
    ```

## Acknowledgements
Developed to empower the community with RDNA2 support on Windows before official upstream integration.
