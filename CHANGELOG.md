# Changelog

## v3.0 — 2026-11-26
- Primeira versão neste repositório: driver ZIP importável (meta.json na raiz) com vkshim + HAL
  r49p1 + blob Mali patchado para G76. Não precisa de módulo nem root.
- vkshim build `portable`: procura `vulkan.r49.so` na pasta do pacote.
- `tools/make_portable.sh`: build reproduzível a partir do zip donor.
