# VIEANA — MASTER ENGINEERING BLUEPRINT

**Document type:** authoritative architecture / implementation contract  
**Product:** Vieana  
**Target:** Android APK only  
**Application code:** native C++26 (`-std=c++2c`)  
**Project status:** architecture and scaffold; implementation has not started merely because files exist  
**Reference archive:** `effectcraft-mobile-main.zip`, inspected from its actual workspace and source manifests  

---

## 0. Purpose and non-negotiable rules

Vieana is an independent motion-graphics, compositing, visual-effects, timeline, audio and 3D application. EffectCraft is a capability and architecture reference only. Vieana is not a Rust-to-C++ translation, a fork, a rebrand, or a line-by-line port. Its algorithms, data structures, file formats, UI, engine and implementation must be designed and authored independently.

### Product rules

1. Android APK is the only product target. No desktop, CLI product, browser, PWA or iOS target is included in the product roadmap unless a future explicit decision changes this rule.
2. Application logic and engine implementation are C++26. No Rust, Kotlin, Java, JavaScript or Python runtime is part of the shipped application.
3. No JNI bridge, Java/Kotlin application bridge, or third-party app framework is to be introduced.
4. No FFmpeg, Android MediaCodec/NDK Media, Qt, ImGui, Skia, OpenCV, FreeType, HarfBuzz, SDL, GLFW, WebView or third-party editor/render engine is to be used.
5. Android system interfaces and hardware drivers remain platform boundaries, not application engines. NativeActivity, Bionic, the Android loader, Vulkan loader/driver, file/permission APIs, and native audio output are OS/hardware interfaces that cannot honestly be described as “zero dependencies”.
6. GPU-first is the preferred rendering path. A CPU reference renderer remains necessary for correctness comparisons, fallback, deterministic tests and devices with unsuitable GPU capability.
7. Testing artifacts never belong under `Vieana/`. Build output never belongs under `Vieana/`. These are permanent directory rules.
8. A subsystem is not complete because it compiles once. It must pass correctness, boundary, malformed-input, memory, concurrency, determinism, stress, integration, performance and Android-device validation appropriate to its scope.
9. Do not copy EffectCraft code, shaders, art, presets, bundled fonts or third-party implementation. Re-implement behavior from independently written specifications and tests. Audit licenses, attribution and codec patent obligations before deciding to ship a format implementation.
10. Never claim that a blank scaffold file is an implemented feature.

### Toolchain and platform boundary

Use an Android-compatible Clang toolchain and CMake/Ninja or equivalent build orchestration to produce ARM64 Android binaries. The NDK compiler/toolchain is distinct from the forbidden Android MediaCodec APIs. NativeActivity may be used to avoid writing a Java/Kotlin Activity, but Android packaging metadata such as `AndroidManifest.xml` and APK signing are still required. Vieana-owned C++ code must own lifecycle, input dispatch, window/surface management, storage abstraction, rendering and audio policy while calling supported Android native OS interfaces.

Vulkan shaders cannot be ordinary C++ code executing directly on a GPU. Vieana may own a compact shader language and compiler in C++ which emits a GPU-compatible intermediate/binary format (for Vulkan, SPIR-V). This is an explicit compiler boundary, not a claim that the GPU executes host C++.

---

## 1. What was actually found in the reference archive

The supplied ZIP contains a Cargo workspace at version `0.16.0`, Rust edition 2024, with **32 crates**, three app workspaces (`effectcraft`, `effectcraft-cli`, `effectcraft-web`) and `xtask`. The crate source inventory is preserved exactly in `Documentation/Research/EffectCraft_Exact_Source_Inventory.md`. The archive includes source files, tests, docs, assets, packaging directories and WGSL shaders; this blueprint does not copy those implementation files.

EffectCraft's own README describes it as an alpha motion-graphics/VFX compositor. The README lists compositions, layers, nested compositions, keyframes, graph editor, motion paths, masks, track mattes, 38 blend modes, motion blur, text animators, expressions, puppet tools, 3D cameras/lights, a render queue, audio preview and 306 effects. It describes `.ecproj` as versioned JSON and reports Lottie import/export. It also states that there is no FFmpeg and describes pure-Rust media/codec paths, some from a separate FilmCraft repository. These are statements from the archive, not independent performance or quality certification.

### 1.1 Exact crate-to-Vieana map

| EffectCraft crate | Vieana destination | Responsibility to preserve / reimplement |
|---|---|---|
| `automation` | `automation/`, `engine/jobs/`, `engine/history/` | Autosave, long jobs, tool registry, bridge boundaries, constrained automation and job state. No default network listener. |
| `av1enc` | `media/codecs/av1/encoder/` | AV1 bit writing, CDF tables, decisions, loop filters, prediction, symbol coding, transforms and output. |
| `color` | `color/`, `rendering/color/` | Color spaces, blending, ICC parsing, transform pipeline and HDR/gamut policy. |
| `effects` | `effects/` | Effect registry/catalog, parameter controls, effect implementations and category-specific systems. |
| `engine` | `engine/editor/`, `engine/commands/`, `engine/history/`, `timeline/`, `ui/panels/` | Session/state, command registry, undo/redo, selection, project/media workflows, render queue and panel model. |
| `export` | `export/`, `media/mux/` | Output pipeline, encoders, output modules, uncompressed paths and WebM output. |
| `expr` | `scripting/expressions/` | Expression grammar, runtime, context, function model, errors and deterministic evaluation. |
| `geom` | `geometry/`, `engine/math/` | Geometry primitives and robust numeric operations. |
| `gpu` | `rendering/gpu/`, `shaders/` | Device/context, render operations, effect kernels, particles, classic/advanced 3D and shader compilation. |
| `hevcenc` | `media/codecs/hevc/encoder/` | Bitstream, CABAC, prediction, motion estimation, deblocking, transforms and parameter controls. |
| `host` | `host/`, `platform/android/` | Host directories, OS adaptation, device information and environment. |
| `interchange` | `interchange/` | Format dispatch, import/export routing and conversion policy. |
| `keyframe` | `animation/keyframes/` | Keyframe values, interpolation and animated text document model. |
| `lottie` | `interchange/lottie/` | Lottie model, import/export, vector shapes, text, animation and packaged file support. |
| `media` | `media/`, `image/codecs/`, `sequences/` | Probe, streams, conversion, pools, animation containers, layered data and specialized image/media types. |
| `model` | `scene3d/model/` | OBJ/FBX/glTF/STL import, primitive generation, extrusion and triangulation. |
| `opusenc` | `media/codecs/opus/encoder/` | Opus bands, energy, MDCT, range coding, rate control, resampling and SILK components. |
| `path` | `geometry/path/` | Boolean path operations, offset, feather, fit, interpolation, tracing and variable strokes. |
| `pdf` | `interchange/pdf/` | PDF objects, page tree, parsing/writing, fonts, images, color, shading and encodings. |
| `plugin` | `plugins/` | Plugin contracts, validation, versioning and sandbox boundaries. Do not adopt WASM unless explicitly approved later. |
| `project` | `project/`, `asset/`, `export/` | Project construction, properties, limits, file validation, sequences, render templates, styles and tracking metadata. |
| `psd` | `interchange/psd/` | PSD layers, descriptors, engine data, placed layers, readers/writers and defensive memory limits. |
| `raster` | `raster/`, `image/` | Blur, channels, cuts, flow, inpainting, scopes, warp and raster operations. |
| `render` | `rendering/`, `rendering/cpu/`, `rendering/compositor/` | Evaluation, render passes, caches, sources, masks, styles, audio, text, shapes, nested comps, 2D/3D and blending. |
| `script` | `scripting/runtime/`, `scripting/ui/` | Script files, include, execution, tasks, socket/control abstractions, script UI and services. The shipping runtime remains C++ only. |
| `segment` | `segmentation/` | Face/object segmentation abstractions and model handling. No external inference framework may be silently introduced. |
| `svg` | `interchange/svg/` | SVG path data, bounds, color and rasterization. |
| `text` | `typography/` | Font discovery/parsing, text layout, selectors, path text and variable fonts. |
| `time` | `engine/time/` | Shared time representation and frame/tick conversion. |
| `track` | `tracking/` | KLT/features, plane and camera tracking, bundle solving, stabilization, face tracking and roto matting. |
| `ui-egui` | `ui/` | Docking, panels, widgets, menus, preferences, localization, viewer, timeline and workspace state. The egui implementation itself is not reused. |
| `vp9enc` | `media/codecs/vp9/encoder/` | VP9 boolean coding, loop filter, motion compensation, probability tables, tiles and transforms. |

### 1.2 Features to preserve in the Vieana plan

**Project/editor:** versioned native project format; project item hierarchy; footage and proxy management; sequences; compositions; nested compositions; layers; parenting; adjustment layers; nulls; camera/light layers; history; command registry; autosave; recovery; render queue; templates; preferences; recent projects; asset relinking; import diagnostics.

**Animation/timeline:** scalar/vector/color/transform properties; linear, hold, Bezier, auto/continuous Bezier, custom interpolation, Easy Ease, velocity controls, roving keys, spatial keys, key labels, copy/paste, motion paths, graph editor with value and speed modes, tangent editing, frame-accurate timeline, rational frame rates, drop-frame timecode, work area, markers, trim, stretch, reverse, freeze frame and time remap.

**Shape/mask/paint/puppet:** vector shape layers; primitive and freeform paths; trim paths; repeaters; round corners; offset; zigzag; twist; wiggle; merge paths; gradient fills/strokes; editable masks; mask modes/feather/expansion; paint strokes; clone/heal; puppet mesh; ARAP-like deformation; pin types; pin/null relationships; roto and mask propagation.

**Effects:** blur/sharpen; channels; color correction and grading; distort/warp/liquify/perspective; generators; noise/grain; keying; matte/object matte/crypto selection; lighting/glow/bloom/rays/shadows; perspective/card-3D/VR; simulations and particles; temporal effects; text effects; transitions; audio effects; layer styles. The archive advertises 306 effects—Vieana must perform its own feature-by-feature inventory and cannot assume all 306 exist or reach production parity merely by creating a folder.

**3D:** 3D layers, camera rigs, one/two-node camera concepts, perspective/orthographic views, lights, materials, shadows, depth-of-field, bokeh, model import, geometry/extrusion/triangulation, classic and advanced scene rendering, environment lighting, particles and stereoscopic/VR projection concepts.

**Tracking/segmentation:** feature detection and matching; KLT; optical flow; planar tracking; camera solving/bundle adjustment; stabilization; face landmarks; Roto Brush; object/foreground segmentation; matte refinement/propagation; model cache and inference memory management.

**Media/export:** image stills and sequences; GIF/WebP animation; DPX/EXR; MP4/MOV/MKV/WebM/AVI/WAV containers; demux/mux; H.264/HEVC/AV1/VP9; AAC/Opus/FLAC/PCM; audio resampling/downmix; alpha and high-bit-depth paths; image sequences; render queue; output modules; integrity verification; storage limits and crash recovery.

**Typography/interchange:** SFNT/OTF/TTF parsing; glyph metrics; Unicode properties; grapheme segmentation; bidi; script/language runs; shaping; kerning/ligatures; font fallback; variable fonts; text on path; SVG; PSD; PDF; Lottie; model interchange; future EDL/FCPXML/OTIO if formats are deliberately scoped.

**UI/automation:** native custom UI; panel docking; timeline and graph editor; viewer tools; widgets; localization; theme; shortcuts; accessibility; workspace persistence; internal commands; automation IDs; optional local-only control/automation API, disabled by default.

---

## 2. Vieana architecture layers and dependency direction

The intended dependency direction is one-way. Lower layers must never depend on editor UI or application panels.

1. **Platform & host (`platform/android`, `host`)** — lifecycle, NativeActivity entry, surface/window, input, storage/permissions, native audio output, device/thermal information.
2. **Core (`engine/core`, `math`, `time`, `io`, `containers`, `diagnostics`, `logging`)** — stable primitives; no UI references.
3. **Memory & concurrency (`engine/memory`, `engine/threading`, `engine/jobs`)** — allocators, ownership, queues, scheduler, cancellation, fences, budgets and safe shutdown.
4. **Data model (`project`, `asset`, `composition`, `timeline`, `animation`, `geometry`)** — serializable editor state, immutable identifiers, transactions and validation.
5. **Processing (`image`, `raster`, `color`, `media`, `audio`, `typography`, `tracking`, `segmentation`, `scene3d`)** — pure domain algorithms and explicit platform-neutral interfaces.
6. **Render (`rendering`, `effects`, `shaders`)** — graph evaluation, CPU/GPU execution, compositor, cache, color pipeline and effect scheduling.
7. **Delivery (`export`, `interchange`, `formats`)** — file readers/writers, muxers, output validation and compatibility reports.
8. **Editor interaction (`engine/editor`, `engine/commands`, `engine/history`, `automation`)** — command-based editing, undo/redo, autosave, selection and task state.
9. **UI (`ui`)** — custom native widgets/panels; UI calls command and query interfaces, never mutates renderer internals directly.
10. **Application (`app`)** — startup, lifecycle orchestration, project opening and top-level service composition.

### 2.1 Architectural invariants

- Model state is authoritative; UI state is a projection plus transient interaction state.
- Every durable edit goes through a validated command/transaction with undo and redo semantics.
- Rendering reads immutable snapshots or versioned state views. A worker must not observe partially mutated project data.
- Resource identity uses stable IDs, not pointers into resizable containers.
- Time is rational/integer based internally. Floating-point seconds are presentation/interchange values, not authoritative timeline identity.
- Cache keys include all semantic dependencies: source revision, effect parameters, frame/time, color transform, render scale, proxy state and nested composition instance route.
- CPU and GPU paths must share a documented semantic contract. Differences require explicit tolerances and tests.
- GPU resources are owned by a device/context lifetime. Device loss/recreation must invalidate dependent resources safely.
- Decoder, parser and importer inputs are untrusted. Every count, offset, allocation, recursion depth, packet size and dimension must be bounded.
- Long operations are cancellable and must release temporary resources on cancellation, error, app backgrounding or low-memory events.
- UI frame scheduling must not block behind export; exports use bounded queues and memory budgets.

---

## 3. Core data models and formats

### 3.1 Project identity and persistence

Design a Vieana-owned project container (suggested extension `.vieana`). The internal schema must be documented before serialization code is written. It should carry a magic/version header, schema version, feature flags, project UUID, timebase, color configuration, assets, compositions, layers, animation, effects, audio routing, render queue and app-generated metadata. Choose between canonical binary chunks plus a readable manifest or canonical JSON-like text only after benchmarking startup, crash resilience, diffability and migration cost. Do not silently copy `.ecproj` or its schema.

Required properties:
- atomic save via temp file + flush/fsync policy + rename where supported;
- journal/checkpoint recovery;
- forward/backward version policy;
- migration reports and unsupported-feature preservation;
- checksums and bounded chunk sizes;
- stable IDs and referential integrity;
- missing media relink workflow;
- corruption detection before mutating project state;
- autosave cadence and storage quotas;
- deterministic serialization tests.

### 3.2 Time model

Use integer ticks with a declared ticks-per-second constant or a rational timebase. Frame rates are rational numerator/denominator pairs, including fractional rates such as 30000/1001. Conversion must avoid cumulative floating-point drift. Drop-frame timecode is formatting/labeling logic, not dropped stored frames. All render requests specify an unambiguous time, source time mapping, subframe policy and frame rate.

### 3.3 Composition/layer model

A composition owns canvas dimensions, pixel aspect, frame rate, duration, color config, layer graph and work area. A layer has a stable ID, type, enabled/solo/shy/lock/guide flags, in/out points, source mapping, transform, parent relation, effects, masks, matte references and animation properties. Nested composition instances need per-instance animation/route context: one shared source composition may be evaluated differently by multiple instances without mutating the source graph.

### 3.4 Render graph

Represent rendering as a dependency graph with explicit inputs/outputs and resource lifetimes. A typical semantic ordering to validate is source acquisition → source interpretation/color conversion → masks and source operations → effects and layer styles → transforms/3D projection → motion blur/temporal processing → matte resolution → blend/composite → output color/display transform. The exact pass order is a Vieana design decision; effect categories must declare ordering constraints and pre/post-transform behavior rather than relying on ad-hoc list order.

### 3.5 Color

Make color space and alpha interpretation explicit for every image/frame. Define transfer functions, primaries, white point, bit depth, scene/display-referred domains, premultiplication, HDR reference luminance and tone mapping. Test blending in linear light where required. ICC profile parsing must be defensive. Do not conflate an image's encoded pixel format with its working color space.

---

## 4. Rendering: CPU reference and Vulkan GPU

### CPU reference path

Build a correct reference implementation first for foundational pixel operations and compositing. Include scalar implementation as the golden baseline; optimized SIMD/parallel paths may follow. CPU renderer is not a discarded fallback: it is the test oracle for supported operations and can help isolate GPU discrepancies.

### GPU path

The Vulkan layer owns instance/device selection, queue families, capability discovery, descriptor/pipeline cache, buffer/image allocation, synchronization, barriers, staging/readback, shader compilation, device loss handling and recovery. It must query capabilities instead of assuming a device class. Avoid persistent allocations without a budget. Support graceful feature fallback.

### Shader compiler

Define a versioned internal shader grammar, type checker, reflection system, compiler diagnostics and deterministic output. Validate memory access, resource binding, numerical range, precision requirements and dispatch bounds. Shader compile output lives under `Build/Shaders`, never source. Maintain a known-good binary cache and invalidate it when shader source, compiler version, device capability or pipeline key changes.

### Renderer acceptance

- CPU golden tests for every foundational kernel.
- GPU-vs-CPU comparison with per-operation error budgets.
- Boundary images: 1x1, odd dimensions, non-power-of-two, maximum supported dimensions, alpha-only, HDR, negative/overshoot values where valid.
- Tiled and full-frame output equivalence within declared tolerance.
- Render graph cycle detection and dependency validation.
- Device loss/recreation; surface resize/orientation; app background/foreground.
- Multi-layer and nested-composition route tests.
- Cancellation and memory pressure tests.
- Benchmarks with thermal state, resolution, render scale and device metadata recorded.

---

## 5. Effects system

Each effect must have a stable identifier, schema version, category, display metadata, typed parameters, defaults, bounds, animation policy, CPU implementation availability, GPU implementation availability, color-space requirements, alpha behavior, temporal behavior, ROI expansion, cache dependencies, serialization contract, error behavior and test vectors.

Required execution features:
- effect stack reorder/enable/disable/bypass;
- parameter animation and keyframe interpolation;
- parameter validation and safe migration;
- CPU/GPU implementation registration;
- ROI and tile dependency declaration;
- temporal frame history where needed;
- preset serialization and compatibility reporting;
- deterministic mode for regression tests;
- diagnostic timing and memory counters.

Never claim a catalog count as a completed implementation count. A catalog entry is complete only when semantic spec, UI controls, serialization, CPU/GPU policy, validation, rendering tests and performance tests exist.

### Effect families

Blur; sharpen; color correction/grading; channel operations; distortion/warp; keying; matte refinement/object matte/cryptomatte; noise/grain; lighting/glow/bloom; stylize; transitions; generators; particles; physical simulation; temporal effects; paint/clone/heal; puppet deformation; roto; typography effects; spatial/VR; audio effects; layer styles; utility/expression controls.

---

## 6. Media and codec strategy

A fully independent codec stack is a multi-year, conformance-heavy engineering program. Do not create empty H.264/HEVC/AV1/VP9/AAC/Opus folders and then treat those formats as supported. Each codec has distinct entropy coding, prediction, transforms, reference management, packetization, bit depth, color metadata, profile/level restrictions and malformed-stream attack surface.

### Safe implementation sequence

1. Raw pixel buffers, PNG/BMP-class simple image formats and image sequences; PCM/WAV for audio; robust timestamp/container foundations.
2. Image formats with broader metadata and animation; lossless audio (subject to independent design and conformance corpus).
3. Demux/container readers with strict bounds; decoder architecture and reference-frame management.
4. One target video codec at a time, starting with a consciously chosen profile subset; complete decoding before broad encoder claims.
5. Audio codecs one at a time; verify channel mapping, resampling, delay/padding, metadata and round-trip behavior.
6. Encoder optimizations only after bit-exact or conformance-valid output, memory safety and measurable speedup.

For every codec maintain: public specification references, exact supported profile/level/bit-depth/chroma list, conformance input corpus, reference outputs, malformed corpus, fuzz harness, decode/encode round-trip checks, resource limits, performance baseline and legal/patent review. Codec patents and distribution rights need jurisdiction-specific legal review; “written from scratch” does not automatically remove patent obligations.

No NDK MediaCodec path is to be used. Native Android audio output is a separate OS boundary and not a media decoder substitute.

---

## 7. Typography, text and Unicode

A custom typography stack is not just glyph drawing. It requires safe SFNT/OTF/TTF parsing; table validation; Unicode scalar decoding; grapheme segmentation; script and language itemization; bidirectional ordering; shaping; substitutions/ligatures; positioning/kerning; variation axes; fallback; line breaking; paragraph layout; text-on-path; glyph cache; rasterization; hinting policy; atlas management; and color/emoji fallback policy. The initial release must clearly state the supported font table set and script coverage. Do not imply all fonts/languages work before a broad test corpus has passed.

---

## 8. Audio

Separate editor audio timeline semantics, PCM buffers, DSP graph, mixer, sample-rate conversion, channel layout, automation, waveform analysis and platform output. Keep processing sample-accurate within the selected timeline/sample-rate model. Handle underruns, device route changes, background/foreground transitions and audio-clock drift. The output interface must be a thin native Android platform adapter; mixing and DSP remain Vieana-owned C++.

---

## 9. Tracking, segmentation and model inference

Start with deterministic classical algorithms: feature detection/matching, pyramids, optical flow, KLT, planar transform fitting and robust camera solve. Define confidence, outlier rejection, failure modes and reproducible test clips. Segmentation/model inference must have a replaceable internal backend boundary but cannot quietly introduce OpenCV, MediaPipe, TFLite or external frameworks. If a neural model is later supported, audit its format, weights license, runtime implementation, memory, device compatibility and download consent. Face/object data should remain local unless the user explicitly exports it.

---

## 10. UI and interaction architecture

Implement a retained or hybrid immediate/retained C++ UI based on a documented element tree, layout constraints, hit testing, invalidation, event routing, gesture recognizers, focus, accessibility semantics, text input, clipboard, docking tree, workspace serialization and a GPU-backed renderer. Avoid using a named forbidden UI framework. UI panels must be views over model/query interfaces and send commands for edits. Timeline and graph editor must have independent interaction models and testable gesture state machines.

Key panels to retain: Project/Media Browser, Composition/Viewer, Timeline, Layer, Effect Controls, Effects & Presets, Properties, Graph Editor, Audio/Waveform, Render Queue, Character/Paragraph/Text, Align, Info/Scopes, Tracking/Camera, Paint, Roto, Puppet, Shape, Model/3D, Script/Automation, Settings, Learn/Help.

The Android UI must account for touch-first operation, high-DPI scaling, safe areas/insets, keyboard attachment, gesture conflict, accessibility and constrained visible area. The complete editor remains the product workspace; do not create a separate “beginner mode”.

---

## 11. Interchange and file compatibility

Implement separate bounded parsers/writers and validators for each format. PSD, PDF, SVG, Lottie, EXR, DPX, model formats and timeline interchange are not a single generic file parser. Each needs its own conformance matrix, resource limits, error reporting, import fidelity expectations and unsupported-feature report. Preserve unknown-but-safe metadata where feasible; never allocate from unchecked file-supplied dimensions/counts.

After Effects proprietary `.aep/.aepx` parsing is not assumed. It requires a separate clean-room, license and technical decision. No proprietary project support can be promised from an archive feature list.

---

## 12. Testing and validation policy (outside source)

All test code and artifacts live under `Testing/`, not `Vieana/`. Test sources may include Vieana public headers and link internal test builds, but fixtures, generated files, logs, coverage, render output and crash dumps remain external.

Test tiers:
- **Unit:** math, time, parser, serializer, allocator, algorithms and effect kernels.
- **Integration:** project → timeline → renderer → cache → export; audio playback; Android surface and lifecycle.
- **Regression:** every fixed bug gets a focused test and, where applicable, a golden render/audio output.
- **Golden/fidelity:** small deterministic compositions with documented input, output and tolerance; visually reviewed cases are recorded as such, not called automated proof.
- **Stress:** layer count, nested depth, long timeline, huge frames, memory pressure, parallel render, long-running exports and thermal throttling.
- **Fuzz:** project, media/container, codec bitstream, font, SVG, PDF, PSD and interchange parsers.
- **Memory:** leaks, stale references, double free, use-after-free, fragmentation, budget enforcement and cancellation cleanup.
- **GPU:** API validation, synchronization, barriers, format support, shader compilation, device loss and GPU/CPU equivalence.
- **Device compatibility:** Android API levels, GPU vendors/architectures, memory classes, refresh rates, rotation, audio routes and thermal behavior.
- **Performance:** reproducible release benchmarks with device/build/configuration/input metadata; never compare numbers without same conditions.

### Definition of done

A feature is complete only when: specification exists; API and serialization are versioned; happy path and edge cases are implemented; malformed and failure paths are safe; cancellation/cleanup is verified; thread ownership is documented; unit/integration/regression tests pass; GPU and CPU behavior are compared if both exist; benchmark is recorded; memory budget is measured; Android device matrix is tested; user-visible limitations are documented.

---

## 13. Build and directory governance

```text
Vieana-Workspace-Complete/
├── Vieana/          # production C++26 source, internal shader source, production assets only
├── Testing/         # every test and test artifact; never inside Vieana/
├── Build/           # object files, generated code, SPIR-V, libraries, APK, caches
└── Documentation/   # specs, architecture, research, APIs and engineering policies
```

`Vieana/` contains no test directories, benchmark output, generated APK, profiling capture, crash dump or temporary build data. `Testing/` never stores production source. `Build/` can be deleted and regenerated without losing source or tests. `Documentation/` records decisions and implementation evidence.

CMake should be modular with small `CMakeLists.txt` files per subsystem. Do not populate every module into one giant target. Build graph layering should enforce the dependency direction above. Android ABI, API level, compiler, optimization mode, sanitizers and shader compiler versions must be recorded in build metadata. A release APK must be reproducible enough to audit and must include software license notices for all included third-party materials (where any are permitted).

---

## 14. Recommended milestone order

**M0 — Repository/bootstrap:** source layout, C++26 compiler check, Android package skeleton, NativeActivity lifecycle, logging, error/result type, CI-independent local test runner.

**M1 — Core correctness:** math/time/rational frame clock, memory ownership model, allocator diagnostics, thread/job scheduler, file IO, project IDs and serialization prototype.

**M2 — Project/editor model:** project database/manifest, asset registry, composition, layers, nested composition route, command registry, transactions, undo/redo and autosave recovery.

**M3 — 2D CPU renderer:** pixel buffers, color/alpha semantics, source layer, transforms, masks, blend modes, render graph and CPU reference golden corpus.

**M4 — Timeline/animation:** keyframes, interpolation registry, graph evaluation and interaction, motion paths, time remap/speed ramp, markers, trim/stretch and preview clock.

**M5 — Native UI foundation:** input/layout/renderer/widgets/docking/gesture/focus and editor panels; connect every durable mutation through commands.

**M6 — Vulkan renderer:** device/capability selection, resource allocator, shader compiler, compositor/effects, synchronization, recovery, CPU/GPU equivalence.

**M7 — Images/effects:** codecs for selected stills, resampling, color management, core effect families and complete per-effect contracts.

**M8 — Audio:** PCM engine, waveform, mixing, DSP, Android output, audio/video sync and recovery.

**M9 — Media ingest/export:** container foundations, decoder/encoder subsets chosen by feasibility, image sequence and export queue; no broad format claims before conformance.

**M10 — Typography/interchange:** font parsing/shaping phases, SVG/Lottie/PSD/PDF, import diagnostics and compatibility reports.

**M11 — Tracking/3D/segmentation:** classical tracking first, then 3D, then carefully scoped segmentation/model inference.

**M12 — Production hardening:** stress/fuzz/device matrix, crash recovery, performance/thermal governance, storage constraints, signed APK and release audit.

Milestones are gates, not calendar promises. A later feature must not be marked complete while a lower-level dependency remains unstable.

---

## 15. Open decisions to resolve before implementation

1. Project container: readable manifest + binary chunks, or canonical text with external media/cache?
2. Working color space and HDR target policy for mobile.
3. Maximum composition dimensions, layer counts, nested depth and cache budget per device memory tier.
4. Initial still-image format subset and media/container parser scope.
5. Codec priority based on patent review, device performance and achievable conformance.
6. Initial typography scripts/font table coverage and fallback strategy.
7. Shader DSL grammar, type system, diagnostics and SPIR-V emitter approach.
8. Plugin decision: disabled initially, signed native modules (high risk), or a future sandboxed VM built internally.
9. Local automation interface: internal only, or explicitly opt-in local socket/control API.
10. Minimum Android API level and supported GPU capability tiers.

Do not bury these decisions in code before their trade-offs and tests are written.

---

## 16. Audit limitations and evidence labels

- The exact source paths are listed in `EffectCraft_Exact_Source_Inventory.md`; the ZIP itself was inspected. File paths reveal implementation areas but do not prove every feature works.
- The README's claims (for example 306 effects, 38 blend modes, codec coverage and alpha status) are archive-authored descriptions. Treat them as reference claims pending code-level and test-level verification.
- Some codec/media dependencies in the Cargo workspace point to separate Git repositories (not embedded in this ZIP). Their implementation cannot be fully audited from this archive alone.
- This Vieana package contains a complete planning/specification scaffold and source file map, not a completed editor. Generated C++ stubs are intentionally empty; no compile or runtime capability is implied.
- The archive inventory keeps test module paths for traceability. Vieana test files are created only under the external `Testing/` tree.

---

## 17. Source-to-Vieana traceability

Use the exact inventory document alongside the crate mapping table. For every source module in the archive, create a traceability row in the future feature matrix with: original path, inferred responsibility, Vieana subsystem, design decision (adopt behavior / redesign / exclude), independent specification reference, test case IDs, and status. A source path with no corresponding row is a missing audit item. A Vieana folder with no design specification is not an implemented system.

## 18. Crate-level source inventory and responsibility notes

### `automation` — 18 source files

Autosave, backend bridge, tool endpoints, long-running jobs, confinement, server/control surface. Vieana scope: internal automation, recovery, task control; no network service by default.

### `av1enc` — 15 source files

AV1 encoding primitives: bit writer, CDF/probability tables, mode decisions, loop filter, prediction, symbols, transforms and writer.

### `color` — 4 source files

Color spaces, ICC profiles and blend math. Vieana scope: color science core, profile parsing, transforms, HDR/gamut handling and deterministic blending.

### `effects` — 68 source files

Effect catalog, controls, CPU effect implementations, blur, color, channel, distortion, generation, keying, matte, noise, perspective, paint, puppet, roto, simulation, time, transition, VR and audio effects.

### `engine` — 175 source files

Editor session/state, commands, undo/history, jobs, project/footage lifecycle, render queue, preferences, menus, shortcuts, media browser, preview and panel-facing model operations.

### `export` — 19 source files

Encoding/export pipeline, output modules, uncompressed formats, WebM and codec-specific output paths.

### `expr` — 9 source files

Expression host, parsing/rewriting, runtime, sampling and tests. Vieana scope: bounded expression language/VM, type system and deterministic evaluation.

### `geom` — 1 source files

Foundational geometry types and operations used by path, rendering and tracking.

### `gpu` — 60 source files

GPU device/context, render operations, 3D paths, particles, effects and WGSL shader catalog.

### `hevcenc` — 12 source files

HEVC encoder elements: bitstream, CABAC, deblocking, frames, inter/intra prediction, motion estimation, parameter sets and transforms.

### `host` — 15 source files

Host application directories and platform integration.

### `interchange` — 4 source files

Import/export interchange routing and compatibility boundaries.

### `keyframe` — 5 source files

Keyframe value types and text-document animation support.

### `lottie` — 9 source files

Lottie animation model, JSON import/export, shape/text handling, base64 and packaged archive support.

### `media` — 17 source files

Media conversion, downmix, DPX/EXR channels, GIF/WebP animation, layered media, pools, probing and streaming.

### `model` — 10 source files

3D primitives, extrusion, OBJ/FBX/glTF/STL reading, triangulation and model tests.

### `opusenc` — 18 source files

Opus encoder building blocks: bands, energy, MDCT, range coding, rate control, resampling and SILK analysis/pulses.

### `path` — 10 source files

Path boolean operations, feathering, fitting, interpolation, offsets, general operations, tracing and variable-width strokes.

### `pdf` — 20 source files

PDF parser/writer, objects, pages, images, fonts, Type 1/CFF, PostScript, CCITT, shading, color and encoding.

### `plugin` — 3 source files

Plugin surface and sandboxed WebAssembly module support.

### `project` — 12 source files

Project building, properties, file checks, limits, render queue, templates, sequence, styles and tracking metadata.

### `psd` — 10 source files

PSD layer/descriptor parsing, engine data, placed-layer handling, read/write and memory tests.

### `raster` — 8 source files

Raster blur, channels/3D, cuts, optical flow, inpainting, scopes and warp operations.

### `render` — 47 source files

CPU compositor, evaluation, passes, source layers, cache/disk cache, color, audio, masks, styles, shape/text, nested comps and 2D/3D rendering.

### `script` — 20 source files

Scripting runtime, include, model, task, socket, script files, script UI and panel/control integration.

### `segment` — 7 source files

Face/object segmentation abstraction and optional neural model adapters (the original tree contains MobileSAM, MediaPipe and TFLite integration modules).

### `svg` — 6 source files

SVG bounds, color, path data and rasterization.

### `text` — 12 source files

Font discovery and parsing, layout, shaping-facing text model, path text, selectors and variable font handling.

### `time` — 2 source files

Shared time primitives and frame/tick semantics.

### `track` — 24 source files

Feature tracking, KLT, planar tracking, camera bundle solve, geometry/linear algebra, stabilization, face tracking and Roto Brush matting/refinement.

### `ui-egui` — 178 source files

Desktop-oriented egui UI: panels, menus, widgets, docks, state, localization, workspace persistence, viewer, timeline, waveform and automation controls. Vieana will replace the framework with its own native C++ UI.

### `vp9enc` — 8 source files

VP9 encoder elements: boolean coding, loop filter, motion compensation, probability tables, tiles and transforms.

