# Python's Star Operator
###### A feature that _packs_ a punch (pun intended)

[Here](https://servicenow.zoom.us/rec/share/JlV3xDx_Z675T1o3O-29RtFZiYTlW_3pLmbg4y-CkUoqAzPVsL_4X3cIaL63LrzF.T7YnqNZUYom6xJAd?startTime=1686854001000) is a recording of me (Joseph) presenting this content.

## Introduction

You have probably seen the `*` operator in something like this:
```py
def f(*args, **kwargs):
    pass
```

I'll start somewhere else completely, and we'll build our understanding up to the function arguments.

The `*` operator is a.k.a.
* packing/unpacking
* spreading
* destructuring
* variadic arguments (for Python function arguments specifically)
* the `...` operator in JavaScript (I'll also give examples in JavaScript)

Different terms, different operators, sometimes implicit... but really one concept. I'll try to showcase it my own words, hopefully connecting some dots and giving you ideas on where to use it.

## Let's start simple

You have also probably seen two variables declared on a single line, like this:

<table>
<tr><th>Python</th><th>JavaScript</th></tr>

<tr>
<td>

```py
first, second = 1, 2
```
</td>
<td>

```js
[first, second] = [1, 2]
```
</td>
</tr>

<tr><td colspan="2">

It is particularly cool to switch the values of two variables, without the need of a third (temporary) variable:
</td></tr>

<tr>
<td>

```py
first, second = second, first
```
</td>
<td>

```js
[first, second] = [second, first]
```
</td>
</tr>

<tr><td colspan="2">

That hints at the fact there is some magic going on. Another clue is that if the number of items doesn't match on both sides of the `=` operator, you get an error like "too many values to unpack (expected 2)". That's it: _packing_ and _unpacking_. In Python, the comma operator implicitly instantiates a tuple. This is what happened here on the right side of the `=` operator. On the left side, we are _unpacking_ the tuple into two variables.

The JavaScript syntax that I used to get a similar result is a lot more explicit, with the array brackets.

So, we can generalize that assigning an iterable to a tuple of variables will implicitly _unpack_ the values into the respective variables. I say "assigning" in general, like for the `=` operator, but also for what happens for example to `k, v` in `for k, v in ...`, or for function arguments when you call a function.
</td></tr>
<tr>
<td>

```py
first, second = my_iterable
```
</td>
<td>

```js
[first, second] = my_array
```
</td>
</tr>

<tr><td colspan="2">

Another way to _unpack_ an iterable, is explicitly using the `*` operator (or `...` in JavaScript):
</td></tr>
<tr>
<td>

```py
my_tuple = first, *items
```
</td>
<td>

```js
my_array = [first, ...items]
```
</td>
</tr>

<tr><td colspan="2">

This is equivalent to enumerating all the items, but without having to know how many items there are.
</td></tr>
<tr>
<td>

```py
# pseudocode
my_tuple = first, items[0], items[1], ..., items[n]
```
</td>
<td>

```js
// pseudocode
my_array = [first, items[0], items[1], ..., items[n]]
```
</td>
</tr>

<tr><td colspan="2">

And this syntax is symmetric!

Earlier, I mentioned that the number of items on both sides has to match, otherwise there are "too many values to unpack", but there are ways around this. You can re-_pack_ the rest of the items in a new tuple.
</td></tr>
<tr>
<td>

```py
first, *rest = my_iterable
```
</td>
<td>

```js
[first, ...rest] = my_array
```
</td>
</tr>

<tr><td colspan="2">

This is equivalent to enumerating all the "cells" where we want the items to go.
</td></tr>
<tr>
<td>

```py
# pseudocode
first, rest[0], rest[1], ..., rest[n] = my_iterable
```
</td>
<td>

```js
// pseudocode
[first, rest[0], rest[1], ..., rest[n]] = my_array
```
</td>
</tr>

<tr><td colspan="2">

In Python, you can also ignore the rest with the variable `_` (by convention).

JavaScript doesn't complain if there are "too many values to unpack", so it's not even necessary to save the rest in a variable.
</td></tr>
<tr>
<td>

```py
first, *_ = my_iterable
```
</td>
<td>

```js
[first] = my_array
```
</td>
</tr>

<tr><td colspan="2">

Obviously, you don't need that fancy syntax to access the first element of an iterable, but it comes in handy in more complex situations like follows. I find JavaScript remarkably straightforward here.
</td></tr>
<tr>
<td>

```py
first, _, third, *_ = my_iterable
```
</td>
<td>

```js
[first, , third] = my_array
```
</td>
</tr>

<tr><td colspan="2">

You get an error if you write "multiple starred expressions in assignment". For example, in `*_, a, *_ = my_iterable`, Python wouldn't know which item to assign to `a`. But, as long you have a single `*`, Python accepts it anywhere; at the beginning, at the end, or in the middle.
</td></tr>
<tr>
<td>

```py
first, *_, second_last, last = my_iterable
```
</td>
<td>

😢
</td>
</tr>

<tr><td colspan="2">

Unfortunately, in JavaScript, the "Rest element must be last element". It can't be anywhere else. To work around that and _unpack_ the end of the array, you can slice it first, like in Python.
</td></tr>
<tr>
<td>

```py
second_last, last = my_iterable[-2:]
```
</td>
<td>

```js
[second_last, last] = my_array.slice(-2)
```
</td>
</tr>

<tr><td colspan="2">

## Back to function arguments!

Positional arguments work just like in a tuple.
</td></tr>
<tr>
<td>

```py
def f(first, *args):
    pass
```
</td>
<td>

```js
function f(first, ...args) {
}
```
</td>
</tr>

<tr><td colspan="2">

This can be viewed as _packing_ all the remaining arguments in one `args` tuple/array.
</td></tr>
<tr>
<td>

```py
# pseudocode
def f(first, args[0], args[1], ..., args[n]):
    pass
```
</td>
<td>

```js
// pseudocode
function f(first, args[0], args[1], ..., args[n]) {
}
```
</td>
</tr>

<tr><td colspan="2">

This is still symmetric, so you can also _unpack_ arguments, just like instantiating a tuple.
</td></tr>
<tr>
<td>

```py
f(first, *args)
```
</td>
<td>

```js
f(first, ...args)
```
</td>
</tr>

<tr><td colspan="2">

## Double up!

In Python, the `**` operator is very similar to the `*` operator, but for dictionaries instead of tuples. In fact, JavaScript uses the same `...` operator for both objects and arrays.

This can be used to _unpack_ a dictionary/object into a new one.
</td></tr>
<tr>
<td>

```py
my_dict = { "foo": 42, "bar": 5, **another_dict }
```
</td>
<td>

```js
my_object = { foo: 42, bar: 5, ...another_object }
```
</td>
</tr>

<tr><td colspan="2">

Only JavaScript also supports _unpacking_ objects into multiple variables (on the left side of the assignment). That's awesome!
</td></tr>
<tr>
<td>

😭
</td>
<td>

```js
{ foo, bar, ...rest } = my_object

// Equivalent to
foo = my_object.foo
bar = my_object.bar
// pseudocode
rest = {
    key0: my_object.key0,
    key1: my_object.key1,
    ...,
    keyN: my_object.keyN
}
```
</td>
</tr>

<tr><td colspan="2">

Python has a special syntax for functions' keyword arguments (a.k.a. named arguments).
</td></tr>
<tr>
<td>

```py
f(foo=42, bar=5)
```
</td>
<td>

🤔
</td>
</tr>

<tr><td colspan="2">

You can think of this as passing a single dictionary of arguments to the function, for example `dict(foo=42, bar=5)`. In the function, you can then _unpack_ this dictionary, grab `foo`, and re-_pack_ the rest of the keyword arguments in `**kwargs`. That's exactly what I did here in Javascript to get the same result.
</td></tr>
<tr>
<td>

```py
def f(foo, **kwargs):
    print(foo, kwargs)


f(foo=42, bar=5)
# 42 {'bar': 5}
```
</td>
<td>

```js
function f({ foo, ...kwargs }) {
    console.log(foo, kwargs);
}

f({ foo: 42, bar: 5 })
// 42 {'bar': 5}
```
</td>
</tr>

<tr><td colspan="2">

And of course positional arguments and keyword arguments can be combined.
</td></tr>
<tr>
<td>

```py
def f(first, *args, kwarg1, kwarg2, **kwargs):
    print(first, args, kwarg1, kwarg2, kwargs)


f(1, 2, 3, kwarg1=42, kwarg2=5, kwarg3=0)
# 1 (2, 3) 42 5 {'kwarg3': 0}
```
</td>
<td>

```js
function f({ kwarg1, kwarg2, ...kwargs }, first, ...args) {
    console.log(first, args, kwarg1, kwarg2, kwargs);
}

f({ kwarg1: 42, kwarg2: 5, kwarg3: 0 }, 1, 2, 3)
// 1 [2, 3] 42 5 {'kwarg3': 0}
```
</td>
</tr>
</table>

In Python, you pass positional arguments first, then the keyword arguments.

So, all arguments defined after the `*args` have to be passed as keyword arguments (those are called keyword-only arguments). That can be leveraged, for example, to avoid ambiguity between two arguments. Bear with me, I'll give an example in a moment.

If you don't need the `*args`, you could use
- `*_` like before to pack additional positional arguments into oblivion and ignore them; or
- `*` alone to forbid additional positional arguments. This is a special syntax specific to function arguments. You can think of it as packing additional positional arguments into... well, there is no variable to collect them, so it raises an exception.

For example, the following `assert_privileges()` function has two arguments of the same type, `actual` and `expected`, but the `>` operator used inside the function is not commutative, so it is important that the caller doesn't flip the arguments. Therefore, I used the `*` to force callers to specify `expected` as a keyword argument.
```py
def assert_privileges(actual: Privileges, *, expected: Privileges):
    if PRIVILEGES_DESCENDING_ORDER.index(actual) > PRIVILEGES_DESCENDING_ORDER.index(expected):
        raise HTTPException

# So that works:
assert_privileges(actual=privileges, expected=expected_privileges)
# Or at least that:
assert_privileges(privileges, expected=expected_privileges)

# But that doesn't work:
assert_privileges(privileges, expected_privileges)
# TypeError: assert_privileges() takes 1 positional argument but 2 were given
assert_privileges(expected_privileges, privileges)
# TypeError: assert_privileges() takes 1 positional argument but 2 were given
assert_privileges(expected_privileges)
# TypeError: assert_privileges() missing 1 required keyword-only argument: 'expected'
```

## Conclusion

The `*` operator is quite versatile, although quite simple once you grasp it. I hope that helped.

Next time you're extracting items from iterables, aim for the stars!
