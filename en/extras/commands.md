# Your own commands

A script can add a client command — the same kind as `.party` or `.macros`: with Tab completion.

## A simple command

```kotlin
command("ping") {
    runs { reply("pong") }
}
```

Done: `.ping` in chat answers "pong".

## Subcommands

```kotlin
command("home") {
    usage("<set|go|list>")
    alias("h")

    sub("set") {
        usage("<name>")
        runs {
            storage.put("home." + arg(0), player.position().toString())
            storage.save()
            reply("home " + arg(0) + " saved")
        }
    }

    sub("go") {
        usage("<name>")
        runs {
            val target = storage.get("home." + arg(0), "")
            if (target.isEmpty()) replyError("no home named " + arg(0)) else reply("home is at " + target)
        }
    }

    sub("list") {
        runs {
            val homes = storage.keys().filter { it.startsWith("home.") }
            reply("homes: " + homes.size)
        }
    }
}
```

`usage` is the hint in the error message, `alias` is a second name for the command. Subcommands can nest inside each other.

## Arguments

Inside `runs { }` you get:

```kotlin
arg(0)                  // first argument, errors if missing
argOr(0, "default")
intArg(0)
doubleArg(0)
booleanArg(0)           // true/on/yes/1 and false/off/no/0
args()                  // every argument as a list
argCount()
rest()                  // everything after the command as one string
rest(1)                 // everything from the second argument on
label()                 // how the command was called
```

`arg(0)` and the typed variants throw a clear error by themselves when an argument is missing or the wrong shape — you do not write those checks.

## Answering

```kotlin
reply("done")
replyError("that will not work")
```

Both are seen by you only.

## Tab completion

```kotlin
sub("go") {
    completes(0) {
        storage.keys()
            .filter { it.startsWith("home.") }
            .map { it.removePrefix("home.") }
    }
    runs { ... }
}
```

`completes(index) { ... }` supplies options for one argument. The lambda runs while the player types, so keep it cheap — no world walks, no disk reads.

A fixed list is shorter:

```kotlin
completes(0, "toggle", "hold", "action")
```

## Taken names

A command name is claimed across the whole client. If it already exists — in another script or in the client itself — registration fails with a clear error. Names are compared without case, so `.Home` and `.home` are the same thing.

## Commands and the switch

A script's commands only work while the script is **on**. Switch it off and the command stops answering and disappears from completions, but the name stays claimed so nobody else takes it.
