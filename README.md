# ROCm 7.2.1 Windows Support for RDNA2 (gfx1031 / RX 6700 XT)

This repository contains the necessary patches and source modifications to enable full **gfx1031** (AMD Radeon RX 6700 XT) support in the ROCm 7.2.1 software stack on Windows.

## Overview of Changes

1.  **Toolchain Support**: Patched `GCNProcessors.td` in `amd-llvm` to enable the `gfx1031` architecture.
2.  **Math Libraries**:
    *   **rocRAND**: Added `gfx1031` to architecture lookup tables and enums.
    *   **rocFFT**: Updated solution-shipping logic for architecture fallbacks.
    *   **hipBLASLt**: Enabled `gfx1031` in TensileLite generators and MSVC runtime configurations.
    *   **hipBLAS**: Fixed Windows portability issues in `host_alloc.cpp` (`popen`/`pclose` aliases and memory status).
3.  **Linker Overrides**: Applied `/FORCE:UNRESOLVED` to core math libraries (`rocBLAS`, `rocFFT`, `rocSPARSE`, `rocSOLVER`) to bypass missing OpenBLAS symbols during verification tool builds.
4.  **MIOpen**: Enabled target property propagation for RX 6700 XT.

## How to Apply

1.  **Clone the target ROCm repository**:
    ```bash
    git clone https://github.com/ROCm/TheRock.git
    cd TheRock
    # Ensure you are on the 7.2.x release branch
    ```
2.  **Copy these patches**:
    Overlay the contents of this repository onto your local clone:
    ```powershell
    # From this patch directory
    Copy-Item -Path ".\*" -Destination "C:\Path\To\TheRock\" -Recurse -Force
    ```
3.  **Build**:
    Use the standard `ninja` build command:
    ```powershell
    ninja -j 24 -C build MIOpen rocFFT rocSPARSE
    ```

## Contributing
These patches were developed to bypass common Windows-specific build hurdles and enable RDNA2 hardware that is currently "unofficial" in the main ROCm distribution. Feel free to submit PRs for other RDNA2/3 targets.
