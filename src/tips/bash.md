# Bash

_Disclaimer: this entry is Bash-specific; some of the tips outlined below might or might not intersect with other shells_\
_NB: `Ctrl+` shall be annotated `C-`. `Alt+` shall be annotated `M-` (as in Meta)_

## Using an $EDITOR for editing [long] commands

POV you're typing up a long command, and the terminal starts playing tricks on you. Tricks like the cursor moving all the way back to the beginning of the line, [visually] destroying your PS1, and part or all of the command being input. This is often a byproduct of terminal resizing. Let's say you started a maximized terminal window, say 50 rows 190 columns, then you decided to split the terminal vertically. The terminal size is now, say, 50 by 94. Dynamic sizing may not always be 100% reliable in some cases, so what's the best options here?

We can edit the commands using an editor, as opposed to using the shell prompt. I personally prefer using VIM for many reasons, with keybinds and convenience specifically taking the cake. Your preferred editor might be something else, so we're gonna wanna make sure we set the default editor first before proceeding

```sh
$ sudo update-alternatives --config editor
There are 4 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path                Priority   Status
------------------------------------------------------------
  0            /bin/nano            40        auto mode
  1            /bin/nano            40        manual mode
  2            /usr/bin/nvim        30        manual mode
* 3            /usr/bin/vim.basic   30        manual mode
  4            /usr/bin/vim.tiny    15        manual mode

Press <enter> to keep the current choice[*], or type selection number:
```

It's simple, really; all you have to do is input the Selection index that corresponds to the editor you want to set as a default. I already had `vim.basic` selected by previously inputting 3. Yours may very well start at 0 (`/bin/nano`), so you can hit Enter to leave the selected option as-is, or input another index

Next, we'll use Bash's handy `edit-and-execute-command` by invoking the shortcut combination (Ctrl+x Ctrl+e), or (C-x C-e) as Bash prefers to annotate it in the man-pages. The concept is simple: enter (or edit) whatever command you have, save the buffer and quit to execute said command, or quit without saving to abort it. What happens when you edit a command using an editor is that this command is saved in a temporary file in `/tmp/bash-fc.{random-identifier}`, then either the command(s) stored in that file are executed (upon saving the buffer and quitting), or the file gets deleted (upon quitting without saving)

## Character repitition

This is for all the buffer overflow lovers out there. Have you ever caught yourself doing something silly like `python -c 'print("A"*200)'`? Let's look at the numbers, Jim; that's about 31 keystrokes, right?

Let's bring those down to 6. Alt+{count}, followed by the character we want repeated, "A" in this case, does just that. Try Alt+200 A; you hold down the Alt key while inputting 2,0,0, release the Alt key, then Shift+a (A)

What about all them hash-passers? I'm talking specifically about the LM portion of NTLM hashes where you want to pass 32 zeros. As you might have already realized, you cannot pass digits as literals for readline. So, how exactly do you do `M-32 0` if it's gonna end up thinking you want to repeat 320 characters, whereas you really want 32 _zeroes_? Well, `C-v`!

The combo goes as follows: `M-32 C-v 0`. That is: count 32 (`M-32`), add the next character (0) typed to the line verbatim (`C-v`)

## Movement

You've entered a somewhat long command, and then you realized you messed up a certain word at the beginning of it, middle, or wherever. Do you keep furiously pressing the arrow keys on your keyboard to move the cursor around? No...No, you do not, my friend. You learn how to do things faster and more efficiently, so let's cover some basics

- `C-a`: move the cursor to the start of the current line
- `C-e`: move the cursor to the end of the current line
- `C-f`: move the cursor one character forward
- `C-b`: move the cursor one character backward
- `M-f`: move the cursor one word forward
- `M-b`: move the cursor one word backward

We can also combine the beauty of readline's convenience with some of the above movements. If you know your way around VIM movements, this part is gonna be breezy for you. Let's have a look at the following example

```sh
$ echo The quick brown fox jumps iver the lazy dog||    # where || indicates the cursor position
```

Let's say we want to jump back to the word "iver" to fix the typo. We can do so by issuing the following combo: `M-4-b`. That is, move the cursor 4 words back, where "dog" is the first word, "lazy" second, "the" third, and finally "iver" fourth

## File/directory name typos

Let's admit it, we've all been there. `ls /user/bin`, or `ls /usr/lin`, followed by the inevitable "ugh". No one likes that. Did you know that Bash supports spelling correction? There are _some_ caveats, but I'll leave those for you to figure out. Hint: chained commands, cursor position, starting versus ending characters

Let's have a look at the following example where we messed up not one, but _three_ words. Normally, we'd either rewrite the entire thing, or go back one character/word at a time, which is not cool

```sh
$ ls -l /user/bim/vash
```

So how do we fix this quickly? `C-x s`. That is `Ctrl+x` then `s`. It really is that simple. We go from `/user/bim/vash` to `/usr/bin/bash`. Pretty cool, eh?

## Text manipulation

- `C-d`: delete the character under the cursor (forward deletion)
- `M-d`: delete the word under the cursor (forward deletion)
- `C-w`: delete one word behind the cursor
- `C-u`: delete everything behind the cursor
- `M-r`: delete everything regardless of cursor position
- `M-#`: comment out the current line (useful if you wanna hold onto a command for later reference)
- `M-t`: transpose words behind the cursor (useful for flipping argument ordering, for example)
- `C-t`: transpose characters behind the cursor (useful for fixing typos like `claer` => `clear`)
- `M-l`: lowercase word under the cursor
- `M-u`: uppercase word under the cursor
- `M-c`: capitalize word under the cursor

## Macros

Let's say there's a handful of repeating commands/functions that you'd like to issue, but you couldn't be bothered to write a script/alias/function for it, or you simply only need them for that one shell session. There is where keyboard macros shine. The process is as follows: start recording the macro, type out (and/or execute) whatever commands to be recorded, end the macro recording, and finally execute the macro at any point later in the same shell. Here's an example

```sh
# Ctrl+x (
$ echo hello world # this is where you start typing
hello world
# Ctrl+x )
# Notice we ended the recording _after_ executing the command
# meaning invoking the macro will _also_ execute the command
# Ctrl+x e
$ echo hello world
hello world
```
