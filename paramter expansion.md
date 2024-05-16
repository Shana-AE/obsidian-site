Here’s a detailed explanation of `${f##*/}`:

-   `${parameter##word}`: This is a parameter expansion that removes the longest match of `word` from the beginning of `$parameter`. In this case, `$parameter` is `$f`, which is the filename of each file found by `find` command, and `word` is `*/`, which matches all leading directory components of `$f`. Therefore, `${f##*/}` removes all leading directory components from `$f`, leaving only the filename.

Here’s a detailed explanation of `${f%.*}`:

-   `${parameter%word}`: This is a parameter expansion that removes the shortest match of `word` from the end of `$parameter`. In this case, `$parameter` is `$f`, which is the filename of each file found by `find` command, and `word` is `.*`, which matches the extension of `$f`. Therefore, `${f%.*}` removes the extension from `$f`, leaving only the filename.