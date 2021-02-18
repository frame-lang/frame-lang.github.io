---
layout: post
title:  "Frame Basics"
date:   2021-02-11 00:00:00 -0800
categories: language-basics
---


## Comments

Currently Frame only has single line comments (no multiline) which start with three dashes:

```
--- this is a comment
--- this is another one
```

## Whitespace separators

One important difference between Frame and other languages is the lack of any commas or semicolons as separators. Instead Frame relies on whitespace to delineate tokens:

```
--- lists ---
[x y]
[x:int y:string]
(a b c)
(d() e() f())

--- statements ---

a() b() c()
var x:int = 1
```

Unlike other languages where structured whitespace is similarly used (e.g. Python), Frame's use of whitespace is <i>unstructured</i>. Frame simply uses it to separate tokens apart and does not insist on any pattern of use.

The esthetic goal is to be as spare and clean as possible, but it may take some getting used to.

## Variables and Constants

Variable and constant declarations have the following format:

```
    <var | const> <name> : <type_opt> = <intializer>

    var x:int = 1
    const name = "Steve"
```

Variables can be modified after initialization and constants can not. Frame will transpile into the closest semantic equivalents in the target language.

The type is optional but the initializer is not.

If you transpile into a language that requires a type and you don't provide one, a token such as `<?>` is substituted. Conversely, if you add a type and transpile into a language that doesn't require one, the Framepiler ignores it.

## Actions (aka methods)

Frame calls methods that are internal to the system "Actions". All system actions must be declared in the  
`-actions-` "block" of a system, which must be placed after the `-interface-` and `-machine-` blocks, if they are present.

Action declarations have three parts and only the <action_name> is required:

```
    <action_name> [<parameters>] : <return_type>
```

 If there are no parameters, the parameter list brackets may be omitted.

 The same is true for the `: <return_type>` which can be omitted if the return type is the language "default".

Here are the four possible permutations:

```
#ActionsExample

    -actions-

    noParamsOrReturn
    hasParams [x:int y:string]
    emptyParams []
    hasParamsAndReturnValue [x:int y:string] : SomeType
    noParamsHasReturnValue : SomeType

##
```

Generates this code:

`C#`
{% highlight csharp %}
    public partial class ActionsExample {


        //===================== Actions Block ===================//

        protected virtual void noParamsOrReturn_do() { throw new NotImplementedException(); }
        protected virtual void hasParams_do(int x,string y) { throw new NotImplementedException(); }
        protected virtual void emptyParams_do() { throw new NotImplementedException(); }
        protected virtual SomeType hasParamsAndReturnValue_do(int x,string y) { throw new NotImplementedException(); }
        protected virtual SomeType noParamsHasReturnValue_do() { throw new NotImplementedException(); }

    }
{% endhighlight %}
