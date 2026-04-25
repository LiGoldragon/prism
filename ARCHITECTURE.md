# ARCHITECTURE — rsc

**R**ecords-to-**S**ource projector. Reads code records from
sema (Module, Struct, Enum, Fn, …) and emits `.rs` source files
+ `Cargo.toml` + `flake.nix` into a workdir for nix/rustc to
consume. One-way emission — sema is the source of truth, the
emitted text is a transient compile artifact.

## Role

The compile path:

```
records in sema (Opus + transitive OpusDeps + Module + Fn + ...)
       │
       ▼
   rsc projects → workdir/{src/*.rs, Cargo.toml, flake.nix}
       │
       ▼
   lojixd invokes nix build (crane + fenix)
       │
       ▼
   compiled binary lands in /nix/store, then bundled into
   lojix-store with RPATH rewrite
```

rsc is a **library**, not a daemon. It's linked into lojixd
and called as a function with a `Slot` of the root Opus.

## Boundaries

Owns:

- Reading sema records via the criomed-provided read interface.
- Emitting valid Rust syntax: every macro invocation passes
  through verbatim (project authors no macros; ecosystem
  macros are emitted as `#[derive(...)]` etc. without
  expansion).
- Generating `Cargo.toml` from `OpusDep` records.
- Generating `flake.nix` from `Derivation` and
  `RustToolchainPin` records.

Does not own:

- The records it projects (those are criomed's; rsc reads,
  never writes).
- Macro expansion (rustc handles that; rsc is upstream of
  expansion).
- Build invocation (lojixd does that).
- Diagnostic translation back to records (post-MVP; lojixd
  captures rustc JSON and asserts CompileDiagnostic records).

## Macro philosophy

The project authors no macros itself: no `macro_rules!`, no
proc-macro crates. Internal code-gen patterns live as sema
rules that run before rsc emission.

The project freely calls third-party macros — `#[derive(...)]`,
`#[tokio::main]`, `format!`, `println!`, etc. — and rsc emits
those invocations verbatim for rustc to expand.

## Code map

```
src/
└── lib.rs — projection entry point + module emitters
            (currently `todo!()` skeleton)
```

## Status

**Skeleton-as-design.** Empty bodies; structure and types
follow nexus-schema's record kinds. Lands when criomed +
lojixd reach the Compile loop (Stage F per
[mentci/reports/064](https://github.com/LiGoldragon/mentci/blob/main/reports/064-bootstrap-as-iterative-competence.md)
— now in arch.md §10 rung-by-rung).

## Cross-cutting context

- Compile + self-host loop:
  [criome/ARCHITECTURE.md §7](https://github.com/LiGoldragon/criome/blob/main/ARCHITECTURE.md)
- Macro philosophy:
  [criome/ARCHITECTURE.md §1](https://github.com/LiGoldragon/criome/blob/main/ARCHITECTURE.md)
