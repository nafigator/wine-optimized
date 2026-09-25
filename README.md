# Wine (WoW64, x86-64-v4 optimized)

Custom builds of [Wine](https://www.winehq.org/) with `-march=x86-64-v4 -O3` and related flags for AMD Zen 4 CPUs.

Built from upstream Wine source with WoW64 mode (`--enable-archs=i386,x86_64`), using Clang for both Unix and Windows components (via [llvm-mingw](https://github.com/mstorsjo/llvm-mingw)).

## Requirements

- **Architecture:** amd64
- **CPU:** with **AVX-512** support (x86-64-v4)
  - AMD Zen 4 (Ryzen 7000/8000/9000)

Wine binaries will crash with `Illegal instruction` on CPUs without AVX-512. Check support:

```bash
grep -o 'avx512[a-z]*' /proc/cpuinfo | sort -u | head
```

If the output is empty — **do not use these builds**.

## Installation

1. Download the archive from the [Releases](../../releases) page.

2. Extract it:

   ```bash
   tar -xJf wine-*-wow64-amd64-clang.tar.xz -C ~/.local/share/wine-builds/
   ```

3. Add `bin` to `PATH`:

   ```bash
   export PATH="$HOME/.local/share/wine-builds/wine-11.0-wow64-amd64-clang/bin:$PATH"
   ```

4. Verify:

   ```bash
   wine --version
   ```

## Initial setup

Create a Wine prefix and initialize it:

```bash
export WINEPREFIX="$HOME/.local/share/wineprefixes/default"
export WINEARCH=wow64
wineboot --init
```

`WINEARCH=wow64` tells Wine to use the new WoW64 mode, which runs 32-bit Windows applications without 32-bit Unix libraries.

## Verify WoW64 mode

```bash
# List supported architectures
wine --version
# Should report WoW64 support and both i386 and x86_64

# Check DLL overrides work
WINEDLLOVERRIDES="d3d11=n" winecfg
```

## Build Architecture

- **Compiler (Unix part):** Clang (system)
- **Compiler (Windows part):** Clang from [llvm-mingw](https://github.com/mstorsjo/llvm-mingw) 20260922 (UCRT)
- **Linker:** LLD
- **Build flags:** `-march=x86-64-v4, -mtune=znver4, -O3, -fomit-frame-pointer, -falign-functions=32, -falign-loops=32`
- **Configure:** `--enable-archs=i386,x86_64 --disable-winemenubuilder --disable-win16 --disable-tests --without-oss`

## Notes

- Built from upstream Wine source at the tagged release.
- Debug symbols are not included.
- The build disables components not needed for gaming: `winemenubuilder`, `win16`, tests, OSS audio.
- For D3D translation, use [DXVK](https://github.com/doitsujin/dxvk) and [VKD3D-Proton](https://github.com/HansKristian-Work/vkd3d-proton) — see companion repositories.
- Not affiliated with the upstream project. Report build-specific issues in this repository's [Issues](../../issues) tracker.

## Companion projects

For a complete optimized graphics stack on AMD Zen 4:

| Project | Purpose |
|---|---|
| [`mesa-optimized`](https://github.com/nafigator/mesa-optimized) | Mesa (radeonsi, RADV) |
| [`dxvk-optimized`](https://github.com/nafigator/dxvk-optimized) | DXVK (D3D9/10/11 → Vulkan) |
| [`vkd3d-proton-optimized`](https://github.com/nafigator/vkd3d-proton-optimized) | VKD3D-Proton (D3D12 → Vulkan) |

## Clear shader caches after installing new drivers

After updating Mesa, DXVK, or VKD3D-Proton, clear all shader caches:

```bash
rm -rf ~/.cache/mesa_shader_cache* \
       ~/.cache/dxvk/* \
       ~/.cache/vkd3d-proton/*
```

For per-game caches, remove `vkd3d-proton.cache*` next to the game's `.exe`.

## How It Is Built

GitHub Actions workflow:

1. Starts the `devuan/devuan:excalibur` container.
2. Installs `clang`, `lld`, `llvm`, `ccache`, `mingw` runtime libs and Wine build dependencies.
3. Downloads and installs `llvm-mingw` toolchain to `/opt/llvm-mingw`.
4. Restores `ccache` from GitHub Actions cache.
5. Downloads Wine source from `dl.winehq.org`.
6. Configures Wine with WoW64 and optimization flags.
7. Builds and installs into a staging prefix.
8. Packages the result into `wine-<version>-wow64-amd64-clang.tar.xz`.
9. Uploads the archive as an artifact.

Source workflow: [`.github/workflows/build.yml`](.github/workflows/build.yml).

## Important

- These builds are **only for CPUs with AVX-512**. Do not use them on other hardware.
- **Do not use** them on distributions older than Devuan Excalibur — glibc version matters.
- Wine manipulation in online multiplayer games may be considered cheating. **Use at your own risk.**
- Always keep a working stock Wine installation as fallback.

## License

Build scripts and workflows in this repository are licensed under the MIT License.

Wine itself is distributed under the [LGPL-2.1-or-later](https://www.winehq.org/license). The compiled binaries in Releases are redistributions of Wine under its original license.
