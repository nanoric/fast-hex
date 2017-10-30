A fast, SIMD (vectorized) hex string encoder/decoder, available as a stand-alone
C++ module and as a Node.js module. Supports upper-case and lower-case
characters.

Requires x86/64 AVX2 (Haswell or later).

I think these implementations are optimal, but I welcome challenges to that
statement.

---

## C++

Pull in `src/hex.h` and `src/hex.cc`, and adjust your build appropriately
(GCC/Clang/ICC: `-march=haswell` for example; MSVC: set
`EnableAdvancedInstructionSet` to "AVX2" or `/arch:AVX2`).

**Benchmark**
* Decoding ~12.5x over scalar.
* Encoding ~11.5x over scalar.

---

## Node.js

```typescript
const speedily = require("speedily");
speedily.decodeHexVec(output: Uint8Array|Buffer, input: Uint8Array|Buffer): void;
speedily.encodeHexVec(input: Uint8Array|Buffer): string;
```

**Benchmark**
* ~2.12x faster decoding for short string inputs (< 1,031,913 chars).
* ~12x faster decoding for some long strings, Buffers and TypedArrays.
* ~5.5x faster encoding.

Development notes:
* Accessing string bytes from v8 is slow. Node uses external strings in some
  cases, but so far the only scenarios in which I've found them used are (a)
  via `buffer.toString()`, (b) via the C++ API
  `v8::Local<v8::Value> node::Encode(v8::Isolate* isolate, const char* buf, size_t len, enum encoding encoding)`.
  Haven't looked yet at what cases cause v8 to automatically use external
  strings.
* `stream.write` uses `StringBytes::Encode` internally. Can't replace this
  without modifying node.js, unless we intercept `stream.write` in JS.
* Creating strings is also expensive. Still need to look at if there's a better
  way to do this, and definitely need to use external strings over some length threshold.

---

Supported compilers:
* MSVC 2015 or later, or Visual C++ Build Tools 2015 or later
* Clang 3.4.x or later
* GCC 4.8.x or later
* ICC 16 or later
