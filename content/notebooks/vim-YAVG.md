---
title: "Yet Another Vim Guide"
date: 2020-05-20T21:06:05-06:00
draft: false
categories: ["tools"]
Authorbox: false
toc: true # Table of contents
thumbnail: /notebooks/img/vim-thumbnail.PNG
---

**Spending some time learning Vim to save time**
<!--more-->
There are a lot more commands to be known that are not shown here, and there are tons of books already explaining the full functionalities of Vim, but what this short guide delivers is a quick start and reference to the use of vim without even moving the elbow. What we want to achieve is efficiency in writing and editing documents. I spent some time "sharpening the saw", learning more about this fantastic CLI tool, and it was strange at the beginning, but turns out, it's a crazy fast and intuitive tool. The learning curve only feels steep until we understand the "Vim language" and how it works.

Note: Vim requires our commands to be very precise, and some of them are relative to the position of the cursor, so it's convenient to modify the cursor to "block" instead of "line" to avoid confusions.

---

**Use cases and differences between :w, :q, :q! :wq, :wq! and :x**

:w is for write, :q is for quit. :wq combines them. "!" forces the command. x will write + quit, but it will only write if there were any changes in the document.

This :x makes sense: If we open a document only for reading, and wq, it will change the date of last editing, whereas x will not change anything.

The commands at *normal mode* can be "built" by chaining:

**verb + modifier + noun**

> *Example: ```ciw``` (**C**hange **I**nside **W**ord).*

Most of the verbs/modifiers/nouns can be used alone.

> *Example: ```/string``` (Search for the word "string" in the whole document)*

### Verbs

> ```c``` - "Change": It will delete the desired selection and put it into the buffer. Starts "insert" mode.
>
> ```v``` - Visually select: highlights text for manipulation or for visual reference. (See examples)
>
> ```d``` - "Delete": Same as "change", but it keeps the "normal" mode.
>
> ```y``` - "Yank". (Copy selection)

***c** and **d** (change/delete) seem to accomplish the same <<cutting and putting data into the buffer for pasting later>> functionalities, and the only difference I can notice is that "c" enters "insert" mode right after, whereas "d" maintains "normal" mode.*

### Modifiers

> ```i/a``` - Inside and around selection. (See examples)
>
> ```numbers``` - Used to repeat the command for "n" iterations.
>
> ```/``` - search string. (See Examples)
>
> ```t/f``` - to/into search string. (See examples)


### Nouns

> ```w/W``` - words/whole words. (Can also be used for movement. See Moves)
>
> ```s/p``` - Sentences/paragraphs. (Not to confuse with substitute & paste)

### General purpose actions

> ```p/P``` - paste after/before the cursor.
>
> ```i/a/A``` - insert mode before / after the cursor / after the whole line (Think "**A**ppend").
>
> ```o/O``` - Insert new line after/before cursor position + enter Insert mode.
>
> ```r/R``` - replace single character and return to normal mode / replace mode: Keeps replacing character by character until Esc.
>
> ```s/S/C``` - Substitute character (Deletes character under cursor and start Insert mode) / Whole line from, and until the next "." / Rest of the line, starting from the cursor position. (Vim seems to consider sentences as anything between 2 periods, regardless of their distance and position, so watch out since s/S is a greedy command)
>
> ```x/X``` - Deletes character under/before the cursor.
>
> ```u/Ctrl+r``` - Undo/Redo actions.
>
> ```J``` - Join the paragraph with the next. (Delete everything between the end of the current paragraph and the next: spaces, empty lines)
>
> ```.``` - Repeat last command. Can be used with numbers. i.e. ```4.``` will repeat the last command 4 times.
>
> ```~``` - Switch upper/lowercase in current position. For whole lines/phrases/words, needs to be combined with visual mode, or selection.

### Moving around in "normal" mode

> ```h, j, k, l``` - Move the cursor. In order: Left, down, up, right.
>
> ```0/$``` - Move cursor to the beginning / end of line.
>   
> ```t/f``` - Jumps up to/into the following character in the same line. (Example: t9 will jump to the next "9" character, stopping at the preceeding character, and f9 will jump INTO the next "9" character)
>
>```w/W``` - jump-to the next word / Big word. (For example, "/etc/passwd" is a "big word"; "etc" or "passwd" would be just "words")
>
> ```b/B``` - jump-back to previous word / Big word.
>
> ```e``` - Jump-to the last character of the current word.
>
> ```) & }``` - Jump-to the next sentence/paragraph. Works backwards with ( and {.
>
> ```H / L``` - Jump to the Top/Bottom of the screen.
>
> ```gg / G``` - Jump to the Top/Bottom of the Document.
>
> ```:13``` - Jump to the line number 13. (Set **line numbers** with ":set number")

### Searching through the file

> ```\<string>``` - Searching forward.
>
> ```?<string>``` - Searching Backwards.
>
> ```*``` - Searches for the next word, matching the one the cursor is in. (If we are already "in" a specific word, instead of searching for it by typing it, we can ```*``` and it will search the next occurrence)
>
> ```n/N``` - To be used after a search: next / previous result.


### Examples

*Not to be memorized, but to be familiar with how Vim "understands"*

> ```dd``` - Cut the entire row, and put it into the memory buffer. It's often used for simple deletion but it can be pasted elsewhere.
>
> ```yy``` - Copies ("yanks") the entire row.
>
> ```cis``` - Delete "inside sentence". Everything after the last "." and until the last ".". Think "Change inside sentence".
>
> ```d/Rodriguez``` - Will delete everything from the cursor to the word "Rodriguez", excluding "Rodriguez" itself.
>
> ```vf(``` - Select (Visual mode) from cursor to the next "(", including it. ```vt(``` would select everything up to "(" without including it.
>
> ```yiW``` - Yank (copy) the current word. Think "yank inside word"
>
> ```y5aw``` - Copy the following 5 words. Think "Yank 5 (around) words". I'm using ```a``` instead of ```i``` because ```y5iw``` would consider spaces as "words" too, so for example, the string "I have a cow" would need ```y7iw``` to grab it correctly.
>
> ```c2w``` - Delete 2 words from the current cursor position. Think "change 2 words".
>
> ```c2aw``` - Delete 2 words, including spaces Around the current word. Think "change 2 around word(s)".
>
> ```c2iw``` - Delete 2 words, but doesn't include the spaces after the current word (Better use "a" than "i" in most cases). Think "change 2 inside word(s)".
>
> ```dw``` - Cuts the current word. Think "delete word"
>
> ```d$``` - Cuts everything after the cursor, until the end of the line. ($ is strange to memorize, but it can be remapped to something more meaningful).
>
> ```dis``` - Delete inside sentence. Greedy! Will cut everything from the last "." to the next one.
>
> ```vaw ~``` switch upper/lowercase in current word. Think "**V**isual(select) **a**round **w**ord".
>
> ```t(``` - Jumps the cursor to the next "(" in the current line. Think "'til (".
>
> ```dt(``` - Delete everything from the cursor until the next "(". Think "delete 'til (".
>
> ```fC caw``` - Jump-to the next "C", and then delete the whole word. Think "Forward to C, change around word".
>
> ```d:36``` - Will Cut everything from the cursor, until the 36th line. Think "delete until 36"

**Inside tags in programming:**

> ```da}``` - Delete everything inside {} (Including brackets) when coding. We don't need to be positioned at the initial "{".
>
> ```di}``` - Delete everything inside {}, excluding brackets.
>
> ```ca}``` / ```ci}``` - Same as before, but it enters "insert" mode after deletion.

## Macros

The most basic macros process there is:

> ```qz``` ("q" starts recording and "z" is the name of the macros; it could be a,b, etc.)
>
> *start editing the file*
>
> ```q``` (Stop recording)
>
> ```@z``` (Start this macros once)
>
> \* Or ```:%:normal %z``` will repeat the macros until the end of the file.

## File navigation

```:e <tab>``` - Start navigating files in cwd with \<tab\>. Current work needs to be saved or discarded before opening a new file.
```:pwd``` - Print CWD.

## Mapping keys

*It can be done temporarily by just typing the mappings but for it to be permanent we need to edit the vimrc file*

**Use:** ```<mapping command> <old> <new>```

*Examples:*

```:map $ '``` - It will add ```'``` as a new command for ```$``` or End-of-line in normal mode. It depends on the keyboard Layout, but I tested it in one which has ```0``` next to ```'``` so in this case it made sense to map together End-of-line and Beginning-of-line.

```:inoremap jk <Esc>``` Mapping in insert mode: Instead of switching to normal mode with Esc, type ```jk```. The default key delay is ~1s. I've seen people using ```;;``` instead, or ```jj```. It depends on what combination is easier to remember and type, and what is most unlinkely to be typed in text, given one's language.

```ZZ``` is sometimes already re-mapped by default to ":wq"


## Checking the current Vimrc file

```:e $MYVIMRC``` - ~/.vimrc is not always the initialization script, so checking the MYVIMRC variable is sometimes a good idea when working under a new environment.

---

And for a basic usage this is enough. Once some muscle memory is developed, we can speed up and trim down precious minutes of work with Vim.
This quick reference covered the most basic usage, but it can grow in complexity almost infinitely (Check references if you want to go deeper).

![Vim Books](/notebooks/img/vim-books.PNG)

...So yeah, there's A LOT Vim can do.

## References

* [“From Vim Muggle to Wizard in 10 Easy Steps” from Erik Falor (Youtube)](https://www.youtube.com/watch?v=MquaityA1SM)
* [MIT “Missing Semester” lecture on Vim (Youtube)](https://www.youtube.com/watch?v=a6Q8Na575qc)
* [Mastering the Vim language (Youtube)](https://www.youtube.com/watch?v=wlR5gYd6um0)
