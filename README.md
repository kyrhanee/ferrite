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

A modern Windows `.exe` packer. Smaller files, encrypted payload, drop-in replacement for UPX.

## Features

- Stronger compression than UPX
- Encrypted payload (no static signatures)
- Anti-debug protection
- Original file restored byte-for-byte by `unpack`
- CLI, shared library, and REST API

## Usage

```
ferrite pack myapp.exe -o myapp.packed.exe
ferrite unpack myapp.packed.exe
ferrite info myapp.exe
```

## Download

Prebuilt binaries: [Releases](../../releases/latest)

## License

MIT or Apache-2.0.
