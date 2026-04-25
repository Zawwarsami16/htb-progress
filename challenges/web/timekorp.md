# TimeKORP

**Event:** HTB MCP TryOut · **Category:** web · **Points:** 1000 · **Difficulty:** very easy

## TL;DR

PHP page calls `exec("date '+" . $_GET['format'] . "' 2>&1")` with no sanitization. Break out of the single-quoted string, run `cat /flag`, and the flag prints into the page.

## What I saw first

Source review on `TimeController.php` straight to `TimeModel.php`. The constructor builds the shell string from raw user input. Dockerfile shows the flag at `/flag`. There is nothing to enumerate.

## What I tried that did not work

I almost over-engineered this one. Started building a payload that would survive `date`'s format parser, then realized the parser does not run before the shell does. The shell runs the whole concatenated string with `;` and `'` as ordinary metacharacters.

## What worked

```
GET /?format='%3Bcat+/flag%3B'
```

The constructed command becomes `date '+';cat /flag;'' 2>&1`. The first `date` runs and prints nothing useful. The `cat /flag` runs and prints the flag. The trailing empty `date` runs and is harmless. The page renders the concatenated stdout.

## Flag

`HTB{t1m3_f0r_th3_ult1m4t3_pwn4g3_c862bd8dfb10f8f53e08505f8ca8242f}`

## What this taught me

When the source builds a shell string with quoted concatenation, the metacharacters are `'`, `;`, and `$()`. None of them are the language's parser; all of them are the shell's. Do not try to outwit `date`. Outwit `sh`.
