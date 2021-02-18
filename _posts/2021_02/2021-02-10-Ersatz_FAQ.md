---
layout: post
title:  "Ersatz FAQ"
date:   2021-02-10 00:00:00 -0800
categories: language-basics
---

While this site is gelling, I am keeping a FAQ post to log answers to questions.

## Comments

Currently Frame only has single line comments (no multiline) which start with three dashes:

```
--- this is a comment
--- this is another one
```

Be aware - comment output in code is currently basically broken.

I know, I know.

## Punctuation

I am doing an active experiment to see if I can keep commas and semicolons extinguished from the language. I'm not utterly dogmatic on it but I want to give it a good go and see if I can't repattern people to use their overwhelmingly favorite syntactic separator that they are processing just fine right **now** while reading this sentence - <i>whitespace</i>.

So you will have to learn to not separate lists of things with commas as in:

```
create_personnel_record("Mark" "Truluck" 99 'm' "ice cream")
```

We'll see.


## Using the Frampiler output

Currently the <a href="http://framepiler.frame-lang.org" target="_blank">Framepiler</a> does one thing - take a Frame system specification and emit a transpiled version of it in the target language.

To actually use this output without too much fuss it is recommeded to use this as a base class and create a subclass from it. In the subclass it should only be necessary to override the actions and put in your custom behavior there.

LMK if you need more support for your intended uses. Configurability is **very high** on the list of things to do.

## Types and Operations

Frame doesn't understand types at all, really. Instead the Framepiler just passes through whatever you indicate the type is. There is no validation that operations make sense based on type and instead relies on the target language to figure all that out. Which for now is as it should be as every language is its own universe of types, operations and meaning.

## String Concatenation

There is no operator for this yet.
