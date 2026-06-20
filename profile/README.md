<p align="center"><img src="https://raw.githubusercontent.com/go-compressions/brand/main/social/go-compressions.png" alt="go-compressions" width="720"></p>

# go-compressions

🌐 **[Website](https://go-compressions.github.io)** · 📚 **[Documentation](https://go-compressions.github.io/docs/)**

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
| [`lz4`](https://github.com/go-compressions/lz4)           | Pure-Go LZ4 block codec, wire-compatible with `pierrec/lz4` (SIMD match-finder via [go-simd/matchlen](https://github.com/go-simd/matchlen)). |
| [`lzfse`](https://github.com/go-compressions/lzfse)       | Pure-Go LZFSE / LZVN codec (Apple's algorithm — LZFSE = LZVN + entropy stage).    |
| [`lzfsec`](https://github.com/go-compressions/lzfsec)     | CLI wrapper around `lzfse`, mirroring Apple's reference `lzfse` binary.           |
| [`lz4c`](https://github.com/go-compressions/lz4c)         | Reference CLI for the `lz4` block codec (compress by default, `-d` to decompress, stdin/stdout pipes). |

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
- **Multi-arch — six SIMD targets, validated on seven architectures.**
  Pure-Go reference implementations; SIMD / NEON acceleration is layered
  behind build tags where it makes sense. `blake3` (`mix4`) ships SIMD on
  **all six** of Go's 64-bit SIMD targets — amd64 (AVX2), arm64 (NEON),
  riscv64 (RVV), loong64 (LSX), ppc64le (VSX) and s390x (vector facility,
  big-endian) — via [go-asmgen](https://github.com/go-asmgen/asmgen)-generated
  assembly. `b3sum` inherits it per-arch with no code change of its own, and
  `lz4` gets its SIMD match-finder from [go-simd/matchlen](https://github.com/go-simd/matchlen)
  (the general SIMD common-prefix primitive now lives in the go-simd org).
  Beyond the six SIMD targets, every library (`lz4`, `lzfse`, `blake3`/`b3sum`)
  also **builds and passes its tests bit-exact on a seventh architecture,
  ppc64 (big-endian)** — POWER9 silicon, via the portable fallback path —
  proving big-endian correctness distinct from the s390x vector kernel.

### Measured performance

`ppc64le` is now natively measured on real POWER10 silicon
([GCC Compile Farm](https://portal.cfarm.net/), VSX, Go 1.26.4, June 2026):

- **lz4 encode** (matchlen-accelerated): **1.8×** scalar (1174 vs 644 MB/s),
  and **beats `pierrec/lz4`** (1174 vs 1012 MB/s).
- **blake3 `mix4`**: **4.5×** scalar.

`s390x` is qemu-validated for correctness; native throughput numbers are
pending a GitHub-hosted IBM Z runner.

## Who uses it

- [cloud-boot](https://github.com/cloud-boot) uses `lzfse` as the body
  compressor for its PE32+ / UKI packer (`go-coff/efipack`).
- The broader content-addressable-storage stack uses `blake3` as the
  address function for layer / blob lookup.

## Landing page

Project landing page: <https://go-compressions.github.io>.
