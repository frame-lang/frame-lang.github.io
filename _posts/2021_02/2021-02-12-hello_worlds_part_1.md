---
layout: post
title:  "Hello Worlds - Part 1"
date:   2021-02-12 00:00:00 -0800
categories: language-basics
---

# Getting Started With Frame Syntax

In the spirit of too much is never enough, this article is the first of a series that will exhaustively beat the traditional **Hello World!** trope to death. However it is for a good cause, and will go well beyond simply introducing a `main()` function and a `print("Bonjour le monde");` statement.

Frame is intended to be a **<i>system specification markdown language</i>** that can be used to generate both code as well as documentation. To see this in action, an online Frame language transpiler is available for your coding pleasure at <a href='http://framepiler.frame-lang.org' target='_blank'>Framepiler</a>. There you will be able to copy and paste the examples in this tutorial into the left column and see it generate both code and documentation.

With all that in mind, we will begin...

## In the Beginning

To define a **system object** in Frame we use the **`#`** symbol:

```       
#World
##
```

The **`##`** symbol ends the specification document. And although this simple system definition does not generate a <i>model</i> as documentation (it's too simple) it does generate some basic code. We will be looking at `C#` for this series of tutorials, but the <a href="http://framepiler.frame-lang.org" target="_blank">Framepiler</a> (aka Frame Transpiler) supports generating other languages as well (and will support many many more in the future!):

{% highlight csharp %}
public partial class World {
}
{% endhighlight %}

This isn't much of a World yet, but we'll fill it in and eventually say some things to it.

## Frame System Blocks

Frame (currently) specifies four <i>blocks</i> to define the key aspects of a system:

- Interface
- State Machine
- Actions
- Domain (aka data)

Let's add these to our rapidly evolving World system now:

```
#World_v2
  -interface-
  -machine-
  -actions-
  -domain-
##
```

The blocks are optional, but if they are present they must be in the order shown.

This specification still doesn't generate any interesting code or documentation. Let's make a **state**ment and add a state to our state machine.

## In the Beginning...

States are identifiers in the `-machine-` block declared using the `$` symbol like so:

```
#World_v3

  -machine-

  $Begin

##
```

Now we are on the map! This specification results in a model with `$Begin` as the start state:

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8Ld5BJC_CKghbgkQArOXLqTUqW8bmEgNafG5K0)

To try this out yourself, copy the code above and paste into the left panel of the <a href='http://framepiler.frame-lang.org' target='_blank'>Framepiler</a> to see various kinds of output Frame notation can be converted into.

This world still doesn't do anything, but things are shaping up. Let's check out our system code now:

`C#`
{% highlight csharp %}
    public partial class World_v3 {    

        public World_v3() {            
            _state_ = _sBegin_;
        }

        //===================== Machine Block ===================//    

        private void _sBegin_(FrameEvent e) {
        }

        //=========== Machinery and Mechanisms ===========//

        private delegate void FrameState(FrameEvent e);
        private FrameState _state_;

    }
{% endhighlight %}


Ok! This is starting to look a lot more interesting. Let's begin our study of how to rule a single state next.

## The Machine Block

All states must live in the Machine Block. Currently our World just has one state - `$Begin`:

{% highlight csharp %}

    //===================== Machine Block ===================//

    private void _sBegin_(FrameEvent e) {
    }

{% endhighlight %}


Frame is rather opinionated on how to implement state machines. While there are lots of equivalent ways to implement one, Frame notation is designed to express a state like this:

{% highlight csharp %}
    private void _sBegin_(FrameEvent e) {
    }
{% endhighlight %}

What we see is a class method that:

- takes a `FrameEvent` object `e`
- returns nothing

That's it! Cool. But, um, how do we use it?

Each language will be slightly different in the details, but the basic principles will apply in all situations. As we have just a single state we could just always send any FrameEvents to it. However a state machine with just one state is really boring and we will want to have our machine in different states over time. To do that we will instead keep a variable that references the current state of the machine and change it when we want our machine to do something different. To do so, we need to get some...

## Machinery and Mechanisms

By convention Frame generates a `_state_` variable to track the current state and enable us to have more than one. In `C#` we need to create a C# `delegate` type that represents references  to a method signature that is compatible with the requirements of our state method signature. Here this type is called a `FrameState`.


{% highlight csharp %}
    //=========== Machinery and Mechanisms ===========//

    private delegate void FrameState(FrameEvent e);
    private FrameState _state_;  
{% endhighlight %}

So now we have a `FrameState` type and a `_state_` variable that can reference one. Finally we initialize our `_state_` to the `$Begin` start state in the constructor of the World:


{% highlight csharp %}
    public World_v3() {        
        _state_ = _sBegin_;
    }
{% endhighlight %}

Easy peasy.

So we now have achieved a fully operational state machine that does - nothing. Let's continue to add value to our system and make it do something (like, hmm, Hello).

## Events

In our next World, we start to get things happening. Events drive behavior and in Frame the `FrameEvent` class is the defined object that the language manipulates:


{% highlight csharp %}
public class FrameEvent {
    public FrameEvent(String msg, Dictionary<String,object> parameters) {
        this.msg = msg;
        this.parameters = parameters;
    }
    public String msg;
    public Dictionary<String,Object> parameters;
    public Object ret;
}
{% endhighlight %}

The code above shows the simplest implementation of a `FrameEvent` with all of the fields public. What is critical is that the `FrameEvent` has the following fields:

- A `message` object (String, enumeration, other)
- A `parameters` key/value lookup object
- A `return` object

Frame notation uses the `@` symbol to indicate an event (much more on this later). Each of the three attributes has its own symbol as well:

| Symbol | Attribute |
| @ | frameEvent |
| @\|\| | frameEvent.message |
| @[] | frameEvent.parameters |
| @["foo"] | frameEvent.parameters["foo"] |
| @^ | frameEvent.return |

Frame uses shorthand for access to some of these parameters. The  most common situation is selecting for an event message to trigger executing an event handler:


```
#World_v4

  -machine-

  $Begin
    |someMessage| ^

##
```

Above we see `|someMessage|` which is actually shorthand for `@|someMessage|`. Either can be used and are equivalent.

We also see a new symbol: `^`. In Frame this means `return` from the state function.

The `$Begin` state now looks like this:


{% highlight csharp %}
private void _sBegin_(FrameEvent e) {
    if (e.Msg.EqualsEx("someMessage")) {
        return;
    }
}
{% endhighlight %}

Here we can see that the message selector `|someMessage|` is manifested as a simple `if` test on the `FrameEvent.message` field.

Easy peasy.

## And...Actions!

We are now on the brink of...

elocution.

To do so we need to introduce Frame Actions, aka private methods.
Frame syntax for actions will look very familiar:


```
#World_v5

  -machine-

  $Begin
    |speak|
      print("Hello World") ^

  -actions-

  print [msg:String]

##
```

In the `-actions-` block we see a declaration for a `print` action:


{% highlight csharp %}
    -actions-

    print [msg:String]
{% endhighlight %}

Frame's action syntax is to have the <i>delcaration</i> parameter list in brackets `[param1:type1 param2:type2]` while the action call is indicated with parenthesis:

`action(p1 p2)`

(Aside - It should be mentioned at this point that the author of Frame has the belief that we as programmers can rid ourselves of commas and semicolons as programming language punctuation. Time will tell, but for now Frame does not support either as separators of anything.)

So for our `print[msg:String]` declaration this code is generated:


{% highlight csharp %}
    //===================== Actions Block ===================//

    protected virtual void print_do(String msg) {}
    {% endhighlight %}

    And for the call:


    {% highlight csharp %}
    private void _sBegin_(FrameEvent e) {
        if (e.Msg.EqualsEx("speak")) {
            print_do("Hello World");
            return;
        }
    }
{% endhighlight %}

Here we see that the `print()` action is turned into `print_do()`. The <a href="http://framepiler.frame-lang.org" target="_blank">Framepiler</a> adds a standard suffix to action names which can be anything. The reason for suffixes is to allow actions and interface methods to the same name in Frame, but distinguish them in the generated code. In order to do so and avoid name collisions the Frame code generator appends some suffix to the action, in this case `_do`. However, anything would really `_do`.

Another key point is that the <a href="frame-lang.org" target="_blank">Framepiler</a> only generates action stubs, the developer must decide what output mechanism to implement. There are various approaches to this requirement but the simplest would be the following:


{% highlight csharp %}
    protected virtual void print_do(String msg) {
        Console.WriteLine(msg);
    }
{% endhighlight %}


## Interfacing

Despite all of the progress we have made, our system is still what is known as a "closed system" with no way to interact with the outside universe.

To open things up we need to create interface methods. That is accomplished by adding interface specifications to the `-interface-` block:


```
    #World_v6

      -interface-

      speak

    ...

    ##
```

This notation generates the following code:

{% highlight csharp %}
    //===================== Interface Block ===================//

    public void speak() {
        FrameEvent e = new FrameEvent("speak",null);
        _state_(e);
    }
{% endhighlight %}

As a reminder, the interface for a `FrameEvent` is:


{% highlight csharp %}
    public FrameEvent(String msg, Dictionary<String,object> parameters) {
        this.msg = msg;
        this.parameters = parameters;
    }
{% endhighlight %}

Here, for the first time, we see how a `FrameEvent` is generated and sent into the state machine to drive system activity.

## Getting in the Last World

Whew - we've been through a lot in a hurry! But we did succeed in emitting "Hello World" in Frame notation. Let's take a look at where we are at:

This Frame specification here:


```
    #World_v6

      -interface-

      speak

      -machine-

      $Begin
        |speak|
          print("Hello World") ^

      -actions-

      print [msg:string]

    ##
```

can now generate this code:

<iframe width="100%" height="475" src="https://dotnetfiddle.net/Widget/W7TYLg" frameborder="0"></iframe>

## Next steps

We have covered a lot of ground in this article. However, we really haven't created an interesting system yet. In the [next installment]({% post_url /2021-02/2021-02-13-hello_worlds_part_2 %})
we will add more states to our system and see how Frame enables defining complex system behavior with nothing more than a text editor.

### La fin!
