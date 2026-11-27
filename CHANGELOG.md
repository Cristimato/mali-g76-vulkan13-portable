# Changelog

## v4.7 — 2026-11-27
- vkshim v4.7 (o mesmo do módulo rosemary-vulkan13 v4.7), build `portable`.
- Novo `shim/spvemu.c`: `gl_ClipDistance` vira recorte real (varying + discard no FS);
  `maxClipDistances`/`maxCullDistances` = 8.
- depthClamp emulado (z preso a [0, w] por vértice; exato quando a primitiva não cruza near/far).
- multiDrawIndirect com `gl_DrawID` correto (push constant extra no layout).
- fillModeNonSolid (LINE/POINT) por geometry shader gerado.
- Erratas do driver r49p1: MK-16 (cull dinâmico sumia com pontos/linhas, confirmada), MK-07/11/06.
- Probe `tools/probe/e47` (19 casos): v4.0 9/19, **v4.7 19/19** com este zip no aparelho.
- Conhecido: no BT3 (Dragon Rage), uma área do chão perto da câmera fica transparente; suspeita no
  depthClamp por vértice em primitivas que cruzam o near. Contorno: `setprop debug.vkshim.clamp 0`.

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
