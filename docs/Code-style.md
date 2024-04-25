# General guidelines

## File names

File names should be meaningful, written in lowercase characters only and end in .R. Both minus (-) and underscore (_) are good options to separate words within a file name, choosing whatever makes more sense. Never use dot (.), and never ever use whitespace ( ) for that purpose.

```Bash
# Good
fit-models.R
utility_functions.R

# Bad
foo.r
StuFF.r
T3rr1bl3 N4m1ng.r
```

If files need to be run in sequence, prefix them with numbers and underscore (_):

```Bash
0_download.R
1_parse.R
2_explore.R
```

## General Code Style

While you should follow the code style that's already there for files that you're modifying, the following are required for any new code.

### Indentation

Indent 4 spaces. No tabs. If you use the vim configuration in the [dotfiles repository](https://github.com/BU-ISCIII/dotfiles) it will change tabs for spaces automatically.

Use blank lines between blocks to improve readability. For existing files, stay faithful to the existing indentation.

### Line Length and Long Strings

Maximum line length is 80 characters.

If you have to write strings that are longer than 80 characters, this should be done with an embedded newline if possible. Literal strings that have to be longer than 80 chars and can't sensibly be split are okay, but it's strongly preferred to find a way to make it shorter.

#### _Bad:_

```shell
long_string_1="I am an exceptionalllllllllllly looooooooooooooooooooooooooooooooooooooooong string."
```

#### _Good:_

```shell
cat <<END;
I am an exceptionalllllllllllly 
looooooooooooooooooooooooooooooooooooooooong string.
END
```

#### _Good:_

```shell
long_string_2="I am an exceptionalllllllllllly 
 looooooooooooooooooooooooooooooooooooooooong string."
```

## Variables

### Naming Conventions

Meaningful self-documenting names should be used. If the variable name does not make it reasonably obvious as to the meaning of the variable, appropriate comments should be added.

Variable names must be words in lowcase separated with underscore, no camelCase.

#### _Bad:_

```shell
local TitleCase=""
local camelCase=""
```

#### _Good:_

```shell
local snake_case=""
```

Uppercase strings are reserved for global variables.

#### _Bad:_

```shell
local UPPERCASE=""
```

#### _Good:_

```shell
UPPERCASE=""
```

Variable names should not clobber command names, such as `dir` or `pwd`.

#### _Bad:_

```shell
local pwd=""
```

#### _Good:_

```shell
local pwd_read_in=""
```

Variable names for loop indexes should be named similarly to any variable you're looping through.

#### _Good:_

```shell
for zone in ${zones}; do
  something_with "${zone}"
done
```

### Constants and Environment Variable Names

All caps, separated with underscores, declared at the top of the file. Constants and anything exported to the environment should be capitalized.

#### _Constant:_

```shell
PATH_TO_FILES='/some/path'
```

## Functions

### Naming Conventions

Lower-case, with underscores to separate words. Parentheses are required after the function name. The opening brace should appear on the same line as the function name.

#### _Bad:_

```shell
function my_bad_func {
  ...
}
```

#### _Good:_

```shell
function my_good_func() {
  ...
}

```

### Error catching

You MUST always use error catching. This varies a lot between programming languages so this issue will be further address in each programming languages code styles.
