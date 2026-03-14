# Thorn
## Language Reference Manual, Version 0.3 Draft

## 1. Overview

Thorn is a general-purpose programming language designed to be a universal programming language that's actually nice to write in. It has inferred types, C-style curly braces, no semicolons, and a standard library covering I/O, files, networking, and concurrency. Thorn compiles to native binaries via C.

Design goals:
- Readable to any programmer in most fields.
- Easy to debug (hence why sections)
- Simple and English-like keywords.
- Should not require a PhD to understand.

## 2. Variables

No declaration keyword. Assign a value and Thorn creates the variable. The type is inferred from the value.

```
x = 5
name = "Thorn"
active = on
nothing = void
```

The first assignment creates the variable. Every subsequent assignment overwrites it.

## 3. Types

Thorn has six built-in value types. The compiler infers all of them.

| Value | Type |
|---|---|
| `5` | number (integer) |
| `3.14` | number (decimal) |
| `"hello"` | text |
| `on / off` | boolean |
| `void` | null / nothing |
| `[1, 2, 3]` | list |
| `{"key": "val"}` | map |

Booleans use `on` and `off` instead of true/false. `void` represents the absence of a value.

## 4. Tasks (Functions)

Functions are called tasks. Define them with the `task` keyword. Return a value with `send`.

```
task add(x, y) {
    send x + y
}

task greet(name) {
    write(v"hello {name}")
}
```

Capturing a return value:

```
result = double(5)          # result is 10
```

## 5. Output

`write()` prints any value to standard output.

```
write("hello world")
write(42)
write(myVariable)
```

## 6. V-Strings (String Interpolation)

Prefix a string with `v` to embed variables inside it. Regular strings do not interpolate.

```
name = "world"
write(v"hello {name}")           # prints: hello world
```

Any expression can go inside the curly braces.

## 7. Comments

Comments start with `#` and run to the end of the line. There are no block comments.

```
# this is a comment
x = 5    # inline comment
```

## 8. Conditionals

Standard `if` / `elif` / `else`. Conditions go in parentheses.

```
if (x > 5) {
    write("big")
} elif (x == 3) {
    write("three")
} else {
    write("small")
}
```

## 9. Loops

Thorn has a single `loop` keyword that covers every looping pattern.

Forever:

```
loop(on) {
    # runs until broken out of
}
```

N times (no counter):

```
loop(15) {
    # runs exactly 15 times
}
```

N times (with counter), counter starts at 0, goes up to N-1:

```
loop(i = 15) {
    write(i)        # 0 through 14
}
```

Condition:

```
loop(x < 100) {
    x = x + 1
}
```

Over a list:

```
loop(item = myList) {
    write(item)
}
```

Over a map:

```
loop(pair = myMap) {
    write(pair.key)
    write(pair.value)
}
```

## 10. Error Handling

Thorn uses `risk` blocks. Errors become first-class values rather than exceptions.

Basic risk:

```
risk {
    # risky code here
}
```

Custom error prefix:

```
risk("network error: ") {
    # risky code here
}
```

Call a handler on failure:

```
risk(myHandler) {
    # risky code here
}
```

Capture the error as a value:

```
err = risk {
    # risky code here
}
if (err != void) {
    write(v"failed: {err}")
}
```

try/catch (full control):

```
try {
    # risky code
} catch(e) {
    write(e)
}
```

## 11. Concurrency, spawn

`spawn` runs a task independently. Execution continues immediately without waiting.

Fire and forget:

```
spawn(fetchData(url))
spawn(processFiles(folder))
```

Capture the result:

When you use the variable, Thorn waits for the task to finish if it has not already.

```
out = spawn(fetchData(url))
write(v"got: {out}")
```

## 12. The .force Modifier

`.force` can be chained to any command. It means: do this even if something is in the way.

```
file("output.txt").force                        # create even if file already exists
folder.create("dist").force                     # create even if folder already exists
myFolder.delete.force                           # delete even if folder is not empty
reach("https://example.com").force              # skip SSL errors
```

`.force` cannot override an OS-level file lock held by another process.

## 13. Imports

Use `need` to import from other files or from built-in modules.

```
need "myfile"                            # import everything
need "myfile" as m                       # import with alias
need myTask from "myfile"                # import specific item
need time
need math
need random
```

## 14. Sections

Sections group related tasks and variables together. They are Thorn's answer to organisation, a lighter alternative to classes that does not force object-oriented patterns.

```
section networking {
    task connect(url) {
        # ...
    }
}
networking.connect("pawnet://ABC12")
```

Visibility modifiers:

By default a section is private. Three modifiers control access:

```
section.global networking {                  # any section can access this
    # ...
}

section.parent database {                    # only the parent section can access this
    # ...
}

section(ui, main) networking {               # only ui and main can access this
    # ...
}
```

Entry point:

`main` is the special entry section. Code outside any section runs before `main`. `start` is `on` when the file is run directly and `off` when it is imported as a library.

```
section main {
    section.global networking {
        task connect(url) { }
    }
}

if (start) {
    main()
}
```

## 15. Lists

```
nums = [1, 2, 3]
nums[0]                 # get item at index (starts at 0)
nums.add(4)             # append to end
nums.remove(0)          # remove by index
nums.length             # number of items
```

Iterate:

```
loop(item = nums) {
    write(item)
}
```

## 16. Maps

```
person = {"name": "Alice", "age": 30}
person["name"]              # get value by key
person.set("name", "x")    # add or update a key
person.remove("name")       # remove a key
person.has("name")          # on/off
person.keys                  # list of all keys
person.values                # list of all values
person.size                  # number of entries
person.empty                 # on if map has no entries
```

## 17. Strings

```
s = "hello world"
s.length                            # character count
s.upper                             # HELLO WORLD
s.lower                             # hello world
s.toList.letter                     # ["h","e","l","l","o",...]
s.toList.word                       # ["hello","world"]
"hello" + " " + "world"            # concatenation
v"hello {name}"                     # interpolation
```

## 18. Type Conversion

```
x.toStr                     # convert to text
x.toInt                     # convert to integer
x.toChar                    # convert to character
x.toList.letter             # string to list of letters
x.toList.word               # string to list of words
```

## 19. Files

Files are a first-class type. No open/close boilerplate.

```
dcmnt = read("path/to/file")                   # open existing file
dcmnt = file("path/to/file")                   # create new file
dcmnt = file("path/to/file").force             # create, overwrite if exists

dcmnt.line(14)                                  # get line 14
dcmnt.line("NAME: THOR")                       # get line by exact match
dcmnt.line.starts("NAME: ")                    # get line starting with text
dcmnt.line.ends("THOR")                        # get line ending with text
dcmnt.line.contains("THOR")                    # get line containing text
dcmnt.line.match("N*ME: T??R")                 # wildcard match
dcmnt.line.count                               # total line count
dcmnt.line.all                                 # all lines as a list

dcmnt.write("content")                        # overwrite entire file
dcmnt.append("new line")                      # append to end
dcmnt.clear                                    # empty the file
dcmnt.replace("old", "new")                   # find and replace

dcmnt.size                                     # file size
dcmnt.name                                     # filename
dcmnt.path                                     # full path
dcmnt.type                                     # file extension
dcmnt.empty                                    # on if file is empty
dcmnt.exists                                   # on if file exists
dcmnt.copy("path/to/dest")
dcmnt.move("path/to/dest")
dcmnt.delete
```

## 20. Folders

```
f = folder("path/to/dir")                      # open existing folder
f = folder.create("path/to/dir")               # create new folder
f = folder.open("path/to/dir")                 # open or create, never errors

f.files                               # list of all files
f.folders                             # list of subfolders
f.contains("myfile.txt")              # check if file exists inside
f.name / f.path / f.empty
f.delete / f.move("path") / f.copy("path")
```

## 21. HTTP, reach

`reach` is Thorn's built-in HTTP client. Redirects are followed automatically.

```
res = reach("https://example.com")           # GET
res = reach("https://example.com", data)     # POST shorthand
res = reach.post("https://example.com", data)
res = reach.put("https://example.com", data)
res = reach.delete("https://example.com")
res = reach.patch("https://example.com", data)

res = reach("https://example.com") {
    auth: "Bearer mytoken"
    timeout: 5000
    headers: {"Accept": "application/json"}
}

res.body            # response body as text
res.status          # HTTP status code
res.json            # body parsed as JSON
res.headers
res.cookies
```

## 22. Built-In Modules

**time:**

```
need time
now.day / now.month / now.year
now.hour / now.minute / now.second
```

**math:**

```
need math
x.sqrt / x.round / x.abs / x.ceil / x.floor
math.pi
```

**events:**

```
need events
broadcast("overheat", temp)                          # send an event with a value
listen("overheat", task(val) {                       # register a handler
    write(v"got: {val}")
})
```

`listen` is non-blocking. It registers the handler and moves on. `broadcast` fires all registered handlers for that event name.

**random:**

```
need random
roll(10)                # random integer 0 to 10
roll(1, 100)            # random integer between 1 and 100
roll(myList)            # random item from a list
```

## 23. Keyword Reference

| Keyword | Description |
|---|---|
| `task` | Define a function |
| `send` | Return a value from a task |
| `write()` | Print to standard output |
| `if` | Conditional branch |
| `elif` | Additional conditional branch |
| `else` | Fallback conditional branch |
| `loop()` | All looping patterns |
| `risk {}` | Error handling, errors become values |
| `try` | Full-control error handling |
| `catch` | Catch block for try |
| `spawn()` | Run a task concurrently |
| `need` | Import from a file or built-in module |
| `section` | Group related tasks and variables |
| `on` | Boolean true |
| `off` | Boolean false |
| `void` | Null / absence of value |
| `start` | on when run directly, off when imported |
| `.force` | Push through obstacles |
| `reach()` | Built-in HTTP client |
| `read()` | Open an existing file |
| `file()` | Create a new file |
| `folder()` | Open or create a folder |
| `roll()` | Random number or item (need random) |

## 24. Full Example

```
need time
need math

task greet(name) {
    write(v"hello {name}, it is {now.hour}:{now.minute}")
}

task circleArea(r) {
    send math.pi * r * r
}

task readNames() {
    dcmnt = read("names.txt")
    loop(name = dcmnt.line.all) {
        write(name)
    }
}

task fetchData(url) {
    err = risk {
        res = reach(url)
        send res.json
    }
    if (err != void) {
        write(v"fetch failed: {err}")
        send void
    }
}

section.global ui {
    task render(data) {
        loop(item = data) {
            write(item)
        }
    }
}

section main {
    task run() {
        greet("world")
        area = circleArea(5)
        write(v"circle area: {area}")
        readNames()
        data = fetchData("https://api.example.com/items")
        if (data != void) {
            ui.render(data)
        }
    }
}

if (start) {
    main()
}
```
