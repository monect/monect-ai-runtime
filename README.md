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
| `image-runtime-v*-win-x64-cuda.zip` | Native image generation/editing worker (Z-Image-Turbo and FLUX.2 klein): [stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) + pinned ggml + CUDA, MSVC and OpenMP runtime DLLs | MIT (stable-diffusion.cpp, ggml), bundled dependency licenses, NVIDIA CUDA Toolkit EULA (cuBLAS/cudart) |
| `image-runtime-v*-win-x64-cpu.zip` | The same native image worker with CPU inference and MSVC/OpenMP runtime DLLs, without CUDA dependencies | MIT (stable-diffusion.cpp, ggml), bundled dependency licenses |
| `A_cute_futuristic_3D.glb` | Default 3D avatar model for the Monect AI avatar window and peer streaming | Monect in-house asset |

## Image generation

[Image runtime v1.0.0](https://github.com/monect/monect-ai-runtime/releases/tag/image-runtime-v1.0.0)
is available for Windows x64:

| Package | Download | Use |
| --- | --- | --- |
| CUDA | [image-runtime-v1.0.0-win-x64-cuda.zip](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.0.0/image-runtime-v1.0.0-win-x64-cuda.zip) | Selected by the receiver installer; built with CUDA 13.3 for NVIDIA Turing and newer GPUs |
| CPU | [image-runtime-v1.0.0-win-x64-cpu.zip](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.0.0/image-runtime-v1.0.0-win-x64-cpu.zip) | Manual CPU deployment |

Image generation is optional. It can be selected during Monect AI installation
or added from the Monect AI page after AI is installed. The receiver downloads
and extracts the runtime before downloading the separate Z-Image-Turbo
denoiser, Qwen3 text encoder and VAE model weights. These weights are not
included in either runtime archive. An existing runtime of the requested
version is reused.

The worker and its dependencies are installed under
`%LOCALAPPDATA%\Monect\PC Remote Receiver\AIAgent\image-runtime\1.0.0\`.
For manual deployment, extract the selected archive directly into that folder.

Text encoding and tiled VAE decoding run on CPU. Diffusion uses CUDA when
available with sufficient free VRAM; a failed CUDA initialization or generation
attempt releases its resources and retries once on CPU with the same prompt,
seed and image size. CPU generation can take several minutes.

Each job retains native diagnostics in a bounded `worker.log` under
`%LOCALAPPDATA%\Monect\PC Remote Receiver\AIAgent\image-jobs\<job-id>\`.
The adjacent `worker-state.json` records job status and includes the backend,
attempt count and diagnostic message when generation fails.

Version 1.0.0 pins stable-diffusion.cpp to
`3f8527a46c54ecf4cb4ed6003da8e8982283c73c` and ggml to
`89c4413f5da6fb20cc796f16033d37f129be81fd`.

## Image editing

[Image runtime v1.1.0](https://github.com/monect/monect-ai-runtime/releases/tag/image-runtime-v1.1.0)
adds local, one-reference instruction editing with the separately installed
**FLUX.2 klein 4B distilled** model package. It also supports existing
Z-Image-Turbo generation requests. Version 1.0.0 remains available unchanged.

| Package | Download | Use |
| --- | --- | --- |
| CUDA | [image-runtime-v1.1.0-win-x64-cuda.zip](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.1.0/image-runtime-v1.1.0-win-x64-cuda.zip) | Receiver's experimental image-editing add-on; compatible NVIDIA CUDA GPU with at least 8 GB VRAM |
| CPU | [image-runtime-v1.1.0-win-x64-cpu.zip](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.1.0/image-runtime-v1.1.0-win-x64-cpu.zip) | Manual deployment; CPU-only editing performance is not qualified |

Editing can be installed from the Monect AI page after AI is installed.
Windows and Android users can choose a PNG/JPEG or a newly generated image,
describe a change, compare the source and result, continue editing that
version, and save the result. Inference stays on the connected Windows PC.
Original inputs remain intact and completed edits become new PNG versions.
Direct editing does not require the chat model to be loaded. Masks, multiple
references and outpainting are not supported in this release.

The installer verifies the runtime archive and all three model components
against pinned SHA-256 hashes. It reuses the exact verified Qwen3-4B chat
encoder when available; the approximately 2.8 GB denoiser/VAE download remains
separate. No weights or user images are bundled in these runtime archives.

The worker is installed under
`%LOCALAPPDATA%\Monect\PC Remote Receiver\AIAgent\image-runtime\1.1.0\`.
Its `provenance.json` advertises native protocol 2, `generate` and
`instructionEdit`, with `maxReferenceImages: 1`. The editing model manifest
must match this protocol. Version 1.1.0 uses the same upstream
stable-diffusion.cpp/ggml commits listed for 1.0.0 above, and adds the bundled
stb_image decoder/license. Each job retains its diagnostics under
`AIAgent\image-jobs\<job-id>\`.

English and Chinese edits of an illustrated portrait were measured on an
RTX 4060 Laptop GPU (8 GB), with 16 GB system RAM: approximately 52 seconds at
512×512 and 221 seconds at 1024×1024, including loading. A 768×512 edit took
approximately 66 seconds. Text encoding and tiled VAE decoding run on CPU;
diffusion uses CUDA, with the single CPU retry described above. These are
measurements from one machine, not general performance or quality guarantees.
Other GPU profiles, real-photo quality and CPU-only editing remain unverified.

Version 1.1.0 publishes
[SHA256SUMS.txt](https://github.com/monect/monect-ai-runtime/releases/download/image-runtime-v1.1.0/SHA256SUMS.txt):

```text
58b38ccbbb7adad8463e455baa28f3cb34312127c4897be1d2608338a07ba736  image-runtime-v1.1.0-win-x64-cpu.zip
a3dd5d8fc40aac12649b1227ba9c03c9f103f0c66d5acc648e0e885c43923daa  image-runtime-v1.1.0-win-x64-cuda.zip
```

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

Publish a new versioned release for each update. For image generation, update
`imageGeneration.runtimeVersion`, `imageGeneration.runtimeDownloadUrl` in the
receiver's `pc-receiver/ai-agent-install-config.json`, and the compiled-in
defaults in `pc-receiver/src-tauri/src/main.rs` together. Keep the hosted install
configuration in sync and preserve existing release assets. For editing, update
`imageEditing.runtimeVersion`, `runtimeDownloadUrl` and `runtimeSha256`,
the pinned component URLs/byte lengths/hashes, and matching Rust defaults
together. Publish a new immutable version instead of replacing an archive.
