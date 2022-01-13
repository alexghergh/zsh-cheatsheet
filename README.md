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

### 10. Input and output redirection

If job control is active and a command is sent to the background (either by
appending a `&` to the command or by using the background control sequence - by
default `Ctrl-Z` in many terminals), then the command's input and output are
those specified in the environment of the parent shell, possibly modified by
input/output specifications on the command line. For example, if a command's
input is the command line standard input (default file descriptor 0) and it is
sent into the background, then it's input becomes `/dev/null` (standard output
and error remains the same, unless other options are set to suppress background
jobs' output).

There are a number of redirections that can appear anywhere inside a simple
command or may precede or follow a complex command (see above for what those
mean). Expansion occurs before `word` or `digit` are used. If the result
substitution on `word` produces more than one filename, then redirection occurs
for each separate filename.

The redirections are as follows:
- `< word`: Open the file `word` for reading as standard input. It's an error to
  open a file in this fashion if it does not exist.
- `<> word`: Open the file `word` for reading and writing as standard input. If
  the file does not exist then it is created (does not seem to work as
  expected!).
- `> word`: Open the file `word` for writing as standard output. If the file
  does not exist then it is created. If the file exists, and the `CLOBBER`
  option is unset, then an error is raised (this prevents from accidental
  overwrites). Otherwise, the file is truncated to zero length.
- `>| word` or `>! word`: Same as `>`, except that the file is truncated to zero
  length regardless of the `CLOBBER` option.
- `>> word`: Open the file `word` in append mode as standard output. If the file
  does not exist, and the options `CLOBBER` and `APPEND_CREATE` are both unset,
  this causes an error. Otherwise, the file is created.
- `>>| word` or `>>! word`: Same as `>>`, except that the file is created if it
  does not exist, regardless of `CLOBBER` and `APPEND_CREATE`.
- `<<[-] word`: The shell input is read up to a line that is the same as `word`,
  or an end-of-file. No parameter expansion, command substitution or filename
  generation is performed on `word`. The resulting document, called a
  _here-document_ (or _heredoc_), becomes the standard input. The
  _here-document_ is subjected to parameter substitution (but no alias or
  filename expansion). However, if any character of `word` is quoted with double
  or single quotes or a `\`, no interpretation is placed on any of the
  characters in the document. Note that `word` itself does not undergo shell
  expansion. Backquotes and `$'...'` also behave differently (see the manual for
  more details). If `<<- word` is used, then all leading tabs from `word` and
  the document are stripped.
- `<<< word`: Perform shell expansion on word and pass the result to standard
  input. This is different from the above, where the above does not undergo
  shell expansion.
- `<& number` and `>& number`: The standard input/output is duplicated from file
  descriptor `number` (see man page dup2(2)).
- `<& -` and `>& -`: Close the standard input/output.
- `<& p` and `>& p`: The input/output from/to the coprocess is moved to the
  standard input/output. See more in the [pipeline section above](#simple-commands-and-pipelines).
- `>& word` and `&> word`: Redirects both standard output and standard error to
  `word`. The first form is ambiguous (since it can match some of the syntax
  above), so it should be avoided. Note that this does not have the same effect
  as `> word 2>&1` in the presence of _multios_ (see section below for more on
  this). **NOTE:** There is a difference between `& >` and `&>`. The first one
  sends the command before it in the background, and then writes the standard
  input to the file supplied after it, while the second one if explained above.
- `>&| word` and `>&! word` and `&>| word` and `&>! word`: Redirects both
  standard output and standard error in the manner of `>| word`.
- `>>& word` and `&>> word`: Redirects both standard output and standard error
  in the manner of `>> word`.
- `>>&| word` and `>>&! word` and `&>>| word` and `&>>! word`: Redirects both
  standard output and standard error in the manner of `>>| word`.

If any of the above forms are preceded by a digit, then that is used instead of
the standard input/output (file descriptors 0 and 1). The digit comes right
before the redirection (there shouldn't be any whitespace between them). This
becomes interesting in cases such as `6&> file`, which basically says that both
file descriptor 6 and standard error are redirected to the file, however
standard output (file descriptor 1), remains the same.

The order in which redirections are specified matters. The shell evaluates each
redirection in terms of the _(file descriptor, file)_ association _at the time
of the evaluation_. For example:

`... 1> file 2>&1`

associates standard output to the file specified, and then associates standard
error to the same output specified by file descriptor 1 (that is, the file). If
they were reverse, then first standard error would be associated with the file
descriptor 1 (standard output), and then file descriptor 1 would be associated
with the file.

There are various forms of process substitution as well; `<(list)` and
`=(list)` for input and `>(list)` for output. They are often used together with
redirection. For example, if the `word` in a redirection is of the form
`>(list)` then the output of the command is piped to the command represented by
`list`. More on this in [process substitution](https://zsh.sourceforge.io/Doc/Release/Expansion.html#Process-Substitution).

#### Named file descriptors

There is another way to use the redirection mechanism. This is by using named
file descriptors. The syntax is as follows:

`... {myfd}>& word`

This works by creating a named file descriptor (with the id _myfd_) which is
guaranteed to be at least file descriptor number 10. This file descriptor can be
written with the syntax `>&$myfd`. The file descriptor remains open in subshells
and forked external executables. The file descriptor can be closed by
`{myfd}>&-`.

The following shows a typical use of this mechanism:

```zsh
integer myfd
exec {myfd}>my/log/file.txt
print This is some debugging line >&$myfd
exec {myfd}>&-
```

#### Multios

There is an additional mechanism which allows arbitrary commands to write their
output to more than one redirection, similar to how the _tee_ executable works,
provided the `MULTIOS` option is set. For example:

`print Some interesting text > bar > foo`

This opens a process that copies the output of the first command (`print`) to
both files (again, provided the option `MULTIOS` is set). With `MULTIOS` not
set, each redirection replaces the previous redirection for that file
descriptor. However, all files redirected to are actually opened. This means the
above would have the effect of opening _bar_ (truncating it to zero length),
then closing it, and opening _foo_ (again truncating it to zero length), and
finally writing the text inside _foo_.

Note that `MULTIOS` works in the same way for piping:

`print Some interesting text > bar | cat`

would both print the statement on screen, as well as write it to the file `bar`.
With `MULTIOS` not set, the `cat` command would receive no input (and so would
display nothing).

Note that all the files to be written using the redirection above are opened at
the same time by the _multio_ process, and not as they are about to be written.

With the `MULTIOS` option set, the word after a redirection is subjected to
filename expansion. For example:

`print Some text > *.c`

will redirect the text to all the files matching the pattern.

Similarly, if a command wants to read from more than one source, and if the
`MULTIOS` option is set, then the shell opens a process which has the output
piped to the input of the command, and which writes through the pipe all the
outputs of the files specified in the redirections on the command line. For
example:

`process < foo < bar`

will have the _multio_ process open both files immediately, and have the inputs
(in order) piped through to _process_. Note that this behaviour differs from
`cat`, which will open each file separately, as it is about to be read.

`sort < f{oo,ubar}`

is equivalent to

`cat foo fubar | sort`.

Also note that a pipe is an implicit redirection, thus:

`cat bar | sort < foo`

is equivalent to

`cat bar foo | sort` (note the order of inputs).

As noted above, if `MULTIOS` is **not** set, then each redirection replaces the
one before it for the same file descriptor.

There may be problems when an external program writes to multiple files using
the _multio_ process. For example:

```zsh
cat file > bar > foo
cat bar foo
```

Since cat is an external program, and the subshell it runs in returns _before_
the _multio_ process is finished writing (it doesn't wait for it to finish
before returning), then the second line might not fully display the whole
contents of the files. The workaround is to run the command as a job in the
current shell:

`{ cat file } > bar > foo`

Here, the `{...}` job will wait for both files to be written.

#### Redirections with no command

The shell allows for two more forms of the redirections. If redirections are
specfied on the command line without and other command before or after them,
then the redirections directly affect the current shell. For example:

`< file`

will redirect the output of the file into the current shell (i.e. output the
contents of the file to standard output). This is similar to `more file` (in fact
the shell runs `more` behind the scene, see below).

Similar with input:

`> file`

will write to `file`, when reading from the standard input.

However, some options need to be set for this to work. If `NULLCMD` is not set
or the option `CSH_NULLCMD` is set, then this causes an error. This is the
standard _csh_ behaviour that _zsh_ tries to emulate when running in _csh_
emulation mode.

If the option `SH_NULLCMD` is set, then the builtin `:` is inserted at the
beginning of the redirections. This is the default when emulating _sh_ or _ksh_.
This causes a different behaviour than what was described above.

Otherwise, if the parameter `NULLCMD` is set, its value will be used as a
command with the given redirections (this is the _zsh_ way). `READNULLCMD` can
be overwritten as well in order to have the input redirection be prepended by
the command given by `READNULLCMD` instead of that given by `NULLCMD`. The
defaults for these two are `cat` for `NULLCMD` and `more` for `READNULLCMD`.

For more information see:
- [The Zsh Manual on Redirection][]

[The Zsh Manual on Redirection]: https://zsh.sourceforge.io/Doc/Release/Redirection.html#Redirection

### 11. Jobs

If the `MONITOR` option is set, the shell associates each pipeline with a job
(see [commands]() for what a pipeline is). The current jobs can be listed with
the `jobs` command. Each job is assigned an integer number. When a job is
started in the background with `&`, the shell prints a line to standard error
which gives the job integer number and the job pid (command):

`[1] 18361`

If a job is started with the `&|` or `&!`, then the job is immediately disowned.
This means it doesn't appear in the jobs table, and cannot be brought back to
the background, and it is implicitly owned by the parent process.

If a job is running in the foreground, it may be sent to the background by
means of `Ctrl-Z` (this is the default, could be different, check the output of
`stty -a`, in `susp`). This sends the `TSTP` signal to the current job. The
shell will then indicate the job is suspended and prompt the user with another
shell. The job can then be sent to the background by running `bg` (careful here,
if the job is suspended it **doesn't run at all**; if the job is in the
background, it **will run** in the background). Suspending is useful for things
like _vim_, which do not need to run in the background (unless a long job is
started inside them, of course), while also sending jobs to the background is
useful for commands that process something (like a long `cp`, `mv` or something
similar). The job can also be brought back to foreground with the `fg` command.
A `Ctrl-Z` is similar to an interrupt, in that pending output and unread input
are discarded immediately.

A job running in the background will suspend if it tries to read from terminal
(standard input, file descriptor 0).

Notice however the jobs mechanism works a bit different for shell functions. If
a shell function is suspended, then this will cause the shell to fork. This is
necessary to separate the function's environment from the parent shell, so that
the shell can return to the command line prompt. As a result, even if the
function is called back by `fg` the function will no longer be part of the
parent shell, and any variables set by the function will no longer be visible by
the parent shell. Thus the behaviour is different from the case where the
function was never suspended.

Background jobs are normally allowed to produce output, however passing the
option `stty tostop` will disable this, and will suspend backgrounded jobs just
as if they are trying to read input.

If a command is suspended and later resumed with `fg` or `wait`, zsh restores
tty modes that were in effect when the job was suspended. This does not apply
however when the job is resumed via `kill -CONT`, nor when it is continued
through `bg`.

There are a few ways to refer to jobs in the shell:
- `%number`: The job with the specified number.
- `%string`: The last job whose command line begins with `string`.
- `%?string`: The last job whose command line contains `string`.
- `%%`: The current job.
- `%+`: Same as `%%`.
- `%-`: The previous job.

The shell normally informs the user if a job gets stuck and cannot continue
(waiting for input or output if `stty tostop` is set). If the option `NOTIFY` is
not set, then the shell waits to inform the user right before printing the next
prompt.

When the `MONITOR` option is set, each background job that completes triggers
any trap set for `CHLD`.

If the user tries to exit the shell when there are background running jobs, then
the shell inform the user about it. If a second exit is issued immediately
after, then the shell terminates all the running jobs and exits the shell
without and additional warning. To avoid this, then the user can either `disown`
the running jobs or run the jobs with the `nohup` command.

For more information see:
- [The manual page on jobs][]
- The wait builtin manual (`run-help wait`)
- The disown builtin manual (`run-help disown`)
- The nohup command manual (`man nohup`)

[The manual page on jobs]: https://zsh.sourceforge.io/Doc/Release/Jobs-_0026-Signals.html#Jobs-_0026-Signals

### 12. Signals

The `INT` and `QUIT` signals for a command are ignore if the job is started in
the background (using `&`) and the `MONITOR` option is not set. The shell itself
always ignores the `QUIT` signal. Otherwise, the signals have the values
inherited by the shell from its parent (for more information see [the special
functions section](#special-functions)).

There are certain jobs that are ran asynchronously by the shell even if the job
is not running in the background. Such jobs are process substitution (TODO link)
or the _multios_ process (see [redirection](#multios)).

For more information see:
- [The signals section in the Zsh Manual][]

[The signals section in the Zsh Manual]: https://zsh.sourceforge.io/Doc/Release/Jobs-_0026-Signals.html#Signals

### 13. Arithmetic evaluation

The Zsh shell has arithmetic capabilities, using integers and floating point
numbers. The arithmetic evaluation is forced either via the builtin `let` or by
a substitution of the form `$((...))`. The shell is compiled to use integers of
8 bytes where possible, otherwise 4 bytes. This can be tested with a number that
can only be represented using 8 bytes (for example `print $((12345678901))`). If
the number is printed unchanged, the precision is at least 8 bytes. Floating
point always uses the _double_ type, with whatever precision is provided by the
library and the compiler (a double usually has 8 bytes as well, on most modern
computers).

The `let` builtin takes arithmetic expressions as arguments, where each one of
them is evaluated separately. Since most of the arithmetic operators requires
quoting, an alternative form is provided, which begins with `((` and ends with
`))`; all the characters between are treated as quoted, and everything in
between is considered as one argument of the `let` command. The return status is
0 if the result of the mathematical expression is non-zero, 1 if the result is
zero, and 2 if an error occurred.

For example:

`let "i = 1 + 2"`

is equivalent to

`(( i = 1 + 2 ))`

both assigning value 3 to the variable _i_ and returning zero to the shell.

Integers can be specified in bases different than base 10. A leading `0x` or
`0X` denotes hexadecimal, while a leading `0b` or `0B` denotes binary. The base
can also be specified via `base#number`, where `base` is an integer between 2
and 36. For backwards compatibility the form `[base]n` is also accepted.

The integer or number can additionally contain underscores after the first
digit, to aid in visual guidance (for example, `1_000_000` or `0xff_ff_ff_ff`).

It's also possible to specify the output base of a number, with the construct
`[#base]`, for example `[#16]`:

`print $(( [#8] x = 32 ))`

outputs `8#40`.

The same can be achieved through `typeset -i 16 x`, where the option `-i base`
specified the implicit output base of the number.

The output can also be separated by underscores by adding an underscore to the
base, followed by a number specifying how many digits should be in a group. For
example:

`print $(( [#16_4] x = 65536 ** 2))`

outputs `16#1_00C8_2710`.

If the option `C_BASES` is set, hexadecimal number are output in C format, i.e.
`0xFF` instead of `16#FF`. If the option `OCTAL_ZEROES` is set, octal numbers
will also be displayed like `077` instead of `8#77`. These options have no
effect on bases other than 16 and 8, and has no effect on the input, as all the
above variants are understood anyway.

The output base can also be suppressed by doubling the `#`, i.e. `[##16]`.

The operators supported in the native mode of operation are (listed in
decreasing order of precedence):
- `+ - ! ~ ++ --`: unary plus/minus, logical NOT, complement, pre- and post-
  decrement and increment
- `<< >>`: bitwise shifts
- `&`: bitwise AND
- `^`: bitwise XOR
- `|`: bitwise OR
- `**`: exponentiation
- `* / %`: multiplication, division, modulus (remainder)
- `+ -`: addition/subtraction
- `< > <= >=`: comparison
- `== !=`: equality or inequality
- `&&`: logical AND
- `|| ^^`: logical OR, XOR
- `? :`: ternary operator
- `= += -= *= /= %= &= ^= |= <<= >>= &&= ||= ^^= **=`: assignment
- `,`: comma separator

With the option `C_PRECEDENCES` set, the precedences of some of the operators
changes, as follows:
- `+ - ! ~ ++ --`: unary plus/minus, logical NOT, complement, pre- and post-
  decrement and increment
- `**`: exponentiation
- `* / %`: multiplication, division, modulus (remainder)
- `+ -`: addition/subtraction
- `<< >>`: bitwise shifts
- `< > <= >=`: comparison
- `== !=`: equality or inequality
- `&`: bitwise AND
- `^`: bitwise XOR
- `|`: bitwise OR
- `&&`: logical AND
- `^^`: logical XOR
- `||`: logical OR
- `? :`: ternary operator
- `= += -= *= /= %= &= ^= |= <<= >>= &&= ||= ^^= **=`: assignment
- `,`: comma separator

Mathematical functions can be used by loading the module `zsh/mathfunc` using
the `zmodload` builtin. The module provides standard floating point mathematical
functions.

#### Getting the numerical value of characters

The shell can output the numerical value of a certain character sequence such as
`a`, `^A`, or `\M-\C-x` using the form `##x`, where `x` is one of such sequences
mentioned before.

Character values are according to the current locale. For handling multibyte
characters, the value `MULTIBYTE` must be set.

For more information see:
- [The arithmetic evaluation section in the Zsh Manual][]

[The arithmetic evaluation section in the Zsh Manual]: https://zsh.sourceforge.io/Doc/Release/Arithmetic-Evaluation.html#Arithmetic-Evaluation
