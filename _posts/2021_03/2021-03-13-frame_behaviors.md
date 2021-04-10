---
layout: post
title:  "System Behavior Notation in Frame"
date:   2021-03-13 00:00:00 -0800
categories: language-basics
---

At a high level, systems have two kinds of behavior - actions and changes of state. In Frame, changes of state come in two flavors - transitions and state changes.

<a target="_blank" href="https://pdf.sciencedirectassets.com/271600/1-s2.0-S0167642300X00603/1-s2.0-0167642387900359/main.pdf?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEN3%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCzx8s1kIWppjed%2FUWqx8Mm4wwdgOoKYvOPoj%2BDzdFVDgIgcCvCR35wnyQ7h3WhBBIODjzDsnLyT3xtY4AZgjt1DLEqtAMINRADGgwwNTkwMDM1NDY4NjUiDK%2BLZBtbDsM1o2RwvSqRAyBKI6qLTrCmQh8wPwmUPOGn3gXTZY8PJd9hFZwp9g5I5KESpqGtG3QE1iEVkd5y%2BfHOcDp6mzGT9g6uB49Dlf5NSVgSCIws%2BqNf%2FTXS8NoUkcIbNnetBh6mtLP9sGvWvybnu1cSzSqAq2efP22oAbUkDqIvsvGvMdMboJnmLoB3luBVDd%2Bk9YkvsbTlIsPdfmfmptHvr9Gpi%2FSKcx1yu4UfaoHVvUB%2F95T8zNkono%2F6T6t%2BNlg0zPu1PTiwvX0r67YKkaNr%2FOD1al%2Fp259PFxCMNoZp02qm1cSmHQGW6Fg94D%2B%2BDPVMZMWf4hVf6WrCXpHnP2YQy5x%2F4sV9TJ8u97uDSvN9OQyjYu0mjX8jaGMAK1Dv7HwicVZMlZugztdT%2FBmRNNIWW6gSx3F0UT%2FBIoONFMiJeuUyIaHieARIwQgfr%2Bbj6zB5Y4mTSO3mVGvCPPVZedsuDAsIMqrOhMD3EA1J%2BSRepgN%2FReIMnW9rlXv26Enx6QZzHwDX40r1ONOrCWDWyCPH2OmgywBasWvju6%2FdMMeEs4MGOusBJZ09B57bJBuEF5c6YWgePXgj9xD5RbSqfQjWuN8vRLmoiFtw0w5LFn0CFxWLCQKgKVzPtEowrFZLSnkBAmFlgIwEak0NRpjTUcpFMeyiHmnXOJ%2Fc4fjSmU2NWRPyqUeleEgf5kWWyNHYEgSj%2B0N52qv%2FVelSO2%2BX%2FeC2Ihrz7aKmM8GJuGjg3sbpvQETJyklUc7hZ%2BLlDHVNz48cvaWzrjOGF9d8gJmdcqwoK0v%2F1FpaVvvDzADQ0hIolTPYcBK1biNIf6Mk4kOu2S5EYIprZMK0II%2B%2FhNv9p4KOqcI3M5Db%2BCBnWwmQRJhu2Q%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20210406T212429Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAQ3PHCVTYXNRPK4HG%2F20210406%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=0cf0584878a6b7545a274637da757ebed8899424b7849f0b63d30442cfe25c20&hash=d331ccbf8042d88ab9fc252d88cbc4222efd5db01880de4cd51f2cfea85969fa&host=68042c943591013ac2b2430a89b270f6af2c76d8dfd086a07176afe7c76c2c61&pii=0167642387900359&tid=spdf-677946f4-9483-4f3f-8bef-e2cd58774ac0&sid=34e9e9da9959484ec31969417a8d858eb39bgxrqa&type=client">Statecharts</a>, the software modeling technique Frame descends from, distinguishes between "actions" which take very little time to execute and "activities" which are relatively long running asynchronous or background behavior, typically managed by actions. For now we can consider all other categories of things systems can do as being derivative from or enabled by actions and transitions.

We will now explore these two broad categories of behavior and discuss the Frame notation for executing them.

## Actions

Frame promotes a "self-documenting" coding style and generally suggests that Frame specs utilize descriptively named action methods to execute operations. Partly this is due to Frame not (currently) supporting much in the way of data manipulation or test syntax.

Actions are simply private methods, if privacy is supported by the target language, that do lower level operations, tests or functional operations. More structure and differentiation may evolve and further categories of methods may emerge as Frame advances in sophistication, but for now actions fulfill a large number of fine-grained roles for Frame specifications.

## Changes of State

Frame supports two operators for changing state:

|Symbol|Meaning|Operations|
|:-: | :-:| :-: |
|-\>\> "opt_label" $NewState| State Change | Change State
|-\> "opt_label" $NewState| Transition | Send Exit Event, Change State, Send Enter Event |

Both operators update the state variable with the target state. However they are radically different in the machinery that they utilize.

Here is a spec showing both in use:

```
#ChangingState

    -interface-

    transition
    changeState

    -machine-

    $S0
        |<| handleTransitionExitEvent() ^
        |transition| -> "transition" $S1 ^

    $S1
        |>| handleTransitionEnterEvent() ^
        |changeState| ->> "change state" $S0 ^

    -actions-

    handleTransitionExitEvent
    handleTransitionEnterEvent
##
```

To visually differentiate these two operators in documentation, Frame specifies that the change state operator is drawn with a dotted line while the transition with a solid line.

<img width="35%" src="https://cdn.jsdelivr.net/gh/frame-lang/article_content@b3c0230973f4f869705b30167100a8c7bb624fed/changeState.png">

We will now examine how both of these operators work under the hood.

## The Transition Operator

As mentioned, transitions are a powerful but heavyweight operator that do the following steps:

1. Send an Exit Event (`|<|`) to the current state.
2. Update the state variable to be the new state.
3. Send an Enter Event (`|>|`) to the new state.

Let us see this machinery in the C# output for our `#ChangingState` spec:

{% highlight csharp %}
//=============== Machinery and Mechanisms ==============//

private delegate void FrameState(FrameEvent e);
private FrameState _state_;

private void _transition_(FrameState newState) {
    FrameEvent exitEvent = new FrameEvent("<",null);
    _state_(exitEvent);
    _state_ = newState;
    FrameEvent enterEvent = new FrameEvent(">",null);
    _state_(enterEvent);
}
{% endhighlight %}

Above we can see the declaration of the `_state_` member variable that tracks the current state. C# also requires state references to be declared as a C# `delegate`, so the FrameState type is declared as such and used as the type for `_state_`. Other languages have different requirements for how member variables must be declared to reference methods.

Invoking a transition is a simple matter of calling the `_transition_(newState)` method:

`Frame`
```
$S0
    |<| handleTransitionExitEvent() ^
    |transition| -> "transition" $S1 ^
```

`C#`
{% highlight csharp %}
private void _sS0_(FrameEvent e) {
    if (e._message.Equals("<")) {
        handleTransitionExitEvent_do();
        return;
    }
    else if (e._message.Equals("transition")) {
        // transition
        _transition_(_sS1_);
        return;
    }
}
{% endhighlight %}

Above we see the `_transition_(_sS1_)` call that will trigger the `|<|` call for `$S0` and subsequently send the `$S1` state the `|>|` message:

`C#`
{% highlight csharp %}
private void _sS1_(FrameEvent e) {
    if (e._message.Equals(">")) {
        handleTransitionEnterEvent_do();
        return;
    }
    else if (e._message.Equals("changeState")) {
        _changeState_(_sS0_);
        return;
    }
}
{% endhighlight %}

Now let us take a look at the change state mechanisms, which are decidedly simpler.

## The Change State Operator

The machinery for a state change is trivial:

`C#`
{% highlight csharp %}
private void _changeState_(newState) {
    _state_ = newState;
}
{% endhighlight %}

Using a method to change state is arguably inefficient. However the code the Framepiler currently generates is a "reference architecture" for defining the functional pattern Frame promotes. It also provides a choke point for centralized debugging as well. In the future, the Framepiler will provide options to generate optimized code.

The Frame change state notation shown here:

`Frame`
```
$S1
    |changeState| ->> "change state" $S0 ^
```

generates the following C# code:

`C#`
{% highlight csharp %}
if (e._message.Equals("changeState")) {
    _changeState_(_sS0_);
    return;
}
{% endhighlight %}

State changes are useful not only for efficient context switches when enter/exit events are not available but also for certain situations where they *are* available but shouldn't be triggered. This gives the system architect a means to quietly "slip into" and "slip out of" a state. This can be very useful for a number of situations that will be discussed later in more architecture oriented articles.

## Conclusion

Actions and changing state make up the bulk of the capabilities of state machine behavior and align with traditional <a target="_blank" href="https://pdf.sciencedirectassets.com/271600/1-s2.0-S0167642300X00603/1-s2.0-0167642387900359/main.pdf?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEN3%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCzx8s1kIWppjed%2FUWqx8Mm4wwdgOoKYvOPoj%2BDzdFVDgIgcCvCR35wnyQ7h3WhBBIODjzDsnLyT3xtY4AZgjt1DLEqtAMINRADGgwwNTkwMDM1NDY4NjUiDK%2BLZBtbDsM1o2RwvSqRAyBKI6qLTrCmQh8wPwmUPOGn3gXTZY8PJd9hFZwp9g5I5KESpqGtG3QE1iEVkd5y%2BfHOcDp6mzGT9g6uB49Dlf5NSVgSCIws%2BqNf%2FTXS8NoUkcIbNnetBh6mtLP9sGvWvybnu1cSzSqAq2efP22oAbUkDqIvsvGvMdMboJnmLoB3luBVDd%2Bk9YkvsbTlIsPdfmfmptHvr9Gpi%2FSKcx1yu4UfaoHVvUB%2F95T8zNkono%2F6T6t%2BNlg0zPu1PTiwvX0r67YKkaNr%2FOD1al%2Fp259PFxCMNoZp02qm1cSmHQGW6Fg94D%2B%2BDPVMZMWf4hVf6WrCXpHnP2YQy5x%2F4sV9TJ8u97uDSvN9OQyjYu0mjX8jaGMAK1Dv7HwicVZMlZugztdT%2FBmRNNIWW6gSx3F0UT%2FBIoONFMiJeuUyIaHieARIwQgfr%2Bbj6zB5Y4mTSO3mVGvCPPVZedsuDAsIMqrOhMD3EA1J%2BSRepgN%2FReIMnW9rlXv26Enx6QZzHwDX40r1ONOrCWDWyCPH2OmgywBasWvju6%2FdMMeEs4MGOusBJZ09B57bJBuEF5c6YWgePXgj9xD5RbSqfQjWuN8vRLmoiFtw0w5LFn0CFxWLCQKgKVzPtEowrFZLSnkBAmFlgIwEak0NRpjTUcpFMeyiHmnXOJ%2Fc4fjSmU2NWRPyqUeleEgf5kWWyNHYEgSj%2B0N52qv%2FVelSO2%2BX%2FeC2Ihrz7aKmM8GJuGjg3sbpvQETJyklUc7hZ%2BLlDHVNz48cvaWzrjOGF9d8gJmdcqwoK0v%2F1FpaVvvDzADQ0hIolTPYcBK1biNIf6Mk4kOu2S5EYIprZMK0II%2B%2FhNv9p4KOqcI3M5Db%2BCBnWwmQRJhu2Q%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20210406T212429Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAQ3PHCVTYXNRPK4HG%2F20210406%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=0cf0584878a6b7545a274637da757ebed8899424b7849f0b63d30442cfe25c20&hash=d331ccbf8042d88ab9fc252d88cbc4222efd5db01880de4cd51f2cfea85969fa&host=68042c943591013ac2b2430a89b270f6af2c76d8dfd086a07176afe7c76c2c61&pii=0167642387900359&tid=spdf-677946f4-9483-4f3f-8bef-e2cd58774ac0&sid=34e9e9da9959484ec31969417a8d858eb39bgxrqa&type=client">Statechart</a> concepts. Frame will continue to develop new "mechanisms" of various kinds that will become part of the notation which will not be clearly be in either of these camps. In doing so, it is hoped that Frame can expand the set of capabilities that are considered "behavior" by automata.

Here is a working demo of the `#ChangingState` spec:

<iframe width="100%" height="475" src="https://dotnetfiddle.net/Widget/xoBwQp" frameborder="0"></iframe>
