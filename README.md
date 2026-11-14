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
about nine minutes and up to 8.4 GiB of memory. Downloads and the build go to `work/`, or to
the directory given as the first argument: an empty directory or one an earlier build made,
with no space in its path.

The result is in `dist/`:

- `crystal-<tag>-caramel.<revision>-darwin-arm64.tar.gz`: the `crystal` binary, the licence
  texts of Crystal, LLVM, bdw-gc and PCRE2, and `BUILD.json`, which names every input by
  hash and the commit of this recipe;
- a `.json` file beside it with the archive's and the binary's SHA-256, and the macOS and
  compiler versions of the machine that built it.

The archive is packed the same way every time (fixed modes and times, sorted entries, no
owner names), so two builds that produce the same binary from the same commit of this
recipe produce the same archive.

## What the build refuses

`./build` fails unless:

- each download has its pinned SHA-256, and the tag resolves to the pinned commit;
- `llvm-config` names only static libraries inside the pinned LLVM;
- the compiler reports the pinned Crystal version, commit, LLVM version and default target;
- the binary starts on the oldest macOS its default target names (11.0), whatever macOS
  built it;
- the multi-threaded runtime and the threaded code generation path are in the binary;
- the binary loads no library outside `/usr/lib` and `/System/Library`;
- a program it compiles, once with one thread and once with its default, runs and prints
  what it should;
- the official archive's bdw-gc and PCRE2 are the versions whose licence texts are pinned;
- the archive holds exactly the staged files, and both JSON files parse.

## Inputs

`inputs` holds every pin: the Crystal tag and commit, the official archive, the LLVM
package and the licence texts of bdw-gc and PCRE2, each with its SHA-256. The compiler
links those two libraries statically from the official archive, which carries no licence
file for either.

To build another tag, change the tag, its commit and the official archive's URL and hash,
check the two library versions, and set `REVISION` back to 1. To publish the same tag
again, raise `REVISION`.

## Release

A release is built on a GitHub runner, and only a release is: the workflow runs when a tag
is pushed, never on a merge or a pull request.

1. Set the pins in `inputs` and commit.
2. Tag that commit `<tag>-caramel.<revision>`, such as `1.22.1-caramel.1`, and push the tag.
   The workflow refuses a tag that does not name what `inputs` builds.
3. The workflow runs `./build` and publishes the archive and its `.json` as a GitHub
   release of that tag. Caramel pins the hashes that release publishes.

Running the workflow by hand builds without publishing.

## Licence

The recipe is under the [MIT License](LICENSE). The compiler it builds is Crystal, under
the Apache License 2.0 with Swift exception; LLVM is under the Apache License 2.0 with LLVM
exceptions, bdw-gc under its MIT-style licence and PCRE2 under the BSD 3-Clause licence.
