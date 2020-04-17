**SPEC VERSION 1.0** (This file is very messy, and most content is WIP and needs a lot of revision, changes, simplifications, etc)
# UJMap Format Specification
- [Contexts](#contexts)
- [Operators](#operators)
	* [Embedder](#embedder-mode)
	* [Requires](#requires)
	* [Definitions](#definitions)
	* [Embedding](#embedding)
	* [Comments](#comments)
	* [Other](#other)
- [Examples](#examples)

# Terms:
In this file I use the terms "embedder", "embed", "context", "require(s)", "utility program"/"utility implementation"/"utility". Here are simple definitions of my usage of these terms:
* Embedder - Essentially a format or protocol that is used with an "embed"
* Embed - A dynamic object that can be executable, or simply data. For example, a json object, another uj script, text, or images. Embeds can have unique behaviours as implemented by the "utility"
* Context - Something that's unique to executable embeds. This essentially just means all globals, locals, and extra data associated with an executable embed.
* Require - An embed loaded (or the act of loading said embed) from a file or some case handled by the utility uniquely (e.g. builtin modules).
* Utility - This is the term I am using to refer to the whole entire implementation of ujmap, for example, a node js module which contains code to execute and parse ujmap files, as well as its own custom require functionality and built in embedders. Ujmap is intended to be easily transpilable to modern languages such as lua or js, and its also intended to easily have parsers/executors implemented into said modern languages.

# Contexts
Every executable (uj, js, lua, etc where implemented) embed, **including requires** gets its own context. The following items are specific and unique to each context:
* [Embedder mode](#embedder-mode)
* Locally stored embeds

# Operators
Operators are single characters prefixing a line. Some operators can consume multiple lines, as well as other operators.
## Embedder mode
### !
The `!` character signifies an embedder mode change. All text up until the next line is included. Mode name content is dependant on implementation.\
An embedder mode is simply an identifier which must have a referenced behaviour in the ujmap utility. In other words, its the name of the embed processor (Could be literally anything as long as the ujmap utility implements it, e.g. `!json`, `!js`, `!lua`, `!uj`, `!png`, etc)

Example usage:
```sh
!uj
~
#~
	Some
	uj
	code
~#
~

!json
~
[
	{
		test: 123
	},
	"abc"
]
~

!uj: +some_file.uj # This time we require it
```
\
[Contents](#ujmap-specification)
## Requires
### +
Requires load content from a referenced file as ujmap code. The file name is determined by leading whitespace (all content up until the first whitespace is included).\
Examples: `+sitemap.uj`, `+my_uj_file.uj`

Required code is essentially the same as directly wrapping file contents in an embed.
Sub requires can be done by adding another + followed by a global variable name (or export name, e.g. json keys: `+example_file.txt+GLOBALLY_DEFINED_VARIABLE_NAME+NEXT_GLOBAL_NAME`)

The ujmap utility program can additionally list its own modules and names, for example, for built in libraries.
\
\
[Contents](#ujmap-specification)
## Definitions
### =
The `=` character defines a global variable. Prefixing the global variable with an `_` makes it localized (it doesn't get exported into parent embeds, and is always referenced with an _, essentially it can only be used in the current uj script)\
By wrapping global names in parenthesese you can multi-assign variables:
`=(GVAR1, GVAR2, _LVAR1)`
By prefixing variables with a `^` you store the variables from the current context within the next executable embed, and an `_` gets them:
A syntactic sugar version for inline embedder modes is `!embedder_name (VAR1, VAR2, VAR3, etc) EMBED`
```sh
# Some uj code
!uj
!": =message1
~Hello world!~
!": =message2
~Greetings universe!~
!": =_secret
~I'm secret! :D~

=^(message1, message2, _secret #~ Shh, don't tell anyone else about me ;) ~#); !uj: +my_module.uj # Pass some arguments to our module
# Syntatic sugar version:
!uj: (message1, message2, _secret) +my_module.uj

# my_module.uj
=_(message1, message2, _aSecret) # Load all three variables (not name dependant, just order dependant, much like a function call)

!console: (message1, message2, _aSecret) ~log~ # Syntactic sugar for "=^(message1, message2, _aSecret) !console: ~log~"
=^(~~, _aSecret)
```
``
## Embedding
### ~
Embedding is signified by a `~` character. The current embedder mode defines behaviour within definition contexts. (See [Embedder](#embedder-mode)) This character encloses its content similarly to how a quote would in any standard programming language. Additionally, contents can be escaped using a \ character.
This allows ujmap utilities to easily transpile, parse, etc contents into strings. \

Inline embedding can be done using `!embedder_name: ~embedder code~`. \
\
[Contents](#ujmap-specification)
## Comments
### \#
All content after the `#` character on any line is a comment. Long comments can be done using the embedder syntax, suffixed with a final `#` character.\
Note: Long comments **do not** function in any way as embeds, and should not be interpreted as such.
```sh
# Comment
#~ Note: I'm not real embedder code! I shouldn't create a context.
Looooong
comment
~#
```
\
[Contents](#ujmap-specification)
## Other
### ;
The `;` character is simply an alias for a new line (not including within embed contents!), however it is not an enforced ending. This allows code to be condensed down into one line for those who like to minify their code.\
\
[Contents](#ujmap-specification)

# Standard Embedders **TODO**
`math` - Processes a string and executes it as math
`uj` - Executes a uj file
`console` - A console utility for outputting content (content will be converted by the utility implementation)
`json` - Loads json content (Not executable)
`tuple` - A tuple embed (Would be internally used for the `(var1, var2, var3...)` syntax)

# Examples **TODO**
Also see the [examples](examples) folder.
[embed_formats.uj](examples/embed_formats.uj) - Shows examples of embed formats.
[context_example.uj](examples/contexts/context_example.uj) - Shows how uj contexts work (uj embedder within a uj file)
[embedder_embedders.uj](examples/embedder_embedders/embedder_embedders.uj) - Example creating more embedders using an executable embedder
