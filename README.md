# monect-ai-runtime

Distributable runtime artifacts for **Monect AI** (the on-device AI in PC Remote
Receiver). This repo hosts binary release assets only — no source code. The
main product repositories are private; the artifacts published here contain
only open-source and freely redistributable components.

## Releases

Each release is one versioned, immutable artifact (the tag and file name carry
the version and platform; published URLs are never re-pointed at different
bytes). Every archive ships a `provenance.json` recording the exact upstream
commits, build configuration and per-file SHA-256 hashes, plus the full
license texts.

| Artifact | Contents | Licenses |
| --- | --- | --- |
| `qwentts-runtime-v*-win-x64-cuda.zip` | Qwen3-TTS native inference runtime: [qwentts.cpp](https://github.com/ServeurpersoCom/qwentts.cpp) (qwen.dll, qt_* C ABI) + its pinned [ggml](https://github.com/ggml-org/ggml) submodule (CPU + CUDA backends) + NVIDIA cuBLAS/cudart redistributables | MIT (qwentts.cpp, ggml), NVIDIA CUDA Toolkit EULA (cuBLAS/cudart) |

## GPU-only policy

Qwen3-TTS is restricted to GPU (CUDA) installs of Monect AI: neural speech
synthesis on CPU is far too slow for interactive use. The installer enforces
this — CPU installs never download these artifacts and keep the built-in
Windows voices.

## Verification

Each release's notes carry the archive SHA-256. Inside, `provenance.json`
lists every DLL's SHA-256 together with the pinned upstream commits, so any
distributed copy can be re-verified independently.
