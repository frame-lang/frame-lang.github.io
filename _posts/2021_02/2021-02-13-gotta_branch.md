---
layout: post
title:  "Gotta Branch"
date:   2021-02-13 00:00:03 -0800
categories: language-basics
---
# Overview of Frame Control Flow Mechanisms

Frame is an evolving language and the current version (v0.3.23 at this <i>instant</i>) only supports branching syntax of various flavors. At the moment you can't loop (no **for**, **while** or any other way to spin around), but that doesn't really matter for the problem domain Frame is intended to address.

The two flavors of branching are a boolean if-then-else syntax and a similar pattern matching syntax, both loosely inspired by the [ternary operator](https://en.wikipedia.org/wiki/%3F:) which I always thought came from `C` but apparently was stolen from `CPL` (one of the many languages I've never programmed in).

The basic syntax for both classes of test are:

```
    x ?<type> <branches> : <else clause> ::
```

The `:` token is "else" and `::` terminates the statement for all branching statement types.

Let's explore the boolean test first.

## Boolean Tests

The basic boolean test in Frame is:

```
    x ? callIfTrue() : callIfFalse() ::
```
This generates this in `C#`:
```
    if (x) {
        callIfTrue_do();
    } else {
        callIfFalse_do();
    }
```

To reinforce the point that branching in Frame is not an expression evaluation, see how we can call multiple statements inside each branch:

`Frame`
```
x ?
    a()
    b()
:
    c()
    d()
::
```
`C#`
```
if (x) {
    a_do();
    b_do();
} else {
    c_do();
    d_do();
}
```


To negate the test use the `?!` operator:

`Frame`
```
x ?! callIfFalse() : callIfTrue() ::
```
`C#`

```
if (!(x)) {
    callIfFalse_do();
} else {
    callIfTrue_do();
}
```

Next we will explore the Frame equivalent of the switch statement for string matching.

## String Matching

```
name() ?~
    /Alice/ hiAlice()   :>
    /Bob/   hiBob()     :
            whoAreYou() ::
^
```

```
if (name_do() == "Alice") {
    hiAlice_do();
} else if (name_do() == "Bob") {
    hiBob_do();
} else {
    whoAreYou_do();
}
```

Frame has two main control flow
Frame has a control flow syntax inspired by the ternary operator in the `C` language. The important difference is that in `C` and other languages the ternary operator evaluates to an expression whereas (for now) Frame uses it to branch statements.

```
    x = a ? b : c;
```
which is equivalent to:

```
    if (a) {
        x = b;
    } else {
        x = c;
    }
```


The important aspect is that above `b` and `c` are expressions.


```
    x = a ? <expression if true> : <expression if false>;
```

In contrast the `?` operator in Frame dictates the control flow for branching <i>statements</i>:

```
    test ? <statements if true> : <statements if false> ::
```

```
    foo ? doIfFooTrue() : doIfNotFooTrue() ::
```
