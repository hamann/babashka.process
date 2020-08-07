# babashka.process

A Clojure wrapper around `java.lang.ProcessBuilder`.

Status: alpha, WIP. This code may end up in [babashka](https://github.com/borkdude/babashka).

## Example usage

``` clojure
user=> (require '[babashka.process :refer [process]])
```

Invoke `ls`:

``` clojure
user=> (-> (process ["ls"]) :out)
"LICENSE\nREADME.md\nsrc\n"
```

Output as stream:

``` clojure
user=> (-> (process ["ls"] {:out :stream}) :out slurp)
"LICENSE\nREADME.md\ndeps.edn\nsrc\ntest\n"
```

Exit code:

``` clojure
user=> (-> (process ["ls" "foo"]) :exit)
0
```

By default, `process` throws when the exit code is non-zero:

``` clojure
user=> (process ["ls" "foo"])
Execution error (ExceptionInfo) at babashka.process/process (process.clj:54).
ls: foo: No such file or directory
```

By setting `:throw` to `false` you can capture the error output as a string or stream:

``` clojure
user=> (-> (process ["ls" "foo"] {:throw false}) :err)
"ls: foo: No such file or directory\n"

user=> (-> (process ["ls" "foo"] {:throw false :err :stream}) :err slurp)
"ls: foo: No such file or directory\n"
```

``` clojure
user=> (-> (process ["ls" "foo"] {:throw false}) :exit)
1
```

When `:out` or `:err` are set to `:stream`, the exit code is wrapped in a future:

``` clojure
user=> (-> (process ["ls" "foo"] {:throw false :err :stream}) :exit deref)
1
```

Redirect output to stdout:

``` clojure
user=> (do (-> (process ["ls"] {:out :inherit})) nil)
nil
user=> LICENSE		README.md	src
```

Redirect output stream from one process to input stream of the next process:

``` clojure
(let [is (-> (process ["ls"] {:out :stream}) :out)]
  (process ["cat"] {:in is
                    :out :inherit})
    nil)
LICENSE
README.md
deps.edn
src
test
nil
```

## License

Copyright © 2019-2020 Michiel Borkent

Distributed under the EPL License. See LICENSE.
