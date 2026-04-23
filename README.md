# rsc

Projector from sema database records to Rust source. Walks an
opus — the database's compilation-unit concept — and emits a
directory of `.rs` files that can be compiled with rustc.

The capstone for self-hosting: when rsc produces a build of the
database-dwelling source that matches (behaviorally) the
hand-written source, the loop closes.

## License

[License of Non-Authority](LICENSE.md).
