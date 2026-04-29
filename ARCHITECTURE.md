# ARCHITECTURE — prism

A future projector from records to Rust source. Walks records
in sema, emits `.rs` source files. **One subcomponent of
[forge-daemon](https://github.com/LiGoldragon/forge)'s
runtime-creation pipeline** — the code-emission phase.
forge-daemon orchestrates the surrounding work: directory
assembly, dependency resolution (`Cargo.toml`, `flake.nix`),
compiler invocation (cargo / crane / rustc), and artifact
landing in arca.

prism emits source. It does not run the compiler, manage
dependencies, or land artifacts.

Stub today. The body lands once the engine has records to
project + forge-daemon arrives to host the pipeline.
