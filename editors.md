# Editors

- [Editors](#editors)
  - [Notes](#notes)
  - [Visual Studio Code](#visual-studio-code)
    - [bindings](#bindings)
      - [custom](#custom)
  - [Sublime Text](#sublime-text)
    - [bindings](#bindings-1)
      - [search/replace](#searchreplace)
  - [Meld](#meld)
  - [Less](#less)
    - [bindings](#bindings-2)
  - [Vim](#vim)
    - [Misc](#misc)
    - [settings](#settings)
    - [multi-files](#multi-files)
    - [selection/search/sort](#selectionsearchsort)
      - [multi-line selection](#multi-line-selection)
    - [examples](#examples)
    - [history](#history)
    - [vimdiff](#vimdiff)
    - [modeline](#modeline)
    - [plugins](#plugins)
    - [Peepcode notes](#peepcode-notes)

## Notes

The VIM section requires some cleanup.

## Visual Studio Code

### bindings

- `Ctrl+B`				            Show/hide sidebar
- `Ctrl+J`				            Show/hide console
- ``Ctrl+` ``                 Show/hide terminal
- `Ctrl+Shift+V`              Open file preview in tab
- `Ctrl+K, V`                 Show file preview (side-by-side)
- `Ctrl+\`                    Open split screen tab

- `Ctrl+K, M`                 Select language

- `Ctrl+Shift+O`              Go to symbol in current file (includes Markdown headers)
- `Alt+←`/`Alt+→`             Navigate through history
- `Ctrl+↓`/`Ctrl+↑`           Scroll the viewport without moving the cursor

- `Ctrl+K, Ctrl+L/U`		      Change case to Upper/Lower
- `Ctrl+K, Ctrl+X`		        Trim trailing whitespace (extension)

- `Ctrl+Space`                Variables autocompletion (at least in Shell script syntax)

- `Ctrl+K, P`                 Copy file path
- `Ctrl+Alt+C`                Copy file path (from explorer panel)
- `Ctrl+Alt+Shift+C`          Copy file relative path (from explorer panel)

#### custom

`Ctrl + @`:                 Open link in browser (custom)

## Sublime Text

### bindings

`Ctrl+Shift+D`              duplicate line or selection
`Ctrl[+K]+D`                when a word is selected, goes to the next instance and selects the current; if [+K], skips the current
`Ctrl+Alt+(select)`         start rectangular selection
`Ctrl+Shift+(Up/Down)`   		move a line
`Alt+Shift+2`               two columns (split screen)

#### search/replace

Search:

`Alt + R `                  Toggle Regular Expressions
`Alt + C `                  Toggle Case Sensitivity
`Alt + W `                  Toggle Exact Match
`Enter `                    Find Next
`Shift + Enter `            Find Previous
`Alt + Enter `              Find All

Replace:

`Ctrl + Shift + H`          Replace Next
`Ctrl + Alt + Enter`        Replace All

Misc:

`F3`                        Search Forward Using Most Recent Pattern:      
`Shift + F3`                Search Backwards Using Most Recent Pattern:    
`Alt + F3`                  Select All Matches Using Most Recent Pattern:  

## Meld

`Alt + PgUp/Down`           Switch panel

## Less

### bindings

`Shift + F`					        Follow (like tail -f)

## Vim

### Misc

`wq! <filename>`            Save to <filename> and quit
`:qa`                       Quit `a`ll
`:b(p|n)[#]`                Move `p`revious/`n`ext buffer [of `#` buffers]

`w/b`                       Move cursor words (forward/back)

`Ctrl + r`                  Redo
`Ctrl + p/n`                Auto-completion (search `p`revious/`n`ext)

`vim -u NONE`               Ignore configuration files

`:<params><command>`        Execute a command with the given params, eg. `:3,5bd` executes `bd` in the range `3,5`

Comment multiple lines: http://stackoverflow.com/questions/1883896/what-is-your-favorite-way-to-comment-several-lines-in-vim

### settings

`~/.vimrc`                  Startup cfg; store commands without ':'

`:set [no]ignorecase`
`:set [no]number`           Line numbers
`:set [no]wrap`             Search wrapping
`:syntax (on|off)`          Syntax hightlighting
`:set tabstop=x`            Tab size
`:set [no]expandtab`        Soft tabs
`:set shiftwidth=x`         Indent

### multi-files

`:Explore`                  File selection dialog
`:sav`                      Save as
`:buffers`                  Show all the buffers
`:bd bufNum`                Delete a buffer
`:3,5bd`                    Delete buffers in range

### selection/search/sort

`mc`                        Mark a line with the char “c”
`'c`                        Reference line c
`.`                         Current line
`%`                         Entire file
`$`                         Last line

`:itvs/patts/pattr/[c][i]`  replace, [c]onfirmation, [i]gnore case; see selection.examples for itv coding

`:itv sort sort lines for itv`

In order to type Ctrl chars, e.g. when searching/replacing, type Ctrl+V then the character, e.g.:

`:%s/<Ctrl-V><M>//g`

#### multi-line selection

Type `Ctrl+V` in order to enter "Visual mode"

Visual modes:

`v`                         Regular selection
`Ctrl+v`                    Rectangular selection

Search and replace. Vim will automatically fill the range, so that `%` must not be used:

`:s/search/replace`

Shift block of lines:

- `:set sw n`               Set the shift size as n
- (select the lines)
- `<` or `>`                Shift the lines


Add/remove characters
during change, only the first line will show thee difference; it will also take a small time after `Esc`

- (select the lines)
- type `Shift+I`
- make the change, then tap `Esc` to apply

### examples

`g'a`                       Goto line marked a
`:'a,$/ss/rr/g`             Replace ss with rr in the interval from line marked a to the end
`:%s/^M//g`                 Strip dos newline; to enter ^M, use Ctrl+V,Ctrl+M

### history

`q(:|/)`                    Visual history of commands/searches
`(:|/)[prefix]`             Clicking up/down goes through the history; if prefix is present, is used as filter

### vimdiff

`:set diffopt+=iwhite`      `i`gnore `white`space
`Ctrl+W,Ctrl+W`             Change frame


Remotely edit a file; the path after the port is relative to home, so the example file is in home:

`vim scp://user@myserver:port/restore_deleted_bookings.rb`

### modeline

Set in vimrc:

```
set modeline
set modelines=5
```

Format. put on top, or bottom of file:

`vim: setting=value{:setting=value}+`

### plugins

`Command-T`                 Textmate "Go to File" functionality
`snipMate`                  snippets
`BufExplorer`               having window showing open buffers

### Peepcode notes

`d2/end`                    Delete [d] until 2nd [2] occurance [s] (=> search) of word 'end'. the 2nd occurrance is not included.
`d3w`                       Delete [3] words [w]
`:h <command>`              Help
`:bd`                       Delete buffer; e.g. exit from help

Upper case commands are generally "extended" versions of the correspective lower case ones.

`I`                         Insert text at the start of the line
`w/b`                       Move forward/backward one word
`W/B`                       Move forward/backward one \S+ (e.g. myobject.mymethod)

`o`                         Create a new line under the current, and switch to insert mode
`c<cmd>`                    Change <command> (=> goes to insert). e.g. cw = change word
`f<char>`                   Go to the next occurence of <char>
