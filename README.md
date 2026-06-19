# Wagnostic - Build Runtime Dependencies

This repository contains scripts and workflows to build all third-party
runtime libraries needed by Wagnostic runners across multiple platforms.

## Repository Structure

```
wagnostic-prebuilt-libs/
├── build.sh              # Main build script for a single platform
├── build-all.sh          # Build all platforms (requires Docker)
├── .github/workflows/
│   └── build.yml         # GitHub Actions workflow
└── README.md
```

### Required repository secrets

None. All dependencies are downloaded from public sources.

### How it works

The workflow runs a matrix build across 8 platform/architecture targets:

| Platform | Runner | Arch | Prebuilt available |
|---|---|---|---|
| Linux x86_64 | ubuntu-22.04 | 64-bit | ✅ wasmtime, V8 |
| Linux i686 | ubuntu-22.04 (Docker) | 32-bit | ⚠️ wasmtime not available |
| Linux aarch64 | ubuntu-22.04 (Docker) | ARM 64-bit | ⚠️ V8 may not be available |
| Linux armv7 | ubuntu-22.04 (Docker) | ARM 32-bit | ⚠️ limited prebuilts |
| macOS x86_64 | macos-13 | Intel Mac | ✅ wasmtime, V8 |
| macOS arm64 | macos-14 | Apple Silicon | ✅ wasmtime, V8 |
| Windows x86_64 | windows-2022 | 64-bit | ✅ wasmtime, V8 |
| Windows i686 | windows-2022 (Docker) | 32-bit | ⚠️ wasmtime not available |

Each build produces a `wagnostic-libs-<platform>.zip` containing:

```
wagnostic-libs-<platform>.zip
├── wasmtime/
│   ├── include/          # wasmtime.h, wasm.h, wasi.h
│   └── lib/              # libwasmtime.so / .dylib / .dll (+ .a)
├── v8/
│   ├── include/          # v8.h, v8-*.h, libplatform/
│   └── lib/              # libv8_monolith.a (compiled from source)
├── sdl2/
│   ├── include/          # SDL.h, SDL_*.h
│   └── lib/              # libSDL2.so / SDL2.lib / SDL2.dll
└── wasm3/
    ├── include/          # wasm3.h, m3_*.h
    └── lib/              # libwasm3.a (compiled from source)
```

## Usage

### Build single platform

```bash
./build.sh <platform> [output_dir]

# Examples:
./build.sh linux-x86_64
./build.sh macos-arm64 build-output
```

### Build all platforms (requires Docker)

```bash
./build-all.sh [output_dir]
```

### GitHub Actions

The workflow triggers on:
- New releases
- Manual dispatch (with version inputs)

## Dependencies

| Library | Source | Build method |
|---|---|---|
| **wasmtime** | https://github.com/bytecodealliance/wasmtime/releases | Prebuilt download (Rust project) |
| **V8** | https://chromium.googlesource.com/v8/v8 | **Compiled from source** via depot_tools + gn + ninja |
| **SDL2** | https://github.com/libsdl-org/SDL/releases | Prebuilt download |
| **wasm3** | https://github.com/wasm3/wasm3 | **Compiled from source** via cmake |

## Integration with Wagnostic

After building, extract the archives to `examples/runners/lib/`:

```bash
unzip wagnostic-libs-linux-x86_64.zip -d examples/runners/lib/
```

This creates the expected structure:
```
examples/runners/lib/
├── wasmtime/
├── v8/
├── sdl2/
└── wasm3/
```

Then build the runners:

```bash
cd examples
make host              # wasm3 (portable)
make host-wasmtime     # Wasmtime + OpenGL
make host-v8           # V8 + OpenGL
```
