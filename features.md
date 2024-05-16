---
tags:
  - "#vite"
---

## npm Dependency resolving and pre-building

npm module会Pre-build，然后改写其URL
依赖会强缓存

## hot module replacement

提供了hmr api，可以通过这个来热更新

## typescript

原生支持ts，但是只会transpile，不会检查。因为编译无需得知所有的module graph，但是检查需要知道所有的module graph

### Transpile onlhy

如果想检查，可以
  - 生产构建，`tsc --noEmit`
  -  development, `tsc --noEmit --watch` 或使用 [vite-plugin-checker](https://github.com/fi3ework/vite-plugin-checker) 在浏览器中显示
### Typescript Compiler Options

需要注意一些compilerOptions

- `isolatedModules` 必须为`true` 因为`esbuild`只会进行无类型的转义。如果依赖有问题那就加入`skipLibCheck: true`暂时抑制报错
- `useDefineForClasssFields`
- 
