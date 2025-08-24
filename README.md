# lua-nxterm

NXTerminal Handling, Escape Sequences, Cursor, colors and stuff...

[![Lua](https://img.shields.io/badge/Lua-5.1+-blue.svg)](https://www.lua.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

## Description

`lua-nxterm` is a lightweight Lua library for handling terminal escape sequences, cursor control, colors, and other terminal manipulations. It provides an easy-to-use API for creating colorful and interactive terminal applications in Lua. Supports ANSI escape codes for styles, colors (including truecolor), cursor positioning, screen clearing, and more.

- **Version**: lua-nxterm
- **Author**: Sam Orlando
- **License**: MIT License
- **Copyright**: Copyright (C) Sam Orlando 2024

This library is designed for cross-platform use where terminals support ANSI escapes (e.g., Linux, macOS, Windows with modern terminals like Windows Terminal).

## Installation

1. Download `nxterm.lua` from this repository.
2. Place it in your project's directory or Lua module path.
3. Require it in your Lua script:

```lua
local nxterm = require('nxterm')
```

No external dependencies are required beyond standard Lua.

## Usage

### Basic Example

Here's a simple script demonstrating color and style usage (based on the provided example):

```lua
local nxterm = require('nxterm')

local sc = nxterm.color   -- screen colors
print( sc.red .. 'The color red ' .. sc.bg_blue .. 'now blue background' .. sc['reset bold yellow'] .. '!!!' )
print( sc'red bold ul' .. 'The color red bold and underlined' )
print( sc['green blink italic'] .. "The color green blinking and italic" )
print( sc'' .. 'Nice and boring.' )  -- empty will be reset of attributes

print( sc('r200g100b20') .. 'r200g100b20 (Truecolor)' )
print( sc('bg_r100g200b255 black') .. 'bg_r100g200b255 black (Truecolor backgrounds)' .. sc'' ) -- reset for background before return
print( sc('#BB0066') .. '#BB0066 (Truecolor)' )
print( sc('c88') .. 'c88 (palette colors)' )
print( sc('g10') .. 'g5 (palette grays)' )  -- Note: 'g5' in original, but example shows 'g10' – adjust as needed

local es = nxterm.escape_codes  -- escape string codes and return
print( es('This is %{red}red %{bold}bold %{ul}underlined %{reset}all in one.') )
print( es('This is %{r0g255b0}green %{bold}bold %{blink}blinking %{}all in one.') ) -- empty {}'s are resets as 0m is default
print( es('This is %{#0000BB bold blink}blue bold blinking and %{ul}underline %{}all in one.') )
```

This will output styled text in your terminal, demonstrating colors, backgrounds, truecolor, palette colors, and embedded escape sequences via `%{}` syntax.

### Cursor Control Example

Move the cursor around and get terminal size:

```lua
local nxterm = require('nxterm')
local cursor = nxterm.cursor

-- Move cursor up 2 lines
print(cursor.up(2))

-- Set cursor position to (10, 20) (row, column)
print(cursor.set(20, 10))  -- Note: set(x, y) where x is column, y is row

-- Get current cursor position
local pos = cursor.get()
print("Cursor at: x=" .. pos[1] .. ", y=" .. pos[2])

-- Get terminal size (columns, rows)
local size = nxterm.mode.size()
print("Terminal size: " .. size[1] .. "x" .. size[2])
```

### Screen Manipulation Example

Clear screen, erase lines, and scroll:

```lua
local nxterm = require('nxterm')

-- Clear the entire screen
print(nxterm.erase())

-- Erase current line
print(nxterm.line.erase())

-- Scroll up by 3 lines
print(nxterm.scroll.up(3))

-- Set terminal title
print(nxterm.title("My Lua App"))
```

### Color and Style Embedding

Use `nxterm.escape_codes` to embed styles in strings:

```lua
local es = nxterm.escape_codes
print(es('This text is %{red bold}red and bold%{reset}, then normal.'))
```

### Formatting with Styles

Use `nxterm.format` as a styled version of `string.format`:

```lua
local nxt_format = nxterm.format
print(nxt_format('Number: %d, Styled: %{red}%s%{reset}', 42, 'Hello'))
```

## API Reference

### General Functions

- `nxterm.title(s)`: Set the terminal title to `s`.
- `nxterm.reset()`: Reset the terminal.
- `nxterm.erase(n)`: Erase the screen (n=0: from cursor to end, n=1: from start to cursor, n=2: entire screen).
- `nxterm.clear()`: Alias for `nxterm.erase()`.
- `nxterm.line.erase(n)`: Erase the current line (similar modes as above).
- `nxterm.line.insert(n)` / `nxterm.line.delete(n)`: Insert/delete n lines.
- `nxterm.scroll.up(n)` / `nxterm.scroll.down(n)`: Scroll up/down by n lines.
- `nxterm.tab(n)` / `nxterm.backtab(n)`: Move forward/backward by n tabs.
- `nxterm.char.delete(n)` / `nxterm.char.erase(n)` / `nxterm.char.insert(n)` / `nxterm.char.rep(n)`: Manipulate characters at cursor.
- `nxterm.mode.set(m, dec)` / `reset(m, dec)` / `restore(m, dec)` / `save(m, dec)`: Set terminal modes (e.g., ANSI mode).
- `nxterm.tty.sane()` / `nxterm.tty.raw()`: Set TTY to sane/raw mode (Unix-like systems).
- `nxterm.mouse.on()` / `nxterm.mouse.off()`: Enable/disable mouse reporting (SGR mode).

### Cursor Control (`nxterm.cursor`)

- `up(n)` / `down(n)` / `forward(n)` / `back(n)`: Move cursor by n in direction.
- `nl(n)` / `pl(n)`: Next/previous line by n.
- `setx(n, absolute)` / `sety(n, absolute)`: Set x/y position.
- `set(x, y)` / `setlc(l, c)`: Set position (x=column, y=row or l=line, c=column).
- `save()` / `restore()`: Save/restore cursor position.
- `hide()` / `show()`: Hide/show cursor.
- `style(n)`: Set cursor style (0-6, e.g., block, underline).
- `get()`: Get current position as `{x, y}` (column, row).
- `getlc()`: Get as `{line, column}`.

Aliases: `right`=`forward`, `left`=`back`, `next_line`=`nl`, `prev_line`=`pl`.

### Terminal Size

- `nxterm.mode.size()`: Returns `{columns, rows}`.

### Colors and Styles (`nxterm.color` or `nxterm.colour`)

A metatable for generating SGR (Select Graphic Rendition) codes. Use as:

- `nxterm.color.red`: Returns escape for red foreground.
- `nxterm.color['red bold ul']`: Combined styles.
- `nxterm.color('r200g100b20')`: Truecolor RGB foreground.
- `nxterm.color('#BB0066')`: Hex RGB foreground.
- `nxterm.color('c88')`: 256-color palette.
- `nxterm.color('g10')`: Grayscale palette.

Prefixes: `bg_` for background, `ul_` for underline color.

Styles include: `bold`, `italic`, `underline`, `blink`, `strike`, etc. (see code for full list).

- `nxterm.color(keys, str, post_keys)`: Compile SGR for `keys`, wrap `str`, reset with `post_keys`.

### Escape Codes

- `nxterm.escape_codes(...)`: Process strings with `%{keys}` for embedded styles (e.g., `%{red bold}`).
- `nxterm.escape_strip(...)`: Remove `%{}` from strings.

### Printing and Formatting

- `nxterm.format(fstr, ...)`: Styled `string.format` handling `%{}`.
- `nxterm.writef(fstr, ...)`: Write formatted string.
- `nxterm.printf(fstr, ...)`: Print formatted with newline.
- `nxterm.print(...)`: Print with escape codes and newline.

## Contributing

Contributions are welcome! Open an issue or pull request for bugs, features, or improvements.

## License

MIT License

Copyright (c) 2024 Sam Orlando

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
