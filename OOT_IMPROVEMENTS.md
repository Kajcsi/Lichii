# Ocarina of Time improvements (Kajcsi/Lichii fork)

This branch, `oot-improvements`, is [zortos293/Lichii](https://github.com/zortos293/Lichii) `nx2`
plus fixes aimed at The Legend of Zelda: Ocarina of Time (`0400486024DEC000`). Upstream `nx2` is
merged in regularly, so everything from zortos293/Lichii is here as well.

## Changes on top of upstream

| Change | Status |
| --- | --- |
| **Bindless handles read from the host-aligned storage base** (`video_core/shader_handle_program.h`, Vulkan and OpenGL pipelines, unit test in `tests/video_core/texture_handles.cpp`). Handle programs record storage offsets relative to the buffer address aligned down to the host's storage-buffer alignment, as `StorageOffset` does; the CPU evaluator used the unaligned address. | Correct and covered by a unit test. **No visible effect in Ocarina so far:** an A/B run on an RTX 4090 (alignment 16) in Kokiri Forest compared both reads for 524,288 handle evaluations and they never differed, because the game's storage pointers are already 16-byte aligned. The commit message's claim that it fixes the missing vegetation is not borne out. zortos293 found the same issue independently (`fix/oot-fog`, `EDEN_NX2_HANDLE_ALIGN`). |
| **Flush pending query writes before native MME DMA reads**, cherry-picked from Eglantine `oot-jojo` (`50db84983f`). | Affects High GPU accuracy only (with the GPU thread split off). At Low accuracy, report payloads are written immediately and the split already drains before the read. |

## Tested

Windows, RTX 4090, Docked, Low GPU accuracy, `EDEN_NX2_SPLIT=2`, Kokiri Forest (Link's house
area): no shaders rejected (build 75 rejected several gather programs here), no white haze or
patches over Link, no white flashes in 30+ sampled frames.

## Known leads, not fixed yet

- Compiling some new pipelines while the camera pans reads texture descriptors past the TIC
  limit (`index <= tic_limit` asserts followed by garbage formats 119, 127, 21) in
  `GenericEnvironment::ReadTextureInfo`.
- The issues collected in zortos293/Lichii#101 (missing vegetation, the `c72b6ea746c00433`
  half-resolution layer, fog volumes) are still open upstream.
