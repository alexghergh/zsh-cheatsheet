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
parameters, separated by whitespace. If parameter assignments precede the
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

Quoting refers to having a character or a sequence of characters stand for
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

### 14. Conditional expressions

A conditional expression is what comes inside `[[`, usually inside an `if`,
`while` or other complex commands (see the section above on complex commands for
more on this). These are helpful to test properties of different files or
strings.

- `-a <file>`: True if `<file>` exists.
- `-b <file>`: True if `<file>` exists and is a special block file.
- `-c <file>`: True if `<file>` exists and is a special character file.
- `-d <file>`: True if `<file>` exists and is a directory.
- `-e <file>`: True if `<file>` exists.
- `-f <file>`: True if `<file>` exists and is a regular file.
- `-g <file>`: True if `<file>` exists and has its `setgid` bit set.
- `-h <file>`: True if `<file>` exists and is a symbolic link.
- `-k <file>`: True if `<file>` exists and has its `sticky` bit set.
- `-n <string>`: True if length of `<string>` is non-zero.
- `-o <option>`: True if `<option>` is on.
- `-p <file>`: True if `<file>` exists and is a special FIFO file (named pipe).
- `-r <file>`: True if `<file>` exists and is readable by the current process.
- `-s <file>`: True if `<file>` exists and has size greater than zero.
- `-t <fd>`: True if file descriptor `<fd>` is open and is associated with a
  terminal device.
- `-u <file>`: True if `<file>` exists and has its `setuid` bit set.
- `-v <varname>`: True if shell variable `<varname>` is set.
- `-w <file>`: True if `<file>` exists and is writeable by the current process.
- `-x <file>`: True if `<file>` exists and is executable by the current process.
  If the `<file>` is a directory, then the process had permission to search
  inside the directory.
- `-z <string>`: True if length of `<string>` is zero.
- `-L <file>`: True if `<file>` exists and is a symbolic link.
- `-O <file>`: True if `<file>` exists and is owned by the effective user ID of
  the process.
- `-G <file>`: True if `<file>` exists and its group matches the effective group
  ID of the process.
- `-S <file>`: True if `<file>` exists and is a socket.
- `-N <file>`: True if `<file>` exists and its access time is not newer than its
  modification time.
- `<file1> -nt <file2>`: True if `<file1>` exists and is newer than `<file2>`.
- `<file1> -ot <file2>`: True if `<file1>` exists and is older than `<file2>`.
- `<file1> -ef <file2>`: True if `<file1>` and `<file2>` exist and refer to the
  same effective file.
- `<string> = <pattern>` or `<string> == <pattern>`: True if `<string>` matches
  `<pattern>`.
- `<string> != <pattern>`: True if `<string>` does not match `<pattern>`.
- `<string> =~ <regexp>`: True if `<string>` matches the regular expression
  `<regexp>`. This is influenced by a few options:
    - If the option `RE_MATCH_PCRE`, then `<regexp>` is tested as a PCRE
      expression using the `zsh/pcre` module, otherwise it is tested as a POSIX
      standard extended regular expression using the `zsh/regex` module.
    - If the option `BASH_REMATCH` is not set the scalar parameter `MATCH`is set
      to the substring that matched the pattern, and the integer parameters
      `MBEGIN` and `MEND` to the index of the start and end respectively of the
      match in `<string>`, such that if `<string>` is contained in variable
      `var`, then `${var[$MBEGIN,$MEND]}` is exactly equivalent to `$MATCH`. The
      additional Zsh array parameters `$match`, `$mbegin` and `$mend` are also
      set during this process. `$match` is set to all the substrings that
      matched paranthesised subexpressions, while `$mbegin` and `$mend` are set
      to the start and end positions, respectively, of the substring matches.
      For an example:

        ```zsh
        if [[ "this is a short strit" =~ "s(...)t s(...t)" ]]; then
            print $MATCH
            print $MBEGIN
            print $MEND
            print $match
            print $mbegin
            print $mend
        fi
        ```

      will result in:

        ```zsh
        short strit
        11
        21
        hor trit
        12 18
        14 21
        ```
- `<string1> < <string2>`: True if `<string1>` comes before `<string2>` based on
  ASCII values of characters.
- `<string1> > <string2>`: True if `<string1>` comes after `<string2>` based on
  ASCII values of characters.
- `<exp1> -eq <exp2>`: True if `<exp1>` is numerically equal to `<exp2>`. For
  purely mathematical or numerical comparisons, the arithmetic evaluation
  (`((..))`) should be used instead (see [arithmetic
  evaluation](#arithmetic-evaluation) above).
- `<exp1> -ne <exp2>`: True if `<exp1>` is numerically not equal to `<exp2>`.
- `<exp1> -lt <exp2>`: True if `<exp1>` is numerically less than `<exp2>`.
- `<exp1> -gt <exp2>`: True if `<exp1>` is numerically greater than `<exp2>`.
- `<exp1> -le <exp2>`: True if `<exp1>` is numerically less than or equal to `<exp2>`.
- `<exp1> -ge <exp2>`: True if `<exp1>` is numerically greater than or equal `<exp2>`.
- `( <exp> )`: True if `<exp>` is true.
- `! <exp>`: True if `<exp>` is false.
- `<exp1> && <exp2>`: True if both `<exp1>` and `<exp2>` are true.
- `<exp1> || <exp2>`: True if either `<exp1>` or `<exp2>` are true.

If the expression `[[ $var ]]` is used, it is treated as `[[ -n $var ]]`, i.e.
tests whether the variable `var` expands to a non-zero length string.

The expansion on `<string>`, `<file>` and `<pattern>` happens normally, however
they are constrained to be a single word while being evaluated, similar to using
double quotes around the expansions.

Note that normal filename expansion does not happen inside `[[`. It can be
forced by setting the option `EXTENDED_GLOB` and appending a `(#q)` at the end
of the string. The expansion happens regardless inside `[` and `test`.

In the numeric comparisons, the expressions `<exp>` undergo normal expansion as
if they were enclosed inside `$((..))`.

For more information see:
- [The Zsh manual on conditional expressions][]
- [The section above on complex commands, last bullet point](#complex-commands)

[The Zsh manual on conditional expressions]: https://zsh.sourceforge.io/Doc/Release/Conditional-Expressions.html#Conditional-Expressions

### 15. Prompt expansion

There are certain prompt sequences that undergo a special form of expansion.
This expansion type is also available while using the `-P` flag to the `print`
command, so this can be used to see the output of a prompt sequence before
changing the actual prompt.

If `PROMPT_SUBST` is on, then additionally parameter expansion, command
substitution and arithmetic expansion happen inside a prompt.

If `PROMPT_BANG` is on, a `!` in the prompt is replaced by the current history
event number. A literal `!` can be represented as `!!`.

If `PROMPT_PERCENT` is on, certain escape sequences that start with `%` are
expanded.

#### Single prompt escapes (single character percent expansion)

Special cases:
- `%%`: A literal `%`.
- `%)`: A literal `)`.

Login information:
- `%l`: The tty the user is logged in on, without the `/dev/` prefix (Example:
  `pts/8`). If the name starts with `/dev/tty`, the prefix is stripped.
- `%y`: The tty the user is logged in on, without the `/dev/` prefix. However
  this does not treat `/dev/tty` specifically.
- `%M`: The full machine hostname.
- `%m`: The hostname up to the first `.`. An integer may follow the `%` to
  specify how many components of the hostname should be included in the
  string. Can also work with a negative integer, in which case trailing
  components are shown.
- `%n`: The `$USERNAME` variable.

Shell state:
- `%#`: A `#` if the shell runs with privileges, a `$` otherwise.
- `%?`: The return status of the last command.
- `%_`: The status of different constructs (i.e. `if`, `while` etc.) that have
  been started. Most useful in `PS2` prompts or in `PS4` for debugging with
  `XTRACE` option on.
- `%^`: The status of different constructs in reverse. Same as above, except
  it's more useful in `RPS2` prompts.
- `%d` or `%/`: Current working directory. A number following the `%` specifies
  how many directories to show trailing the current directory. Zero means the
  whole path. A negative integer is used for leading components.
- `%~`: Same as above, except if current working directory starts with `$HOME`,
  that part is replaced with `~`. Also works with named directories.
- `%e`: Evaluation depth of the current sourced file, shell function, or `eval`.
  This is incremented or decremented every time the value of `%N` is set or
  reverted to a previous value (sort-of like a call stack). Most useful for
  debugging in `PS4` prompts.
- `%h` or `%!`: Current history event number.
- `%i`: The current line number being executed as part of a sourced file, shell
  function or `eval`. Most useful for debugging in `PS4` prompts (also see `%e`
  above).
- `%I`: The line number being currently executed in file `%x`. Shows the line
  number in the file, even if a shell function is currently executed, whereas
  `%i` above would show the line number in the function.
- `%j`: The current number of jobs.
- `%L`: The current value of `$SHLVL`.
- `%N`: The name of the script, source file or shell function that zsh is
  currently executing, whichever was started more recently. If no script is
  running, then `%N` takes the value of `$0`.
- `%x`: The name of the file containing the source code currently being
  executed. Behaves like `%N`, except that only files are shown, not functions
  or `eval` commands.
- `%c` or `%.` or `%C`: Trailing component of the current working directory. An
  integer may follow the `%` to get more than one component. Unless `%C` is
  used, tilde contraction is performed first. These should not be used anymore
  though, but instead `%1~` for `%c` and `%1/` for `%C` should be used instead.

Date and time:
- `%D`: Current date in `yy-mm-dd` format.
- `%T`: Current time of day, in 24-hour format.
- `%t` or `%@`: Current time of day, in 12-hour, am/pm format.
- `%*`: Current time of day in 24-hour format, with seconds.
- `%w`: The date in `day dd` format.
- `%W`: The date in `mm/dd/yy` format.
- `%D{<string>}`: `<string>` is formatted using the `strftime` format (see `man
  strftime(3)` for more details). The are a couple of extra Zsh extensions with
  no extra leading zero or space if the number is a single digit:
    - `%f`: a day of the month.
    - `%K`: the hour of the day in 24-hour format.
    - `%L`: the hour of the day in 12-hour format.

  In addition to this, if the system supports POSIX `gettimeofday()` system
  call, `%.` provides decimal fractions of a second since the epoch with leading
  zeroes. By default only 3 decimal places are provided, but up to 9 can be
  specified. This means that `%6.` outputs microseconds, and `%9.` outputs
  nanoseconds.

Visual effects:
- `%B (%b)`: Start (stop) bold mode.
- `%E`: Clear to the end of the line.
- `%U (%u)`: Start (stop) underline mode.
- `%S (%s)`: Start (stop) standout mode (inversion of background and foreground
  colors).
- `%F (%f)`: Start (stop) using a different foreground color, if supported by
  the terminal. The colour can be specified in two ways: either as a numeric
  argument, or by a sequence following the escape sequence (i.e. `%F{red}`). The
  available such sequences can be found in the `fg zle_highlight` attribute
  (TODO link to "zle line editor" - "character highlighting"). Numeric colors
  are allowed also in the second format.
- `%K (%k)`: Start (stop) using a different bacKground color. Same as above for
  formats.
- `%{...%}`: Include a string as a literal escape sequence. The string within
  the braces should not change the cursor position. Brace pairs can be nested.
  A positive numeric argument can be specified. Treated the same as below for
  `%G`.
- `%G`: Within a `%{...%}`, include a _glitch_: assume that a single character
  width will be output. This is helpful when trying to output characters that
  are not correctly understood by the shell, like different character sets on
  some terminals. Specifying a number for `%G` will assume a different character
  width (i.e. `%{<seq>%2G%}` will assume a character width of 2).

Conditional substrings in prompts:
- `%v`: The value of the first element of the `psvar` array. An integer can be
  given to specify the element in the array. Negative integers count from the
  end.
- `%(<x>.<true-text>.<false-text>)`: Ternary conditional expression. The
  separator between the elements is arbitrary. The same character should be used
  however for both places. Both `<true>` and `<false>` elements may contain
  arbitrary escape sequences, including further ternary expressions.

  The left parenthesis may be preceded or followed by a positive integer `n`,
  which defaults to zero. A negative integer will be multiplied by `-1`,
  exception being the case below for `l`.

  The test character `<x>` may be any of the following:
    - `!`: True if the shell is running with privileges.
    - `#`: True if the effective uid of the process is `n`.
    - `?`: True if the exit status of the last command was `n`.
    - `_`: True if at least `n` shell constructs have been started.
    - `C` or `/`: True if the current absolute path has at least `n` elements
      relative to the root directory, hence `/` (root) is counted as 0 elements.
    - `c` or `.` or `~`: True if the current path, with prefix replacement, has
      at least `n` elements relative to the root directory.
    - `D`: True if month is equal to `n` (January = 0).
    - `d`: True if day of month is equal to `n`.
    - `e`: True if evaluation depth is at least `n`.
    - `g`: True if the effective gid of the process is `n`.
    - `j`: True if the number of jobs is at least `n`.
    - `L`: True if the `SHLVL` parameter is at least `n`.
    - `l`: True if at least `n` characters have already been printed on the
      current line. When `n` is negative, true if at least `abs(n)` characters
      remain before the opposite margin (thus the left margin for `RPROMPT`).
    - `S`: True if the `SECONDS` parameter is at least `n`.
    - `T`: True if the time in hours is equal to `n`.
    - `t`: True if the time in minutes is equal to `n`.
    - `v`: True if the array `psvar` has at least `n` elements.
    - `V`: True if element `n` of the `psvar` array is set and non-empty.
    - `w`: True if day of the week is equal to `n` (Sunday = 0).
- `%<string<` or `%>string>` or `%[xstring]`: Specified truncation behaviour for
  the remainder of the prompt string. This does not undergo prompt expansion.
  `string` will be displayed instead of the truncated part.

  The numeric argument, which in the third form may appear immediately after
  `[`, specifies the maximum permitted length of the various strings that can be
  displayed in the prompt. If the integer is negative, then the truncation
  length is calculated by subtracting `abs(n)` from the number of remaining
  characters on the current prompt line. If this results in zero or negative
  length, then a length of 1 is used instead. This means that a negative integer
  does so that after truncation, there are still `n` characters before the right
  margin (or the left margin in case of `RPROMPT`).

  The form with `<` truncates at the left of the string, while the form with `>`
  truncates at the right of the string.

  For example:

  ```zsh
  print -P `%8<rob<pikeypikester`
  > robester
  ```

  For something more useful (like path truncation):

  ```zsh
  print -P `%8<..<%/`
  > ..hing/here
  ```

  If `string` happens to be longer than the actual truncation length, it will
  appear in full, replacing the truncated string:

  ```zsh
  print -P `%2<rob<pikeypikester`
  > rob
  ```

  A truncation with argument zero (i.e. `%<<`), marks the end of the part to be
  truncated, while turning truncation off from there on. Note that `%-0<<`
  specifies that the prompt is truncated at the right margin.

### 16. General expansion

There are general expansion types that happen inside the shell. The order of the
expansions is as follows:
- `History expansion`: Only performed in interactive shells.
- `Alias expansion`: Aliases are expanded immediately, before the command line
  is actually parsed (see [aliasing](#aliasing) for more).
- `Process substitution`, `Parameter expansion`, `Command substitution`,
  `Arithmetic expansion`, `Brace expansion`: The five expansions happen in
  left-to-right fashion. What this means is that all process substitutions are
  replaced on the command line, before parameter expansion starts to take place.
  After all the expansions take place, all unquoted occurrences of the
  characters `\`, `'`, `"` are removed.
- `Filename expansion`: If the option `SH_FILE_EXPANSION` is set, the order of
  expansion is modified for compatibility with `sh` and `ksh`. In that case,
  `filename expansion` is performed immediately after `alias expansion`,
  preceding the set of five expansions mentioned above.
- `Filename generation`: This expansion is always done last (commonly referred
  to as _globbing_).

All these expansions are further explained below:

#### History expansion

This form of expansion only applies in interactive shells.

This kind of expansion allows you to use command line history, i.e. use already
interpreted commands to construct the current command. This simplifies
repetition of long commands or spelling errors.

Immediately after being executed, each command is assigned a number and saved in
the history of the shell commands. The size of the history is controlled by the
`HISTSIZE` parameter. Each saved command is called a history _event_. The
history number in prompt expansion is the number to be assigned to the _next_
command (see [prompt expansion](#prompt-expansion) above).

History expansion begins with the first character of `histchars` (by default
`!`). It can occur anywhere inside the command line, including inside double
quotes, but not inside single quotes(`'..'` or `$'..'`), nor when escaped with a
backslash.

The first character is then followed by an optional [event designator](#event-designators) and then an
optional [word designator](#word-designators). If neither of these is present,
then no history expansion occurs.

History expansion lines are echoed before being executed, and before any other
expansions take place. It's this line that gets saved into history.

##### Event designators

An event designator is a reference to an entry in the history list:
- `!`: Start a history expansion, except when followed by a blank space, a
  newline, `=` or `(`.
- `!!`: Refer to the previous command.
- `!<n>`: Refer to command-line `<n>`.
- `!<-n>`: Refer to the current command-line minus `<n>`.
- `!<str>`: Refer to the most recent command starting with `<str>`.
- `!?<str>[?]`: Refer to the most recent command containing `<str>`. The
  trailing `?` is necessary if the next part is not to be considered part of
  `<str>`.
- `!#`: Refer to the current command line typed in so far. The line is treated
  to be complete up to and including the word before the reference `!#`.
- `!{...}`: Encapsulate a history reference from adjacent characters.

##### Word designators

A word designator is a reference to a part of the event specified by the event
designator. If no event designator is specified, then the last history entry
(the last command executed) is to be considered the event designator.

The word designators can be separated from the event designators using `:`. It
may be omitted if the word designator starts with `^`, `$`, `*`, `-` or `%`.

Word designators include:
- `0`: The first input word (the command that was executed).
- `n`: The `n-th` argument.
- `^`: The first argument (similar to the above case when `n` is 1).
- `$`: The last argument.
- `%`: The word matched by the most recent `?<str>` search.
- `x-y`: A range of words; `x` defaults to 0.
- `*`: All the arguments, or a null value if there are none.
- `x*`: Abbreviates to `x-$`.
- `x-`: Like `x*`, but omitting word `$` (omits the last word).

Additionally, the `%` can appear inside a command only if there was an earlier
`!?` expansion (possibly in an earlier command). Otherwise it results in an
error.

---

By default, a history reference with no event designator refers to the same
event as any preceding history reference on the same command line. If it's the
only history reference on the command line, then it refers to the previous
command line event.

With `CSH_JUNKIE_HISTORY` set, then every history reference with no event
specification _always_ refers to the previous command line event. To see with an
example what this means: `!!:1` always refers to the first argument of the
previous command, and `!!:$` always refers to the last argument of the previous
command. With `CSH_JUNKIE_HISTORY` set, the events `!:1` and `!:$` work in the
same manner as the above examples. However, with `CSH_JUNKIE_HISTORY` unset,
then `!:1` and `!:$` refer to the first and last arguments of the event
specified by the nearest history reference preceding those two, or the previous
command if there isn't such a history reference.

##### Modifiers

After the optional word designator, there can additionally be a sequence of one
or more of the following modifiers, separated by `:`. These modifiers also work
on the result of _filename generation_ and _parameter expansion_, except where
noted below.

The modifiers that can appear in a history expansion are:
- `a`: Turn a file name into an absolute path: prepends the current directory,
  and removes any unnecessary `./` or `../` and the segments preceding them.
  Basically, if the path to a file is correctly specified, then the absolute
  path will also work. For example, suppose there is a file `/home/user/file.c`.
  Involving the history reference mechanism from `/home/user/project/subproject`
  with the previous command `echo ../../file.c` works as `echo !!:1:a` and will
  output `echo /home/user/file.c` correctly. The transformation however does not
  care about what actually is in the filesystem.
- `A`: Performs the same transformations as above, but then passes the path
  through `realpath(3)`, in order to follow potential symbolic links (the form
  above doesn't). **Note:** On some inputs, `foo:A` and `realpath(foo)` might be
  different. See the modifier `P` below.
- `c`: Resolve a command name into an absolute path by searching the `PATH`
  variable.
- `e`: Remove everything except the part after the `.` in a filename.
- `h [digits]`: Remove the trailing pathname component similar to `dirname`
  (keep the head of the pathname). If the `h` is followed by any digits, that
  number of leading components is preserved.
- `l`: Convert to lowercase.
- `p`: Print the new command without executing it. Only for history expansion.
- `P`: Turn the file name into an absolute path, similar to `realpath(3)`. Also
  see the `A` and `a` modifiers above.
- `q`: Quote the substituted words, escaping further substitutions.
- `Q`: Remove one level of quotes from the substituted words.
- `r`: Remove a filename extension leaving the root name. Strings with no
  filename extension are not modified.
- `s/l/r[/]`: Substitute `l` for `r`. The substitution is only done for the
  first string that matches `l`. For arrays and for filename generation, this
  applies for each word of the expanded text. The forms `gs/l/r` and `s/l/r/:G`
  perform global substitution, meaning they substitute every occurrence of `l`
  for `r`, not just the first one. There is more information in the manual about
  what `l` and `r` can contain and how the substitution operates in-depth.
- `&`: Repeat the previous `s` substitution. Like `s`, may be preceded
  immediately by a `g`.
- `t[digits]`: Remove all leading pathname components, leaving the final
  component only (_tail_ or _trailing_). This works like `basename`.
- `u`: Converts to uppercase.
- `x`: Like `q`, but break into words at whitespace. Does not work with
  parameter expansion.

---

The rest of the modifiers only work with parameter expansion and filename
generation.

- `f`: Repeats the immediately following until the resulting word doesn't change
  anymore.
- `F:expr:`: Like `f`, but repeats only _n_ times if the expression `expr`
  evaluates to _n_.
- `w`: Makes the immediately following modifier work on each word in the string.
- `W:sep:`: Like `w`, but words are considered to be part of the string that are
  separated by `sep`.

#### Process substitution

A process substitution takes place in command arguments that look like
`<(list)`, `>(list)` or `=(list)`. See [Simple commands](#simple-commands-and-pipelines) for what `list`
refers to.

Note that `<<(list)` is not a special form of process substitution. It is the
same as `< <(list)`, redirecting standard input from the result of process
substitution.

In the case of `<` or `>` forms, the shell runs the commands in `list` as a
subprocess of the current job executing the shell command line. The result is
either placed in a device file corresponding to a file descriptor (if the system
supports the `/dev/fd` mechanism), or a named pipe if the system supports named
pipes (FIFOs). The result of the process substitution is the literal file name,
that has the contents of running the command. In the case of the `/dev/fd`
mechanism, the actual name of the file can be seen by running `echo >(true)`.

For the `>` form, writing on this file will provide input for `list`, while for
the `<` form, the results of running `list` will write the file, that will
provide input for the current command. As an example:

```zsh
paste <(cut -f1 file1) <(cut -f3 file2) | tee >(process1) >(process2) >/dev/null
```

If the `cut` processing was not needed, then simply doing the below would be
enough:

```zsh
paste file1 file2 | ...
```

This illustrates a good point where process substitution is useful. The most
important thing to remember is that the forms `>(...)` and `<(...)` are
**literally** replaced by a file name on the command line following process
substitution (at least in the case of the `/dev/fd` mechanism). There might be
cases where the command cannot read from a file, but rather expects input
redirected to it. The form `<<(...)` can then be used.

The `=(...)` form is very similar to `<(...)`, with the difference that the
former uses a temporary file instead of a device file, which is useful in places
where commands are expected to lseek through the file (see `man lseek(2)`).

There are other places where the `=(...)` is useful, since both the `<(...)` and
the `>(...)` have drawbacks in certain cases. For more see [the process substitution
section in the manual](https://zsh.sourceforge.io/Doc/Release/Expansion.html#Process-Substitution).

A good explanation on process substitution in [this post](https://stackoverflow.com/a/33532589).


For more information see:
- [The Expansion section in the Zsh manual][]

[The Expansion section in the Zsh manual]: https://zsh.sourceforge.io/Doc/Release/Expansion.html#Expansion
