---
layout: post
title:  "Beyond UML"
date:   2021-02-15 00:00:00 -0800
categories: language-basics UML
---

The **Unified Modeling Language (UML)** is an extensive amalgamation of modeling concepts and artifact types initially developed in 1995 by Grady Booch, Ivar Jacobson, and James Rumbaugh while working at Rational Software. Also included in the UML spec was prior work done by Dr. David Harel on the <a href="https://www.sciencedirect.com/science/article/pii/0167642387900359" target="_blank">Statechart</a>,

Frame owes its lineage to Statecharts, but builds upon them with significant differences in philosophy and implementation.

1. Frame is a _textual system specification language_ that can be translated precisely into both code and documentation. In contrast, Statecharts are declared to be a visual formalism and explicitly eschew a textual approach to software modeling.
2. Frame is precise in the meaning of its notation and defines a reference implementation for each of its features.
3. Frame introduces new semantics with states and transitions not included in UML.
4. Frame aspires (an admittedly anthropomorphic statement) to be an information dense _symbolic_ language. While Frame has not realized this goal fully, it is an important part of the charter of the language to evolve in that direction.

We will now discuss the current big new ideas in Frame that push beyond the boundaries of UML notation. Some of these features require the use of a new coding mechanism called the `StateContext` for implementation, which we will hold off discussing until after all of the new features have been introduced.

## Enter Event Parameters

One of the distinct differences of programming _state-oriented software_ is the challenge of passing data between states. This situation can arise when an event occurs in one state that includes data that needs to be processed in another. The usual, very workable solution is to simply cache the data somewhere, transition and then pick the data up in the new state.

In the example below data comes in as a paramter with the `|someEvent|` message. It is then stored off in a member variable and followed by the system transitioning to the `-> $UseData` state to consume it:

```
#CacheData

    -interface-

    someEvent [datum:string]

    -machine-

    $ReceiveData
        |someEvent| [datum:string]
            #.datumCache = datum
            -> $UseData ^

    $UseData
        |>|
            doSomething(#.datumCache) ^

    -actions-

    doSomething[datum:string]

    -domain-

    var datumCache:string = null

##
```
Upon entry to the `$UseData` state the machine processes the datum when calling the `doSomething(#.datumCache)` action. All well and good, but this approach can be cumbersome.

As a more elegant approach, Frame introduces _transition parameters_.

## Enter Event Parameters

Instead of ad-hoc caching, Frame provides notation to directly pass arguments as part of a transition:

`-> (<enter_argument_list>) $NewState`

For instance:

`-> ("Mark") $PrintName`

In `$NewState` the arguments are passed to the enter event handler:

```
    $PrintName
        |>| [name:string]
            print(name) ^
```

Here is a full system specification using this feature:

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

Passing data to a new state via a transition is a common and useful feature. For symmetrical completeness, Frame supports exit event parameters as well.

## Exit Event Parameters

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

Building on the previous full example we can see how `$State` handles both enter and exit transition arguments:

`Frame`
```
#ExitEventParameters

    -interface-

    start @(|>>|)

    -machine-

    $Begin
        |>>| -> ("Hello $State") $State  ^

    $State
        |>| [greeting:string]
            print(greeting)
            ("Goodbye $State") -> $End ^
        |<| [farewell:string]
            print(farewell) ^

    $End

    -actions-

    print[message:string]

##
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

## Conclusion

This article gives an introduction to new Frame syntax that breaks begins to move beyond the core ideas presented in Statecharts. These capabilities overcome some notational sharp edges in Statecharts and simplify system design for the architect. This makes it easier to focus more on the solution being designed and not overcoming typical friction points when working with state machine design.

In the next article I will explore in depth the mechanisms that support these new features.
