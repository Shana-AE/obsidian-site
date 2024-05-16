---
tags:
  - typescript
  - tsconfig
---

用其他transpiler编译ts代码也很常见（如babel），但是这些transpiler只能一次操作一个文件，无法理解整体的类型系统，用ts.transpileModule Api的工具也会有这个问题。

这些问题可能会引起`const enum`和`namespace`相关的runtime问题。
设置这个选项可以让ts在一些代码无法在单独文件中解析的时候warn你

以下是一些`isolatedModules`开启时失败的例子

````ad-example
title: Exports of Non-Value Identifiers
collapse: open

```typescript
import { someType, someFunction } from "someModule";

someFunction();

export { someType, someFunction };
```
因为someType没有值，所以生成文件中的`export`不会有它, js runtime 中就有可能有问题

```typescript
export { someFunction }
```
单文件的transpiler不知道someType会不会生成值，所以只导出类型会报错
````

````ad-example
title: Non-Module Files
collapse: open

如果`isolatedModules`设置后，`namespace`只允许在`modules`中出现（意味着它有一些`import/export`）, 如果不在module文件中，
```typescript
namespace Instantiated {

Namespaces are not allowed in global script files when 'isolatedModules' is enabled. If this file is not intended to be a global script, set 'moduleDetection' to 'force' or add an empty 'export {}' statement.Namespaces are not allowed in global script files when 'isolatedModules' is enabled. If this file is not intended to be a global script, set 'moduleDetection' to 'force' or add an empty 'export {}' statement.

export const x = 1;

}
```

这个限定不限制`d.ts`文件
````


````ad-example
title: References to `const enum` members
collapse: open

在ts中，如果用到了`const enum`， 生成的js文件中会替换为真实数值
```typescript
declare const enum Numbers {

Zero = 0,

One = 1,

}

console.log(Numbers.Zero + Numbers.One);
```
转换为js为
```js
"use strict";

console.log(0 + 1);
```

如果其他transpiler不了解这些值的信息就无法将代码转换为`Numbers`中实际的值。所以`isolateModules`开启时，不能使用`declare const enum`，但是可以`const enum` 可以看这里[typescript - Why is `const enum` allowed with `isolatedModules`? - Stack Overflow](https://stackoverflow.com/questions/56854964/why-is-const-enum-allowed-with-isolatedmodules)的解释
````