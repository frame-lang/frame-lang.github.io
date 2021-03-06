---
layout: post
title:  "The Frame System Interface"
date:   2021-03-14 00:00:00 -0800
categories: language-basics
---

As discussed in previous articles, Frame specs are divided into up to four "blocks", the first of which is the *interface block*.

The interface block implementation in the target language consists of the public methods for the object which are responsible for creating and initializing a *FrameEvent* object, sending it to the internal state machine and then returning a response, if any, to the caller. Let us start by taking a look at the structure of a FrameEvent and then discover how it is used in the interface.

## Frame Spec Overview

The diagram below shows a model Frame spec and it's four blocks, illustrating how the interface is the gateway for information and activity into the system. Here we can see how calls to the interface methods are marshaled into a FrameEvent and sent into the state machine to drive activity:

<img width="70%" src="https://cdn.jsdelivr.net/gh/frame-lang/article_content@06c43a539ddfd16ec235d399b5eba7eca11331a9/FrameSpec.png">


To see this architecture in detail,  we will explore the inner workings of the following Frame spec:

```
#InterfaceSpec

    -interface-

    yell [msg:string] :string @(|whisper|)

    -machine-

    $Echo
        |whisper| [msg:string] :string
            ^(toLowerCase(msg))

    -actions-

    toLowerCase [msg:string] :string
##
```

Let us now take a look at a FrameEvent class definition to understand how it serves as the information transport mechanism for the system and the stimulus for system behavior.

## The FrameEvent

Frame Events are essential to the notation and the implementation of Frame system controllers. Frame notation has syntax for manipulating FrameEvents and assumes three mandatory fields exist:

- A `message` object (String, enumeration, other)
- A `parameters` key/value lookup object
- A `return` object

Here is a basic implementation of the FrameEvent class:

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

The message, by default, is the name of the interface method called and the parameters are the - well - parameters. The return object, if there is one, is created by the behavior of the state machine while processing the FrameEvent and will hold the data to be returned back to the client.

Frame notation uses the `@` symbol to identify a FrameEvent. Each of the three FrameEvent attributes has its own accessor symbol as well:

| Symbol | Meaning/Usage |
|--------|---------|
| @ | frameEvent |
| @\|\| | frameEvent._message |
| @[] | frameEvent._parameters |
| @["foo"] | frameEvent._parameters["foo"] |
| @^ | frameEvent._return |
| ^(value) | frameEvent._return = value; return; |

In this article we will only use the last accessor for the return value. This expression notation can be interpreted as "set the return value on the current event being handled and return". It would also be possible to use the equivalent syntax:

```
@^ = toLowerCase(msg) ^
```

Let's now explore a simple demo to see FrameEvents in action.

## HEY! shh. Its A Demo!

Interface methods have the following syntax:

`iface_method [param1:type1 param2:type2] :ret_type @(|message_alias|)`

As we can see, interface declarations have four parts, the last three being optional:

1. The interface method name.
2. The parameter list
3. The return type
4. The message alias

The #InterfaceSpec system only has a single interface method - yell - with the following attributes:

|Syntax    | Meaning |
|--- | --- |
|`yell` | the interface method name|
`[msg:string]` | method parameters
`:string` | return type
`@(|whisper|)` | message alias

Most of this syntax is self explanatory, however a few nuances are worthy of discussion for each part.

- The interface method name must be compatible with the target languages for the code generator.
- The yell parameter list only has one parameter. Parameter lists separate parameters only with whitespace as in `[param1:type1  param2:type2]`. Note - types are optional for Frame, but possibly not for the target language.
- The return type format is `:return_type`.
- The default message sent with a FrameEvent is the name of the interface method. The *message alias* syntax allows this default behavior to be overriden. The alias notation uses the Frame token for "FrameEvent" `@` and the message selector syntax `|substitute_msg|` to indicate that the default message is being replaced by the new message alias.

Below we see the code that is generated by the `yell` declaration:

`C#`
{% highlight csharp %}
public string yell(string msg) {
    Dictionary<String,object> parameters = new Dictionary<String,object>();
    parameters["msg"] = msg;

    FrameEvent e = new FrameEvent("whisper",parameters);
    _state_(e);
    return (string) e._return;
}
{% endhighlight %}

All interface method implementations follow the same pattern we see above:

1. If there are parameters on the call, create a dictionary object.
2. Add parameters to the dictionary.
3. Create new FrameEvent initialized with the message (interface name or alias) and parameters.
4. Send FrameEvent to the current state of the state machine.
5. Pass the return value, if any, to the caller.

This completes our examination of the Frame syntax and implementation of interface methods. However it is instructive to understand how the FrameEvent created by the interface is used, which we will now show.

## The FrameEvent and Event Handlers

The `yell` interface method creates a new FrameEvent and adds its parameters to it, as well as changes the default message `|yell|` to `|whisper|`.

In the `$Echo` state, we can see there is an event handler that selects for the `|whisper|` message and takes the correct parameters:

```
#InterfaceSpec

    -interface-

    yell [msg:string] :string @(|whisper|)

    -machine-

    $Echo
        |whisper| [msg:string] :string
            ^(toLowerCase(msg))

    -actions-

    toLowerCase [msg:string] :string
##
```


The event handler does one thing - it calls the `toLowerCase(msg)` action with the message that was sent and returns it, presumably after transforming it to lower case in an appropriate way for the target language.

`C#`
{% highlight csharp %}
private void _sEcho_(FrameEvent e) {
    if (e._message.Equals("whisper")) {
        e._return = toLowerCase_do(((string) e._parameters["msg"]));
        return;
    }
}
{% endhighlight %}


Here we see the FrameEvent parameters used to look up the value of the `msg` parameter. Additionally, the FrameEvent `_return` parameter is set to carry the value to the interface method.

## Conclusion

Although there are many different ways to implement automata, Frame's highly opinionated choice to make FrameEvents a first-class aspect of the implementation enables much of the simplicity and power of the language.

While this results in a radically different implementation of an object-oriented class than would be coded by hand, it does allow standardization of object-oriented implementations across all popular object-oriented languages, and possibly a much broader set of languages in the future.

Here is a working example of the `#InterfaceSpec` system showing all the aspects we have discussed.

<iframe width="100%" height="475" src="https://dotnetfiddle.net/Widget/8YuOqs" frameborder="0"></iframe>
