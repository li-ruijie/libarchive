# libarchive

Pre-built binary releases of [libarchive](https://github.com/libarchive/libarchive) for Windows (MSVC / MinGW x64) and Linux (GCC x86_64).

This repository exists purely to produce binary releases — DLLs, static libraries, headers and CLI tools (`bsdtar`, `bsdcpio`, `bsdcat`) — in both static and dynamic variants. No upstream source code is modified.

## Releases

Each release contains six artifacts:

| Artifact | Platform | Compiler | Linking | Contents |
|----------|----------|----------|---------|----------|
| `*-windows-msvc-x64-static.zip` | Windows | MSVC | Static | Standalone CLI tools, static lib, self-contained DLL, headers |
| `*-windows-msvc-x64-dynamic.zip` | Windows | MSVC | Dynamic | DLL-linked CLI tools, shared DLL, import lib, dependency DLLs, headers |
| `*-windows-mingw-x64-static.zip` | Windows | MinGW UCRT64 | Static | Standalone CLI tools, `libarchive.a`, self-contained `libarchive.dll`, headers |
| `*-windows-mingw-x64-dynamic.zip` | Windows | MinGW UCRT64 | Dynamic | DLL-linked CLI tools, `libarchive.dll`, import `.dll.a`, headers |
| `*-linux-gcc-x64-static.tar.gz` | Linux | GCC | Static | Statically-linked CLI tools, `libarchive.a`, headers |
| `*-linux-gcc-x64-dynamic.tar.gz` | Linux | GCC | Dynamic | Dynamically-linked CLI tools, `libarchive.so`, headers |

Download from the [Releases](https://github.com/li-ruijie/libarchive/releases) page.

### Static build library files (Windows)

The Windows static builds include multiple library files:

| File | Purpose |
|------|---------|
| `archive.lib` (MSVC) / `libarchive.a` (MinGW) | Static library — link this to embed libarchive directly into your executable |
| `libarchive.dll` | Self-contained DLL with all dependencies statically linked — no external DLLs required |
| `libarchive.lib` (MSVC) / `libarchive.dll.a` (MinGW) | Import library for the DLL — link this when using the DLL |

Choose **one** approach: either link the static library for a single executable, or distribute the DLL with your application and link against the import library.

## Dependencies

All variants are built with the following libraries:

| Library | Purpose |
|---------|---------|
| zlib | gzip compression |
| bzip2 | bzip2 compression |
| liblzma | xz/lzma compression |
| lz4 | LZ4 compression |
| zstd | Zstandard compression |
| OpenSSL | Cryptography (encrypted ZIP/7zip/RAR, hash verification, XAR signatures) |
| libiconv | Character set conversion |
| libxml2 | XML parsing (xar format) |

**Static builds** embed all eight libraries directly into the binaries. The resulting DLLs, static libraries and CLI tools have no external dependencies beyond Windows system DLLs (`KERNEL32.dll`, `bcrypt.dll`, UCRT) or the Linux system C library. No VC++ Redistributable or MinGW runtime DLLs are required.

**Shared builds** link against these libraries dynamically and ship the dependency DLLs alongside.

## Building

Builds are triggered via GitHub Actions `workflow_dispatch` with an upstream release tag (e.g. `v3.8.5`). The workflow checks out the corresponding tag from the upstream submodule and compiles from source.

### Windows (MSVC)

- Compiler: MSVC (Visual Studio 17 2022, x64)
- Dependencies via vcpkg (`x64-windows-static` triplet for static builds)
- Static variant uses `/MT` CRT (no VC++ Redistributable required)
- Two CMake passes produce both a static `.lib` and a self-contained `.dll`

### Windows (MinGW)

- Compiler: GCC via MSYS2 UCRT64 environment
- Dependencies via pacman
- Static variant uses `-static` linking (no external runtime dependencies)
- Two CMake passes produce both a static `.a` and a self-contained `.dll`

### Linux

- Compiler: GCC (x86_64)
- Dependencies via apt
