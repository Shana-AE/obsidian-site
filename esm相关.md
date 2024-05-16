---
tags:
  - 工程化
  - esm
  - vite
  - tailwindcss
  - postcss
---

## 'import' and 'export' may appear only with 'sourceType: module'

将项目更改为type:module后发现报这个错误，后续查找发现是tailwind版本问题，其依赖的postcss-load-config版本有问题，3.x不支持esm