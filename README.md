# Thorn

Thorn is a general-purpose programming language that compiles to native binaries via C. It has inferred types, no semicolons, and a standard library that covers I/O, files, networking, and concurrency. The idea is that it should be readable to most programmers without a lot of explanation, and it should not get in the way of what you are actually trying to do.

The full language reference is at https://thorn.hikikomori.no and there is a forum at https://thorn.hikikomori.no/forum if you want to ask something or follow development.

---

## A quick example

```thorn
need time
need math

task greet(name) {
    write(v"hello {name}, it is {now.hour}:{now.minute}")
}

task circleArea(r) {
    send math.pi * r * r
}

section main {
    task run() {
        greet("world")
        area = circleArea(5)
        write(v"circle area: {area}")
    }
}

if (start) {
    main()
}
```

---

## Variables

There is no declaration keyword. You assign a value and the variable exists. The type is inferred from the value, and the first assignment is where the variable is created.

```thorn
x = 5
name = "Thorn"
active = on
nothing = void
```

Booleans are `on` and `off`. The absence of a value is `void`.

---

## Tasks

Functions in Thorn are called tasks. You define them with `task` and return a value with `send`. If a task does not send anything, it returns `void`.

```thorn
task add(x, y) {
    send x + y
}

result = add(3, 4)
```

String interpolation works by putting a `v` in front of the string. Any expression can go inside the curly braces.

```thorn
write(v"the result is {add(3, 4)}")
```

---

## Loops

Thorn has one loop keyword that handles every looping pattern you would normally need.

```thorn
loop(on) { }             # runs forever
loop(10) { }             # runs exactly 10 times
loop(i = 10) { }         # runs 10 times, counter i goes from 0 to 9
loop(x < 100) { }        # runs while the condition is true
loop(item = myList) { }  # iterates over a list
loop(pair = myMap) { }   # iterates over a map, pair.key and pair.value
```

---

## Error handling

`risk` blocks turn errors into values. You can capture the error, pass a handler, or add a prefix to the message. If you need full control there is also a standard try/catch.

```thorn
err = risk {
    res = reach("https://example.com")
    send res.json
}

if (err != void) {
    write(v"failed: {err}")
}
```

---

## Concurrency

`spawn` runs a task concurrently and returns immediately. If you read the return value before the task has finished, execution blocks until it is ready.

```thorn
out = spawn(fetchData(url))
write(v"got: {out}")
```

---

## Sections

Sections group related tasks and variables together. They are Thorn's answer to organisation, and a lighter alternative to classes that does not force object-oriented patterns on you. Visibility is controlled explicitly.

```thorn
section.global networking {
    task connect(url) {
        # accessible from anywhere
    }
}

section(ui, main) renderer {
    task draw(data) {
        # accessible only from ui and main
    }
}
```

`main` is the special entry section. `start` is `on` when the file is run directly, and `off` when it is imported as a library.

---

## HTTP

`reach` is the built-in HTTP client. It follows redirects automatically and does not need to be imported.

```thorn
res = reach("https://example.com")
res = reach.post("https://example.com", data)
write(res.json)
```

---

## Files

Files are a first-class type in Thorn. There is no open/close boilerplate.

```thorn
dcmnt = read("data.txt")
loop(line = dcmnt.line.all) {
    write(line)
}

dcmnt.append("new content")
dcmnt.replace("old", "new")
```

---

## Standard library

```thorn
need time    # now.hour, now.day, now.month, etc.
need math    # math.pi, x.sqrt, x.round, x.abs, x.ceil, x.floor
need random  # roll(10), roll(1, 100), roll(myList)
need events  # broadcast("name", value), listen("name", handler)
```

---

## CLI

```
thorn run file.thorn
thorn build file.thorn
thorn --version
thorn help
```

---

## Status

Thorn is in active development and the v0.3 spec is still a draft. The compiler is written in C. A Java Swing IDE is being developed alongside it. Things will change.

---

## Licence

Modified AGPLv3 with a Public Distribution Clause. See `LICENSE` for the full terms.
