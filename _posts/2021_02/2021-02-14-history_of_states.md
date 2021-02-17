---
layout: post
title:  "The History of States (there won't be a quiz) "
date:   2021-02-14 00:00:00 -0800
categories: language-basics
---
# State History Mechanism

The statechart was invented by Dr. David Harel in his 1987 paper, which proposed a number of additional features to pure state machines as well as visual formalisms for expressing them. One of these important new ideas was the "history" mechanism for returning to the previous state.

## The Problem

State machines are an inherently limited kind of system in that they have no memory except what you build into the states themselves.  Consider this system:

`Frame`
```
#History101

  -machine-

    $A
        |e1| -> $C ^

    $B
        |e1| -> $C ^

    $C
        |return| ^

##
```

![The Problem](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8L71MgkMgXR2SajZEOxQYWgsi7P5ifg2aR6fbOfnf2Q2udN18EgNafGDC1)

Here we see that `$C` has no way to know what state preceded it. To solve this problem for a <i>pure</i> state machine we would have to do something like this:

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8L71MgkMgXR2SajdCYCYS9p75KqDMr0ybOAQWf6ngPMASQGcWk9uXC4gQCSo9OoX4kKvHQKbgK1vDD0iiwOPTrICrB0ReK0000)

`$Ca` and `$Cb` would be identical except for the response to the `|return|` message. This is obviously inefficient.

## The Solution

Automata theory holds that there are are three levels of increasing complexity and capability for abstract machines:

1. Finite State Machines
2. Pushdown Automata
3. Turing Machines

Pushdown Automata and Turning Machines share the trait of being able to store information for future use. Pushdown Automata specifically use a stack for storing history while Turning Machines theoretically have a "tape" to store information on. In reality if a system can store off data and access it later to make a decision it is effectively a Turing Machine.

For our problem with remembering the last state, a stack will do nicely thus giving us the power of a Pushdown Automata. To support this, Frame has two special operators:

|Operator|Name|
|$$[+]|State Stack Push|
|$$[-]|State Stack Pop|

Let's see how these are used:

`Frame`
```
#History201

  -machine-

    $A
        |e1| $$[+] -> "$$[+]" $C ^

    $B
        |e1| $$[+] -> "$$[+]" $C ^

    $C
        |return| -> "$$[-]" $$[-] ^

##
```

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuU82iafI5HmLghbgeMmd9BOpcEseeAjh1sHRAQYeH6l7SZcXyPt1_6WFhLY8a6uibqDgNWhGV000)

What we see above is that the state stack push token precedes a transition to a new state:

```
$$[+] -> $NewState
```

while the state stack pop operator produces the state to be transitioned into:

```
-> $$[-]
```


 Recalling that `FrameState` is a delegate typedef in C# to allow pointers to methods, we can see that Frame generates a `_stateStack_` variable which is initialized to a `Stack<FrameState>()` data structure.

 `C#`
 {% highlight csharp %}
     //=========== Machinery and Mechanisms ===========//

     ...

     private Stack<FrameState> _stateStack_ = new Stack<FrameState>();

     private void _stateStack_push(FrameState state) {
         _stateStack_.Push(stateContext);
     }

     private FrameState _stateStack_pop() {
         FrameState state =  _stateStack_.back();
         return _stateStack_.Pop();
     }     
 {% endhighlight %}

 Also generated are the `push` and `pop` functions for the state stack operations.

`Frame`
 ```
 #History202

  -interface-

  gotoA
  gotoB
  gotoC
  goBack

  -machine-

    $Waiting
        |>| print("In $Waiting") ^
        |gotoA| print("|gotoA|") -> $A ^
        |gotoB| print("|gotoB|") -> $B ^

    $A
        |>| print("In $A") ^
        |gotoB| print("|gotoB|") -> $B ^
        |gotoC| print("|gotoC|") $$[+] -> "$$[+]" $C ^

    $B
        |>| print("In $B") ^
        |gotoA| print("|gotoA|") -> $A ^
        |gotoC| print("|gotoC|") $$[+] -> "$$[+]" $C ^

    $C
        |>| print("In $C") ^
        |goBack| print("|goBack|") -> "$$[-]" $$[-] ^

    -actions-

    print [msg:string]

##
```

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuU82iafI5S8JCqioyz8LghbgeIAEI6md9BOpc1sj5QkWgsi7qyS5fK5YG9rM2chAXaOcrkdv9VcE42QA2YSK5KvG5Ou4vPo1SYegqTgnN4vuR792K-iCvaTxQCL2X7HZkHnIyrB0lkS20000)

 <iframe width="100%" height="475" src="https://dotnetfiddle.net/Widget/aofLnO" frameborder="0"></iframe>

## Refactoring Common Behavior

 In the sprit of staying <a href="https://en.wikipedia.org/wiki/Don%27t_repeat_yourself" target="_blank">DRY</a> let's refactor out the common transition shared by `$A` and `$B` into a parent state `$AB`:

 ```
 #_History203_

    -interface-

    gotoA
    gotoB
    gotoC
    goBack

    -machine-

    $Waiting
        |>| print("In $Waiting") ^
        |gotoA| print("|gotoA|") -> $A ^
        |gotoB| print("|gotoB|") -> $B ^

    $A => $AB
        |>| print("In $A") ^
        |gotoB| print("|gotoB|") -> $B ^

    $B => $AB
        |>| print("In $B") ^
        |gotoA| print("|gotoA|") -> $A ^

    $AB
        |gotoC| print("|gotoC| in $AB") $$[+] -> "$$[+]" $C ^

    $C
        |>| print("In $C") ^
        |goBack| print("|goBack|") -> "$$[-]" $$[-] ^

    -actions-

    print [msg:string]

 ##
```

We can see that the duplicated `|gotoC|` event handler is now moved into `$AB` and both `$A` and `$B` inherit behavior from it.

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuU82iafI5S8JCqioyz8LghbgeIAEJa2E0X10kL1UBPAO4qmChiaPR42qLgo2hguTp50kA0qMSrImKb1JDZGoiKxFBybtX31HL3YXg722gd348-U4nsH7YAGpK5959LexbiiPp8_sq8g52Ed6SZcavgM0mW80)

Below we can see  that the system reports out it is transitioning from `$AB` now:

<iframe width="100%" height="475" src="https://dotnetfiddle.net/Widget/U1axyV" frameborder="0"></iframe>

NOTE: History203 demonstrates the recommended best practice of using a Frame specification to define a base class (in this case `_History203_`) and then derive a subclass to provide the implemented actions for behavior. 

## Conclusion

The History mechanism is one of the most valuable contributions of Statecharts to the evolution of the state machine formalism.

However, whereas Statecharts were declared to be a visual formalism <a href="https://www.sciencedirect.com/science/article/pii/0167642387900359" target="_blank">(Harel, 1987)</a> Frame is intended to be a symbolic language. As such, the Frame notation will favor a terse but (hopefully) clear symbology that is both clear and meaningful.
