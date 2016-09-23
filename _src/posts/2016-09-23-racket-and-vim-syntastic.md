    Title: Racket and vim-syntastic
    Date: 2016-09-23T21:48:27
    Tags: racket,vim

The current Syntastic checker for Racket syntax in Vim is the `racket`
executable itself. This means that the current buffer is executed by Racket
whenever it is checked for syntax, and this can have unintended consequences,
e.g. entering an infinite loop, if you happen to open the wrong file.

In order to just get the syntax errors, add this to your `.vimrc`:

```
let g:syntastic_racket_racket_args="--load"
```
