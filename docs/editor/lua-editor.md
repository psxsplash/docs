# The Lua Editor

![The Lua editor with the autocomplete list open](../assets/images/lua.png)

Lua files open in a tab next to **Scene** and **UI**. Open one by double-clicking it in the Project panel, or with **Edit script** on a Script component. **New Lua** on a Script component writes `scripts/<object name>.lua` with `onCreate` ready to fill in and opens it. See [Lua scripting](../lua/index.md) for the language and the API.

You can write every script in Lua, build every script from [blocks](block-editor.md), or mix them in one scene. Neither is the default, and both end up as the same Lua on the console.

## What the editor does for you

- **Highlighting** for Lua 5.2 and the psxsplash API.
- **Compile check.** A moment after you stop typing, the script is compiled the way the exporter will compile it. The bar at the bottom reads **Compiles for the PS1**, or shows the first error with its line; click it to jump there, and a squiggle marks the spot.
- **Autocomplete.** Type `Entity.` and the list shows every function in the table, with its signature. **Ctrl+Space** opens the list on demand. **Enter** or **Tab** accepts. Your own functions and variables are listed too.
- **Signature hints** while you type the arguments of a call.
- **Go to definition.** **Ctrl+click** a name, or press **F12**, to jump to where your script defines it. Hovering an API function shows what it does.
- **Find and replace.** **Ctrl+F** finds, **Ctrl+H** replaces. Options: **Match case**, **Whole word**, regular expression (with `$1` in the replacement) and **Preserve case**, plus **Find in selection**. **F3** and **Shift+F3** step through matches; **Ctrl+Alt+Enter** replaces all in one undo step.
- **Ctrl+/** comments the selection. **Tab** and **Shift+Tab** indent and outdent.
- **Ctrl+S** saves; **Ctrl+Z** and **Ctrl+Y** undo and redo. The status bar shows the line and column, and `Lua 5.2 (psxlua)`.

Press **Play** and open scripts are saved first.

## Files changed outside the editor

If a script changes on disk while it is open (your own text editor, git) and the tab has unsaved edits, the tab says **Changed on disk** and offers **Reload** or **Keep mine**. **Ctrl+Z** after a reload brings your edits back. A file deleted on disk can be written again with **Save**.

Closing a tab with unsaved edits asks first: **Save**, **Don't save** or **Cancel**.

## Using VS Code or another editor

**View > Set up external editor (VS Code)** writes two files into the project folder:

- `psxsplash.d.lua`, the psxsplash API as [lua-language-server](https://luals.github.io/) definitions: completion and signatures for every table and function, with hover docs where the API reference has them, made from the same list the editor's autocomplete uses.
- `.luarc.json`, which sets Lua 5.2 and turns off the libraries the PS1 does not have (`math`, `io`, `os`, `require`). It is only written when the project has none, so your own settings are kept.

Open the project folder in VS Code with the **Lua** extension (sumneko) installed. When a newer editor opens the project, it brings `psxsplash.d.lua` up to date on its own.

## Fixed-point numbers

The PS1 has no floating-point unit, so a decimal like `0.5` is not a number in these scripts. Read [Fixed-Point Math](../lua/fixed-point.md) first. The compile check accepts what the compiler accepts, so it will not flag a script that does the wrong arithmetic.
