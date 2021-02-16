---
layout: post
title:  "Hello Worlds - Part 2"
date:   2021-02-13 00:00:01 -0800
categories: language-basics
---

# Stateful machines

In the [last article]({% post_url /2021_02/2021-02-12-hello_worlds_part_1 %}) we successfully had a Frame system say "Hello World". In this one we will guild the lily a bit and explore some of the core Frame syntax for implementing basic state machines.

## Event Message Aliasing

By default the Frame syntax for interface methods sends an event with a message that is the same name as the interface method:

```
#World_v6

  -interface-

  speak

  ...

##
```

The Frame notation above generates the following `C#` code:

{% highlight csharp %}
//===================== Interface Block ===================//

public void speak() {
    FrameEvent e = new FrameEvent("speak",null);
    _state_(e);
}
{% endhighlight %}

Frame **event message aliases** provide a way to send a different message into the state machine:

```
#World_v7

  -interface-

  speak @(|parler|)

  -machine-

  $Begin
    |parler|
      print("Coucou le monde") ^

  -actions-

  print [msg:string]

##
```

{% highlight csharp %}
    public partial class World_v7 {

        ...

        //===================== Interface Block ===================//

        public void speak() {
            FrameEvent e = new FrameEvent("parler",null);
            _state_(e);
        }    

        //===================== Machine Block ===================//

        private void _sBegin_(FrameEvent e) {
            if (e.Msg.Equals("parler")) {
                print_do("Coucou le monde");
                return;
            }
        }

        ...
    }
{% endhighlight %}

Here we see that the interface method `speak` now sends `parler` instead of the default `speak` message.

Aliasing allows us to decouple the interface from the details of the event handling in the state machine. This feature enables Frame to support a few symbolic messages that are reserved for certain standard semantic operations:

| Message | Meaning |
| > | Enter State |
| < | Exit State |
| \>\> | Start System |
| \<\< | Stop System |
| \>\>\> | Save System |
| \<\<\< | Restore System |

Let's convert our World to use this new ability:

```
#World_v8

  -interface-

  start @(|>>|)

  -machine-

  $Begin
    |>>|
      print("Hello World") ^

  -actions-

  print [msg:string]

##
```
Now we are able to send the standard Frame `start` message `>>` to trigger our greeting:

{% highlight csharp %}
    public partial class World_v8 {

        ...

        //===================== Interface Block ===================//

        public void start() {
            FrameEvent e = new FrameEvent(">>",null);
            _state_(e);
        }

        //===================== Machine Block ===================//

        private void _sBegin_(FrameEvent e) {
            if (e.Msg.Equals(">>")) {
                print_do("Hello World");
                return;
            }
        }

        ...
    }
{% endhighlight %}

Cool. However, we don't want to be a one state pony. Instead we are going to stretch our boundaries and introduce a new `$Working` state to do the heavy lifting of printing our greeting.

```
#World_v9

  -interface-

  start @(|>>|)

  -machine-

  $Begin
    |>>| ^

  $Working
    |>|
      print("Hello World") ^

  -actions-

  print [msg:string]

##
```

In the `$Working` state we see the use of the new enter message `>` that calls the `print()` action now.

{% highlight csharp %}
    private void _sWorking_(FrameEvent e) {
        if (e.Msg.Equals(">")) {
            print_do("Hello World");
            return;
        }
    }
{% endhighlight %}

This is the same event handler pattern we have seen before but there is a big mystery here - where does the enter message come from? So far we have only seen messages on events created by the interface. What's worse is there is the major problem of not knowing how to get to the `$Working` state in the first place.

The answer to both questions is the inner workings of **the transition**.

## Transitions

Central to state machine theory is the idea of a <i>transition</i> from one state to another. Frame notates transitions with the `->` symbol. So to get from `$Begin` to `$Working` we can do this:

```
    $Begin
      |>>|
        -> $Working ^
```

This notation generates this code:
{% highlight csharp %}
    private void _sBegin_(FrameEvent e) {
        if (e.Msg.Equals(">>")) {
            _transition_(_sWorking_);
            return;
        }
    }
{% endhighlight %}

Transitions do three core activities:

- Send an exit event to the current state
- Change the `_state_` variable to the new state
- Send an enter event to the new state

Here is the code for the `_transition_()` method:

{% highlight csharp %}
    private void _transition_(FrameState newState) {
        FrameEvent exitEvent = new FrameEvent("<",null);
        _state_(exitEvent);
        _state_ = newState;
        FrameEvent enterEvent = new FrameEvent(">",null);
        _state_(enterEvent);
    }
{% endhighlight %}

Enter and exit events are, essentially, constructors and destructors for states and enable powerful mechanisms for managing resources. For instance the following code is a template for how to use a timer properly to do an activity report:

```
#TimeStudy

  -machine-

  $Working
    |>|
      startTimer() ^
    |<|
      stopTimer()
      generateTimeStudy() ^

##
```
You can try it yourself in the <a href='http://frame-lang.org' target='_blank'>Framepiler</a>.

We now have all the tools to understand how "Hello World" can be emitted from a true state machine:

```
---------------------------------------

#World_v10

  -interface-

  start @(|>>|)

  -machine-

  $Begin
    |>>|
      -> $Working ^

  $Working
    |>|
      print("Hello World") ^

  -actions-

  print [msg:string]

##
```

Here is what it looks like in `C#`:

{% highlight csharp %}
    public partial class World_v10 {

        public World_v10() {

            _state_ = _sBegin_;
        }

        //===================== Interface Block ===================//

        public void start() {
            FrameEvent e = new FrameEvent(">>",null);
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

You can see a working demo of this on <a href="https://dotnetfiddle.net/SqOwD7" target="_blank">.NET fiddle</a>. (NOTE: In the online version I have added code to the `print()` action to actually write the greeting. More to come on how to use Frame state machines in real world development later).

## Conclusion

We have now seen a couple of ways to implement "Hello World" which have allowed us to explore some of Frame's core capabilities.

In the [next and final installment]({% post_url /2021_02/2021-02-13-hello_worlds_part_3 %}) of our "Hello World" saga, we will dive deep into the power of **hierarchical state machines** as well as bring the world to a tidy `$End` with **stop** events using Frame notation.
