# Ferrite

```
 _____  _____ ______ ______  _____  _____  _____
|  ___||  ___|| ___ \| ___ \|_   _||_   _||  ___|
| |_   | |_   | |_/ /| |_/ /  | |    | |  | |_
|  _|  |  _|  |    / |    /   | |    | |  |  _|
| |    | |___ | |\ \ | |\ \  _| |_   | |  | |___
\_|    \____/ \_| \_|\_| \_| \___/   \_/  \____/
```

![Build](https://img.shields.io/badge/build-passing-brightgreen)
![License](https://img.shields.io/badge/license-MIT%20%7C%20Apache--2.0-blue)
![Platform](https://img.shields.io/badge/platform-Windows%20x64-lightgrey)
![Compression](https://img.shields.io/badge/compression-up%20to%2060%25-red)

A modern Windows `.exe` packer. Smaller files, encrypted payload, drop-in replacement for UPX.

## ⚙️ Features

- 📦 **Stronger compression than UPX** — up to **60%+** on real binaries
- 🔒 **Encrypted payload** — every packed file gets a fresh ChaCha20 key
- 🧩 **Three integration modes** — CLI, native shared library (DLL), self-hosted REST API
- 🪶 **Tiny footprint** — minimal runtime overhead
- ✅ **No external dependencies** — single executable

## 📥 Download

Prebuilt binaries: **[Releases](../../releases/latest)**

| File | What |
| --- | --- |
| `ferrite.exe` | Command-line packer |
| `ferrite.dll` | Native shared library (FFI) |
| `ferrite-api.exe` | REST API server |
| `libferrite.dll.a` | Import library for cgo / MinGW |
| `ferrite.h` | C header for the shared library |

## 🚀 Quick start

```
ferrite pack myapp.exe -o myapp.packed.exe
```

That's it. Run `myapp.packed.exe` like the original — it works the same way, just smaller.

## 🖥️ Commands

### `pack` — compress an executable

```
ferrite pack <input.exe> [-o output.exe] [-l level]
```

Example output:

```
$ ferrite pack myapp.exe

  [*] input    : myapp.exe
  [*] output   : myapp.packed.exe
  [*] algorithm: zstd (level 22)

  [+] 2.23 MiB -> 899.50 KiB
  [+] saved 1.35 MiB (60.62%)
  [*] elapsed  : 892 ms

  [+] packed successfully
```

### `info` — inspect a file

```
ferrite info <file.exe>
```

Shows machine, subsystem, base address, sections, and packed metadata if it is a Ferrite container.

### `test` — check whether a file is packed

```
ferrite test <file.exe>
```

Exit code `0` if the file is packed by Ferrite, `1` otherwise. Useful in scripts.

### `version` — print build info

```
ferrite version
```

## 🎚️ Options

| Flag | Default | Description |
| --- | --- | --- |
| `-o, --output <path>` | `<input>.packed.exe` | Output path |
| `-l, --level <level>` | `ultra` | Compression level: `fast`, `normal`, `max`, `ultra` |
| `--no-encrypt` | off | Disable payload encryption |
| `--keep-debug` | off | Keep the debug data directory |
| `--keep-certificates` | off | Keep the Authenticode certificate |

## 📊 Compression results

Real-world benchmarks (Zstd, level 22):

| Source | Size | Packed | Reduction |
| --- | --- | --- | --- |
| `ferrite.exe` | 1.63 MiB | 675 KiB | **59.59%** |
| `ferrite-api.exe` | 3.36 MiB | 1.37 MiB | **59.13%** |
| `cmd.exe` | 283 KiB | 171 KiB | **39.58%** |

## 📚 Native library (DLL)

For embedding from Go (cgo), Python (ctypes), C/C++, or any language with FFI. Header in `ferrite.h`:

```c
const char*    ferrite_version(void);
FerriteOptions ferrite_default_options(void);

int32_t ferrite_pack_file(const char* in, const char* out,
                          const FerriteOptions* opts, FerriteReport* report);
int32_t ferrite_pack_buffer(const uint8_t* in, size_t in_len,
                            const FerriteOptions* opts,
                            uint8_t* out, size_t cap, size_t* out_len,
                            FerriteReport* report);
int32_t ferrite_is_packed(const char* path);
int32_t ferrite_inspect(const char* path, FerriteReport* report);
int32_t ferrite_last_error(char* buf, size_t buf_len);
```

Returns `0` on success, negative on error.

## 🌐 REST API

For remote/automated packing — run `ferrite-api.exe`:

```
ferrite-api.exe --host 0.0.0.0 --port 7474
```

Endpoints:

| Method | Path | Body | Returns |
| --- | --- | --- | --- |
| `GET` | `/healthz` | — | `{"status":"ok"}` |
| `POST` | `/v1/pack` | multipart `file=`, optional `options=` | packed PE binary |
| `POST` | `/v1/inspect` | multipart `file=` | JSON manifest |

```
curl -F file=@app.exe -F 'options={"level":22}' \
     -o app.packed.exe \
     http://localhost:7474/v1/pack
```

Supports optional `Authorization: Bearer <TOKEN>` (set via `FERRITE_TOKEN` env).

## 🖥️ Compatibility

- ✅ **Windows 10 x64**
- ✅ **Windows 11 x64**
- ✅ **Windows Server 2019 / 2022**
- ⏳ x86 (32-bit) — planned
- ⏳ ARM64 — planned

> **Note:** system executables that depend on `%SYSTEMROOT%` (`notepad.exe`, `cmd.exe`, etc.) require their resource files to be co-located. They behave the same way without packing — this is a Windows limitation, not a Ferrite limitation. Pack your own applications instead.

## 📜 License

Dual-licensed under [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE).

For legitimate software-distribution purposes only.
