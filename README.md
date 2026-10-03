# monect-ai-runtime

Distributable runtime artifacts for **Monect AI** (the on-device AI in PC Remote
Receiver). This repo hosts binary release assets only — no source code. The
main product repositories are private; the artifacts published here are either
open-source components (with their license texts preserved inside each
archive) or Monect-owned assets.

## Releases

Each release is one versioned, immutable component and may contain CPU and CUDA
variants. Tags and archive file names carry the version and platform; published
URLs are never re-pointed at different bytes. Binary archives ship a
`provenance.json` recording the exact upstream commits, build configuration and
per-file SHA-256 hashes, plus the upstream license texts.

| Artifact | Contents | Licenses |
| --- | --- | --- |
| `qwentts-runtime-v*-win-x64-cuda.zip` | Qwen3-TTS native inference runtime: [qwentts.cpp](https://github.com/ServeurpersoCom/qwentts.cpp) (qwen.dll, qt_* C ABI) + its pinned [ggml](https://github.com/ggml-org/ggml) submodule (CPU + CUDA backends) + NVIDIA cuBLAS/cudart redistributables | MIT (qwentts.cpp, ggml), NVIDIA CUDA Toolkit EULA (cuBLAS/cudart) |
| `image-runtime-v1.1.0-win-x64-cuda.zip` | Native FLUX.2 klein 4B generation/editing worker: [stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) + pinned ggml + CUDA, MSVC and OpenMP runtime DLLs | MIT (stable-diffusion.cpp, ggml), bundled dependency licenses, NVIDIA CUDA Toolkit EULA (cuBLAS/cudart) |
| `image-runtime-v1.1.0-win-x64-cpu.zip` | The same native image worker with CPU inference and MSVC/OpenMP runtime DLLs, without CUDA dependencies | MIT (stable-diffusion.cpp, ggml), bundled dependency licenses |
| `A_cute_futuristic_3D.glb` | Default 3D avatar model for the Monect AI avatar window and peer streaming | Monect in-house asset |

## Image generation and editing

Current receivers use **FLUX.2 klein 4B distilled** for both text-to-image
generation and one-reference instruction editing, with
[image runtime v1.1.0](https://github.com/monect/monect-ai-runtime/releases/tag/image-runtime-v1.1.0).

| Package | Download | Use |
| --- | --- | --- |
| CUDA | [image-runtime-v1.1.0-win-x64-cuda.zip](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.1.0/image-runtime-v1.1.0-win-x64-cuda.zip) | Selected by the receiver installer; compatible NVIDIA CUDA GPU with at least 8 GB VRAM. The 8 GB profile is experimental. Built with CUDA 13.3 for Turing and newer GPUs. |
| CPU | [image-runtime-v1.1.0-win-x64-cpu.zip](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.1.0/image-runtime-v1.1.0-win-x64-cpu.zip) | Manual deployment; CPU-only editing performance is not qualified. |

Image installation is optional. It can be selected during Monect AI installation
or added from the Monect AI page afterwards. One generation/editing feature
card and install/resume/cancel flow use one model package, enabling both
operations without downloading a second set of image weights. An existing
verified FLUX editing installation also supplies generation. Older
Z-Image-Turbo installations show an upgrade option in current receivers,
and their existing generated images remain usable.

The installer verifies the runtime archive and all three model components
against pinned SHA-256 hashes. It reuses the exact verified Qwen3-4B chat
encoder when available; the denoiser/VAE download is approximately 2.8 GB.
Both operations use four sampling steps and CFG 1.0. No model weights or user
images are bundled in the runtime archives. The FLUX.2 klein 4B model, Qwen3-4B
encoder and repackaged VAE are published under Apache 2.0; the receiver installs
their license and attribution notices separately from the runtime licenses.

The runtime is installed under
`%LOCALAPPDATA%\Monect\PC Remote Receiver\AIAgent\image-runtime\1.1.0\`.
For manual deployment, extract the selected archive directly into that folder.
The shared model package is under `AIAgent\image-models\flux2-klein-4b\1.0.0\`.
Its manifest records native protocol 2 and generation/editing capabilities;
legacy FLUX `instructionEdit` manifests are also recognized by current receivers.
Runtime `provenance.json` advertises `generate` and `instructionEdit`, with
`maxReferenceImages: 1`.

Windows and Android users can choose a PNG/JPEG or a newly generated image,
describe a change, compare the source and result, continue editing that
version, and save the result. Inference stays on the connected Windows PC.
Original inputs remain intact and completed edits become new PNG versions.
Direct editing does not require the chat model to be loaded. Masks, multiple
references and outpainting are not supported in this release.

Text encoding and tiled VAE processing run on CPU. Diffusion uses CUDA when
available with sufficient free VRAM; a failed CUDA initialization or generation
attempt releases its resources and retries once on CPU with the same prompt,
seed and image size. CPU generation can take several minutes.

Each job retains native diagnostics in a bounded `worker.log` under
`%LOCALAPPDATA%\Monect\PC Remote Receiver\AIAgent\image-jobs\<job-id>\`.
The adjacent `worker-state.json` records job status, operation, backend,
attempt count and diagnostic message when generation fails.

English and Chinese edits of an illustrated portrait were measured on an
RTX 4060 Laptop GPU (8 GB), with 16 GB system RAM: approximately 52 seconds at
512×512 and 221 seconds at 1024×1024, including loading. A 768×512 edit took
approximately 66 seconds. These measurements cover editing on one machine;
they are not a comparative generation benchmark. Other GPU profiles,
real-photo quality and CPU-only editing remain unverified.

Version 1.1.0 pins stable-diffusion.cpp to
`3f8527a46c54ecf4cb4ed6003da8e8982283c73c` and ggml to
`89c4413f5da6fb20cc796f16033d37f129be81fd`, and includes the stb_image
decoder/license. Its immutable archive bytes are unchanged by the receiver's
switch to FLUX generation.

[SHA256SUMS.txt](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.1.0/SHA256SUMS.txt):

```text
58b38ccbbb7adad8463e455baa28f3cb34312127c4897be1d2608338a07ba736  image-runtime-v1.1.0-win-x64-cpu.zip
a3dd5d8fc40aac12649b1227ba9c03c9f103f0c66d5acc648e0e885c43923daa  image-runtime-v1.1.0-win-x64-cuda.zip
```

### Legacy Z-Image-Turbo compatibility

[Image runtime v1.0.0](https://github.com/monect/monect-ai-runtime/releases/tag/image-runtime-v1.0.0)
is a legacy release retained for older Z-Image-Turbo receivers. Its model
weights are downloaded separately by those receivers. New receivers use FLUX.2
klein 4B. Runtime 1.1.0 also retains native compatibility with older Z-Image
generation requests. Current receivers use the shared FLUX package and install configuration schema 3, with
one `image` section. Matching legacy FLUX `imageGeneration`/`imageEditing`
sections are accepted and normalized; conflicting sections are rejected. Older
receivers reject schema 3 and retain their compiled defaults. The service keeps
separate generation/editing operation status fields for Android compatibility.

## Qwen3-TTS GPU policy

Qwen3-TTS is restricted to GPU (CUDA) installs of Monect AI: neural speech
synthesis on CPU is far too slow for interactive use. The installer enforces
this — CPU installs never download the Qwen3-TTS runtime and keep the built-in
Windows voices. Image generation has the CPU fallback described above.

## Verification

Release notes carry archive SHA-256 hashes. Image runtime v1.0.0 also publishes
[SHA256SUMS.txt](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.0.0/SHA256SUMS.txt):

```text
40292cf022e7c2e745c77423de894b3c92ae6e68196e31a0ad9af8d7542ed1ea  image-runtime-v1.0.0-win-x64-cpu.zip
5ba694021cf04c882de5e68c1fef4334e9721ec68e9a76c641e604798b966324  image-runtime-v1.0.0-win-x64-cuda.zip
```

On Windows, use `Get-FileHash -Algorithm SHA256 <archive-path>` to compare a
download with the published checksum. Inside each runtime archive,
`provenance.json` lists the executable, DLL and license file hashes together
with the pinned upstream commits, so extracted files can also be verified.

## Updating a runtime

Publish a new versioned release for each runtime update. Current generation
and editing share one package: update the `image` section
(runtime version, download URL, archive SHA-256 hash and pinned model
URLs/byte lengths/hashes) and matching Rust defaults together. Keep the hosted
`pc-receiver/ai-agent-install-config.json` synchronized. Preserve existing
release assets; publish a new immutable version instead of replacing an archive.
