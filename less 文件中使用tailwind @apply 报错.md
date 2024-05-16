```log
The `px-` class does not exist. If `px-` is a custom class, make sure it is defined within a `@layer` directive.

  5777 | }
  5778 | .van-dialog .van-dialog__header {
> 5779 |   @apply text-text-90 px- [20rem];
       |   ^
  5780 | }
  5781 | 
```

Hey! This looks specific to Less, if you remove `lang="less"` it works. Less must be compiling it in a way where it is either removing the `[20px]` part before Tailwind processes it, or adding a space after the `p-` part.

[https://stackblitz.com/edit/vitejs-vite-rufwr3?file=src/App.vue](https://stackblitz.com/edit/vitejs-vite-rufwr3?file=src/App.vue)

Going to close as not a Tailwind issue unfortunately, nothing we can really do here. In general we recommend avoiding using any preprocessor with Tailwind:

[https://tailwindcss.com/docs/using-with-preprocessors](https://tailwindcss.com/docs/using-with-preprocessors)

[Use @apply in LESS, If prefix-[Arbitrary values] is not after prefix-[color] ，Throw error：The `h-` class does not exist..😥 · Issue #8738 · tailwindlabs/tailwindcss (github.com)](https://github.com/tailwindlabs/tailwindcss/issues/8738)