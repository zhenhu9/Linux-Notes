
# How to use tmux

### NAME

     tmux — terminal multiplexer

### SYNOPSIS

     tmux [-2CluvV] [-c shell-command] [-f file] [-L socket-name]
          [-S socket-path]

     tmux [command [flags]]

### DESCRIPTION

tmux is a terminal multiplexer: it enables a number of terminals to be cre‐
ated, accessed, and controlled from a single screen.  tmux may be detached
from a screen and continue running in the background, then later reattached.

When tmux is started it creates a new session with a single window and dis‐
plays it on screen.  A status line at the bottom of the screen shows informa‐
tion on the current session and is used to enter interactive commands.

A session is a single collection of pseudo terminals under the management of
tmux.  Each session has one or more windows linked to it.  A window occupies
the entire screen and may be split into rectangular panes, each of which is a
separate pseudo terminal (the pty(4) manual page documents the technical
details of pseudo terminals).  Any number of tmux instances may connect to
the same session, and any number of windows may be present in the same ses‐
sion.  Once all sessions are killed, tmux exits.

Each session is persistent and will survive accidental disconnection (such as
ssh(1) connection timeout) or intentional detaching (with the ‘C-b d’ key
strokes).  tmux may be reattached using:

      $ tmux attach

In tmux, a session is displayed on screen by a client and all sessions are
managed by a single server.  The server and each client are separate pro‐
cesses which communicate through a socket in /tmp.


**Firgure  1**

```
   $ tmux

   -----------------------------------------------------------------------
   | ------------------------------------------------------------------- |
   | | --------------------------------------------------------------- | |
   | | |                      Pane0   |                       Pane1  | | |
   | | |                              |                              | | |
   | | |                              |                              | | |
   | | |                              |                              | | |
   | | |                              |                              | | |
   | | |                              |                              | | |
   | | |                              |                              | | |
   | | |                              |                              | | |
   | | |                              |                              | | |
   | | |                              |                              | | |
   | | |                              |                              | | |
   | | |                              |                              | | |
   | | | :`bash*` or `command`        |                              | | |
   | | --------------------------------------------------------------- | |
   | | 1                         <==  marked from 1.     Windows       | |
   | ------------------------------------------------------------------- |
   | [0]                         <==  marked from 0.     Sessions        |
   -----------------------------------------------------------------------
```

**Firgure 2**

```
tmux basic stucture:

     server	Each user namespace has only one server.
     client	Each teminal has many clients.
     session	Each client has many sessions.
     window	Each session has many windows, but only one occupys the screen.
     pane	Each window can split into several panes, working places.

tmux basic commands:

     --help

     list-commands
     list-clients    -sessions    -windows    -panes
     rename-session               -window

     attach-session
               -t target-session

     kill-server      -session     -window    -pane

     join-pane			used to reverse break-pane.
               -h/-v		join the window horizontally or vertically.
               [-s src-pane]
               [-t dst-pane]

     **Alternate**: pressing `C-b` + `:` to enter Tmux command line, then just
               input subcommands.
```

### KEY BINDINGS

tmux may be controlled from an attached client by using a key
combination of a prefix key, ‘C-b’ (Ctrl-b) by default, followed by a
command key.

The default command key bindings are:

C-b         Send the prefix key (C-b) through to the application.

**Clients**:

     r           Force redraw of the attached client.
     d           Detach the current client.
     D           Choose a client to detach.
     s           Select a new session for the attached client interactively.
     C-z         Suspend the tmux client.

**Sessions**:

     $           Rename the current session.
     (           Switch the attached client to the previous session.
     )           Switch the attached client to the next session.
     L           Switch the attached client back to the last session.

**Windows**:

     c           Create a new window.
     &           Kill the current window.
     w           Choose the current window interactively.
     ,           Rename the current window.

     f           Prompt to search for text in open windows.
     .           Prompt for an index to move the current window.

     0 to 9      Select windows 0 to 9.
     p           Change to the previous window.
     n           Change to the next window.

     l           Move to the previously selected window.
     '           Prompt for a window index to select.
     i           Display some information about the current window.

**Panes**:

     q           Briefly display pane indexes.

     "           Split the current pane into two, top and bottom.
     %           Split the current pane into two, left and right.

     Left, Right
                 Change to the pane above, below, to the left, or to the
                 right of the current pane.

     z           Toggle zoom state of the current pane.

     !           Break the current pane out of the window.
     x           Kill the current pane.

     C-o         Rotate the panes in the current window forwards.
     {           Swap the current pane with the previous pane.
     }           Swap the current pane with the next pane.

     o           Select the next pane in the current window.
     ;           Move to the previously active pane.

**Copy mode**:

     [           Enter copy mode to copy text or view the history.
     ]           Paste the most recently copied buffer of text.

     Page Up     Enter copy mode and scroll one page up.
     Up, Down

**useful key bindings**:

     ?           List all key bindings.

     t           Show the time.

     :           Enter the tmux command prompt.

     ~           Show previous messages from tmux, if any.

**Others**:

     #           List all paste buffers.
     -           Delete the most recently copied buffer of text.
     =           Choose which buffer to paste interactively from a list.

     m           Mark the current pane (see select-pane -m).
     M           Clear the marked pane.
     M-1 to M-5  Arrange panes in one of the five preset layouts: even-hori‐
                 zontal, even-vertical, main-horizontal, main-vertical, or
                 tiled.
     Space       Arrange the current window in the next preset layout.
     M-n         Move to the next window with a bell or activity marker.
     M-o         Rotate the panes in the current window backwards.
     M-p         Move to the previous window with a bell or activity marker.
     C-Up, C-Down
     C-Left, C-Right
                 Resize the current pane in steps of one cell.
     M-Up, M-Down
     M-Left, M-Right
                 Resize the current pane in steps of five cells.

     Key bindings may be changed with the bind-key and unbind-key commands.

**The options are as follows**:

```
-2            Force tmux to assume the terminal supports 256 colours.

-C            Start in control mode (see the CONTROL MODE section).  Given
              twice (-CC) disables echo.

-c shell-command
              Execute shell-command using the default shell.  If necessary,
              the tmux server will be started to retrieve the default-shell
              option.  This option is for compatibility with sh(1) when tmux
              is used as a login shell.

-f file       Specify an alternative configuration file.  By default, tmux
              loads the system configuration file from /etc/tmux.conf, if
              present, then looks for a user configuration file at
              ~/.tmux.conf.

              The configuration file is a set of tmux commands which are exe‐
              cuted in sequence when the server is first started.  tmux loads
              configuration files once when the server process has started.
              The source-file command may be used to load a file later.

              tmux shows any error messages from commands in configuration
              files in the first session created, and continues to process
              the rest of the configuration file.

-L socket-name
              tmux stores the server socket in a directory under TMUX_TMPDIR
              or /tmp if it is unset.  The default socket is named default.
              This option allows a different socket name to be specified,
              allowing several independent tmux servers to be run.  Unlike -S
              a full path is not necessary: the sockets are all created in
              the same directory.

              If the socket is accidentally removed, the SIGUSR1 signal may
              be sent to the tmux server process to recreate it (note that
              this will fail if any parent directories are missing).

-l            Behave as a login shell.  This flag currently has no effect and
              is for compatibility with other shells when using tmux as a
              login shell.

-S socket-path
              Specify a full alternative path to the server socket.  If -S is
              specified, the default socket directory is not used and any -L
              flag is ignored.

-u            When starting, tmux looks for the LC_ALL, LC_CTYPE and LANG
              environment variables: if the first found contains ‘UTF-8’,
              then the terminal is assumed to support UTF-8.  This is not
              always correct: the -u flag explicitly informs tmux that UTF-8
              is supported.

              Note that tmux itself always accepts UTF-8; this controls
              whether it will send UTF-8 characters to the terminal it is
              running (if not, they are replaced by ‘_’).

-v            Request verbose logging.  Log messages will be saved into
              tmux-client-PID.log and tmux-server-PID.log files in the cur‐
              rent directory, where PID is the PID of the server or client
              process.  If -v is specified twice, an additional
              tmux-out-PID.log file is generated with a copy of everything
              tmux writes to the terminal.

              The SIGUSR2 signal may be sent to the tmux server process to
              toggle logging between on (as if -v was given) and off.

-V            Report the tmux version.

command [flags]
              This specifies one of a set of commands used to control tmux,
              as described in the following sections.  If no commands are
              specified, the new-session command is assumed.
```

### FILES

```
     ~/.tmux.conf       Default tmux configuration file.
     /etc/tmux.conf     System-wide configuration file.
```

### EXAMPLES

To create a new tmux session running vi(1):

      $ tmux new-session vi

A session may be detached using ‘C-b d’ (or by an external event such as
ssh(1) disconnection) and reattached with:

      $ tmux attach-session

Typing ‘C-b ?’ lists the current key bindings in the current window; up and
down may be used to navigate the list or ‘q’ to exit from it.

Commands to be run when the tmux server is started may be placed in the
~/.tmux.conf configuration file.  Common examples include:

Changing the default prefix key:

      set-option -g prefix C-a
      unbind-key C-b
      bind-key C-a send-prefix

Turning the status line off, or changing its colour:

      set-option -g status off
      set-option -g status-style bg=blue

Setting other options, such as the default command, or locking after 30 min‐
utes of inactivity:

      set-option -g default-command "exec /bin/ksh"
      set-option -g lock-after-time 1800

Creating new key bindings:

      bind-key b set-option status
      bind-key / command-prompt "split-window 'exec man %%'"
      bind-key S command-prompt "new-window -n %1 'ssh %1'"

### SEE ALSO
     pty(4)
