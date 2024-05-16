```json
"exports": {
	".": {
		"build": "./index.ts",
		"import": "./dist/index.mjs",
		"require": "./dist/index.cjs",
		"default": "./dist/index.mjs",
		"development": "./index.ts"
	},
	"./*": {
		"build": "./*.ts",
		"import": "./dist/*.mjs",
		"require": "./dist/*.cjs",
		"default": "./dist/*.mjs",
		"development": "./*.ts"
	}
}
```
exports 为暴露给外侧的引用，其中`"."`和`"./*"`为路径，`/`后要有`*`，但是根据实践应该不遵循glob匹配规则，需要后续再看看
monorepo中，如果需要将内部重导出，同时需要在构建时保留原有的引用，以免报以下类似错误，需要在内部加入更多的”条件“， 比如上边的build，同时在vite中加`入, 此时vite会找conditions，根据conditions 的值进行解析
```js
resolve: {
  conditions: ['build'],
},
```
```log
ERROR @xxx/constant build error [vite]: Rollup failed to resolve import "@xxx/types-site/project" from "/Users/xxx/Documents/projects/xxx/packages/constants/optionsData.ts".
```