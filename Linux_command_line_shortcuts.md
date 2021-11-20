## Common used command line shortcuts

### Functional commands

```
Tab	Commands, pathnames, filenames completions.
Ctrl-l	Clear the screen, reprinting the current line at the top.
Ctrl-s	Freeze the current terminal session.
Ctrl-q	Restore the current terminal session.

Ctrl-c	Terminate the current task.
Ctrl-z	Suspend the current job.
```

### Moving cursor

```
Ctrl-a	Move to the start of the line.
Ctrl-e	Move to the end of the line.
Ctrl-x-x	Switch cursor between the start and end of the line.

Alt-f	Move forward a word,where a word is composed of letters and digits.
Alt-b	Move backward a word,where a word is composed of letters and digits. 
```
### Command line editing

```
Ctrl-u	Cut the text from the current cursor position to the start of the line.
Ctrl-k	Cut the text from the current cursor position to the end of the line.

Ctrl-w	Cut from the cursor to the start of the current word, or, if between words, to the start of the previous word.
Alt-d	Cut from the cursor to the end of the current word, or, if between words, to the end of the next word.

Ctrl-y	Yank the most recently cutted text at the cursor.

Alt-u	Uppercase the current (or following) word.
Alt-l	Lowercase the current (or following) word.
Alt-c	Capitalize the current (or following) word.
```

### Searching for, or feferring to specified command line in history

```
Ctrl-p	Fetching the previous command in history record chain, by repeating to move back in the chain.
Ctrl-n	Fetching the next command in history record chian, by repeating to move forward in the chain.

Ctrl-r	Reverse-i-search mode. Reversely searching for the command including the string which you input.
Ctrl-g	Exit Reverse-i-search mode.

Alt-.	Insert the last argument of the previous command. Rotating the history record by repeating. 

!n	Refer to command line n.
!`string`:p	print the most recent command line containing `string`.
!#	The entire command line typed so far.
```

Notice: Most of the contents above is from the `info bash`. You also can find them on GNU website https://www.gnu.org/.

------
