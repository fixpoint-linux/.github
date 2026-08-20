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

## The stack

| Component | What it is |
|---|---|
| **[`dhall-c`](https://github.com/fixpoint-linux/dhall-c)** | A subset interpreter for the Dhall configuration language, written in C. `typecheck`, `normalize`, `to-json`/`toml`/`yaml`. The typed-config foundation everything else builds on. |
| **[`datalog-dafsa`](https://github.com/fixpoint-linux/datalog-dafsa)** | A DAFSA-backed Datalog engine in C. Load facts into an on-disk minimal-acyclic-DAFSA store, compile Datalog rules to a small VM, materialize derived relations, serve reads from an mmap'd snapshot. |
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
