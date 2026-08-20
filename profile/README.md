<div align="center">

# ⚙️ fixpoint-linux

**A Linux system that is *a fixed point*: deterministically built, from source, by itself.**

</div>

---

`fixpoint-linux` is a collection of small, self-contained components written in **C11** that
assemble into a coherent Linux userspace. Every binary is compiled with
[**cosmocc**](https://github.com/jart/cosmopolitan) into a single portable
[**Actually Portable Executable**](https://justine.lol/ape.html) (APE) — one file that runs on
Linux, macOS, Windows, and the BSDs with **no VM, no runtime, no interpreter, and no dependencies**.

Everything is configured in **[Dhall](https://dhall-lang.org/)**, a strongly-typed, total
configuration language. Configs are typechecked, normalized, and *terminate* — they are programs,
not property files.

The name comes from the two ideas at the heart of the stack:

- **Fixpoint** — the least-fixed-point semantics of [Datalog](https://en.wikipedia.org/wiki/Datalog);
  a system is its own build artifact, deterministic and reproducible.
- **DAFSA** — the [minimal acyclic finite-state automaton](https://en.wikipedia.org/wiki/Deterministic_acyclic_finite_state_automaton)
  that backs the data stores: compact, exact, and fast.

## ⏱️ A system that never forgets itself

`fixpoint-linux` is **content-addressed by construction and time-travelling by
default**. Every change is one atomic snapshot of the whole system; the timeline
is the system's complete history. You can inspect any past state with an as-of
query, roll back to any earlier point, and undo the rollback itself — without
ever losing the record of what happened.

- **Roll-forward rollbacks** — the timeline is an append-only ledger; going back
  is recorded as history and always undoable.
- **Boot rollback** — if the latest activation fails to come up, init rolls back
  to the last good state automatically.
- **Generation GC** — keep N bootable generations, prune the rest, done.

Powered by `datalog-dafsa`'s native snapshot time-travel
(`dl_publish_snapshot` / `dl_snapshot_versions` / `dl_query_version`).

## The stack

| Component | What it is |
|---|---|
| **[`fixpoint-linux`](https://github.com/fixpoint-linux/fixpoint-linux)** | **The system itself** — a Dhall-specified, self-hosting Linux distro. Like Nix's *model* (pure derivations, content-addressed store, hermetic builds) without the Nix language. **Time-travelling** — the whole system remembers and rolls back ([§10](https://github.com/fixpoint-linux/fixpoint-linux/blob/main/DESIGN.md)). [Read the design.](https://github.com/fixpoint-linux/fixpoint-linux/blob/main/DESIGN.md) |
| **[`dhall-c`](https://github.com/fixpoint-linux/dhall-c)** | A subset interpreter for the Dhall configuration language, written in C. `typecheck`, `normalize`, `to-json`/`toml`/`yaml`. The typed-config foundation everything else builds on. |
| **[`datalog-dafsa`](https://github.com/fixpoint-linux/datalog-dafsa)** | A DAFSA-backed Datalog engine in C. Load facts into an on-disk minimal-acyclic-DAFSA store, compile Datalog rules to a small VM, materialize derived relations, serve reads from an mmap'd snapshot. **Native time travel** — versioned snapshots + as-of queries make the system timeline and rollbacks possible. |
| **[`dhake`](https://github.com/fixpoint-linux/dhake)** | A Make-like build tool whose buildfile is a Dhall program (`Dhakefile.dhall`). Typed actions, incremental mtime up-to-date checks, dependency ordering, phony targets. **Self-hosting** — it builds itself. |
| **[`compendium`](https://github.com/fixpoint-linux/compendium)** | A small, self-contained authoritative **DNS server** (UDP, RFC 1035), configured in Dhall, shipped as a single APE binary. |
| **[`visage`](https://github.com/fixpoint-linux/visage)** | A compact **email alias & forwarding server** — disposable `alias@domain` addresses backed by a DAFSA store. Daemon and store in one small APE binary. |
| **[`dafsa`](https://github.com/fixpoint-linux/dafsa)** | The [Carrasco–Forcada](https://en.wikipedia.org/wiki/Deterministic_acyclic_finite_state_automaton) incremental DAFSA — minimal automaton with add/delete/lookup, persistence and DOT export. |
| **[`shen-meta`](https://github.com/fixpoint-linux/shen-meta)** | A self-hosted [Shen](https://shenlanguage.org/) implementation — a **sequent-calculus Lisp**. Evaluates itself, compiles itself to native bytecode via its own `shen->kl` compiler, and runs on a native C VM with a custom GC. Sequent calculus provides the inference kernel (cut elimination as computation). |

## Design principles

- **One binary, zero deps.** Cosmocc + APE means each tool is self-contained and portable across OSes.
- **Config is typed code.** Dhall gives typechecking, imports, and reusable functions — and it *always* terminates.
- **Logic is declarative.** Datalog + DAFSA keep the data plane compact and exact.
- **Self-hosting.** Tools build themselves (see `dhake`'s self-hosting buildfile).
- **Small and legible.** Each component fits in your head; none pulls in a framework or a heavyweight runtime.

## The system — read the design

The org's apex is the **`fixpoint-linux`** distro itself: a self-hosting Linux
system whose spec, builds, and store are all Dhall + Datalog + DAFSA —
content-addressed by construction.

👉 **[Read the full architecture design](https://github.com/fixpoint-linux/fixpoint-linux/blob/main/DESIGN.md)**

## Getting started

Each repository is independently buildable. For example, to build the Dhall interpreter and the
build tool:

```sh
cd dhall-c && make && make test   # builds dhall.com (APE) + runs the test suite
cd dhake   && make                # self-hosting: builds dhake.com from its Dhakefile.dhall
```

## License

Per-repository; see each component for details.

---

<div align="center">

Built with ❤️ and a single `cosmocc` invocation.

</div>
