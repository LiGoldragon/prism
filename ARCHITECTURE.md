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

## Shape

prism is **a library, linked by forge**. Not a CLI. Not a
proc-macro crate. Not a daemon. The current binary skeleton
(`src/main.rs` with `fn main() {}`) is a placeholder —
prism's body lands as a `lib.rs` once the records are wide
enough to project. forge depends on prism via `Cargo.toml`
and calls into it as a Rust API.

Per [criome ARCHITECTURE.md](https://github.com/LiGoldragon/criome/blob/main/ARCHITECTURE.md)
§4 / §7 / §10:

- forge owns the build pipeline; it links prism and calls
  into it for the records → `.rs` step.
- criome never links prism. Code emission is effect-bearing
  work; criome communicates, it never runs.
- prism never opens files, spawns subprocesses, or talks to
  the network. It is a pure function: records in, source
  text out.

## Contract (planned)

Input: a `FlowGraphSnapshot` — a coherent slice of sema
holding one `Graph` plus its `Nodes` and `Edges` (the
`signal::flow` data-kinds, materialised into Rust structs by
forge before the call).

```rust
// Sketch — actual types land in src/lib.rs.
pub struct FlowGraphSnapshot {
    pub graph: Graph,
    pub nodes: Vec<Node>,
    pub edges: Vec<Edge>,
}
```

Output: an `Emission` — a list of emitted source files,
each carrying a relative path inside the workdir and the
file's UTF-8 source text. forge writes these to disk as
part of workdir assembly.

```rust
pub struct Emission {
    pub files: Vec<EmittedFile>,
}

pub struct EmittedFile {
    pub path: String, // relative to workdir root
    pub source: String, // UTF-8 Rust source
}
```

Top-level entry point (sketch):

```rust
pub fn emit(snapshot: &FlowGraphSnapshot) -> Emission;
```

One-way emission: sema → Rust source. Nothing in the engine
ever reads the emitted text back (criome ARCH Invariant A).

## Macro-programming framing

prism is the workspace's **records-as-macros** layer, in the
sense criome ARCH §1 names: in the eventual self-hosting
state, code-generation patterns live as sema rules that prism
emits as plain Rust; macro-like behaviour happens at the
sema-to-Rust boundary, not as proc-macro expansion inside
rustc.

Concretely: **per node-kind, prism carries one hand-coded
emission template** (a Rust function in prism's own source).
The walker iterates the snapshot, dispatches each node to
its kind's template, threads edges through the templates as
wiring, and concatenates the per-template outputs into the
emitted file set.

- One Rust function per kind. Hand-coded in prism. No
  meta-templates, no string interpolation DSL, no
  dynamically-loaded codegen — the templates *are* the
  source of truth, version-controlled in this repo.
- The walker is the only generic part. Adding a kind = a new
  template function + extending the dispatch match.
- Third-party authored macros (`#[derive(Serialize)]`,
  `#[tokio::main]`, `format!`, `println!`) are emitted
  verbatim into the output text — rustc expands them
  downstream (criome ARCH §1 macro philosophy).

## First five node-kind templates

The opening templates planned for the records → `.rs`
walker, matching the `signal::flow` node-kinds:

- **Source** — produces values. Emits an actor/function that
  yields downstream over its outgoing edges.
- **Transformer** — value-in / value-out. Emits a function
  applied to each input from incoming edges; result flows
  along outgoing edges.
- **Sink** — consumes values. Emits the terminal handler;
  no outgoing edges.
- **Junction** — fan-in / fan-out routing. Emits the
  multi-edge plumbing (merge, split, broadcast) that wires
  the surrounding nodes together.
- **Supervisor** — lifecycle + restart policy over a
  sub-graph. Emits the parent actor that owns the children
  named on its outgoing edges.

These five cover the smallest expressive flow-graph
vocabulary; further kinds (state machines, persistent
stores, external-system adapters) land as additional
templates over the same walker.

## Status

Stub today (`fn main() {}`). The body lands once the engine
has records to project + forge-daemon arrives to host the
pipeline. Don't refactor or scaffold further until then.
