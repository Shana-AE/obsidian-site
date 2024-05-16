问题出在typescript的### Resolve package.json Exports -`resolvePackageJsonExports` 上

`--resolvePackageJsonExports` forces TypeScript to consult [the `exports` field of `package.json` files](https://nodejs.org/api/packages.html#exports) if it ever reads from a package in `node_modules`.

This option defaults to `true` under the `node16`, `nodenext`, and `bundler` options for `--moduleResolution`.

[`--moduleResolution bundler` (formerly known as `hybrid`) by andrewbranch · Pull Request #51669 · microsoft/TypeScript (github.com)](https://github.com/microsoft/TypeScript/pull/51669)

之后vuex及mitt报错，应该是exports中没有对types进行导出

或者可以在package.json中定义typesVersions
[TypeScript: Documentation - Publishing (typescriptlang.org)](https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html#version-selection-with-typesversions)