# simple_os

A small x86_64 kernel written in Rust, with no standard library (`#![no_std]`). It boots with the `bootloader` crate and writes text straight to the VGA text buffer.

This guide covers building and running it on Linux.

## Prerequisites

### 1. Rust (nightly)

The project needs nightly Rust because it builds `core` from source and uses a custom target. You do not have to switch your default toolchain: `rust-toolchain.toml` tells `rustup` to use nightly inside this folder.

Install `rustup` if you don't have it:

```sh
curl https://sh.rustup.rs -sSf | sh
```

Then, from the project root, add the two components the build needs:

```sh
rustup component add rust-src llvm-tools-preview
```

- `rust-src` lets Cargo compile `core` and `compiler_builtins` for the custom target.
- `llvm-tools-preview` is needed by `bootimage` to create the bootable disk image.

### 2. bootimage

`bootimage` links the kernel with the bootloader and produces a bootable disk image.

```sh
cargo install bootimage
```

### 3. QEMU

QEMU runs the kernel in a virtual machine.

| Distro | Command |
| --- | --- |
| Debian / Ubuntu | `sudo apt install qemu-system-x86` |
| Fedora | `sudo dnf install qemu-system-x86` |
| Arch | `sudo pacman -S qemu-system-x86` |

Check that it is installed:

```sh
qemu-system-x86_64 -version
```

## Build

Clone the repo:

```sh
git clone https://github.com/Palmkonde/simple_os.git
cd simple_os
```

Compile the kernel only:

```sh
cargo build
```

This produces the kernel ELF file at `target/x86_64-simple_os/debug/simple_os`. It cannot boot by itself yet.

Create the bootable disk image:

```sh
cargo bootimage
```

This produces `target/x86_64-simple_os/debug/bootimage-simple_os.bin`.

For an optimized build, add `--release` to either command. The output then goes to `target/x86_64-simple_os/release/`.

## Run

The easiest way is:

```sh
cargo run
```

`.cargo/config.toml` sets `bootimage runner` as the runner, so this builds the disk image and starts QEMU in one step.

You can also start QEMU yourself:

```sh
qemu-system-x86_64 -drive format=raw,file=target/x86_64-simple_os/debug/bootimage-simple_os.bin
```

If everything works, a QEMU window opens and shows `Hello, Furry san!` in yellow on the bottom row of the screen.

## How the build is configured

| File | What it does |
| --- | --- |
| `rust-toolchain.toml` | Pins the project to nightly Rust. |
| `x86_64-simple_os.json` | Custom target: bare metal x86_64, no OS, no red zone, SIMD disabled (soft float), linked with `rust-lld`. |
| `.cargo/config.toml` | Builds `core` and `compiler_builtins` from source, sets the custom target as the default, and sets `bootimage runner` as the runner. |
| `Cargo.toml` | Uses `bootloader` 0.9 and sets `panic = "abort"` for both dev and release builds. |

## Troubleshooting

**`can't find crate for core`**
The `rust-src` component is missing. Run `rustup component add rust-src`.

**`bootimage` says it could not find the `llvm-tools` component**
Run `rustup component add llvm-tools-preview`.

**Components go missing after a fresh clone or a toolchain update**
You can let `rustup` install them automatically by adding a `components` line to `rust-toolchain.toml` (not tested yet in this project):

```toml
[toolchain]
channel = "nightly"
components = ["rust-src", "llvm-tools-preview"]
```

**Errors about unknown unstable options or target spec fields**
Your nightly may be too old. Run `rustup update nightly`.

**`qemu-system-x86_64: command not found`**
QEMU is not installed. See the QEMU section above.

## Credits

This project follows Philipp Oppermann's blog series [Writing an OS in Rust](https://os.phil-opp.com/).
