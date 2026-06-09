# go-compressions

**Pure-Go byte-stream transformation primitives.**

Despite the name, `go-compressions` houses the broader family of
low-level **byte-stream transforms** — both lossless compression codecs
*and* content-addressable hash functions. They live together because
they sit at the same layer of the stack: pure-Go, allocation-aware,
zero-copy-friendly primitives that everything else builds on.

The umbrella covers two adjacent concerns:

- **Compression codecs** — pure-Go implementations of stream
  compressors (LZFSE/LZVN today; more to come) for projects that need a
  Go-native body compressor without dragging in cgo bindings.
- **Content-addressable hashes** — pure-Go implementations of modern
  cryptographic / keyed hashes (BLAKE3 today) used as the address
  function in content-addressable storage layers.

Different math, same place in the stack: turn a byte stream into
another, smaller or fingerprinted, byte stream.

## Repositories

### Compression codecs

| Repo                                                      | Role                                                                              |
| --------------------------------------------------------- | --------------------------------------------------------------------------------- |
| [`lzfse`](https://github.com/go-compressions/lzfse)       | Pure-Go LZFSE / LZVN codec (Apple's algorithm — LZFSE = LZVN + entropy stage).    |
| [`lzfsec`](https://github.com/go-compressions/lzfsec)     | CLI wrapper around `lzfse`, mirroring Apple's reference `lzfse` binary.           |

### Content-addressable hashes

| Repo                                                      | Role                                                                              |
| --------------------------------------------------------- | --------------------------------------------------------------------------------- |
| [`blake3`](https://github.com/go-compressions/blake3)     | Pure-Go [BLAKE3](https://github.com/BLAKE3-team/BLAKE3) hash + keyed-hash + KDF.   |
| [`b3sum`](https://github.com/go-compressions/b3sum)       | Pure-Go `b3sum` CLI matching the reference Rust [`b3sum`](https://crates.io/crates/b3sum). |

### Docs

| Repo                                                      | Role                                                                              |
| --------------------------------------------------------- | --------------------------------------------------------------------------------- |
| [`docs`](https://github.com/go-compressions/docs)         | Org-level documentation, design notes, NEON / archsimd notes.                     |

## Project standards

- **Pure Go.** No cgo. Bottom-of-stack code should compile and run
  anywhere Go does.
- **100% test coverage** is the bar on every module. Codecs are
  fuzz-tested against upstream reference implementations; hashes are
  cross-checked against the spec test vectors.
- **BSD-3-Clause** on all source files.
- **Allocation-aware.** Hot paths avoid per-call allocations; APIs
  expose buffered variants where it matters.
- **Multi-arch.** Pure-Go reference implementations; SIMD / NEON
  acceleration is layered behind build tags where it makes sense.

## Who uses it

- [cloud-boot](https://github.com/cloud-boot) uses `lzfse` as the body
  compressor for its PE32+ / UKI packer (`go-coff/efipack`).
- The broader content-addressable-storage stack uses `blake3` as the
  address function for layer / blob lookup.

## Landing page

Project landing page: <https://go-compressions.github.io>.
