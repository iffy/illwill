# illwill

## Overview 

**illwill** is a *(n)curses* inspired simple terminal library that aims to make
writing cross-platform text mode applications easy. Having said that, it's
*much* simpler than (n)curses and it's not as robust *by far* in terms of
supporting different encodings, terminal types etc. The aim was to write
something small and simple in pure Nim that works for 99.99% of users without
requiring any external dependencies or a terminfo database.

![nimmod running on Windows](img/modplayer.png)

For "serious" applications, the best is always to write different backends for
*nix and Windows (one of the reasons being that the Windows Console is buffer
based, not file based). But I think this library is perfect for small
cross-platform programs and utilities where you need something more than the
basic blocking console I/O but you don't actually want to bother with
a full-blown GUI.

Everybody likes lists, so here's the bullet point version:

**Use it if**

* you just want something simple that lets you write full-screen terminal
  apps with non-blocking keyboard input that works on Windows, Mac OS X and
  99.99% of modern Linux distributions
* you don't need to support any fancy encodings and terminal types
* you're developing a custom UI so you don't need any predefined widgets
* you don't mind the [immediate mode UI](https://github.com/ocornut/imgui)
  style/mindset
* you don't want any external dependencies

**Don't use it if**

* you need ultimate robustness in terms of supporting obscure terminals,
  character encodings and Linux/Unix distributions
* you need predefined widgets
* you like complicating your life :)


### Main features

* Non-blocking keyboard input.

* Support for key combinations and special keys available both in the standard
  Windows Console (cmd.exe) and most common POSIX terminals.

* Virtual terminal buffers with double-buffering support (only display changes
  from the previous frame and minimise the number of attribute changes to
  reduce CPU usage).

* Simple graphics using UTF-8 box drawing symbols.

* Full-screen support with restoring the contents of the terminal after exit.

* The module depends only on the standard terminal module. However, you should
  not use any terminal functions directly, neither should you use echo, write
  or other similar functions for output. You should only use the interface
  provided by the module to interact with the terminal.

The following symbols are exported from the terminal module (these are safe to
use):

terminalWidth(), terminalHeight(), terminalSize(), hideCursor(), showCursor(),  Style

### Limitations

* Suspend/resume (SIGTSTP/SIGCONT handling) works, but it doesn't properly
  reset the terminal when suspending the app.

* The contents of the terminal is not restored after exiting the app on
  Windows in full-screen mode.

## Installation

The best way to install the library is by using `nimble`:

```
nimble install illwill
```

## Usage

This is a simple example on the general structure of a fullscreen terminal
application. Check out the [examples](/examples) for more advanced use cases
(e.g. using box drawing buffers, handling terminal resizes etc.).


```nimrod
import os, strutils
import illwill

# 1. Initialise terminal in fullscreen mode and make sure we restore the state
# of the terminal state when exiting.
proc exitProc() {.noconv.} =
  illwillDeinit()
  showCursor()
  quit(0)

illwillInit(fullscreen=true)
setControlCHook(exitProc)
hideCursor()

# 2. We will construct the next frame to be displayed in this buffer and then
# just instruct the library to display its contents to the actual terminal
# (double buffering is enabled by default; only the differences from the
# previous frame will be actually printed to the terminal).
var cb = newTerminalBuffer(terminalWidth(), terminalHeight())

# 3. Display some simple static UI that doesn't change from frame to frame.
cb.setForegroundColor(fgBlack, true)
cb.drawRect(0, 0, 40, 5)
cb.drawHorizLine(2, 38, 3, doubleStyle=true)

cb.write(2, 1, fgWhite, "Press any key to display its name")
cb.write(2, 2, "Press ", fgYellow, "ESC", fgWhite,
               " or ", fgYellow, "Q", fgWhite, " to quit")

# 4. This is how the main event loop typically looks like: we keep polling for
# user input (keypress events), do something based on the input, modify the
# contents of the terminal buffer (if necessary), and then display the new
# frame.
while true:
  var key = getKey()
  case key
  of Key.None: discard
  of Key.Escape, Key.Q: exitProc()
  else:
    cb.write(8, 4, ' '.repeat(31))
    cb.write(2, 4, "Key pressed: ", fgGreen, $key)

  cb.display()
  sleep(20)

```

## License

Copyright © 2018 John Novak <<john@johnnovak.net>>

This work is free. You can redistribute it and/or modify it under the terms of
the **Do What The Fuck You Want To Public License, Version 2**, as published
by Sam Hocevar. See the `COPYING` file for more details.

