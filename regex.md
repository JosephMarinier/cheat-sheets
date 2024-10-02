# Regular Expression

## Introduction

A regular expression (a.k.a. regex) is a pattern used to match text. It's a very powerful and concise (but cryptic) language.

> [!NOTE]
> Different programming languages and regex libraries have different flavors of regex, with slight variations in syntax and features. This document focuses on Python and on the common regex features that are consistent between languages.

Without further ado, let's jump right in with examples.

## Exact match

Say we have the following string of text.

```
Words should all match.
The pattern should recognise that swordfish is a different word, only it has _word_ at the centre.
```

The simplest example of a regex is a simple string, with no special characters, requiring an exact match.

```py
pattern = "word"
```

Using a regex for exact match is probably overkill, but it's a good place to start building our understanding. We can perform a number of different operations with a regex, but with a simple exact-match example like that, we could perform the same operations without a regex, simply using Python's `str` standard library. Here is how those operations map.

Operation                                | Using `str`                    | Using `re`
-----------------------------------------|--------------------------------|---------------------------------------
Test if a string matches the pattern     | `string == pattern`            | `re.fullmatch(pattern, string)`¹
Test if a string starts with the pattern | `string.startswith(pattern)`   | `re.match(pattern, string)`¹
Test if a string contains the pattern    | `pattern in string`            | `re.search(pattern, string)`¹
Find where the pattern is in a string    | `string.index(pattern)`        | `re.search(pattern, string).span()[0]`
Split a string on the matches            | `string.split(pattern)`        | `re.split(pattern, string)`
Replace the matches in a string          | `string.replace(pattern, new)` | `re.sub(pattern, new, string)`
Find all matches in a string             | `string.count(pattern)`        | `re.findall(pattern, string)`² or<br/>`re.finditer(pattern, string)`

1. The regex returns a [`Match`](https://docs.python.org/3/library/re.html#re.Match) object rather than a `bool`. We can check `match is not None` if we are only interested in whether it matched, or we can use `match.group()` to get the string that matched, and more.
2. The regex returns a list (or an iterable) with the matches. We can use `len(matches)` if we are only interested in the count, or we can use the matches.

> [!TIP]
> If a pattern will be used multiple times, it is best to "compile" it once to improve performance.
> 
> ```py
> COMPILED_PATTERN = re.compile(pattern)
> 
> 
> def function_called_multiple_times(string):
>     match = COMPILED_PATTERN.fullmatch(string)
>     # ...
> ```

But, the power of regex is to define more flexible patterns (rather than exact match)...

## Disjunction (vertical bar)

For example, let's match the word "center" (US), as well as "centre" (UK).

A vertical bar is a boolean "or".

```py
re.compile(r"center|centre")
```

## Groups (parentheses)

We can use parentheses to define precedence of the operators (among other uses).

```py
re.compile(r"cent(er|re)")
```

These parentheses are called "groups" and they play different roles.

Simple `(...)` groups are "capturing", meaning the `Match.group(1)` will hold whatever the group has matched in the string (more on that later). If we don't need that, it is good practice to use a non-capturing group `(?:...)` to avoid that extra computation time and space.

```py
re.compile(r"cent(?:er|re)")
```

## Character class, a.k.a. set (square brackets)

Let's match the word "recognize" (US), as well as "recognise" (UK). There is a special syntax for a single-character "or", using a set of characters inside square brackets.

```py
re.compile(r"recogni[sz]e")
```

The character class syntax also allows for ranges of characters, using a dash. For example, this would match any lowercase letter.

```py
re.compile(r"[a-z]")
```

There can be multiple ranges inside the brackets. For example, this will match an hexadecimal character.

```py
re.compile(r"[0-9A-Fa-f]")
```

We can even mix ranges and single characters inside the brackets. For example, this will match any character that is valid in a variable name.

```py
re.compile(r"[0-9A-Za-z_]")
```

## Character class escape

The example above is known to as a "word" character. It is often useful, so there is a shortcut for it: `\w`.

```py
re.compile(r"\w")
```

> [!WARNING]
> In Python, `\w` also matches letters with accents, so it's not strictly equivalent to `[0-9A-Za-z_]`.

Similar shortcuts include `\d` for digits and `\s` for whitespaces of all kinds (space, tab, newline, non-breaking space, etc.).

> [!TIP]
> In Python, we usually define regex patterns with a raw string literal `r"..."` so that `\` is interpreted as a literal `\`, and not an escape sequence like in `\n`.

## Complement character class, a.k.a. negative set

We can invert the brackets syntax with a `^` at the start and we can invert the shortcuts by using a capital letter instead. For example, `[^0-9]` and `\D` will both match any character that is not a digit.

## Wildcard

There is also a wildcard that matches any character except the newline character `\n`.

```py
re.compile(r".")
```

The `re.DOTALL` flag (or the `s` inline flag for "*s*ingle-line") makes the wildcard `.` match even the newline character. We can apply the flag to the whole regex or to a specific group (inline flag).

```py
re.compile(r".", re.DOTALL)
re.compile(r"(?s:.)")
```

# Escape sequence for metacharacters

The special characters used in regex patterns are called metacharacters. If you want to match a special character literally, instead of it being a metacharacter, you must escape it with a `\`. For example, this will match a literal `.`, and nothing else.

```py
re.compile(r"\.")
```

> [!CAUTION]
> The `.` character is particularly error-prone since the wildcard `.` does match the literal character `.`, so you might not notice if you forget to escape it.

## Quantifiers

A quantifier specifies how many times an element is allowed to repeat. The following quantifiers are available.

Quantifier  | Signification
------------|---------------------------
`?`         | zero or one
`*`         | zero or more
`+`         | one or more
`{n}`       | exactly `n`
`{min,}`    | `min` or more
`{,max}`    | up to `max`
`{min,max}` | `min` or more, up to `max`

All quantifiers except `{n}` match a variable number of times. By default, these are "greedy", meaning they will match as many times as possible before backtracking until the rest of the pattern doesn't match.

If these quantifiers are followed by a `?`, they become "non-greedy", meaning they will match as few times as possible, stopping as soon as the rest of the pattern matches.

For example, let's try to find JSON objects.

```py
# non-greedy (doesn't work with nested objects):
re.findall(r"\{.*?\}", 'JSON with a nested object: {"a": {"b": 1}, "c": 2}')
# ['{"a": {"b": 1}']

# greedy (works for a single JSON object):
re.findall(r"\{.*\}", 'JSON with a nested object: {"a": {"b": 1}, "c": 2}')
# ['{"a": {"b": 1}, "c": 2}']

# greedy (limitation, doesn't work with multiple JSON objects):
re.findall(r"\{.*\}", 'Simple JSON: {"a": 1} and JSON with a nested object: {"a": {"b": 1}, "c": 2}')
# ['{"a": 1} and JSON with a nested object: {"a": {"b": 1}, "c": 2}']
```

> [!NOTE]
> See [Annex 1](#annex-1---json) for a robust JSON pattern, but also keep in mind that instead you should probably use a proper JSON decoder like `json.JSONDecoder().raw_decode(string, start)`. Regex is not always the answer.

Now, let's try to match spans of markdown text that are in italic, knowing there may be multiple spans within the text.

```py
# non-greedy (works with multiple italic spans):
re.findall(r"_.+?_", "The result of _2 * 3_ is _6_, not *5*.")
# ['_2 * 3_', '_6_']

# greedy (doesn't work with multiple italic spans):
re.findall(r"_.+_", "The result of _2 * 3_ is _6_, not *5*.")
# ['_2 * 3_ is _6_']
```

> [!TIP]
> In many cases, making a quantifier greedy or not won't impact the final match, but it may impact performance (one way or the other). A regex editor like [regex101.com](https://regex101.com) can help comparing the number "steps" needed to run different patterns on different inputs.

## Capturing groups, backreferences, and replacement

Capturing groups capture parts of a match. There are 3 uses:
1. Backreference a captured group within the regex itself
2. Get the captured group
3. Replace with a captured group

```py
RE_MD_ITALIC = re.compile(r"([_*])(.+?)\1")

[m.group(2) for m in RE_MD_ITALIC.finditer("The result of _2 * 3_ is _6_, not *5*.")]
# ['2 * 3', '6', '5']

RE_MD_ITALIC.sub(r"<i>\2</i>", "The result of _2 * 3_ is _6_, not *5*.")
# 'The result of <i>2 * 3</i> is <i>6</i>, not <i>5</i>.'
```

> [!NOTE]
> The syntax for replacing with a match or captured group varies between languages. For example, here is Python vs JavaScript.
> 
> Replacing with            | Python          | JavaScript
> --------------------------|-----------------|-----------
> Whole match               | `\g<0>`         | `$&`
> Nth group (for example 1) | `\g<1>` or `\1` | `$1`
> 
> ```js
> const RE_MD_ITALIC = /([_*])(.+?)\1/g
> 
> [..."The result of _2 * 3_ is _6_, not *5*.".matchAll(RE_MD_ITALIC).map((m) => m[2])]
> // ['2 * 3', '6', '5']
> 
> "The result of _2 * 3_ is _6_, not *5*.".replaceAll(RE_MD_ITALIC, "<i>$2</i>")
> // 'The result of <i>2 * 3</i> is <i>6</i>, not <i>5</i>.'
> ```

## Named capturing groups, backreferences, and replacement

The capturing groups can be named to make the code more readable.

```py
RE_MD_ITALIC = re.compile(r"(?P<italic>[_*])(?P<inside>.+?)(?P=italic)")

[m.group("inside") for m in RE_MD_ITALIC.finditer("The result of _2 * 3_ is _6_, not *5*.")]
# ['2 * 3', '6', '5']

RE_MD_ITALIC.sub(r"<i>\g<inside></i>", "The result of _2 * 3_ is _6_, not *5*.")
# 'The result of <i>2 * 3</i> is <i>6</i>, not <i>5</i>.'
```

> [!NOTE]
> The syntax for named capturing groups varies between languages. For example, here is Python vs JavaScript. :weary:
> 
> Operation       | Python        | JavaScript
> ----------------|---------------|------------
> Capturing group | `(?P<name>)`  | `(?<name>)`
> Backreference   | `(?P=name)`   | `\k<name>`
> Replacement     | `\g<name>`    | `$<name>`
> 
> ```js
> const RE_MD_ITALIC = /(?<italic>[_*])(?<inside>.+?)\k<italic>/g
> 
> [..."The result of _2 * 3_ is _6_, not *5*.".matchAll(RE_MD_ITALIC).map((m) => m.groups["inside"])]
> // ['2 * 3', '6', '5']
> 
> "The result of _2 * 3_ is _6_, not *5*.".replaceAll(RE_MD_ITALIC, "<i>$<inside></i>")
> // 'The result of <i>2 * 3</i> is <i>6</i>, not <i>5</i>.'
> ```

## Case sensitivity

Now, let's try to find all the occurrences of `word` in the text.

```py
re.compile(r"word")
```

This misses the first word since it is capitalized. By default, regular expressions are case sensitive. We could use a set again, like `[Ww]`, but there is another general solution:

The `re.IGNORECASE` flag (or the `i` inline flag for "*i*gnore case" or "case-*i*nsensitive") makes the regex (or a specific group) case-insensitive.

```py
re.compile(r"word", re.IGNORECASE)
re.compile(r"(?i:w)ord")
```

## Word boundaries

Let's avoid matching `swordfish` by adding word boundaries: `\b`. That matches an empty string between a word character and a non-word character.

```py
re.compile(r"\bword\b", re.IGNORECASE)
```

Because of the ending `\b`, we now lost the plural `Words`. Let's support the optional `s` with the quantifier `?`.

```py
re.compile(r"\bwords?\b", re.IGNORECASE)
```

We are also missing `_word_` since `_` is considered a word character (in the context of variable names). Let's be more precise.

```py
re.compile(r"[^a-z]words?[^a-z]", re.IGNORECASE)
```

## Input (or line) boundaries

We are now missing the first `Words` again. Being at the beginning of the text, it is not preceded by a non-letter character `[^a-z]`. We could use some `|`s to also match the beginning of the text `^`, and the end of the text `$`.

```py
re.compile(r"(?:^|[^a-z])words?(?:[^a-z]|$)", re.IGNORECASE)
```

By default, `^` and `$` match the beginning and the end of the whole input. With the `re.MULTILINE` flag (or the `m` inline flag), they also match the beginning and the end of each line, delimited by newline characters `\n`.

## Look-around assertions

That works, but we don't want those two groups in the final match. We could make do of a capturing group for the `(words?)` and retrieve what the group matches, but there is something better: look-around assertions. What they match doesn't en up in the final match.

Assertion | Look-behind | Look-ahead
----------|-------------|-----------
Positive  | `(?<=...)`  | `(?=...)`
Negative  | `(?<!...)`  | `(?!...)`

To adapt our regex pattern, we would like to use positive look-around assertions.

```py
re.compile(r"(?<=^|[^a-z])words?(?=[^a-z]|$)", re.IGNORECASE)
```

In Python, this pattern raises a `re.error: look-behind requires fixed-width pattern`. Indeed, `[^a-z]` has a width of 1 while `^` has a width of 0 (empty string). A solution would be to write the look-behind assertion within a `(?:...|...)`.

```py
re.compile(r"(?:^|(?<=[^a-z]))words?(?=[^a-z]|$)", re.IGNORECASE)
```

> [!TIP]
> An alternative Python library called `regex` can be installed. It does support variable-width look-behind assertions, as well as other cool advanced features and optimizations. Check it out at https://pypi.org/project/regex/.

However, there is a simpler solution. Instead of negating the character set with `^`, we can use negative look-around assertions by replacing the `=` signs with `!`.

```py
re.compile(r"(?<![a-z])words?(?![a-z])", re.IGNORECASE)
```

Success! :tada:

## Conclusion

As you can see, writing a regular expression is often an iterative process. It is normal to refactor regular expressions. After all, it's code! It may be intimidating at first, but with practice comes mastery. Give it try, iterate on it, and don't hesitate to invite others to review.

## References

- General syntax: https://en.wikipedia.org/wiki/Regular_expression
- Python native `re` library: https://docs.python.org/3/library/re.html
- Python alternative `regex` library: https://pypi.org/project/regex/
- Regex editor: https://regex101.com

## Annex 1 - JSON

Here is a regular expression that will match only valid JSON. It leverages the `re.VERBOSE` flag for clarity, as well as the more advanced `regex` library and its extra features of recursive patterns `(?R)` and `(?&name)`.

```py
import regex

RE_JSON = regex.compile(
    r"""
    (?P<whitespace>[ \n\r\t]*)
    (?:
        (?P<string>"(?:[^\x00-\x31"\\]|\\["\\/bfnrt]|\\u[0-9A-Fa-f]{4})*")
        |
        (?P<number>-?(?:0|[1-9]\d*)(?:\.\d+)?(?:[eE][+-]?\d+)?)
        |
        (?P<object>\{(?:(?&whitespace)|(?P<key_value>(?&whitespace)(?&string)(?&whitespace):(?R))(?:,(?&key_value))*)\})
        |
        (?P<array>\[(?:(?&whitespace)|(?R)(?:,(?R))*)\])
        |
        (?P<boolean>true|false)
        |
        (?P<null>null)
    )
    (?&whitespace)
    """,
    regex.VERBOSE,
)
```
