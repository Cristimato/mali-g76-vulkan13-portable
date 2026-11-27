# Mali-G76 Vulkan 1.3 — driver portátil (r49p1 + vkshim v5.7)

Driver Vulkan **importável** para a **Mali-G76** (Bifrost, MT6785 / Helio G90T, testado no
POCO/Redmi rosemary). Funciona em launchers e emuladores que aceitam **"Import driver ZIP"**
(estilo adrenotools, com `meta.json` na raiz). **Não precisa de root nem de módulo.**

> **v5.7 (2026-11-27):** desempenho por jogo no `Android/data/<pacote>/vkshim.conf`: `fps=30`
> (limitador com ritmo estável), `aniso=4` / `mip=near` / `lodbias=0.5` (samplers mais baratos; no
> benchmark p57 `aniso=4` deixa a carga de textura 8,4x mais rápida) e `fp16=1` (experimental).
> Tudo desligado por padrão. Medições no CHANGELOG do
> [módulo](https://github.com/Cristimato/rosemary-vulkan13/blob/main/CHANGELOG.md).
>
> **v5.0 (2026-11-27):** mesmo vkshim v5.0 do módulo
> [rosemary-vulkan13](https://github.com/Cristimato/rosemary-vulkan13): `VK_EXT_robustness2`
> (exigida pelo DXVK), push descriptors, **descriptor indexing completo**, `draw_indirect_count`,
> depthClamp exato, `depth_clip_enable`/`_control`, multi draw, orçamento de memória, HUD e config
> por jogo. Probe `c92`: 22 OK / 1 limite (r49 puro: 7 OK). As versões anteriores continuam nas Releases.

| | driver do sistema (r32p1) | v3.0 | v4.0 | v4.7 | **v5.0 / v5.7** |
|---|---|---|---|---|---|
| Vulkan | 1.1.177 | 1.3.278 | 1.3.278 | 1.3.278 | **1.3.278** |
| dynamicRendering / sync2 / maintenance4 | — | sim | sim | sim | sim |
| shaders bindless (RuntimeDescriptorArray) | falha | convertidos | convertidos | convertidos | convertidos |
| shader com `gl_ClipDistance` | — | derruba o driver | compila (recorte ignorado) | ✅ recorte real | ✅ recorte real |
| erratas do compilador Mali (Arm 3843237 e outros) | — | — | ✅ corrigidas no shader | ✅ + erratas do driver MK-16/07/11/06 | ✅ + erratas do driver MK-16/07/11/06 |
| depthClamp / fillModeNonSolid / multiDrawIndirect | não | não | anunciados (ignorados) | ✅ emulados de verdade | ✅ emulados de verdade (depthClamp exato por pixel) |
| texturas BC1–BC7 | não | não | ✅ na GPU, sem perda (PixelBC) | ✅ | ✅ |
| cache de pipelines entre sessões | — | — | ✅ | ✅ | ✅ |
| robustness2 / push descriptor / multi_draw | — | — | — | — | ✅ |
| descriptor indexing (não uniforme, partially bound, UAB) | — | — | — | — | ✅ (UAB de buffer: valor do bind) |
| draw_indirect_count / depth_clip_enable / depth_clip_control | — | — | — | — | ✅ |
| HUD e config por jogo (`Android/data/<pacote>/vkshim.conf`) | — | — | — | — | ✅ |
| limitador de FPS, samplers e FP16 por jogo (v5.7) | — | — | — | — | ✅ (desligados por padrão) |

## Instalação

1. Baixe `vk49_g76-v5.7-portatil.zip` na [Release mais recente](../../releases/latest).
2. No launcher: *Driver / GPU driver → Import driver ZIP* → escolha o zip.
3. Selecione **"Mali-G76 r49p1 Vulkan 1.3 (vkshim v5.7)"** e abra o jogo.

> O `meta.json` usa `backend: panvk` e `panvkFrontend: bifrost-jm` **só** para passar no filtro
> de família desses launchers (eles reconhecem Turnip/PanVK/RADV/PowerVR pelo texto). O driver
> real é o blob ARM, não o Mesa.

## ⚠️ Avisos

- **Desempenho:** na v3.0, em jogo pesado, ele ficou **pior** que o driver do sistema. A v4 não
  muda a GPU: os ganhos são compatibilidade e menos travadas por compilação (cache de pipelines).
  Use quando o jogo exigir Vulkan 1.3 ou falhar no r32, e compare antes de deixar fixo.
- Só para **Mali-G76**. O blob vem de um vendor de G52 com a checagem de GPU patchada
  (G52 e G76 têm a mesma ISA Bifrost v7).
- Os avisos `user 11.46, kernel 11.38` e `large_page_conf` no log são normais (kernel r32p1).
- Blob proprietário da ARM/MediaTek; uso por sua conta e risco.

## Conteúdo do zip

| arquivo | o que é |
|---|---|
| `meta.json` | descrição do pacote para o launcher |
| `vulkan.mali_r49.so` | **vkshim v5.7** (entrada do driver): corrige SPIR-V, emula recursos, cache de pipelines |
| `vulkan.pixelbc.so` | **PixelBC**: texturas BC1–BC5/BC7 decodificadas na GPU, sem perda |
| `vulkan.r49.so` | HAL Vulkan r49p1 |
| `libGLES_ma49.so` | blob Mali r49p1 (GPU-ID patchado, SONAME renomeado) |
| `libgp49.so`, `libgpd9.so`, `android.hardware.*-ndk.so` | dependências do blob |

## Como funciona

- **Blob:** a checagem de GPU do r49p1 em `0x1f29624` vira `mov w0,1; b 0x1f296dc`, e as libs
  são renomeadas (mesmo tamanho de string) para não colidir com o driver r32 do sistema.
- **vkshim v4:** em cada shader, (1) troca `OpTypeRuntimeArray` de imagens/samplers por arrays
  fixos (8192/256); (2) retira `gl_ClipDistance`/`gl_CullDistance`, que derrubam o compilador
  Mali (`shim/clipcull.c`); (3) aplica os passes do fork
  [leegao/SPIRV-Tools](https://github.com/leegao/SPIRV-Tools) contra bugs do compilador Mali, só
  quando o padrão existe no shader. Também anuncia e emula depthClamp, fillModeNonSolid,
  clip/cull distance e multiDrawIndirect, e guarda pipelines em `cache/vkshim_pcache.bin`.
  No build `portable` ele procura `vulkan.pixelbc.so` e `vulkan.r49.so` na pasta do pacote.
- **PixelBC** ([supplefrog/PixelBC](https://github.com/supplefrog/PixelBC), MIT): variante sem
  perda, com patch para achar o `vulkan.r49.so` ao lado dele (`patches/`).
- Ajustes (root): `setprop debug.vkshim.bcn 0` (sem BC), `debug.vkshim.spv 0` (sem passes Mali),
  `debug.vkshim.pcache 0` (sem cache), `debug.vkshim.fake none` (sem recursos emulados).
  Pesquisa e validação: [docs/PESQUISA-V4.md](docs/PESQUISA-V4.md).
- Diagnóstico: `setprop debug.vkshim 1` (root) → logs em `logcat -s vkshim` e dumps de pipelines
  que falham em `/data/user/0/<pacote>/cache/vkdump`.

## Compilar do zero (Termux)

Requer `clang`/`clang++`, `cmake`, `ninja`, `git`, `glslangValidator`, `python3`, `patchelf`,
`zip`, `unzip` e os headers Vulkan, mais o zip donor `Mali-r49p1-MT6768-v0.1.0.zip` (sha256
conferido pelo script). As dependências (SPIRV-Tools do fork e PixelBC) são baixadas em commits
fixos por `tools/build_deps.sh`.

```sh
tools/make_portable.sh Mali-r49p1-MT6768-v0.1.0.zip   # -> out/vk49_g76-v5.7-portatil.zip
tests/run.sh                                          # testes offline dos passes SPIR-V
```

Testar o driver sem launcher (root):
```sh
clang -O1 -o v4test tools/probe/v4test.c -ldl
mkdir x && unzip out/vk49_g76-v5.7-portatil.zip -d x
# shaders de teste: compile tests/*.vert|frag|comp com glslangValidator -V (veja tests/run.sh)
su -c "LD_LIBRARY_PATH=$PWD/x:/vendor/lib64:/vendor/lib64/mt6785 ./v4test $PWD/x/vulkan.mali_r49.so <pasta_dos_spv>"
# esperado: RESULTADO: TUDO OK
```

## Problemas comuns

- **"Unknown driver family" / "families do not match":** você importou outro zip (ou só o `.so`).
  Use o zip da Release, ele já traz o `meta.json` certo.
- **Driver aparece mas não carrega:** `logcat -s vkshim` — a linha `vkshim ativo sobre …` mostra
  de onde o HAL foi aberto.
- **`vkCreateGraphicsPipelines` -3 num jogo:** ligue o diagnóstico (`setprop debug.vkshim 1`) e
  veja `fail_NNN.txt`/`.spv` em `cache/vkdump`. Teste desligando partes (`debug.vkshim.spv 0`,
  `debug.vkshim.bcn 0`) para isolar.

## Créditos

- Blob r49p1 e libgpud shim: pacote *Mali-r49p1-MT6768* de **Dracen-W** (vendor Redmi gale).
- Passes para o compilador Mali: fork do SPIRV-Tools de **leegao** (Apache-2.0).
- Decodificação BC: **PixelBC** de supplefrog (MIT).
- vkshim, `clipcull.c`, patch da G76 e empacotamento: Cristimato. Veja [THIRD_PARTY.md](THIRD_PARTY.md).
