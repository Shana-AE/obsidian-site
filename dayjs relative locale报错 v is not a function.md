relative time plugin throws: asynchronous error TypeError: v is not a function

[relative time plugin throws: asynchronous error TypeError: v is not a function · Issue #1545 · iamkun/dayjs (github.com)](https://github.com/iamkun/dayjs/issues/1545)

Got my issue solved. Not sure if it is related to this issue but it had a similar error message.  
In my case it was because I was updating only some of the default locale settings for `relativeTime` like this:

```
dayjs.updateLocale('en', {
	relativeTime: {
		d: '%d d',
		dd: '%d d',
		M: '%d mo.',
		MM: '%d mos.',
		y: '%d y',
		yy: '%d y',
	}
});
```

I figured out that if you don't set **ALL** of the locale options for the `relativeTime` it will throw this error (or at least very similar error) when dayjs tries to use one of the `relativeTime` locale options that you didn't set on `updateLocale` and doesn't find it.

This is because `updateLocale` will replace the default `relativeTime` locale object with the object you set in the `updateLocale`. So for example in this case it would not have locale options for h, hh, s, ss etc.

The way I fixed this in my use case was to get the default `relativeTime` locale values and just spread those and update the ones I need to update.

```
const localeList = dayjs.Ls;

dayjs.updateLocale('en', {
	relativeTime: {
		...localeList['en'].relativeTime,
		d: '%d d',
		dd: '%d d',
		M: '%d mo.',
		MM: '%d mos.',
		y: '%d y',
		yy: '%d y',
	}
});
```

Hope this helps someone else as well.