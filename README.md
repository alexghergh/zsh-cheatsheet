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
