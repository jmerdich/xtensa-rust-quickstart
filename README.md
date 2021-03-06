
# Rust on the xtensa architecture

Need help? Join the esp-rs room on matrix, https://matrix.to/#/#esp-rs:matrix.org.

## Overview
ESP32 and ESP8266 chips come with one or two Xtensa CPUs, which (unlike x86 or ARM) are not supported by the official Rust compiler.

These instructions describe how to build Rust compiler (from [LLVM](https://en.wikipedia.org/wiki/LLVM) and rustc) that supports Xtensa architecture, as well as  how to build and flash blinking LED program to the ESP32.

## Building Rust compiler
### System Requirements
Building Rust compiler is CPU and memory intense process. The compilation can take up to 15 minutes on higher-grade CPUs and even several hours on resource-starved VMs. Besides that, the compilation may fail on systems with less than 6GB of RAM.

The required software components and resulting artefacts can take up to 10GB of disk space.

## Recommended build method

The fork now uses the xtensa enabled fork as its llvm submodule, so its now possible to build the entire toolchain in a few commands.

### Recommended build method - UNIX

All that is required to build rustc for linux is the steps below. 

```bash
$ git clone https://github.com/MabezDev/rust-xtensa
$ cd rust-xtensa
$ ./configure --experimental-targets=Xtensa
$ ./x.py build
```
Before cross-compiling an xtensa target, you must set the following vars, which are set in the setenv script in this project:
```
XARGO_RUST_SRC=/path/to/rust-xtensa/src
RUSTC=/path/to/rust-xtensa/build/x86_64-unknown-linux-gnu/stage2/bin/rustc
```

### Recommended build method - Windows

After making sure you have Visual Studio Community and python3 installed, in a cmd.exe run:

```cmd
$ git clone https://github.com/MabezDev/rust-xtensa
$ cd rust-xtensa
$ CALL "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvars64.bat"
$ python3 src/bootstrap/configure.py --experimental-targets=Xtensa
$ python3 x.py build
```

Before cross-compiling an xtensa target, you must set the following vars either for your system or within your IDE:
```
XARGO_RUST_SRC=\path\to\rust-xtensa\src
RUSTC=\path\to\rust-xtensa\build\x86_64-pc-windows-msvc\stage2\bin\rustc
```

## Manual llvm build instructions

If you would like to build the llvm fork separately, follow the instructions below.

### llvm-xtensa
If you don't have them already, you'll first have to install `ninja-build` and a C++ compiler (such as `g++`).
Please refer to [LLVM project](https://llvm.org/docs/GettingStarted.html) for more information.

    $ git clone https://github.com/MabezDev/llvm-project
    $ cd llvm-project/llvm
    $ mkdir build
    $ cd build
    $ cmake .. -DLLVM_TARGETS_TO_BUILD="X86" -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="Xtensa" -DCMAKE_BUILD_TYPE=Release -G "Ninja"
    $ cmake --build .

### rust-xtensa
Please refer to the [rust-xtensa](https://github.com/MabezDev/rust-xtensa) project for authoratative instructions.

Assuming you built llvm in your home directory:

    $ git clone https://github.com/MabezDev/rust-xtensa
    $ cd rust-xtensa
    $ ./configure --llvm-root=$HOME/llvm-project/llvm/build
    $ ./x.py build

## Installing tools
### xtensa-esp32-elf toolchain
Instructions can be found [on Espressif's web site](https://docs.espressif.com/projects/esp-idf/en/release-v3.0/get-started/linux-setup.html).

Download the archived toolchain file, and extract it to the directory of your choice. Then add the toolchain's bin/ directory to your `$PATH`. For example:

    $ mkdir ~/esp
    $ tar -xzf ~/Downloads/xtensa-esp32-elf-linux64-1.22.0-80-g6c4433a-5.2.0.tar.gz -C ~/esp
    $ PATH="$PATH:$HOME/esp/xtensa-esp32-elf/bin"

### xargo or cargo xbuild

    $ cargo install xargo

When you install xargo, it may complain that this requires the nightly channel. The fix for this is to switch your rust installation to use the nightly channel as follows:

    $ rustup default nightly
    $ rustup update


or

    $ cargo install cargo-xbuild

### esptool
Esptool is python-based command line tool for flashing ESP32 and ESP8266 chips.
Full installation instructions are available on [the website](https://github.com/espressif/esptool), but it's usually as straightforward as:

    $ pip install esptool

## Blinking LED
### Starting a new project
    $ git clone https://github.com/MabezDev/xtensa-rust-quickstart

### Workflow
Update `CUSTOM_RUSTC` in `setenv` to point to the version of rust you compiled earlier. Then load the environment variables with

    $ source setenv

If you installed `xbuild` instead of `xargo`, you will need to update `flash` and `flash_release` accordingly.
    
You should now be able to call xargo (or cargo xbuild) to build the project:

    $ xargo build

You will need to change the parameter `BLINKY_GPIO` to match your board's LED pin. Unfortunately, this may require adjustments to the chip's IO_MUX peripheral, which will mean consulting the ESP32 Technical Reference Manual. See [this issue](https://github.com/MabezDev/idf2svd/issues/11) for more information.

Then after editing the flash script, you should be able to flash the ESP as follows (you map need to edit the device):

    $ ./flash

## Resources

- The [esp-rs](https://github.com/esp-rs) organization has been formed to develop runtime, pac and hal crates for the esp32 and eventually esp8266.
- Checkout @lexxvir's [project](https://github.com/lexxvir/esp32-hello) for an example of using the esp-idf bindings in a Rust application.

## FAQ

- `error: intermittent IO error while iterating directory` - try creating that directory
- `undefined reference to .L` see [this issue](https://github.com/MabezDev/xtensa-rust-quickstart/issues/1)
