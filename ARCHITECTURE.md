# ARCHITECTURE — prism

A future projector from records to Rust source. Walks records
in sema, emits `.rs` source files. **One subcomponent of
[lojix-daemon](https://github.com/LiGoldragon/lojix)'s
runtime-creation pipeline** — the code-emission phase.
lojix-daemon orchestrates the surrounding work: directory
assembly, dependency resolution (`Cargo.toml`, `flake.nix`),
compiler invocation (cargo / crane / rustc), and artifact
landing in lojix-store.

prism emits source. It does not run the compiler, manage
dependencies, or land artifacts.

Stub today. The body lands once the engine has records to
project + lojix-daemon arrives to host the pipeline.
