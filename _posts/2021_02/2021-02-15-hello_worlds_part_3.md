---
layout: post
title:  "Hello Worlds - Part 3"
date:   2021-02-15 00:00:00 -0800
categories: language-basics
---

In the last installment of our conversation with the World, the denouement concludes with a demonstration of how to implement a key feature of Statecharts called **hierarchical state machines** and the use of the **stop message**.

## The Stop Message

Formal computational theory defines that state machines should have one or more <i>stop states</i>. A stop state is like any other state except that it is considered "correct" if a machine ends in one and "incorrect" if it does not.

These semantics are often related to validation of languages or other streams of activity. In the more mundane activity of developing an application, we often just want to turn the system off no matter what state it is currently in.

To do so in a standardized way, Frame defines the `<<` message token to be a reserved message meaning "stop". However, system designers are free to ignore this convention as nothing bad will happen if other means are used to stop the system. However, its adoption allows Frame tools to recognize the semantic intent and utilize for other automated capabilities.

Here we see its use in our `$Working` state and how it drives our system to a proper `$End` state:

```

  $Working
    |>|
      print("Hello World") ^
    |<<|
      -> $End ^

  $End
    |>|
      print("End of the World") ^
```

Here is the next World in all its manifestations:

```
#World_v11

  -interface-

  start @(|>>|)
  stop @(|<<|)

  -machine-

  $Begin
    |>>|
      -> $Working ^

  $Working
    |>|
      print("Hello World") ^
    |<<|
      -> $End ^

  $End
    |>|
      print("End of the World") ^

  -actions-

  print [msg:string]

##
```

Here is the `C#` generated from this system definition:

{% highlight csharp %}
public partial class System_v11 {

    public World_v11() {

        _state_ = _sBegin_;
    }

    //===================== Interface Block ===================//

    public void start() {
        FrameEvent e = new FrameEvent(">>",null);
        _state_(e);
    }

    public void stop() {
        FrameEvent e = new FrameEvent("<<",null);
        _state_(e);
    }    

    //===================== Machine Block ===================//

    private void _sBegin_(FrameEvent e) {
        if (e.Msg.Equals(">>")) {
            _transition_(_sWorking_);
            return;
        }
    }

    private void _sWorking_(FrameEvent e) {
        if (e.Msg.Equals(">")) {
            print_do("Hello World");
            return;
        }
        else if (e.Msg.Equals("<<")) {
            _transition_(_sEnd_);
            return;
        }
    }

    private void _sEnd_(FrameEvent e) {
        if (e.Msg.Equals(">")) {
            print_do("End of the World");
            return;
        }
    }

    //===================== Actions Block ===================//

    public virtual void print_do(string msg) {}

    //=========== Machinery and Mechanisms ===========//

    private delegate void FrameState(FrameEvent e);
    private FrameState _state_;

    private void _transition_(FrameState newState) {
        FrameEvent exitEvent = new FrameEvent("<",null);
        _state_(exitEvent);
        _state_ = newState;
        FrameEvent enterEvent = new FrameEvent(">",null);
        _state_(enterEvent);
    }    
}
{% endhighlight %}

And the equivalent UML statechart:

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8Ld5BJC_CKghbgeVpm_ABipBnq917Nl1GmBrehLa5NrmwYWmjCWlXm7LOAQig6HYRMTdOGcWig0L84CWIkmCO6gi0XDIy5w180)

## Hierarchical State Machines (HSMs)

The pinnacle of Statechart mods to the standard state machine is the idea of factoring common behavior between two or more states into a parent state. This is very similar to object-oriented inheritance but is strictly related to behavior and not data.

UML implements this concept visually:

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8L71MgkMgXR2SajZEOpUMeeAjh1-HOAQWf6ngPMAT2A2ud7E8EgNafGCC1)

In Frame notation that system looks like this:

```
#FlatMachine

    -machine-

    $A
        |e| -> $C ^

    $B
        |e| -> $C ^

    $C

##
```

As we can see, there is redundant behavior between states `$A` and `$B`.  **Hierarchical State Machines (HSMs)** allow system designers to factor out common behavior into a parent state using the "dispatch" symbol `=>`:

```
#HSM

    -machine-

    $A => $Parent

    $B => $Parent

    $C

    $Parent
        |e| -> $C ^    

##
```

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8Ld1MgkMgXx834ejIy4g200X10X1oXl5eaCIUO650Z5rIFhguTq2Wh1JLbGoCJwrG8nUMGcfS2j0e0)

Thanks to the approach Frame takes with implementing state machines, the mechanism for coding HSM functionality is trivial:


{% highlight csharp %}

    //===================== Machine Block ===================//

    private void _sA_(FrameEvent e) {
        _sParent_(e);
    }

    private void _sB_(FrameEvent e) {
        _sParent_(e);
    }

    private void _sParent_(FrameEvent e) {
        if (e.Msg.Equals("e")) {
            _transition_(_sC_);
            return;
        }
    }

    private void _sC_(FrameEvent e) {
    }
{% endhighlight %}

Let's apply this to the final incarnation of our World.

## Getting in the Final World

And now, the magnum opus of our methodical exploration of  "Hello World" in Frame notation:

```
#FinalWorld

    -interface-

    start @(|>>|)
    stop @(|<<|)
    tick [source:Object elapsedEvent:ElapsedEventArgs]

    -machine-

    $Begin
        |>>|
            print("In the Beginning...")
            -> $Working ^

    $Working => $Default
        |>|
            startTimer(500)
            print("Hello World") ^
        |<|
            stopTimer()
            print("Done working") ^
        |tick|
            -> $Resting ^

    $Resting => $Default
        |>|
            startTimer(1000)
            print("I'm resting") ^
        |<|
            stopTimer()
            print("Done resting") ^
        |tick|
            -> $Working ^

    $Default
        |<<|
            -> $End ^

    $End
        |>|
            print("End of the World") ^

    -actions-

    print [msg:string]
    startTimer [interval:int]
    stopTimer  [interval:int]

    -domain-

    var timer:Timer = null

##
```

Let us explore each of the states to review what we have learned so far.

The `$Begin` state utilizes the "start system" message `>>` symbol to kick things into gear and then transitions to the `$Working` state to go do stuff.

```
$Begin
    |>>|
        print("In the Beginning...")
        -> $Working ^
```

The `$Working` state exercises the enter `>` and exit `<` messages by managing a timer with a custom interval.  Upon timeout, the event hander for the `tick` message transitions the system to the `$Resting` state for a well deserved break.

```
$Working => $Default
    |>|
        startTimer(500)
        print("Hello World") ^
    |<|
        stopTimer()
        print("Done working") ^
    |tick|
        -> $Resting ^
```

The `$Resting` state is functionally equivalent to the `$Working` state and just cycles back to `$Working` after a long nap.

```
$Resting => $Default
    |>|
        startTimer(1000)
        print("I'm resting") ^
    |<|
        stopTimer()
        print("Done resting") ^
    |tick|
        -> $Working ^
```

Both `$Working` and `$Resting` factor out the code to handle the stop system message `<<` to the parent `$Default` state. This event handler simply transitions the system to the `$End` state.

```
$Default
    |<<|
        -> $End ^
```

In the `$End` the World says its goodbyes and goes quietly off into `/dev/null`.

```
$End
    |>|
        print("End of the World") ^
```

Here is the Framepiler generated UML documentation for the `#FinalWorld`:

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8Ld5BJC_CKghbgeNoNrBJ4qfmIe8W24434mlEBiZFpqg5YjKWoGQd59KWoS5DSyrB0PaPhnIhewjf1RE42ao0-t4Gh1JLbGoCJQpix2Cq5bG0fWXgEK5IIcPmDLGQLWfk5GndKCo1b82V1bTZOG1KufEQb0CC20000)


And at last, here is the final `C#` output and its running code (with a hand coded `print()` action):


<iframe width="100%" height="475" src="https://dotnetfiddle.net/Widget/75tD78" frameborder="0"></iframe>

## Conclusion

This concludes our exploration of advanced approaches to the classic "Hello World" program for new languages.

By making full use of both the stop message `<<` as well as our HSM mechanisms we have seens what may well be the most embellished "Hello World" ever.

In the spirit if not the brevity of the original "Hello World", not enough information has been presented yet to really do much.

In the next article we will explore syntax for conditional branching that will provide the bare minimum architects and developers need to start to utilize Frame notation for system design and programming.
