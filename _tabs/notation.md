---
title: Frame Notation
icon: fas fa-info
order: 2
---

# Frame System Notation (FSN) v0.3

**Frame System Notation** (FSN) is a domain specific markdown language for defining the behavior of software (and other) systems. Frame technology generates both system controller code as well as corresponding documentation from a system specification document. Frame currently supports generating C#, C++, JavaScript and UML documentation.

## Comments

Comments are single line and indicated by three (3) dashes:
```
--- this is a single line comment
--- as is this
```

## Variable and Parameter Declarations

Variables and parameter declarations share a core common syntax.

### Parameter Declarations

Parameters are declared as follows and separated by whitespace:
```
param:type
```

The name is required but the :type is optional. Parameter lists are one or more parameter declarations enclosed in brackets:

```
[<param_list>]
```

Therefore parameter lists can be declared either of these two ways:

```
[param1 param2]

--- or ---

[param1:type1 param2:type2]
```
### Variable Declarations

Variable and constant declarations have the following format:

```
    <var | const> <name> : <type_opt> = <intializer>

    var x:int = 1
    const name = "Steve"
```

Variables can be modified after initialization and constants can not. Frame will transpile into the closest semantic equivalents in the target language.

The type is optional but the initializer is not.

If you transpile into a language that requires a type and you don't provide one, a token such as `<?>` is substituted. Conversely, if you add a type and transpile into a language that doesn't require one, the Framepiler ignores it.

## Methods

 All methods (for all blocks) have a similar syntax:

```
<method-name> <parameters-opt> <return-value-opt>

```

As implied above, the parameters and return value are optional. Here are the permutations for method declarations:

```
method_name
method_name [param]
method_name [param:type]
method_name [param1 param2]
method_name [param1:type param2:type]
method_name : return_value
method_name [param1:type param2:type] : return_value
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

Unlike other languages where structured whitespace is significant (e.g. Python), Frame's use of whitespace is <i>unstructured</i>. Frame only separates tokens with whitespace and does not insist on any pattern of use.

The esthetic goal is to be as spare and clean as possible, but it may take some getting used to.

## Lists

List come in two flavors - *parameter lists* and *expression lists*.

Frame uses square brackets to denote parameter lists:

```
[x y]
[x:int y:string]
```


## System Controller Architecture

Now with the basics explained we can take a look at the big picture and work in.

The `#` token prefix tags an identifier as a <i>system</i>. the `##` token indicates the end of the system specification.

```
#MySystemController
##
```

Currently Frame generates an object-oriented class for each system specification document. However, this does not preclude other ways to implement system controllers which will be an interesting research topic for the future.

Frame is highly opinionated about the internal structure of system controllers and currently specifies a composition of four sequential (but optional) "blocks":

1. Interface
2. State Machine
3. Actions
4. Domain

Here is the syntax for these blocks:

```
#MySystemController
    -interface-
    -machine-
    -actions-
    -domain-
##  
```

Once again, there can be 0-4 blocks, but if present they must be in this order. Tying all of these together is the use of the FrameEvent which we will explore next.

## Frame Events

Frame Events are essential to the notation and the implementation of Frame system controllers. Frame notation assumes three mandatory fields for a FrameEvent:

- A `message` object (String, enumeration, other)
- A `parameters` key/value lookup object
- A `return` object

Here is a basic implementation of this class:

`C#`
{% highlight csharp %}
public class FrameEvent {
    public FrameEvent(String message, Dictionary<String,object> parameters) {
        this._message = message;
        this._parameters = parameters;
    }
    public String _message;
    public Dictionary<String,Object> _parameters;
    public Object _return;
}
{% endhighlight %}

Frame notation uses the `@` symbol to identify a FrameEvent. Each of the three FrameEvent attributes has its own accessor symbol as well:

| Symbol | Meaning/Usage |
|--------|---------|
| @ | frameEvent |
| @\|\| | frameEvent._message |
| @[] | frameEvent._parameters |
| @["foo"] | frameEvent._parameters["foo"] |
| @^ | frameEvent._return |
| ^(value) | frameEvent._return = value; return; |

Frame has six special reserved messages for important operations:

| Message Symbol | Meaning | Mandatory |
|--- | ---| --- |
|> | Enter state | Yes |
|< | Exit state | Yes |
|\>\> | Start system | No |
|\<\< | Stop system | No |
|\>\>\> | Save system | No |
|\<\<\<| Restore system | No |

The semantics of the `|>|` and `|<|` events are understood by the Framepiler and functionally supported. The remaining messages are optional may be unused or replaced by other messages with the same semantics if desired.

## Interface Block

The Interface Block contains a list of zero or more public methods with an optional <i>alias</i> annotation.

Here is a simple `#Dog` with a single public interface method:

`Frame`
```
#Dog

    -interface-

    speak
##
```

`C#`
{% highlight csharp %}
public partial class Dog {

    //===================== Interface Block ===================//

    public void speak() {
        FrameEvent e = new FrameEvent("speak",null);
        _state_(e);
    }

}
{% endhighlight %}

This code snippet isn't actually functional as it references a missing `_state_` variable which wasn't generated. However it does show the most basic syntax for an interface method.

To get our `#Dog` to respond we can send it a text string of what we want it to say and return an AudioClip of <a href="https://www.youtube.com/watch?v=hiMjgLaN9cY" target="_blank">it's attempt</a>:

`Frame`
```
#Dog

    -interface-

    speak [name:string] : AudioClip
##
```

`C#`
{% highlight csharp %}
public partial class Dog {

    //===================== Interface Block ===================//

    public AudioClip speak(string name) {
        Dictionary<String,object> parameters = new Dictionary<String,object>();
        parameters["name"] = name;

        FrameEvent e = new FrameEvent("speak",parameters);
        _state_(e);
        return (AudioClip) e.Return;
    }

}
{% endhighlight %}

### Message Aliases

By default, Frame generates a message identical to the name of the interface method. In order to send a different message Frame supports the message alias syntax:

`@(|substitute_msg|)`

This can be used to change the messages that the interface sends to the state machine:

`Frame`
```
#SpanishDog

    -interface-

    speak [name:string] : AudioClip @(|hable|)
##
```

`C#`
{% highlight csharp %}
public partial class SpanishDog {

    //===================== Interface Block ===================//

    public AudioClip speak(string name) {
        Dictionary<String,object> parameters = new Dictionary<String,object>();
        parameters["name"] = name;

        FrameEvent e = new FrameEvent("hable",parameters);
        _state_(e);
        return (AudioClip) e.Return;
    }

}
{% endhighlight %}

As we can see interface methods do not provide any actual functionality. Instead, the interface simply transmits the message and parameters from the outside world to the system controller's internal state machine. When the machine completes it's activity the interface returns whatever values come back from it on the FrameEvent's return attribute.

Frame takes advantage of the message alias capability to define a standardized pattern of starting and stopping a system:

`Frame`
```
#System

    -interface-

    start @(|>>|)
    stop @(|<<|)
##
```

`C#`
{% highlight csharp %}
public partial class System {

    //===================== Interface Block ===================//

    public void start() {
        FrameEvent e = new FrameEvent(">>",null);
        _state_(e);
    }

    public void stop() {
        FrameEvent e = new FrameEvent("<<",null);
        _state_(e);
    }
}
{% endhighlight %}

This feature gives flexibility in naming the interface methods while maintaining a standardized internal protocol for system start/stop using `>>` and `<<`. We will revisit how these are used after introducing Transitions below.

## Machine Block

The Machine Block houses the system state machine and defines the behavior of the system. The machine block is comprised of zero or more states. If it exists, the first state is defined to be the "start state" which is what the machine's `_state_` variable is initiazed to.

```
#LaMachine

    -machine-

    $Begin
##
```

![](https://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8Ld5BJC_CKghbgkQArOXLqTUqW8bmEgNafG5K0)

### States

State identifiers are indicated by the `$` prefix:

`$Begin`

In the Frame reference implementation, states are implemented as methods with this signature:

`void state(FrameEvent e)`

In languages that support or require it, this signature is typedef'd with a name:

`C#`
{% highlight csharp %}
    private delegate void FrameState(FrameEvent e);
{% endhighlight %}

The machine implementation must contain a private member variable for maintaining a reference to the current state:

`C#`
{% highlight csharp %}
    private FrameState _state_;
{% endhighlight %}

The `_state_` variable must be initialized in the constructor or before the object is used to the start state.

`C#`
{% highlight csharp %}
    public LaMachine() {
        _state_ = _sBegin_;
    }
{% endhighlight %}

After being initialized, the `_state_` variable should only be modified as part of a "transition" or "state change" (see below). Without a transition or state change, the machine will only stay in its initial state. Here is a `C#` minimal state machine:

`C#`
{% highlight csharp %}
    public partial class LaMachine {
        public LaMachine() {

            _state_ = _sBegin_;
        }


        //===================== Machine Block ===================//

        private void _sBegin_(FrameEvent e) {
        }

        //=============== Machinery and Mechanisms ==============//

        private delegate void FrameState(FrameEvent e);
        private FrameState _state_;

    }
{% endhighlight %}

### Event Handlers

Event Handlers are sections of code called in response to a message.

`Frame`
```
#Dog

    -interface-

    speak [name:string] : AudioClip

    -machine-

    $Begin
        |speak| --- Event Handler message selector
            ^       --- return
##
```

`C#`
{% highlight csharp %}
        //===================== Machine Block ===================//

        private void _sBegin_(FrameEvent e) {
            if (e.Msg.Equals("speak")) {
                return;
            }
        }
{% endhighlight %}

Event handlers have two terminator tokens - return and continue.

|Token|Name|C# code|
|----|----|----|
|`^`|return|`return;`|
|`^(42)`|return value| `frameEvent._return = 42;`<br>`return;``|
|`:>`|continue|`break;`|

The basic return token simply returns from the state function while the return value variant first sets the FrameEvent's return value. The continue token allows the code to break out of the message tests and, as we will see later, enable further processing in parent states.

`Frame`
```
    ...

    -machine-

    $State
        |continueEvent|
            :>       --- continue terminator
        |returnEvent|
            ^       --- return terminator
        |returnValueEvent|
            ^(42)   --- return value terminator

    ...
```

`C#`
{% highlight csharp %}
    //===================== Machine Block ===================//

    private void _sState_(FrameEvent e) {
        if (e.Msg.Equals("continueEvent")) {
            break; //  continue terminator
        }

        else if (e.Msg.Equals("returnEvent")) {
            return; //  return terminator
        }          
        else if (e.Msg.Equals("returnValueEvent")) {
            e.Return = 42 // --- return value terminator
            return;

        }
    }
{% endhighlight %}

### Calling Actions

Actions are protected methods that can be called in event handlers to implement system behavior:


`Frame`
```
#Dog

    -interface-

    speak [name:string] : AudioClip

    -machine-

    $Begin
        |speak| speak() ^
##
```

`C#`
{% highlight csharp %}
        //===================== Machine Block ===================//

        private void _sBegin_(FrameEvent e) {
            if (e.Msg.Equals("speak")) {
                speak_do();
                return;
            }
        }
{% endhighlight %}

The Framepiler appends a standard suffix (in this case `_do`) so that the interface can call actions with the same name and not have a namespace collision.

### Transitions

State machines require the ability to update the `_state_` variable so as to point to a different `FrameState` method. In addition to simply switching state, transitions adhere to the following Statechart semantics:

1. send an exit message `<` to the current state
2. change the machine to the new state
3. send and enter message `>` to the new state

The `<` event handler, if present, can deallocate resources and do other clean up of the state that is being exited.

The `>` event handler, if present, can allocate resources and do other initialization of the state that is being entered.


`Frame`
```
#Dog

    -interface-

    start @(|>>|)
    speak
    hush

    -machine-

    $Begin
        |>>| -> $Quiet ^

    $Quiet
        |speak| -> $Barking ^

    $Barking
        |>| startBarking() ^
        |<| stopBarking() ^
        |hush| -> $Quiet ^
##
```

`C#`
{% highlight csharp %}
public partial class Dog {
    public Dog() {

        _state_ = _sBegin_;
    }

    //===================== Interface Block ===================//

    public void start() {
        FrameEvent e = new FrameEvent(">>",null);
        _state_(e);
    }

    public void speak() {
        FrameEvent e = new FrameEvent("speak",null);
        _state_(e);
    }

    public void hush() {
        FrameEvent e = new FrameEvent("hush",null);
        _state_(e);
    }


    //===================== Machine Block ===================//

    private void _sBegin_(FrameEvent e) {
        if (e.Msg.Equals(">>")) {
            _transition_(_sQuiet_);
            return;
        }
    }

    private void _sQuiet_(FrameEvent e) {
        if (e.Msg.Equals("speak")) {
            _transition_(_sBarking_);
            return;
        }
    }

    private void _sBarking_(FrameEvent e) {
        if (e.Msg.Equals(">")) {
            startBarking_do();
            return;
        }
        else if (e.Msg.Equals("<")) {
            stopBarking_do();
            return;
        }
        else if (e.Msg.Equals("hush")) {
            _transition_(_sQuiet_);
            return;
        }
    }

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

Below is a complete example of system lifecycle management utilizing many of the concepts that have been introduced so far:

`Frame`
```
#System

    -interface-

    start @(|>>|)
    stop @(|<<|)

    -machine-

    $Begin
        |>>| -> $Working ^

    $Working
        |<<| -> $End ^        

    $End   
##
```

`C#`
{% highlight csharp %}
    public partial class System {
        public System() {

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
            if (e.Msg.Equals("<<")) {
                _transition_(_sEnd_);
                return;
            }
        }

        private void _sEnd_(FrameEvent e) {
        }

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

### State Changes

Sometimes it is desirable to change states without triggering the full enter/exit machinery involved with a transition. The state change operator enables this capability:

`->> $NewState`

A simple filter system is a good example of when changing state is more appropriate than a full transition. Here we can see that the `#Filter` system simply oscillates between the `$Off` and `$On` states. It's only behaivor is enabling and disabling transmission of objects through it so no state resource management is necessary:

`Frame`
```
#Filter

    -interface-

    open
    close
    send [obj:object]

    -machine-

    $Closed
        |open| ->> $Open ^

    $Open
        |close| ->> $Closed ^

        |send| [obj:object]
            send(obj) ^

    -actions-

    send [obj:object]
##
```

Frame notates a state change `->>` visually using a dashed line:

![](https://www.plantuml.com/plantuml/png/SoWkIImgAStDuUBY0Z9BKXMSS_ABKrCKghbgeGB-1QbvO6wqLgo2hguTL0KNLA5kT4fYSKPgIYnG1gpKIa5DsT38n3eVo86mk43Y28Lm8-1Aaq5Sg5g7rBmKe7i0)

The implementation of the state change mechanism consists of the addition of the `_changeState_()` method that only updates the `_state_` variable (no enter/exit events) as well as its use in the event handlers:

`C#`
{% highlight csharp %}
    public partial class Filter {
        public Filter() {

            _state_ = _sClosed_;
        }

        //===================== Interface Block ===================//

        public void on() {
            FrameEvent e = new FrameEvent("on",null);
            _state_(e);
        }

        public void off() {
            FrameEvent e = new FrameEvent("off",null);
            _state_(e);
        }

        public void send(object obj) {
            Dictionary<String,object> parameters = new Dictionary<String,object>();
            parameters["obj"] = obj;

            FrameEvent e = new FrameEvent("send",parameters);
            _state_(e);
        }


        //===================== Machine Block ===================//

        private void _sClosed_(FrameEvent e) {
            if (e.Msg.Equals("on")) {
                _changeState_(_sOpen_);
                return;
            }
        }

        private void _sOpen_(FrameEvent e) {
            if (e.Msg.Equals("off")) {
                _changeState_(_sClosed_);
                return;
            }
            else if (e.Msg.Equals("send")) {
                send_do((object) e.Parameters["obj"]);
                return;
            }
        }

        //===================== Actions Block ===================//

        protected virtual void send_do(object obj) { throw new NotImplementedException(); }


        //=========== Machinery and Mechanisms ===========//

        private delegate void FrameState(FrameEvent e);
        private FrameState _state_;

        private void _changeState_(newState) {
            _state_ = newState;
        }

    }
{% endhighlight %}

### Branching Control Flow

Frame notation supports two general syntaxes for branching control flow:

- Boolean Tests
- Pattern Matching

The basic syntax for both classes of test are:

```
    x ?<type> <branches> : <else clause> ::

```    

The `:` token is "else" and `::` terminates the statement for all branching statement types.

Let's explore the boolean test first.

#### Boolean Tests

The basic boolean test in Frame is:

```
    x ? callIfTrue() : callIfFalse() ::
```
This generates this in `C#`:
{% highlight csharp %}
    if (x) {
        callIfTrue_do();
    } else {
        callIfFalse_do();
    }
{% endhighlight %}

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
{% highlight csharp %}
    if (x) {
        a_do();
        b_do();
    } else {
        c_do();
        d_do();
    }
{% endhighlight %}


To negate the test use the `?!` operator:

`Frame`
```
x ?! callIfFalse() : callIfTrue() ::
```
`C#`
{% highlight csharp %}
    if (!(x)) {
        callIfFalse_do();
    } else {
        callIfTrue_do();
    }
{% endhighlight %}

Next we will explore the Frame equivalent of the switch statement for string matching.

#### Pattern Matching Statements

Frame uses a novel but easy to understand notation for switch-like statements:

```
test ?<type>
    /pattern1/ statements :>
    /pattern2/ statements :
               statements ::
```

The currently supported operators are `?~` for string matching and `?#` for number/range matching. The `:` token indicates else/default and `::` terminates the pattern matching statement.

##### String Matching

The string matching statement looks like this:

`Frame`
```
name() ?~
    /Elizabeth/ hiElizabeth()   :>
    /Robert/    hiRobert()      :
                whoAreYou()     ::
```
And results in this code:

`C#`
{% highlight csharp %}
    if (name_do() == "Elizabeth") {
        hiElizabeth_do();
    } else if (name_do() == "Robert") {
        hiRobert_do();
    } else {
        whoAreYou_do();
    }
{% endhighlight %}

Frame also permits multiple string matches per pattern:

`Frame`
```
name() ?~
    /Elizabeth|Beth/ hiElizabeth()   :>
    /Robert|Bob/     hiRobert()      :
                     whoAreYou()     ::
```
With this output:

`C#`
{% highlight csharp %}
    if (name_do() == "Elizabeth") || (name_do() == "Beth") {
        hiElizabeth_do();
    } else if (name_do() == "Robert") || (name_do() == "Bob") {
        hiRobert_do();
    } else {
        whoAreYou_do();
    }
{% endhighlight %}

##### Number Matching

Number matching is very similar to string pattern matching:

`Frame`
```
n ?#
    /1/ print("It's a 1")   :>
    /2/ print("It's a 2")   :
        print("It's a lot") ::
```
The output is:

`C#`
{% highlight csharp %}
    if (n == 1)) {
        print_do("It's a 1");
    } else if (n == 2)) {
        print_do("It's a 2");
    } else {
        print_do("It's a lot");
    }
{% endhighlight %}

Frame can also pattern match multiple numbers to a single branch as well as compare decimals:

`Frame`
```
n ?#
    /1|2/           print("It's a 1 or 2")  :>
    /101.1|100.1/   print("It's over 100")  :
                    print("It's a lot")     ::
```
The output is:

`C#`
{% highlight csharp %}
    if (n == 1) || (n == 2)) {
        print_do("It's a 1 or 2");
    } else if (n == 101.1) || (n == 100.1)) {
        print_do("It's over 100");
    } else {
        print_do("It's a lot");
    }
{% endhighlight %}

#### Branches and Transitions

The default behavior of Frame is to label transitions with the message that generated the transition. This is fine when an event handler only contains a single transition:

`Frame`
```
#GottaBranch

  -machine-

    $A
        |e1| -> $B ^

    $B

##
```

![](https://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8L71MgkMgXR2SmErehLa5Nrqx1aSiHH0D5hHJKb0sDJAnJ3I4qbqDgNWhG2000)

However this leads to ambiguity with two or more transitions from the same event handler:

`Frame`
```
#GottaBranch_v2

  -machine-

    $Uncertain
        |inspect|
            foo() ?
                -> $True
            :
                -> $False
            :: ^

    $True

    $False

##
```

![](https://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8LGlEIKujA4ZFp5AgvQg5Y8KMbgKXSjyISOWW_MYjMGLVN3g692yu2YKCqMYceAHiQcLXdvXKNf2QNG3Ye2i56ubBfa9gN0dGV0000)

Transition labels provide clarity as to which transition is which:

`Frame`
```
#GottaBranch_v3

  -machine-

    $Uncertain
        |inspect|
            foo() ?
                -> "true foo" $True
            :
                -> "foo not true" $False
            :: ^

    $True

    $False

##
```

![](https://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8LGlEIKujA4ZFp5AgvQg5Y8KMbgKXSjyISOWW_MYjMGLVN3g692yu2YKCqMYcKWAYq_7nKMQWvLY0PXRpy4h0oBeVKl1IWQm00)



## Declaring Actions

Actions are declared in the `-actions-` block and observe all of the method declaration syntax discussed above. However actions do not support the alias notation:

`Frame`
```
#Kinetic

  -actions-

  walk
  run [speed:float]
  isHurt : bool

##
```


`C#`
{% highlight csharp %}
    public partial class Kinetic {


        //===================== Actions Block ===================//

        protected virtual void walk_do() { throw new NotImplementedException(); }
        protected virtual void run_do(float speed) { throw new NotImplementedException(); }
        protected virtual bool isHurt_do() { throw new NotImplementedException(); }

    }
{% endhighlight %}

Frame does not (yet) support language features for actually defining useful methods. Therefore the best practice to utilize the system controller Frame emits is to create a subclass that inherits from the controller and overrides the action declarations.

Notice that the generated controller code supports a default defensive programming mechanism by throwing an exception if the base class method is called.

## Domain System Variables

The final block in a Frame system specification is the `-domain-` where system level (member) variables are declared:

```
    <var | const> <name> : <type_opt> = <intializer>
```

Frame supports variables (`var`) which can be changed after initializations and constants (`const`) which cannot.

`Frame`
```
#SumData

    -domain-

    var i = 0
    var temp:float = 98.6
    const name:string = "Bob"

##
```

`C#`
{% highlight csharp %}
    public partial class SumData {


        //===================== Domain Block ===================//

        <?> i = 0;
        float temp = 98.6;
        string name = "Bob";
    }
{% endhighlight %}

As discussed previously, variables and constants can be untyped but not uninitalized. If a type is required by a target language the Framepiler will emit a token like `<?>` to indicate a missing required but unspecified type.


## Transition Parameters

Transition parameters allow system designers to specify that data that should be sent to enter `|>|` and exit `|<|` event handlers during a transition.

This capability simplifies managing data passing when the current event handler shouldn't process information in the current context.

### Enter Event Parameters

Frame provides notation to directly pass arguments to the new state as part of a transition:

`-> (<enter_argument_list>) $NewState`

For instance:

`-> ("Mark") $PrintName`

This list is sent as arguments to the enter event in the target state:

`Frame`
```
#EnterEventParameters

    -interface-

    start @(|>>|)

    -machine-

    $Begin
        |>>| -> ("Hello $State") $State ^

    $State
        |>| [greeting:string]
            print(greeting) ^

    -actions-

    print[message:string]

##
```

### Exit Event Parameters

Though not as common an operation as sending data forward to the next state, Frame also enables sending data to the exit event hander of the current state as well:

`(<exit_argument_list>) -> $NewState`

For instance:

`("cya") -> $NextState`

as in

`Frame`
```
    $OuttaHere
        |<| [exitMsg:string]
            print(exitMsg) ^

        |gottaGo|
            ("cya") -> $NextState ^
```



## State Parameters

In addition to parameterizing the transition operator, Frame enables passing arguments to states themselves. State arguments are passed in an expression list after the target state identifier:

`-> $NextState(<state_args>)`

State parameters are declared as a parameter list for the state:

`$NextState [<state_params>]`

Unlike transition parameters which are scoped to the enter/exit event handlers, state parameters are scoped to the lifecycle of the state itself and therefore in scope for any state event handler.

`Frame`
```
#StateParameters

    -interface-

    start @(|>>|)
    stop @(|<<|)

    -machine-

    $Begin
        |>>| -> $State("Hi! I am $State :)")  ^

    $State [stateNameTag:string]
        |>|  print(stateNameTag) ^
        |<|  print(stateNameTag) ^
        |<<|
             print(stateNameTag)
             -> $End ^

    $End

    -actions-

    printAll[message:string]

    -domain-

    var systemName = "#Variables"
##
```
Above we see that the `stateNameTag` is accessible in the enter, exit and stop event handlers. It will also be in scope for all other event handlers for the state as well.

## Frame Variables

Frame has three scopes for variable declarations:

- System domain variables
- State variables
- Event handler variables

### System Domain Variables

In object-oriented terminology, domain variables are simply member variables. As such, their scope is system wide.

`Frame`
```
#DomainVariables

    -interface- 

    start @(|>>|)

    -machine-

    $Begin
        |>>|
            print(systemWide)
            -> $S0 ^

    $S0
        |>|
            print(systemWide)
            -> $S1 ^

    $S1
        |>|
            print(systemWide)
            -> $End ^            

    $End
        |>|
            print(systemWide)
            -> $End ^   

    -domain-

    var systemWide:string = "bigtime"
##
```

## State Variables

The next scope for declaring variables is the state. States can declare variables above the first event handler:

`Frame`
```
    ...
    $State
        var x:int = 0

        |>| ^
    ...
```

State variable scope is across all event handlers for the lifecycle of the state:

`Frame`
```
#StateVariableExample

    -machine-

    $Working
        var stateName:string = "$Working"

        |>| print(stateName) ^
        |<| print(stateName) ^
        |<<| print(stateName) ^
##
```
As we can see, the state variable `stateName` stays in scope for the active lifecycle of the state, just like state parameters.

## Event Handler Variables

Event handler variables are scoped to the event handler they are declared in:

`Frame`
```
#EventHandlerVariableExample

    -machine-

    $Working
        |>|
            var id:string = "12345"

            print(getFirstName(id)) ^
        |e1|
            print(getFirstName(id)) ^ --- Error!            
##
```

## Scope Identifiers

If variables have unique names then Frame resolves them by searching in the following priority order for scopes for the first match:

1. Event Handler Variables
2. Event Handler Parameters
3. State Variables
4. State Parameters
5. System Domain Variables

To disambiguate variables with the same name in different scopes, Frame uses the following symbols:

|Symbol|Scope|Example|
|:--|---|---|
|`||.<id>`|Event Handler Variable|`||.d`|
|`||[<id>]`|Event Handler Parameter|`||[c]`|
|`$.<id>`|State Variable|`$.b`|
|`$[<id>]`|State Parameter|`$[a]`|
|`#.<id>`|System Variable|`#.e`|

Here we can see the use of each of these scope identifiers:

`Frame`
```
#ScopeIdentifiers

    -interface-

    start @(|>>|)

    -machine-

    $Begin
        |>>| -> (2) $Scopes(4) ^

    $Scopes[d:int] --- 4

        var c:int = 3

        |>| [b:int] --- 2

            var a:int = 1

            output(||.a ||[b] $.c $[d] #.e) ^

    -actions-

    output[a:int b:int c:int d:int e:int]

    -domain-

    var e:int = 5
##
```

You can see the generated controller here:

<iframe width="100%" height="475" src="https://dotnetfiddle.net/Widget/AH20m9" frameborder="0"></iframe>
