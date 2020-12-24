---
title: 'Tip: Emitting the notify signal in Vala'
date: 2017-08-13 05:09:09
comments: true
published: true
categories:
- Vala
tags:
- glib
- gobject
author: desiderantes
---
I needed a way to emit the `notify` signal of one of my objects from another place, and Vala didn't show me a straightforward way to do it. If you need it for some reason, here's a code snippet showing you how:

```vala
using GLib;
 
public class TestClass : GLib.Object {
    public string test1 {get;set;default = "test1";}
    //Ths one won't emit on assignment
    [CCode (notify = false)]
    public string test2 {get;set;default = "test2";}
 
    public static void main (string[] args) {
        var test = new TestClass ();
        test.notify["test1"].connect (() => GLib.print ("test1 notification\n"));
        test.notify["test2"].connect (() => GLib.print ("test2 notification\n"));
        test.test1 = "Ahoy";
        test.test2 = "Újale";
        test.test2 = "Ajúa";
        ParamSpec pspec = ((ObjectClass) typeof (TestClass).class_ref ()).find_property ("test2");
        GLib.print ("First try, will not work %s\n", pspec.name);
        test.notify(pspec);
        GLib.print ("second one, this is how it works");
        test.notify["test2"] (pspec);
        return;
    }
}
```
