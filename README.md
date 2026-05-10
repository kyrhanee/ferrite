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
![Platform](https://img.shields.io/badge/platform-Windows%20x86%20%2F%20x64-lightgrey)
![Compression](https://img.shields.io/badge/compression-up%20to%2073%25-red)

A modern Windows `.exe` packer. Compresses executables, polymorphises the payload with ChaCha20, ships as a single binary.

## ⚙️ Features

- 📦 **Strong compression** — up to **73%** size reduction (Zstd 22 + BCJ x86 filter)
- 🎲 **Polymorphic payload** — every packed file gets a fresh ChaCha20 key, so two
  identical inputs produce byte-different outputs (defeats static signatures)
- 🛡️ **Tamper-evident** — SHA-256 of the payload is verified by the stub at every launch
- 🖥️ **Both x86 (32-bit) and x64 (64-bit)** — packer auto-detects architecture and
  embeds the right unpacker stub
- ⚡ **Three speed presets** — from instant to maximum
- 🧩 **Three integration modes** — CLI, native shared library (DLL), self-hosted REST API
- 🪶 **~50 KiB runtime** — Rust `no_std` stub, only depends on `kernel32.dll` + `bcrypt.dll`
- ✅ **Zero external dependencies** — single executable

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
ferrite app.exe
```

Packs `app.exe` into `app.packed.exe`. Run it like the original — it works the same, only smaller.

## 🖥️ Usage

```
ferrite [OPTIONS] FILE
```

| Flag | Description |
| --- | --- |
| `-o, --output PATH` | Output file (default: `FILE.packed.exe`) |
| `-l, --level LEVEL` | `fast` \| `max` \| `ultra` (default: `ultra`) |
| `-i, --info` | Show file info / metadata |
| `-t, --test` | Exit `0` if FILE is packed, `1` otherwise |
| `-V, --version` | Print build info |
| `-h, --help` | Show help |

### Examples

```
ferrite app.exe                    # pack into app.packed.exe (ultra)
ferrite app.exe -l fast            # fast pack — seconds
ferrite app.exe -l max             # balanced
ferrite app.exe -o tiny.exe        # custom output name
ferrite -i app.packed.exe          # show metadata
ferrite -t app.packed.exe          # check if packed (exit 0/1)
```

### Example output

```
$ ferrite node.exe

  [*] input    : node.exe
  [*] output   : node.packed.exe
  [*] algorithm: zstd (level 22)

  [+] 87.45 MiB -> 23.59 MiB
  [+] saved 63.85 MiB (73.02%)
  [*] elapsed  : 41255 ms

  [+] packed successfully
```

## 📊 Compression results

Real-world benchmark on a 87.45 MiB x64 binary:

| Level | Size | Reduction | Time |
| --- | --- | --- | --- |
| `fast` | 34.13 MiB | 60.97% | 1.1 s |
| `max` | 27.10 MiB | 69.01% | 2.7 s |
| `ultra` | **23.59 MiB** | **73.02%** | 41 s |

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

For remote / automated packing — run `ferrite-api.exe`:

```
ferrite-api.exe                                # listens on 127.0.0.1:7474 (loopback)
ferrite-api.exe --host 0.0.0.0 --token SECRET  # exposed; token mandatory
```

By default the server binds to `127.0.0.1` (loopback only). Binding to a
non-loopback address requires `--token` / `FERRITE_TOKEN`, otherwise startup
is refused.

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

When a token is set, send it in the request:

```
curl -H "Authorization: Bearer SECRET" -F file=@app.exe \
     -o app.packed.exe \
     http://your-host:7474/v1/pack
```

## 🖥️ Compatibility

- ✅ Windows 10 / 11 (x64 and x86)
- ✅ Windows Server 2019 / 2022
- ✅ x86 (32-bit) inputs — packed with a 32-bit stub
- ✅ x64 (64-bit) inputs — packed with a 64-bit stub
- ⏳ ARM64 — planned

## 📜 License

Dual-licensed under [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE).

For legitimate software-distribution purposes only.
