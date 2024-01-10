# PackCC #

## Overview ##

**PackCC** is a parser generator for C.
Its main features are as follows:

- Generates your parser in C from a grammar described in a **PEG**,
- Gives your parser great efficiency by **packrat parsing**,
- Supports direct and indirect **left-recursive** grammar rules.

The grammar of your parser can be described in a **PEG** ([Parsing Expression Grammar](http://en.wikipedia.org/wiki/Parsing_expression_grammar)).
The PEG is a [top-down parsing language](http://en.wikipedia.org/wiki/Top-down_parsing_language),
and is similar to the [regular-expression](http://en.wikipedia.org/wiki/Regular_expression) grammar.
Compared with a bottom-up parsing language, like Yacc's one, the PEG is much more intuitive and cannot be ambiguous.
The PEG does not require tokenization to be a separate step, and tokenization rules can be written in the same way as any other grammar rules.

Your generated parser can parse inputs very efficiently by **packrat parsing**.
The packrat parsing is the [recursive descent parsing](http://en.wikipedia.org/wiki/Recursive_descent_parser) algorithm
that is accelerated using [memoization](http://en.wikipedia.org/wiki/Memoization).
By using packrat parsing, any input can be parsed in linear time.
Without it, however, the resulting parser could exhibit exponential time performance in the worst case due to the unlimited look-ahead capability.

Unlike common packrat parsers, PackCC can support direct and indirect **left-recursive** grammar rules.
This powerful feature enables you to describe your language grammar in a much simpler way.
<small>(The algorithm is based on the paper [*"Packrat Parsers Can Support Left Recursion"*](http://www.cs.ucla.edu/~todd/research/pub.php?id=pepm08)
authored by A. Warth, J. R. Douglass, and T. Millstein.)</small>

Some additional features are as follows:

- Thread-safe and reentrant,
- Supports UTF-8 multibyte characters (version 1.4.0 or later),
- Generates more ease-of-understanding parser source codes,
- Consists of just a single compact source file,
- Under MIT license. (not under a certain contagious license!)

The generated code is beautified and as ease-of-understanding as possible.
Actually, it uses lots of *goto* statements, but the control flows are much more traceable
than *goto* spaghetti storms generated by Yacc or other parser generators.
This feature is irrelevant to common users, but helpful for PackCC developers to debug it.

PackCC itself is under MIT license, but you can distribute your generated code under any license you like.

## Installation ##

You can obtain the executable `packcc` by compiling [`src/packcc.c`](src/packcc.c) using your favorite C compiler.
For convenience, the build environments using GCC, Clang, and Microsoft Visual Studio are prepared under [`build`](build) directory.

### Using GCC ###

#### Other than MinGW ####

`packcc` will be built in both directories `build/gcc/debug/bin` and `build/gcc/release/bin` using `gcc` by executing the following commands:

```
cd build/gcc
make
make check  # bats-core and uncrustify are required (see tests/README.md)
```

`packcc` in the directory `build/gcc/release/bin` is suitable for practical use.

#### MinGW ####

`packcc` will be built in both directories `build/mingw-gcc/debug/bin` and `build/mingw-gcc/release/bin` using `gcc` by executing the following commands:

```
cd build/mingw-gcc
make
make check  # bats-core and uncrustify are required (see tests/README.md)
```

`packcc` in the directory `build/mingw-gcc/release/bin` is suitable for practical use.

### Using Clang ###

#### Other than MinGW ####

`packcc` will be built in both directories `build/clang/debug/bin` and `build/clang/release/bin` using `clang` by executing the following commands:

```
cd build/clang
make
make check  # bats-core and uncrustify are required (see tests/README.md)
```

`packcc` in the directory `build/clang/release/bin` is suitable for practical use.

#### MinGW ####

`packcc` will be built in both directories `build/mingw-clang/debug/bin` and `build/mingw-clang/release/bin` using `clang` by executing the following commands:

```
cd build/mingw-clang
make
make check  # bats-core and uncrustify are required (see tests/README.md)
```

`packcc` in the directory `build/mingw-clang/release/bin` is suitable for practical use.

### Using Microsoft Visual Studio ###

You have to install Microsoft Visual Studio 2019 in advance.
After that, you can build `packcc.exe` by the following instructions:
- Open the solution file `build\msvc\msvc.sln`,
- Select a preferred solution configuration (*Debug* or *Release*) and a preferred solution platform (*x64* or *x86*),
- Invoke the *Build Solution* menu item.

`packcc.exe` will appear in `build\msvc\XXX\YYY` directory.
Here, `XXX` is `x64` or `x86`, and `YYY` is `Debug` or `Release`.
`packcc.exe` in the directory `build\msvc\XXX\Release` is suitable for practical use.

## Usage ##

### Command ###

You must prepare a PEG source file (see the following section).
Let the file name `example.peg` for example.

```
packcc example.peg
```

By running this, the parser source `example.h` and `example.c` are generated.

If no PEG file name is specified, the PEG source is read from the standard input, and `-.h` and `-.c` are generated.

The base name of the parser source files can be changed by `-o` option.

```
packcc -o parser example.peg
```

By running this, the parser source `parser.h` and `parser.c` are generated.

If you want to disable UTF-8 support, specify the command line option `-a` or `--ascii` (version 1.4.0 or later).

If you want to insert `#line` directives in the generated source and header files, specify the command line option `-l` or `--lines` (version 1.7.0 or later).
It is helpful to trace compilation errors of the generated source and header files back to the codes written in the PEG source file.

If you want to confirm the version of the `packcc` command, execute the below.

```
packcc -v
```

### Tree syntax ###
This syntax lets you define ASTs for an easier development experience.

First, Add this to your peg file: `%value "TREE_VALUE*"`. This makes sure your return values are always a tree node.

**Create a number value with `number(num)`. Example: `number(5)`**

**Create a text value with `text(txt)`. Example: `text("Hello World!")`**

**Create a tree value with `make(type, left, right)`. The structure only supports binary trees.**

**Create a null value whenever needed (rarely by the user) with `CREATE_NULL()`.**

**Create a node (a tree value with the left value being your value and the right being null) with `CREATE_NODE(type, data)`.**

**Use `PRINT_VALUE(expression)` to print your syntax tree at any point.**

#### Tree API ####

```c
enum VALUE_TYPE { NUMBER, STRING, TREE };

// --- IMPORTANT PARTS --- //
typedef struct tree {
   string type;
   TREE_VALUE *left;
   TREE_VALUE *right;
} TREE_PART;

typedef struct value {
   enum VALUE_TYPE    type;
   union _value *value;
} TREE_VALUE;
// --- IMPORTANT PARTS --- //

typedef union _value {
   int   number;
   string _text;
   TREE_PART *tree;
} _TREE_VALUE;
```

### Syntax ###

A grammar consists of a set of named rules.
A rule definition can be split into multiple lines.

**_rulename_ `<-` _pattern_**

The _rulename_ is the name of the rule to define.
The _pattern_ is a text pattern that contains one or more of the following elements.

**_rulename_**

The element stands for the entire pattern in the rule with the name given by _rulename_.

**_variable_`:`_rulename_**

The element stands for the entire pattern in the rule with the name given by _rulename_.
The _variable_ is an identifier associated with the semantic value returned from the rule by assigning to `$$` in its action.
The identifier can be referred to in subsequent actions as a variable.
The example is shown below.

```
term <- l:term _ '+' _ r:factor { $$ = l + r; }
```

A variable identifier must consist of alphabets (both uppercase and lowercase letters), digits, and underscores.
The first letter must be an alphabet.
The reserved keywords in C cannot be used.

**_sequence1_ `/` _sequence2_ `/` ... `/` _sequenceN_**

Each _sequence_ is tried in turn until one of them matches, at which time matching for the overall pattern succeeds.
If no _sequence_ matches then the matching for the overall pattern fails.
The operator slash (`/`) has the least priority.
The example is shown below.

```
'foo' rule1 / 'bar'+ [0-9]? / rule2
```

This pattern tries matching of the first sequence (`'foo' rule1`).
If it succeeds, then the overall pattern matching succeeds and ends without evaluating the subsequent sequences.
Otherwise, it tries matching of the next sequence (`'bar'+ [0-9]?`).
If it succeeds, then the overall pattern matching succeeds and ends without evaluating the subsequent sequence.
Finally, it tries matching of the last sequence (`rule2`).
If it succeeds, then the overall pattern matching succeeds.
Otherwise, the overall pattern matching fails.

**`'`_string_`'`**

A character or string enclosed in single quotes is matched literally.
The ANSI C escape sequences are recognized within the characters.
The UNICODE escape sequences (ex. `\u20AC`) are also recognized including surrogate pairs,
if the command line option `-a` is not specified (version 1.4.0 or later).
The example is shown below.

```
'foo bar'
```

**`"`_string_`"`**

A character or string enclosed in double quotes is matched literally.
The ANSI C escape sequences are recognized within the characters.
The UNICODE escape sequences (ex. `\u20AC`) are also recognized including surrogate pairs,
if the command line option `-a` is not specified (version 1.4.0 or later).
The example is shown below.

```
"foo bar"
```

**`[`_character class_`]`**

A set of characters enclosed in square brackets matches any single character from the set.
The ANSI C escape sequences are recognized within the characters.
The UNICODE escape sequences (ex. `\u20AC`) are also recognized including surrogate pairs,
if the command line option `-a` is not specified (version 1.4.0 or later).
If the set begins with an up-arrow (`^`), the set is negated (the element matches any character not in the set).
Any pair of characters separated with a dash (`-`) represents the range of characters from the first to the second, inclusive.
The examples are shown below.

```
[abc]
[^abc]
[a-zA-Z0-9_]
```

**`.`**

A dot (`.`) matches any single character.
Note that the only time this fails is at the end of input, where there is no character to match.

**_element_ `?`**

The _element_ is optional.
If present on the input, it is consumed and the match succeeds.
If not present on the input, no text is consumed and the match succeeds anyway.

**_element_ `*`**

The _element_ is optional and repeatable.
If present on the input, one or more occurrences of the _element_ are consumed and the match succeeds.
If no occurrence of the _element_ is present on the input, the match succeeds anyway.

**_element_ `+`**

The _element_ is repeatable.
If present on the input, one or more occurrences of the _element_ are consumed and the match succeeds.
If no occurrence of the _element_ is present on the input, the match fails.

**`&` _element_**

The predicate succeeds only if the _element_ can be matched.
The input text scanned while matching _element_ is not consumed from the input and remains available for subsequent matching.

**`!` _element_**

The predicate succeeds only if the _element_ cannot be matched.
The input text scanned while matching _element_ is not consumed from the input and remains available for subsequent matching.
A popular idiom is the following, which matches the end of input, after the last character of the input has already been consumed.

```
!.
```

**`(` _pattern_ `)`**

Parentheses are used for grouping (modifying the precedence of the _pattern_).

**`<` _pattern_ `>`**

Angle brackets are used for grouping (modifying the precedence of the _pattern_) and text capturing.
The captured text is numbered in evaluation order, and can be referred to later using `$1`, `$2`, etc.

**`$`_n_**

A dollar (`$`) followed by a positive integer represents a text previously captured.
The positive integer corresponds to the order of capturing.
A `$1` represents the first captured text.
The examples are shown below.

```
< [0-9]+ > 'foo' $1
```

This matches `0foo0`, `123foo123`, etc.

```
'[' < '='* > '[' ( !( ']' $1 ']' ) . )* ( ']' $1 ']' )
```

This matches `[[`...`]]`, `[=[`...`]=]`, `[==[`...`]==]`, etc.

**`{` _c source code_ `}`**

Curly braces surround an action.
The action is arbitrary C source code to be executed at the end of matching.
Any braces within the action must be properly nested.
Note that braces in directive lines and in comments (`/*`...`*/` and `//`...) are appropriately ignored.
One or more actions can be inserted in any places between elements in the pattern.
Actions are not executed where matching fails.

```
[0-9]+ 'foo' { puts("OK"); } 'bar' / [0-9]+ 'foo' 'baz'
```

In this example, if the input is `012foobar`, the action `{ puts("OK"); }` is to be executed, but if the input is `012foobaz`,
the action is not to be executed.
All matched actions are guaranteed to be executed only once.

In the action, the C source code can use the predefined variables below.

- **`$$`**
    The output variable, to which the result of the rule is stored.
    The data type is the one specified by `%value`.
    The default data type is `int`.
- **`auxil`**
    The user-defined data that has been given via the API function `pcc_create()`.
    The data type is the one specified by `%auxil`.
    The default data type is `void *`.
- _variable_
    The result of another rule that has already been evaluated.
    If the rule has not been evaluated, it is ensured that the value is zero-cleared (version 1.7.1 or later).
    The data type is the one specified by `%value`.
    The default data type is `int`.
- **`$`**_n_
    The string of the captured text.
    The _n_ is the positive integer that corresponds to the order of capturing.
    The variable `$1` holds the string of the first captured text.
- **`$`**_n_**`s`**
    The start position in the input of the captured text, inclusive.
    The _n_ is the positive integer that corresponds to the order of capturing.
    The variable `$1s` holds the start position of the first captured text.
- **`$`**_n_**`e`**
    The end position in the input of the captured text, exclusive.
    The _n_ is the positive integer that corresponds to the order of capturing.
    The variable `$1e` holds the end position of the first captured text.
- **`$0`**
    The string of the text between the start position in the input at which the rule pattern begins to match
    and the current position in the input at which the element immediately before the action ends to match.
- **`$0s`**
    The start position in the input at which the rule pattern begins to match.
- **`$0e`**
    The current position in the input at which the element immediately before the action ends to match.

An example is shown below.

```
term <- l:term _ '+' _ r:factor { $$ = l + r; }
factor <- < [0-9]+ >            { $$ = atoi($1); }
_ <- [ \t]*
```

Note that the string data held by `$`_n_ and `$0` are discarded immediately after evaluation of the action.
If the string data are needed after the action, they must be copied in `$$` or `auxil`.
If they are required to be copied in `$$`, it is recommended to define a structure as the type of output data using `%value`,
and to copy the necessary string data in its member variable.
Similarly, if they are required to be copied in `auxil`, it is recommended to define a structure as the type of user-defined data using `%auxil`,
and to copy the necessary string data in its member variable.

The position values are 0-based; that is, the first position is 0.
The data type is `size_t` (before version 1.4.0, it was `int`).

**_element_ `~` `{` _c source code_ `}`**

Curly braces following tilde (`~`) surround an error action.
The error action is arbitrary C source code to be executed at the end of matching only if the preceding _element_ matching fails.
Any braces within the error action must be properly nested.
Note that braces in directive lines and in comments (`/*`...`*/` and `//`...) are appropriately ignored.
One or more error actions can be inserted in any places after elements in the pattern.
The operator tilde (`~`) binds less tightly than any other operator except alternation (`/`) and sequencing.
The error action is intended to make error handling and recovery code easier to write.
In the error action, all predefined variables described above are available as well.
The examples are shown below.

```
rule1 <- e1 e2 e3 ~{ error("e[12] ok; e3 has failed"); }
rule2 <- (e1 e2 e3) ~{ error("one of e[123] has failed"); }
```

**`%header` `{` _c source code_ `}`**

The specified C source code is copied verbatim to the C header file before the generated parser API function declarations.
Any braces in the C source code must be properly nested.
Note that braces in directive lines and in comments (`/*`...`*/` and `//`...) are appropriately ignored.

**`%source` `{` _c source code_ `}`**

The specified C source code is copied verbatim to the C source file before the generated parser implementation code.
Any braces in the C source code must be properly nested.
Note that braces in directive lines and in comments (`/*`...`*/` and `//`...) are appropriately ignored.

**`%common` `{` _c source code_ `}`**

The specified C source code is copied verbatim to both of the C header file and the C source file
before the generated parser API function declarations and the implementation code respectively.
Any braces in the C source code must be properly nested.
Note that braces in directive lines and in comments (`/*`...`*/` and `//`...) are appropriately ignored.

**`%earlyheader` `{` _c source code_ `}`**

**`%earlysource` `{` _c source code_ `}`**

**`%earlycommon` `{` _c source code_ `}`**

Same as `%header`, `%source` and `%common`, respectively.
The only difference is that these directives place the code at the very beginning of the generated file,
before any code or includes generated by PackCC.
This can be useful for example when it is necessary to modify behavior of standard libraries via a macro definition.

**`%value` `"`_output data type_`"`**

The type of output data, which is output as `$$` in each action and can be retrieved from the parser API function `pcc_parse()`,
is changed to the specified one from the default `int`.

**`%auxil` `"`_user-defined data type_`"`**

The type of user-defined data, which is passed to the parser API function `pcc_create()`,
is changed to the specified one from the default `void *`.

**`%prefix` `"`_prefix_`"`**

The prefix of the parser API functions is changed to the specified one from the default `pcc`.

**`#`_comment_**

A comment can be inserted between `#` and the end of the line.

**`%%`**

A double percent `%%` terminates the section for rule definitions of the grammar.
All text following `%%` is copied verbatim to the C source file after the generated parser implementation code.

<small>(The specification is determined by referring to [peg/leg](http://piumarta.com/software/peg/) developed by Ian Piumarta.)</small>

### Macros ###

Some macros are prepared to customize the parser.
The macro definition should be in <u>`%source` section</u> in the PEG source.

```
%source {
#define PCC_GETCHAR(auxil) get_character((auxil)->input)
#define PCC_BUFFERSIZE 1024
}
```

The following macros are available.

**`PCC_GETCHAR(`**_auxil_**`)`**

The function macro to get a character from the input.
The user-defined data passed to the API function `pcc_create()` can be retrieved from the argument _auxil_.
It can be ignored if no user-defined data.
This macro must return a character code as an `int` type, or `-1` if the input ends.

The default is defined as below.

```C
#define PCC_GETCHAR(auxil) getchar()
```

**`PCC_ERROR(`**_auxil_**`)`**

The function macro to handle a syntax error.
The user-defined data passed to the API function `pcc_create()` can be retrieved from the argument _auxil_.
It can be ignored if no user-defined data.
This macro need not return a value.
It may abort the process (by using `exit()` for example) when a fatal error occurs, and can also return normally to deal with warnings.

The default is defined as below.

```C
#define PCC_ERROR(auxil) pcc_error()
static void pcc_error(void) {
    fprintf(stderr, "Syntax error\n");
    exit(1);
}
```

**`PCC_MALLOC(`**_auxil_**`,`**_size_**`)`**

The function macro to allocate a memory block.
The user-defined data passed to the API function `pcc_create()` can be retrieved from the argument _auxil_.
It can be ignored if no user-defined data.
The argument _size_ is the number of bytes to allocate.
This macro must return a pointer to the allocated memory block, or `NULL` if no sufficient memory is available.

The default is defined as below.

```C
#define PCC_MALLOC(auxil, size) pcc_malloc_e(size)
static void *pcc_malloc_e(size_t size) {
    void *p = malloc(size);
    if (p == NULL) {
        fprintf(stderr, "Out of memory\n");
        exit(1);
    }
    return p;
}
```

**`PCC_REALLOC(`**_auxil_**`,`**_ptr_**`,`**_size_**`)`**

The function macro to reallocate the existing memory block.
The user-defined data passed to the API function `pcc_create()` can be retrieved from the argument _auxil_.
It can be ignored if no user-defined data.
The argument _ptr_ is the pointer to the previously allocated memory block.
The argument _size_ is the new number of bytes to reallocate.
This macro must return a pointer to the reallocated memory block, or `NULL` if no sufficient memory is available.
The contents of the memory block should be left unchanged in any case even if the reallocation fails.

The default is defined as below.

```C
#define PCC_REALLOC(auxil, ptr, size) pcc_realloc_e(ptr, size)
static void *pcc_realloc_e(void *ptr, size_t size) {
    void *p = realloc(ptr, size);
    if (p == NULL) {
        fprintf(stderr, "Out of memory\n");
        exit(1);
    }
    return p;
}
```

**`PCC_FREE(`**_auxil_**`,`**_ptr_**`)`**

The function macro to free the existing memory block.
The user-defined data passed to the API function `pcc_create()` can be retrieved from the argument _auxil_.
It can be ignored if no user-defined data.
The argument _ptr_ is the pointer to the previously allocated memory block.
This macro need not return a value.

The default is defined as below.

```C
#define PCC_FREE(auxil, ptr) free(ptr)
```

**`PCC_DEBUG(`**_auxil_**`,`**_event_**`,`**_rule_**`,`**_level_**`,`**_pos_**`,`**_buffer_**`,`**_length_**`)`**

The function macro for debugging (version 1.5.0 or later).
Sometimes, especially for complex parsers, it is useful to see how exactly the parser processes the input.
This macro is called on important *events* and allows to log or display the current state of the parser.
The argument `rule` is a string that contains the name of the currently evaluated rule.
The non-negative integer `level` specifies how deep in the rule hierarchy the parser currently is.
The argument `pos` holds the position from the start of the current context in bytes.
In case of `event == PCC_DBG_MATCH`, the argument `buffer` holds the matched input and `length` is its size.
For other events, `buffer` and `length` indicate a part of the currently loaded input, which is used to evaluate the current rule.

**Caution:** Since version 1.6.0, the first argument _auxil_ is added to this macro.
The user-defined data passed to the API function `pcc_create()` can be retrieved from this argument.

There are currently three supported events:
 - `PCC_DBG_EVALUATE` (= 0) - called when the parser starts to evaluate `rule`
 - `PCC_DBG_MATCH` (= 1) - called when `rule` is matched, at which point buffer holds entire matched string
 - `PCC_DBG_NOMATCH` (= 2) - called when the parser determines that the input does not match currently evaluated `rule`

A very simple implementation could look like this:

```C
static const char *dbg_str[] = { "Evaluating rule", "Matched rule", "Abandoning rule" };
#define PCC_DEBUG(auxil, event, rule, level, pos, buffer, length) \
    fprintf(stderr, "%*s%s %s @%zu [%.*s]\n", (int)((level) * 2), "", dbg_str[event], rule, pos, (int)(length), buffer)
```

The default is to do nothing:

```C
#define PCC_DEBUG(auxil, event, rule, level, pos, buffer, length) ((void)0)
```

**`PCC_BUFFERSIZE`**

The initial size (the number of characters) of the text buffer.
The text buffer is expanded as needed.
The default is `256`.

**`PCC_ARRAYSIZE`**

The initial size (the number of elements) of the internal arrays other than the text buffer.
The arrays are expanded as needed.
The default is `2`.

### API ###

The parser API has only 3 simple functions below.

```C
pcc_context_t *pcc_create(void *auxil);
```

Creates a parser context.
This context needs to be passed to the functions below.
The `auxil` can be used to pass user-defined data to be bound to the context.
`NULL` can be specified if no user-defined data.

```C
int pcc_parse(pcc_context_t *ctx, int *ret);
```

Parses an input text (from standard input by default) and returns the result in `ret`.
The `ret` can be `NULL` if no output data is needed.
This function returns `0` if no text is left to be parsed, or a nonzero value otherwise.

```C
void pcc_destroy(pcc_context_t *ctx);
```

Destroys the parser context.
All resources allocated in the parser context are released.

The type of output data `ret` can be changed.
If you want change it to `char *`, specify `%value "char *"` in the PEG source.
The default is `int`.

The type of user-defined data `auxil` can be changed.
If you want change it to `long`, specify `%auxil "long"` in the PEG source.
The default is `void *`.

The prefix `pcc` can be changed.
If you want change it to `foo`, specify `%prefix "foo"` in the PEG source.
The default is `pcc`.

After the above settings, the API functions change like below.

```C
foo_context_t *foo_create(long auxil);
```

```C
int foo_parse(foo_context_t *ctx, char **ret);
```

```C
void foo_destroy(foo_context_t *ctx);
```

The typical usage of the API functions is shown below.

```C
int ret;
pcc_context_t *ctx = pcc_create(NULL);
while (pcc_parse(ctx, &ret));
pcc_destroy(ctx);
```

## Examples ##

### Desktop calculator ###

A simple example which provides interactive four arithmetic operations of integers is shown here.
Note that **left-recursive** grammar rules are defined in this example.

```
%prefix "calc"

%source {
#include <stdio.h>
#include <stdlib.h>
}

statement <- _ e:expression _ EOL { printf("answer=%d\n", e); }
           / ( !EOL . )* EOL      { printf("error\n"); }

expression <- e:term { $$ = e; }

term <- l:term _ '+' _ r:factor { $$ = l + r; }
      / l:term _ '-' _ r:factor { $$ = l - r; }
      / e:factor                { $$ = e; }

factor <- l:factor _ '*' _ r:unary { $$ = l * r; }
        / l:factor _ '/' _ r:unary { $$ = l / r; }
        / e:unary                  { $$ = e; }

unary <- '+' _ e:unary { $$ = +e; }
       / '-' _ e:unary { $$ = -e; }
       / e:primary     { $$ = e; }

primary <- < [0-9]+ >               { $$ = atoi($1); }
         / '(' _ e:expression _ ')' { $$ = e; }

_      <- [ \t]*
EOL    <- '\n' / '\r\n' / '\r' / ';'

%%
int main() {
    calc_context_t *ctx = calc_create(NULL);
    while (calc_parse(ctx, NULL));
    calc_destroy(ctx);
    return 0;
}
```

### AST builder for Tiny-C ###

You can find the more practical example in the directory [`examples/ast-tinyc`](examples/ast-tinyc).
It builds an AST (abstract syntax tree) from an input source file
written in [Tiny-C](http://www.iro.umontreal.ca/~felipe/IFT2030-Automne2002/Complements/tinyc.c) and dump the AST.
