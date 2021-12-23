# Zsh tips, tricks and features

Here are some tips, tricks and features of the Z Shell. Most of the tips are
taken from the internet, either from [the official zsh website][zsh] or from other
places online. If you find anything that could be considered personal property
and would like it taken down, send an email at <alexghergh@gmail.com>.

[zsh]: https://www.zsh.org/

--------------------------------------------------------------------------------

### 1. Passing options to the `zsh` command

Options can be passed to the zsh command in different forms, either as short
options or as long GNU-style options. The special options `--version` and
`--help` are handled separately. For the other options, they can be specified by
name using the `-o` flag, followed by the option name, to turn the option _on_.
To turn it _off_, the option should be preceded by `+o` instead:

`zsh -o shwordsplit` (turn `shwordsplit` on)

or

`zsh +o shwordsplit` (turn `shwordsplit` off)

The long option style is `--option-name`. To turn it off, the same as above can
be applied, i.e. substitute the first `-` for a `+` (`+-option-name`), which is
equivalent to `--no-option-name`.

Passing `--help` to the command lists a set of options that can be turned on or
off.

For more information see [Invocation][].

[Invocation]: https://zsh.sourceforge.io/Doc/Release/Invocation.html#Invocation

### 2. Interactive and login shells

Depending on how a particular shell instance is started, the shell can be
interactive or non-interactive, login or non-login. When you manually type
commands in front of a shell, it is interactive. When you run a script in a
shell, it happens to be non-interactive (though careful, the first shell that
launches the command - say `zsh script` - is interactive, and it hangs around
while the second shell, which runs the script and is non-interactive, finishes).

When a system boots up, and after you type the password, the program
`/bin/login` reads the file `/etc/passwd` and automatically launches a **login**
shell. Other shells launched from this shell are all non-login shells. To tell
the difference, the `/bin/login` program usually sticks a `-` in front of the
login shell. You can get the name of the shell currently active by `echo $0`. If
it contains the `-` at the beginning, it is a login shell.

**Note**: There may possibly be non-interactive login shells, when for example
windowing systems might lunch some startup scripts, and then launch the real
interactive login shell.

**Note 2**: Programs like `tmux` launch by default login shells. This happens
because, for example, you might have a tmux session running, and then logout
(which means `.zlogout` is ran), but the tmux server is still running in the
background. If the shells would then not be login shells, they might not
properly run after logging in again. This means shells running inside `tmux` are
interactive login shells.

To test if a shell is interactive/login:

```zsh
# To check if a shell is a login shell
if [[ -o login ]]; then
    print "Login"
else
    print "Non-login"
fi

# To check if a shell is an interactive shell
if [[ -o interactive ]]; then
    print "Interactive"
else
    print "Non-interactive"
fi
```

For more information see:
- [Interactive and Login Shells in the Zsh Guide][]
- [Tmux creating login shells on StackExchange][]

[Interactive and Login Shells in the Zsh Guide]: https://zsh.sourceforge.io/Guide/zshguide02.html
[Tmux creating login shells on StackExchange]: https://superuser.com/questions/968942/why-does-tmux-create-new-windows-as-login-shells-by-default

### 3. Startup/Shutdown files

The order the files are read by `zsh` is:

- `/etc/zshenv`: This is the first file read, and this cannot be changed.
Subsequent files are read according to the variables `RCS` and `GLOBAL_RCS` (if
`RCS` is unset in any of the files, no other files will be read after it; if
`GLOBAL_RCS` is unset in any of the files, no other global files - `/etc/<file>`
\- will be read after it).
- `$ZDOTDIR/.zshenv`: If `$ZDOTDIR` is unset, `$HOME` is used instead.
- `/etc/zprofile`, then `$ZDOTDIR/.zprofile` (if the shell is a login shell).
- `/etc/zshrc`, then `$ZDOTDIR/.zshrc` (if the shell is an interactive shell).
- `/etc/zlogin`, then `$ZDOTDIR/.zlogin` (if the shell is a login shell).
- `$ZDOTDIR/.zlogout`, then `/etc/zlogout` (when a login shell exits).

However, if the shell terminates by `exec`'ing another process, the logout files
are not used.

**Note**: Some of the files may be located in `/etc/zsh/..` rater than just
`/etc/..`.

**Note 2**: `PATH` shouldn't be modified in any `zshenv` file in Arch Linux, as
the `/etc/zprofile` file builds it from scratch.

Generally, users should only touch the `$ZDOTDIR/<file>` files, while the
`/etc/<file>` files should only be managed by administrators.

More on what the files should contain:
- `zshenv`: Sourced on all invocations of the shell, unless the `-f` option is
  set. Should contain commands to set the search path and other important
  environment variables. _It should not contain commands that produce output_.
- `zshrc`: Sourced in interactive shells. Should contain commands to set up
  aliases, functions, options, keybindings, etc.
- `zlogin`: Sourced in login shells. Should contain commands that should only be
  executed in login shells (TODO example?).
- `zprofile`: Similar to `zlogin`, except is is sourced before `zshrc`. This
  file is meant as an alternative to `.zlogin` for ksh fans. The two are not
  intended to be used together, though this can be done. `zlogin` should not
  define aliases, commands, options, environment variables etc. It should rather
  be used to set the terminal type and run a series of commands (`fortune`,
  `msgs` etc.). `zprofile` should be used for things that are inherited by
  non-`zsh` processes (like `PATH` of program-specific environment variables).

For more information see:
- [The Startup/Shutdown Files Manual][]
- [The old _"Startup/Shutdown Files"_ Manual][]
- [The Startup/Shutdown Files Zsh Guide][]
- [Arch Linux's Guide on Startup/Shutdown Files][]
- [A nice graph illustrating Bash's and Zsh's startup files][]
- [A great answer on StachExchange on what each file should contain][]

[The Startup/Shutdown Files Manual]: https://zsh.sourceforge.io/Doc/Release/Files.html#Files
[The old _"Startup/Shutdown Files"_ Manual]: https://zsh.sourceforge.io/Intro/intro_3.html
[The Startup/Shutdown Files Zsh Guide]: https://zsh.sourceforge.io/Guide/zshguide02.html
[Arch Linux's Guide on Startup/Shutdown Files]: https://wiki.archlinux.org/title/Zsh#Startup/Shutdown_files
[A nice graph illustrating Bash's and Zsh's startup files]: https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html#implementation
[A great answer on StachExchange on what each file should contain]: https://unix.stackexchange.com/questions/71253/what-should-shouldnt-go-in-zshenv-zshrc-zlogin-zprofile-zlogout

### 4. Commands and pipelines

#### Simple commands and pipelines

A `simple command` is a word (the actual command) followed by additional
parameters, separated by whitespace. If parameter assignments preced the
command, these modify the environment in which the command in running. It's
important to note that commands (careful, commands that are **not** builtins)
run in a subshell, with a different environment from the parent shell. The
value of a simple command is its exit status, or 128 plus the signal number if
it was terminated by a signal.

A `pipeline` is either a single command, or a sequence of simple commands where
each command is separated from the next one by either `|` or `|&`. In the case
of `|`, the standard output of the first command is connected to the standard
input of the second command, while `|&` is shorthand for `2>&1 |`, which
redirects both standard output and standard error of the first command to the
input of the second command. The value of a pipeline is the value of the last
command, unless the pipeline is preceded by `!`, in which case the value of the
pipeline is the logical inverse of the value of the last command. For example:

`echo foo; echo $?`

returns 0, while

`! echo foo; echo $?`

returns 1.

This is useful in if statements in scripts.

If the pipeline is preceded by the `coproc` word, it is executed as a
coprocess, which means a two-way pipe is established between it and the parent
shell. The shell can read and write through the coprocess pipe by the means of
`>&p` and `<&p` or with `print -p` and `read -p`. A pipeline cannot be
preceded by both `coproc` and `!`.

A `sublist` is either a single pipeline, or a sequence of one or more
pipelines separated by `||` or `&&`. If two pipelines are separated by a `&&`,
the second pipeline executes only if the first pipeline was successfull
(returns zero). If two pipelines are separated by `||`, then the second pipeline
executes only if the first one fails (returns non-zero). Both operatos have
equal precedence and are left associative. The value of the sublist is the
value of the last pipeline executed.

A `list` is a sequence of zero or more sublists, in which each sublist is
terminated by `;`, `&`, `&|`, `&!`, or a newline. When the sublist is terminated
by `;`, the shell waits for it to finish before executing the next sublist. If a
sublist is terminated by a `&`, `&|`, `&!`, the shell executes the last pipeline
in it in the background, and does not wait for it to finish (note the difference
from other shells where the whole sublist is executed in the background). A
backgrounded pipeline always returns zero.

#### Complex commands

A complex command in zsh usually refers to the shell language used for scripting
(i.e. the `if` statements, the `while` and `for` loops etc.).

Zsh has quite a comprehensive list of complex commands (careful though, every
`<list>` ends in the terminators mentioned above, i.e. `;`, `&`, `&|`, `&!`, or
a newline):
- `if <list> then <list> [elif <list> then <list>] ... [else <list>] fi`: The
  classic if statement seen in shells.
- `for <name> ... [in <word> ...] <term> do <list> done`: The classic for
  statement found in the shells language. `<term>` refers to either a `;` or one
  or multiple newlines. `<name>` expands to each `<word>` separately, and the
  loop is executed once for each such expansion. There can be multiple `<name>`s
  specified after the for loop, in which case each `<name>` is assigned a
  variable from the `<words>` list in order. If there are more `<name>`s than
  `<words>` in the list, the additional `<name>`s are assigned the empty string.
- `for (( [expr1]; [expr2]; [expr3] )) do <list> done`: A standard C-like for
  loop.
- `while <list> do <list> done`: The classic while statement found in the shells
  language. Runs the body of the loop (the second `<list>`) while the first
  `<list>` returns a zero value.
- `until <list> do <list> done`: Runs the body of the loop (the second `<list>`)
  until the first `<list>` returns a zero value.
- `repeat <word> do <list> done`: `<word>` is expanded and treated as an
  arithmetic expression, which must evaluate to a number. `<list>` is then
  executed that number of times.
- `case <word> in [[(]<pattern>[ | <pattern>] ... ) <list> (<;;>|<;&>|<;|>)] ...
  esac`: A C-like switch statement. `<word>` is matched against `<pattern>` and,
  if it matched, then `<list>` is executed. The pattern matching is the same as
  that of filename generation (TODO link to tip number). If the `<list>` that is
  executed is terminated with `;&` instead of the usual `;;`, the following
  `<list>` is also executed (regardless if `<pattern>` matches `<word>` or not).
  The rules for terminators are then repeated until the final `esac` is reached.
  If the `<list>` executed terminates with `;|`, then the shell continues to
  scan `<patterns>` until either a new pattern-matching occurred, or the final
  `esac` was reached.
- `select <name> [in <word> .. <term>] do <list> done`: Offers a selection
  dialog on the command line, with each `<word>` being one of the options.
  `<term>` again refers to either `;` or one or multiple newlines. On correct
  selection of one of the options, the `<name>` parameter is set to the
  corresponding option. The `$REPLY` variable is set to the contents of the line
  read from the input.
- `( <list> )`: Executes `<list>` in a subshell. Traps set by the `trap` builtin
  are reset to their default values while executing `<list>`. Builtins execute
  in a subshell too.
- `{ <list> }`: Executes `<list>`.
- `{ <try-list> } always { <always-list> }`: A try-finally block. First executes
  `<try-list>`, and then executes `<always-list>`, regardless of errors,
  `continue` or `break` found. More details about this construct can be found in
  the manual.
- `function <word> ... [()] [<term>] { <list> }` or `<word> ... () [<term>] {
  <list> }` or `<word> ... () [<term>] <command>`: Defines a function which is
  referenced by any of the `<word>`s. `<term>` again refers to either `;` or one
  or multiple newlines.
- `term [<pipeline>]`: `<pipeline>` is executed, then timing statistics are
  shown in the command line, according to the `TIMEFMT` parameter. If
  `<pipeline>` is omitted, print statistics about the shell and its children.
- `[[ exp ]]`: Evaluate the conditional expression `exp` and return zero if it's
  true. This is usually helpful in conjunction with the `if`, `while`, `for`
  statements mentioned above.

There are some alternate forms for the complex commands described above, however
these are not standard and should be strongly avoided. More information in the
manual.

For more information see:
- [The Shell Grammar in the Zsh Manual][]

[The Shell Grammar in the Zsh Manual]: https://zsh.sourceforge.io/Doc/Release/Shell-Grammar.html#Shell-Grammar

### 5. Precommand modifiers

Simple commands may be preceded by _precommand modifiers_, which alter how the
command is interpreted. These are shell builtins, except for `nocorrect` which
is a reserved word.

- `-`: The command is executed with a `-` prepended to its `argv[0]` string
  (e.g. `- echo hello`).
- `builtin`: The command word is taken to be a builtin command rather than a
  shell function or an external command.
- `command` [`-pvV`]: The command word is taken to be an external command,
  rather than a shell function or a builtin. The `-p` flag causes a default path
  to be searched, instead of that in `$PATH`. The `-v` flag causes a command to
  be displayed as how it would be run (synonym to `whence`). The `-V` switch
  displays a more verbose output (synonym to `whence -v`).
- `exec`: Run the command in place of the current shell, instead of as a
  subprocess. The shell does not fork, but is completely replaced. It does not
  invoke a `TRAPEXIT`, nor does it invoke `zlogout` files.
- `nocorrect`: No spelling correction is done on any words.
- `noglob`: No filename generation/globbing is performed.

For more information see:
- [Precommand Modifiers in the Zsh Manual][]
- The man pages of the different modifiers (`run-help <modifier>`)

[Precommand Modifiers in the Zsh Manual]: https://zsh.sourceforge.io/Doc/Release/Shell-Grammar.html#Precommand-Modifiers

### 6. Shell functions

Shell functions are defined with the `function` builtin or the `funcname ()`
syntax. Alias names are resolved when the function is first read. This means
aliases defined after a function has been read have no effect inside the
function.

Functions execute in the same process as the caller shell, unlike the external
commands, which execute in subprocesses. This means changes to environment
inside a function have effect on the shell executing it.

All the current functions can be listed with the `functions` builtin. A function
can be deleted by `unfunction`.

Functions can be marked as undefined using the `autoload` builtin (or `typeset
-fu` or `functions -u`). Such functions have no body, but the shell searches for
its definition the first time the function is executed in the `$fpath`
variable. Thus to autoload a function:

```zsh
fpath=(~/<dir/to/my/funcs> $fpath)
autoload func1 func2
```

The alias expansion during reading can be suppressed by passing the `-U` flag to
the `autoload` builtin.

For each _item_ in `$fpath`, the shell looks at the following files to load the
definition of a function:
- `_item_.zwc`
- `_item_/_function_.zwc`
- `_item_/_function_`

For what these files should contain and how the shell interprets the code inside
them, see [The Functions section in the Zsh Manual][].

#### Special functions

There are certain functions that get executed when, for example, some commands
are ran. These functions are called hook functions. For example:
- `chpwd`: Executed whenever the current directory is changed.
- `periodic`: If `$PERIOD` is set, this function is executed every `$PERIOD`
  seconds, just before a prompt.
- `precmd`: Executed before each prompt.
- `preexec`: Executed just after a command has been read and is about to be
  executed.
- `zshaddhistory`: Executed when a command line has been read interactively,
  but before it is executed.
- `zshexit`: When the main shell is about to exit normally.

Other types of functions are trap functions. These are executed when the shell
catches a specific signal, as defined by the `kill` command.

For example:

```zsh
TRAPINT() {
    print "caught SIGINT"
    return $(( 128 + $1 ))
}
```

For more information see:
- [The Functions section in the Zsh Manual][]
- The man pages of the Zsh functions (`run-help functions` and `run-help function`)

[The Functions section in the Zsh Manual]: https://zsh.sourceforge.io/Doc/Release/Functions.html#Functions

### 7. Comments

In non-interactive shells, or in interactive shells which set the
`INTERACTIVE_COMMENTS` option, a word beginning with the third character of the
`histchars` variable (usually `#`) marks a comment, which causes all the
following characters until the next newline to be ignored by the shell.

### 8. Aliasing

Usually, everything can be aliased on the command line. The command is checked
for aliases and the text is replaced if an alias was found. For a parameter of
the command to be replaced, the alias has to be global (defined with `alias
-g`). For precisely what is subject to alias expansion, see the manual page on
Aliasing.

Alias expansion is done on the shell input before any other expansion, except
history expansion. If an alias is defined for the word `foo`, alias expansion
can be avoided by `\foo`, although nothing stops anything from creating an alias
for `\foo` as well.

With `POSIX_ALIASES` set, only plain unquoted strings are eligible for aliasing.

For more information see:
- [Aliasing in the Zsh Manual][]
- The `alias` man page (`run-help alias`)

[Aliasing in the Zsh Manual]: https://zsh.sourceforge.io/Doc/Release/Shell-Grammar.html#Aliasing

### 9. Quoting

Quoting referes to having a character or a sequence of characters stand for
themselves. A character can be _quoted_ (i.e. stand for itself), by preceding it
with a `\`.

A string enclosed between `$'` and `'` is processed in the same way as string
arguments to the `print` builtin, and the resulting string is considered to be
entirely quoted (i.e. no expansion happens aside from the expansion that would
happen using the `print` builtin). All characters enclosed between a pair of `'`
are considered to be quoted (no expansion happens at all). However, no `'` can
appear inside this string (even escaped with `\`).

If the option `RC_QUOTES` is set, then a pair of single quotes inside another
pair of single quotes is turned into a single quote. Without the option set, no
single quotes can appear inside a pair of quotes.

For example:

`echo ''''`

with `RC_QUOTES` set prints a single quote, otherwise it prints nothing (a
newline).

Inside a pair of double quotes (`"`), alias expansion does not happen, however
environment variables expansion happens as usual. Filename generation and
globbing also does not happen inside double quotes.

For more information see:
- [The Zsh Manual on Quoting][]

[The Zsh Manual on Quoting]: https://zsh.sourceforge.io/Doc/Release/Shell-Grammar.html#Quoting
