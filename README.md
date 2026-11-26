# Mali-G76 Vulkan 1.3 — driver portátil (r49p1 + vkshim)

Driver Vulkan **importável** para a **Mali-G76** (Bifrost, MT6785 / Helio G90T, testado no
POCO/Redmi rosemary). Funciona em launchers e emuladores que aceitam **"Import driver ZIP"**
(estilo adrenotools, com `meta.json` na raiz). **Não precisa de root nem de módulo.**

| | driver do sistema (r32p1) | este pacote (r49p1) |
|---|---|---|
| Vulkan | 1.1.177 | **1.3.278** |
| dynamicRendering / sync2 / maintenance4 | — | sim |
| shaders bindless (RuntimeDescriptorArray) | falha | convertidos pelo vkshim |

## Instalação

1. Baixe `vk49_g76-v3.0-portatil.zip` na [Release mais recente](../../releases/latest).
2. No launcher: *Driver / GPU driver → Import driver ZIP* → escolha o zip.
3. Selecione **"Mali-G76 r49p1 Vulkan 1.3 (vkshim)"** e abra o jogo.

> O `meta.json` usa `backend: panvk` e `panvkFrontend: bifrost-jm` **só** para passar no filtro
> de família desses launchers (eles reconhecem Turnip/PanVK/RADV/PowerVR pelo texto). O driver
> real é o blob ARM, não o Mesa.

## ⚠️ Avisos

- **Desempenho:** em jogo pesado ele ficou **pior** que o driver do sistema. Use quando o jogo
  exigir Vulkan 1.3 ou falhar no r32; compare antes de deixar fixo.
- Só para **Mali-G76**. O blob vem de um vendor de G52 com a checagem de GPU patchada
  (G52 e G76 têm a mesma ISA Bifrost v7).
- Os avisos `user 11.46, kernel 11.38` e `large_page_conf` no log são normais (kernel r32p1).
- Blob proprietário da ARM/MediaTek; uso por sua conta e risco.

## Conteúdo do zip

| arquivo | o que é |
|---|---|
| `meta.json` | descrição do pacote para o launcher |
| `vulkan.mali_r49.so` | **vkshim** (entrada do driver): corrige SPIR-V e repassa ao HAL |
| `vulkan.r49.so` | HAL Vulkan r49p1 |
| `libGLES_ma49.so` | blob Mali r49p1 (GPU-ID patchado, SONAME renomeado) |
| `libgp49.so`, `libgpd9.so`, `android.hardware.*-ndk.so` | dependências do blob |

## Como funciona

- **Blob:** a checagem de GPU do r49p1 em `0x1f29624` vira `mov w0,1; b 0x1f296dc`, e as libs
  são renomeadas (mesmo tamanho de string) para não colidir com o driver r32 do sistema.
- **vkshim:** intercepta `vkCreateShaderModule` e `vkCreateGraphicsPipelines`. Troca
  `OpTypeRuntimeArray` de imagens/samplers por arrays fixos (8192/256), remove as capabilities
  de descriptor indexing que a G76 não tem e a decoração `NonUniform`. No build `portable` ele
  procura `vulkan.r49.so` na própria pasta do pacote.
- Diagnóstico: `setprop debug.vkshim 1` (root) → logs em `logcat -s vkshim` e dumps de pipelines
  que falham em `/data/user/0/<pacote>/cache/vkdump`.

## Compilar do zero (Termux)

Requer `clang`, `python3`, `patchelf`, `zip`, `unzip` e os headers Vulkan, mais o zip donor
`Mali-r49p1-MT6768-v0.1.0.zip` (sha256 conferido pelo script).

```sh
tools/make_portable.sh Mali-r49p1-MT6768-v0.1.0.zip   # -> out/vk49_g76-v3.0-portatil.zip
```

Testar o driver sem launcher (root):
```sh
clang -O1 -o shimvk tools/probe/shimvk.c -ldl
mkdir x && unzip out/vk49_g76-v3.0-portatil.zip -d x
su -c "LD_LIBRARY_PATH=$PWD/x:/vendor/lib64:/vendor/lib64/mt6785 ./shimvk $PWD/x/vulkan.mali_r49.so"
# esperado: GPU Mali-G76 API 1.3.278
```

## Problemas comuns

- **"Unknown driver family" / "families do not match":** você importou outro zip (ou só o `.so`).
  Use o zip da Release, ele já traz o `meta.json` certo.
- **Driver aparece mas não carrega:** `logcat -s vkshim` — a linha `vkshim ativo sobre …` mostra
  de onde o HAL foi aberto.
- **`vkCreateGraphicsPipelines` -3 num jogo:** ligue o diagnóstico e veja `fail_NNN.txt`/`.spv` em
  `cache/vkdump` (G76 não tem clip/cull distance, `depthClamp`, `fillModeNonSolid`).

## Créditos

- Blob r49p1 e libgpud shim: pacote *Mali-r49p1-MT6768* de **Dracen-W** (vendor Redmi gale).
- vkshim, patch da G76 e empacotamento: Cristimato.
