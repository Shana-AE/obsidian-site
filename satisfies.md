`satisfies` 类似于加强版的`as` ，能够结合字面量与`satisfies`的对象进行类型推断

```ts
interface Person {
  name: string;
  sex: string;
}

const a = { name: 'string' } satisfies Partial<Persion> { name: string };
```

```ts
interface Persion {
  name: string;
  id: number | string;
}

const a = { name: 'test', id: 123 } satisfies Persion; { name: string, id: number }
```