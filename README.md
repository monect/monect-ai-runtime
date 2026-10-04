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
| `image-runtime-v1.2.0-win-x64-cuda.zip` | Native FLUX.2 klein 4B generation/editing worker: [stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) + pinned ggml + CUDA, MSVC and OpenMP runtime DLLs | MIT (stable-diffusion.cpp, ggml), bundled dependency licenses, NVIDIA CUDA Toolkit EULA (cuBLAS/cudart) |
| `image-runtime-v1.2.0-win-x64-cpu.zip` | The same native image worker with CPU inference and MSVC/OpenMP runtime DLLs, without CUDA dependencies | MIT (stable-diffusion.cpp, ggml), bundled dependency licenses |
| `A_cute_futuristic_3D.glb` | Default 3D avatar model for the Monect AI avatar window and peer streaming | Monect in-house asset |

## Image generation and editing

Current receivers use **FLUX.2 klein 4B distilled** for both text-to-image
generation and one-reference instruction editing, with
[image runtime v1.2.0](https://github.com/monect/monect-ai-runtime/releases/tag/image-runtime-v1.2.0).

| Package | Download | Use |
| --- | --- | --- |
| CUDA | [image-runtime-v1.2.0-win-x64-cuda.zip](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.2.0/image-runtime-v1.2.0-win-x64-cuda.zip) | Selected by the receiver installer. At least 16 GB dedicated VRAM is recommended; lower/unknown VRAM permits an explicit experiment when CUDA is available. The 8 GB profile is experimental. Built with CUDA 13.3 for Turing and newer GPUs. |
| CPU | [image-runtime-v1.2.0-win-x64-cpu.zip](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.2.0/image-runtime-v1.2.0-win-x64-cpu.zip) | Selected by the receiver installer on Windows x64, independently of the chat backend. CPU inference is slower; broader CPU-only performance qualification remains pending. |

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
`%LOCALAPPDATA%\Monect\PC Remote Receiver\AIAgent\image-runtime\1.2.0\`.
New installers use `cpu/` or `cuda/` subdirectories and record the relative
worker filename in the model manifest. For manual deployment, match that
manifest path when extracting the selected archive.
The shared model package is under `AIAgent\image-models\flux2-klein-4b\1.0.0\`.
Its manifest records native protocol 2 and generation/editing capabilities;
legacy FLUX `instructionEdit` manifests are also recognized by current receivers.
Runtime `provenance.json` advertises `generate` and `instructionEdit`, with
`maxReferenceImages: 1`, `progressVersion: 1` and `detailedProgress`.

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

Version 1.2.0 pins stable-diffusion.cpp to
`3f8527a46c54ecf4cb4ed6003da8e8982283c73c` and ggml to
`89c4413f5da6fb20cc796f16033d37f129be81fd`, and includes the stb_image
decoder/license. It adds telemetry schema 1 and advertises `detailedProgress`; the immutable
1.1.0 release remains available for existing installations.

[SHA256SUMS.txt](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.2.0/SHA256SUMS.txt):

```text
9a78b8f4bd22b7f5b380d6a2b8d2247487db89bdc6571d3817da8d2672dec2a9  image-runtime-v1.2.0-win-x64-cpu.zip
6f2ac0b227c76c41fc5f79004e71fb4e323f99ffbeead0f6b3f80b5e5dd5340f  image-runtime-v1.2.0-win-x64-cuda.zip
```


### Detailed progress

Updated Windows and Android apps show live generation/editing progress in chat
and the image editor, with expandable details. Runtime 1.2.0 reports model
loading, source/instruction encoding, sampling, VAE decoding and saving;
tensor/step/tile counters when available; sampler step timings and remaining
sampling estimates; backend/CPU retry and attempt count; resolved size/seed;
elapsed/phase/attempt durations; and worker/system memory. One-second heartbeats
cover long operations without numerical callbacks. GPU free/total memory is
explicitly measured at backend selection, not presented as a live gauge.

Percentages describe the current phase. Completing sampling does not mean the
PNG is ready: decoding, saving, validation and publication still follow. No
whole-job percentage or total ETA is fabricated. Prompts, raw library logs and
host paths are excluded from progress events. Old workers/clients retain coarse
states; the Windows feature card offers an update that reuses verified weights.

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
