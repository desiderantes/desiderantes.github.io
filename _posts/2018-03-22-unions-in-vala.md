---
layout: post
title: Unions in Vala
date: 2018-03-22 02:32:050
published: true
comments: true
categories:
- Vala
tags:
- kotlin
- rust
- unions
author: desiderantes
---
I’ve started to use Kotlin professionally, and keeping an eye on Rust. Both offer a lot of niceties that I wish we could adapt for Vala, but there’s one that keeps popping up my mind everytime: pattern matching.

The simplest pattern matching we have is C unions, and a lot of c-libs use them. Unfortunately, the current handling of unions in Vala is a disgrace, and there’s no alternative for it. But I believe we can import some syntax from both Kotlin and Rust. Here is my proposal of how should unions work in Vala


```vala
//Opening a bracket defines an "anonymous struct"
public union OrderStatus {
    ACCEPTED,
    CANCELLED {string reason, string cancelled_by},
    REJECTED {string reason},
    COMPLETED {DateTime completed_at}
    ON_HOLD {string reason, Datetime until}
}
 
match (order.status) {
    ACCEPTED -> info("Cool!");
    CANCELLED -> debug(@"Not okay, it was cancelled because of $(it.reason) by $(it.cancelled_by)");
    REJECTED as that -> info (@"Rejected: $that.reason");
    default -> error("What is this?? There's no implicit \"it\" here because it's a catch-all!")
}
 
public union NestedUnion {
    SIMPLE,
    union COMPLEX {
        SOFT,
        HARD{string reason}
    }
}
 
//The additional field belongs to the wrapping struct
 
public union ComplexUnion {
    FIRST,
    SECOND,
    THIRD {string reason};
    //parent field, children cannot have a field with the same name
    uint32 timestamp;
}
 
//Maybe this is not a good idea
 
public union VeryComplex {
    FIRST {
        override void run() {
            log("Uy!")
        }
    },
    SECOND {
        override void run() {
            log("Ouch!");
        }
    },
    THIRD {
        override void run() {
            log("Ay!");
        }
        override void do() {
            debug ("Yay!");
        }
    }
     
    //They are all required to implement it!
    abstract void run();
     
    //Optionally overriden
    virtual void do() {
         
    }
    //Can't touch this
    public void execute() {
    }
}
 
//In this case, they reuse existing datatypes
 
public union ExternalUnion {
    STRING(string),
    NUMBER(uint64),
    THINGY(GLib.Object)
}
 
public void method () {
    var order = new Order(OrderStatus.ON_HOLD("reason", DateTime.now()));
    var other_order = new Order(OrderStatus.CANCELLED(cancelled_by = "desiderantes", reason = "who knows!")); 
     
    var nested = NestedUnion.COMPLEX.HARD(reason = "no reason at all");
    //'match' can return a value, but all branches should return the same type
    //this 'match' in particular is exhaustive, so no default needed, but if you return a value from 'match', you have to either
    //cover all cases or have a default branch
     
    NestedUnion another_nested = get_from_network();
    var reason = match (another_nested) {
        SIMPLE -> "Just because";
        COMPLEX -> match (it) {
            SOFT -> "Really easy";
            //if not renamed, then you'll lose access to the it from outer context, as it'll be shadowed
            HARD as that -> that.reason;
        }
    }
     
    //This errors
    var complex = ComplexUnion(123456789);
    var complex = ComplexUnion();
    var complex = ComplexUnion.FIRST();
 
    //This should work
    var complex = ComplexUnion.THIRD(123456789, "I can");
    var complex = ComplexUnion.THIRD(reason = "Just because", timestamp = 321654987);
     
     
    match (complex) {
        //properties from the parent are only accessible from the parent reference, no implicit parent var
        FIRST -> debug(@"$(complex.timestamp)");
        SECOND -> debug ("Oops");
        THIRD -> debug @("$(complex.timestamp) by $(it.reason)");
    }
     
    var external = ExternalUnion.STRING("this string is required");
}
```

The internal structure (C-wise) of my proposed tagged union is not anything new, it has been done a lot before in C land ([here is an
explanation from the Rust viewpoint](http://patshaughnessy.net/2018/3/15/how-rust-implements-tagged-unions))
