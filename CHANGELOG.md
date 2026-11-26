# Changelog

## v4.0 — 2026-11-26
- vkshim v4 (o mesmo do módulo rosemary-vulkan13 v4.0), build `portable`.
- Corrigido: shader com `gl_ClipDistance`/`gl_CullDistance` derrubava o driver; agora esses
  built-ins são retirados (`shim/clipcull.c`).
- Corrigido: shaders afetados por bugs do compilador Mali (erratum Arm 3843237 e 3779215, shift
  folding, spec-constant composite, lockstep de compute), via passes do fork leegao/SPIRV-Tools.
- Novo: depthClamp, fillModeNonSolid, clip/cull distance e multiDrawIndirect emulados.
- Novo: `vulkan.pixelbc.so` (PixelBC sem perda) para texturas BC1–BC5/BC7; validado na G76
  (535/0 e 20046/0).
- Novo: cache de pipelines persistente; props `debug.vkshim.fake/.spv/.pcache/.bcn`.
- Build: `tools/build_deps.sh` (commits fixos), `tests/run.sh`, probe `v4test`.

## v3.0 — 2026-11-26
- Primeira versão neste repositório: driver ZIP importável (meta.json na raiz) com vkshim + HAL
  r49p1 + blob Mali patchado para G76. Não precisa de módulo nem root.
- vkshim build `portable`: procura `vulkan.r49.so` na pasta do pacote.
- `tools/make_portable.sh`: build reproduzível a partir do zip donor.
