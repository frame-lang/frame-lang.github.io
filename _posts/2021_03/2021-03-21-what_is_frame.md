---
layout: post
title:  "What is Frame?"
date:   2021-03-21 00:00:00 -0800
categories: language-basics
---

Frame is a simple yet powerful _system specification language_ for defining the dynamic behavior of systems. With Frame notation it is easy to quickly design state machines that comply with core UML statechart concepts through a decidedly advantageous new approach.

## A Markdown Language For System Designers

UML and other modeling specifications promote a visual-first paradigm. However this approach to system design requires (sometimes expensive) diagramming and modeling tools. Additionally - let's just say it - working with boxes and lines to code can be a pain when the systems get complex.

With Frame, anyone with a text editor can quickly sketch out a system design concept - notepad is just fine!

`Frame`
```
#Lamp

    -interface-

    turnOn
    turnOff

    -machine-

    $Off
        |turnOn| -> $On ^

    $On
        |turnOff| -> $Off ^
##
```   

The true power of Frame, however, is realized by the ability to generate both documentation and code from Frame specification documents:

`UML`

![](http://www.plantuml.com/plantuml/png/SoWkIImgAStDuG8oIb8L_DFI5AgvQc6yF30dMYjMGLVN3YJ91SGWDaZAIa5DsT38nBgaj2ZFFm_2vWAAGvMYo0FvK0KEgNafGFi0)

`C#`
{% highlight csharp %}
    public partial class Lamp {
        public Lamp() {

            _state_ = _sOff_;
        }

        //===================== Interface Block ===================//

        public void turnOn() {
            FrameEvent e = new FrameEvent("turnOn",null);
            _state_(e);
        }

        public void turnOff() {
            FrameEvent e = new FrameEvent("turnOff",null);
            _state_(e);
        }


        //===================== Machine Block ===================//

        private void _sOff_(FrameEvent e) {
            if (e.Msg.Equals("turnOn")) {
                _transition_(_sOn_);
                return;
            }
        }

        private void _sOn_(FrameEvent e) {
            if (e.Msg.Equals("turnOff")) {
                _transition_(_sOff_);
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

## The Framepiler

Frame comes with a supporting transpiler called the {{site.data.site_config.framepiler_url}} (clever huh?). The Framepiler is written in Rust and is deployed online as a WebAssembly module on the page itself - no server interactions are required to transpile!

Additionally, the Framepiler is written as both a library as well as standalone executable. This allows Frame to support browser based use cases as well as to be included as part of arbitrary build toolchains.

## Code Generation

Frame currently supports generating UML, C#, C++ and JavaScript. In the future Frame will support a much wider range of target languages.

## Software Architecture and Engineering.  

Traditionally software projects struggle with maintaining code and documentation that must be kept in sync. With Frame, these activities become one.

### UML Statecharts

Frame notation directly supports the following statechart concepts and features:

- Hierarchical State Machines
- State Actions (enter/exit events)
- State History

It should be noted that Frame does not (and will not) adhere strictly to the UML and W3C standards. Although functionally compliant with the best parts of the specifications, Frame intends to break new ground not covered with those approaches.

### Systems Engineering

At a functional level, Frame currently provides the ability to easily restructure object-oriented classes to support state machines.

This capability, in and of itself, is interesting and valuable. However, by taking the viewpoint of these classes as <i>system controllers</i>, the larger utility and direction of Frame becomes vast and very exciting.

## The Future of Frame

The roadmap for Frame will be a journey that starts with the evolution of the language to define the capabilities and semantics of individual systems as implemented by object-oriented classes. Additionally, the project will explore other possible targets for system controllers than just object-oriented classes. These will include but are not limited to lower-level languages for embedded systems and hardware.

Looking into the future, the Frame project will explore how to design, generate and document <a href="https://en.wikipedia.org/wiki/System_of_systems#:~:text=System%20of%20systems%20is%20a,sum%20of%20the%20constituent%20systems." target="_blank">systems-of-systems</a> that control vastly more complex environments in a powerful and understandable way.
