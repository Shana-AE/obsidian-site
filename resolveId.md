---
tags:
  - rollup
  - build
  - rollup-hooks
---

| type:     | ResolveIdHook                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| kind:     | async, first                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Previous: | [`buildStart`](https://rollupjs.org/plugin-development/#buildstart) if we are resolving an entry point, [`moduleParsed`](https://rollupjs.org/plugin-development/#moduleparsed) if we are resolving an import, or as fallback for [`resolveDynamicImport`](https://rollupjs.org/plugin-development/#resolvedynamicimport). Additionally, this hook can be triggered during the build phase from plugin hooks by calling [`this.emitFile`](https://rollupjs.org/plugin-development/#this-emitfile) to emit an entry point or at any time by calling [`this.resolve`](https://rollupjs.org/plugin-development/#this-resolve) to manually resolve an id |
| Next:     | [`load`](https://rollupjs.org/plugin-development/#load) if the resolved id has not yet been loaded, otherwise [`buildEnd`](https://rollupjs.org/plugin-development/#buildend)                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
````ad-note
title: type definition
collapse: close
```typescript
type ResolveIdHook = (
	source: string,
	importer: string | undefined,
	options: {
		attributes: Record<string, string>;
		custom?: { [plugin: string]: any };
		isEntry: boolean;
	}
) => ResolveIdResult;

type ResolveIdResult = string | null | false | PartialResolvedId;

interface PartialResolvedId {
	id: string;
	external?: boolean | 'absolute' | 'relative';
	attributes?: Record<string, string> | null;
	meta?: { [plugin: string]: any } | null;
	moduleSideEffects?: boolean | 'no-treeshake' | null;
	resolvedBy?: string | null;
	syntheticNamedExports?: boolean | string | null;
}
```

````

`source` is the importee exactly as it's written in the import statement,
```js
import { foo } from '../bar.js'// source '../bar.js';
```

The `importer` is the fully resolved id of the importing module.

(When resolving entry points, importer will usually be `undefined`. An exception here are entry points generated via #TOREAD [`this.emitFile`](https://rollupjs.org/plugin-development/#this-emitfile) as here, you can provide an `importer` argument.) 

For those cases, the `isEntry` option will tell you if we are resolving a user defined entry point, an emitted chunk, or if the `isEntry` parameter was provided for the #TOREAD [`this.resolve`](https://rollupjs.org/plugin-development/#this-resolve) context function.

You can use this for instance as a mechanism to define custom proxy modules for entry points. 
````ad-example
title: example about proxy all entry points to inject a polyfill import
collapse: close
```typescript
// We prefix the polyfill id with \0 to tell other plugins not to try to load or
// transform it
const POLYFILL_ID = '\0polyfill';
const PROXY_SUFFIX = '?inject-polyfill-proxy';

function injectPolyfillPlugin() {
	return {
		name: 'inject-polyfill',
		async resolveId(source, importer, options) {
			if (source === POLYFILL_ID) {
				// It is important that side effects are always respected
				// for polyfills, otherwise using
				// "treeshake.moduleSideEffects: false" may prevent the
				// polyfill from being included.
				return { id: POLYFILL_ID, moduleSideEffects: true };
			}
			if (options.isEntry) {
				// Determine what the actual entry would have been.
				const resolution = await this.resolve(source, importer, options);
				// If it cannot be resolved or is external, just return it
				// so that Rollup can display an error
				if (!resolution || resolution.external) return resolution;
				// In the load hook of the proxy, we need to know if the
				// entry has a default export. There, however, we no longer
				// have the full "resolution" object that may contain
				// meta-data from other plugins that is only added on first
				// load. Therefore we trigger loading here.
				const moduleInfo = await this.load(resolution);
				// We need to make sure side effects in the original entry
				// point are respected even for
				// treeshake.moduleSideEffects: false. "moduleSideEffects"
				// is a writable property on ModuleInfo.
				moduleInfo.moduleSideEffects = true;
				// It is important that the new entry does not start with
				// \0 and has the same directory as the original one to not
				// mess up relative external import generation. Also
				// keeping the name and just adding a "?query" to the end
				// ensures that preserveModules will generate the original
				// entry name for this entry.
				return `${resolution.id}${PROXY_SUFFIX}`;
			}
			return null;
		},
		load(id) {
			if (id === POLYFILL_ID) {
				// Replace with actual polyfill
				return "console.log('polyfill');";
			}
			if (id.endsWith(PROXY_SUFFIX)) {
				const entryId = id.slice(0, -PROXY_SUFFIX.length);
				// We know ModuleInfo.hasDefaultExport is reliable because
				// we awaited this.load in resolveId
				const { hasDefaultExport } = this.getModuleInfo(entryId);
				let code =
					`import ${JSON.stringify(POLYFILL_ID)};` +
					`export * from ${JSON.stringify(entryId)};`;
				// Namespace reexports do not reexport default, so we need
				// special handling here
				if (hasDefaultExport) {
					code += `export { default } from ${JSON.stringify(entryId)};`;
				}
				return code;
			}
			return null;
		}
	};
}
```
````
