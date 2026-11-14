# caramel-crystal

Builds the Crystal compiler that Caramel's managed toolchain runs on Apple Silicon macOS.

It is the Crystal tag Caramel pins, unpatched, built by the official compiler of the same
tag, with two differences from the official macOS archive:

- code generation runs on several threads (the official macOS binary is built with
  `-Dwithout_mt`, so `--threads` does nothing there);
- LLVM is linked statically from the conda-forge `llvmdev` package Caramel already pins.

## Build

```sh
./build
```

It needs macOS on Apple Silicon, Apple's Command Line Tools and the network. A build takes
about nine minutes and up to 9 GB of memory. Downloads and the build go to `work/`, or to the
directory given as the first argument; its path must have no space.

The result is in `dist/`:

- `crystal-<tag>-caramel.<revision>-darwin-arm64.tar.gz`: the `crystal` binary, the licences
  of Crystal and LLVM, and `BUILD.json`, which names every input by hash;
- a `.json` file beside it with the archive's and the binary's SHA-256.

## What the build refuses

`./build` fails unless:

- each download has its pinned SHA-256, and the tag resolves to the pinned commit;
- `llvm-config` names only static libraries inside the pinned LLVM;
- the compiler reports the pinned Crystal version, commit, LLVM version and default target;
- the multi-threaded runtime and the threaded code generation path are in the binary;
- the binary loads no library outside `/usr/lib` and `/System/Library`;
- a program it compiles, once with one thread and once with its default, runs and prints
  what it should.

## Inputs

`inputs` holds every pin: the Crystal tag and commit, the official archive and the LLVM
package, each with its SHA-256. To build another tag, change the tag, its commit and the
official archive's URL and hash, and set `REVISION` back to 1. To publish the same tag
again, raise `REVISION`.

The compiler also links bdw-gc and PCRE2 statically, from the official archive. `BUILD.json`
records their versions; the official archive carries no licence file for either, so the
archive built here does not yet either.

## Licence

The recipe is under the [MIT License](LICENSE). The compiler it builds is Crystal, under
the Apache License 2.0, with LLVM under the Apache License 2.0 with LLVM exceptions.
