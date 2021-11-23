## Common command-line shortcuts

### Functional commands

```
Tab	Commands, pathnames, filenames completions.
Ctrl-l	Clear the screen, reprinting the current line at the top.
Ctrl-s	Freeze the current terminal session.
Ctrl-q	Restore the current terminal session.

Ctrl-c	Terminate the current task.
Ctrl-z	Suspend the current job and put it on the background.

Ctrl-Alt-Fn	Switch between consoles.
```

### Moving cursor

```
Ctrl-a	Move the cursor to beginning of line.
Ctrl-e	Move the cursor to end of line.
Ctrl-x-x	Switch the cursor between the line beginning and end.

Alt-f	Cursor moves forward by word.
Alt-b	Cursor moves back by word.
```
### Edit command line

```
Ctrl-u	Cut the text to the start of the line.
Ctrl-k	Cut the text to the end of the line.

Ctrl-w	Delete forward by word.
Alt-d	Delete backwards by word.

Ctrl-y	Yank the most recently cutted text at the cursor.

Alt-u	Uppercase the current (or following) word.
Alt-l	Lowercase the current (or following) word.
Alt-c	Capitalize the first char of the current (or following) word.
```

### Searching for, or feferring to specified command line in history

```
Ctrl-p	Fetch the previous executed command.
Ctrl-n	Fetch the next executed command.

Ctrl-r	Reverse-i-search mode. Reversely searching for the command matching the string which you input.
Ctrl-g	Exit Reverse-i-search mode.

Alt-.	Insert the last argument of the previous command. Rotating the history record by repeating. 

!n	Refer to the nth command in command line history.
!`string`:p	print the most recent command line containing `string`.
!#	The entire command line typed so far.
```

Notice: Most of the contents above is from the `info bash`. You also can find them on GNU website https://www.gnu.org/.

------
