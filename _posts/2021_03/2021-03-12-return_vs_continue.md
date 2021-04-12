---
layout: post
title:  "Frame Event Handler Termination"
date:   2021-03-12 00:00:00 -0800
categories: language-basics
---

Frame event handlers are terminated with two operators:

|Operator | Meaning|
|:-:|:-:|
|^|return|
|:>|continue|

Return is, by far, the most common of the two. Lining up the Frame notation with the C# code it generates we can see how they align:

<table>
    <tr>
        <th style="text-align:center;">Frame Notation</th>
        <th style="text-align:center; ">C# code</th>
    </tr>
    <tr>
        <td style="vertical-align:top;">
<pre >
$EventTerminators
    |return|
        ^

    |continue|
        :>
</pre>
        </td>
        <td>
<pre align="left">
private void _sEventTerminators_(FrameEvent e) {
    if (e._message.Equals("return")) {
        return;
    }
    else if (e._message.Equals("continue")) {

    }
}
</pre>
        </td>
    </tr>
</table>

Note that the continue is, from a code generation standpoint, a no-op in that it doesn't generate anything. This enables the execution to simply fall through to whatever code is below the tests for event handler messages.

This approach enables a simple way for a child state event handler to do something and then pass the event on for further processing by the parent state:
<table>
    <tr>
        <th style="text-align:center;">Frame Notation</th>
        <th style="text-align:center; ">C# code</th>
    </tr>
    <tr>
        <td style="vertical-align:top;">
<pre >

#ReturnVsContinue





    -interface-

    _return @(|return|)




    _continue @(|continue|)





    -machine-

    $Child => $Parent
        |return|
            log("saw return in $Child")
            ^

        |continue|  
            log("saw continue in $Child")
            :>





    $Parent
        |return|
            log("saw return in $Parent")
            ^

        |continue|
            log("saw continue in $Parent")
            :>



    -actions-

    log [msg:string]
##
</pre>
        </td>
        <td>
<pre align="left">
// emitted from framec_v0.3.39
public partial class ReturnVsContinue {
    public ReturnVsContinue() {

        _state_ = _sChild_;
    }

    //===================== Interface Block ===================//

    public void _return() {
        FrameEvent e = new FrameEvent("return",null);
        _state_(e);
    }

    public void _continue() {
        FrameEvent e = new FrameEvent("continue",null);
        _state_(e);
    }


    //===================== Machine Block ===================//

    private void _sChild_(FrameEvent e) {
        if (e._message.Equals("return")) {
            log_do("saw return in $Child");
            return;
        }
        else if (e._message.Equals("continue")) {
            log_do("saw continue in $Child");

        }
        _sParent_(e);

    }

    private void _sParent_(FrameEvent e) {
        if (e._message.Equals("return")) {
            log_do("saw return in $Parent");
            return;
        }
        else if (e._message.Equals("continue")) {
            log_do("saw continue in $Parent");

        }
    }

    //===================== Actions Block ===================//

    protected virtual void log_do(string msg) { throw new NotImplementedException(); }


    //=============== Machinery and Mechanisms ==============//

    private delegate void FrameState(FrameEvent e);
    private FrameState _state_;

}
</pre>
        </td>
    </tr>
</table>

The \_return and \_continue interface methods above are prefixed with a "_" as both return and continue are reserved words in many target languages. We use the alias notation to send the `|return|` and `|continue|` messages.

This code can be seen running in this online demo:

<iframe width="100%" height="475" src="https://dotnetfiddle.net/Widget/Qq69UX" frameborder="0"></iframe>
