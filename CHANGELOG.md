# Changelog

## v5.7 — 2026-11-27
- vkshim v5.7 (o mesmo do módulo rosemary-vulkan13 v5.7), build `portable`.
- Por jogo em `Android/data/<pacote>/vkshim.conf` (desligado por padrão):
  `fps=N` (limitador com intervalo constante), `aniso=N`, `mip=near`, `lodbias=f`, `fp16=1`.
- Benchmark `tools/probe/p57` (1280x720, PSNR com `p57 cmp`): vkshim sem custo sobre o r49 puro;
  textura com aniso 16x: `aniso=4` 8,4x mais rápido, `mip=near` 2,1x (37 dB); `fp16` +5% no máximo.
- Em jogo limitado pela CPU (BT3 em 1x) os ajustes de textura não mudam o FPS; o HUD mostra qual é
  o caso (`espera_ms` e `gpu_uso_pct`; no portátil o uso da GPU fica -1 sem root).
- c92 22 OK / 1 limite e e47 23/23 com este zip.

## v5.0 — 2026-11-27
- vkshim v5.0 (o mesmo do módulo rosemary-vulkan13 v5.0), build `portable`. Detalhes completos no
  CHANGELOG do módulo; resumo:
- `VK_EXT_robustness2` (nullDescriptor por recursos fantasma; o r49 trava com qualquer descritor nulo),
  `VK_KHR_push_descriptor`, `VK_EXT_multi_draw`, `VK_EXT_memory_budget`/`_priority`, divisor EXT.
- Descriptor indexing completo: índice não uniforme de imagens/texel por laço waterfall no shader,
  partially bound preenchido com fantasmas, update-after-bind (real para imagens/texel; buffers
  usam o descritor do momento do bind), limites de 1.000.000 descritores.
- `drawIndirectCount` (endereço do contador por push constant; o VS descarta os draws extras),
  `VK_EXT_depth_clip_enable` e `VK_EXT_depth_clip_control`.
- depthClamp exato por pixel (`gl_FragDepth`): corrige o chão transparente do BT3 (Dragon Rage).
- HUD (`hud=1`) e config por jogo em `Android/data/<pacote>/vkshim.conf` (no portátil a frequência
  e o uso da GPU ficam -1: ler o `ged` precisa da regra de SELinux do módulo com root).
- Probes `c92` (22 OK / 0 BUG / 1 limite com este zip; r49 puro 7 OK) e `e47` (23/23).

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
