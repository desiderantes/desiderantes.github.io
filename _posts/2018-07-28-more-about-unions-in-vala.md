---
title: More about Unions in Vala
date: 2018-07-28 16:07:37
published: true
comments: true
status: published
categories:
- Programming
- Vala
tags:
- coding
- unions
author: desiderantes
---
[In a previous
post](/2018/03/22/unions-in-vala/)
I speculated about adding tagged unions to Vala. Let's do that again.

In order to support the C usecase, Vala would need to include 2 types of
tagged unions, a legacy one, and an idiomatic one. The legacy should
look like the C version of a tagged union:

```c
struct tagged {
    short tag;
    union variants {
        string text;
        int num;
    }
} tagged;
```

This is needed to bind the C libs out there. The type of the tag should be something that can be easily compared IMO (numeric datatypes, C enums, and string). The binding would then look like this:

```vala
[CCode (cname = "tagged", union_tag_field = "tag" union_tag_type = "short" union_field = "variants")]
union Tagged {
    [CCode (cname = "text", union_tag_id = "1")]
    TEXT(string),
    [CCode (cname = "num", union_tag_id = "2")]
    NUM(int)
}
```

The idiomatic ones, however, would actually look [like a Rust
struct](http://patshaughnessy.net/2018/3/15/how-rust-implements-tagged-unions#comment-3806350222),
so if we declare:

```vala
public union OrderStatus {
    ACCEPTED,
    CANCELLED {string reason, string cancelled_by},
    REJECTED {string reason},
    COMPLETED {DateTime completed_at},
    ON_HOLD {string reason, Datetime until}
}
```

We should get

```vala
enum OrderStatusTag {
    ACCEPTED,
    CANCELLED,
    REJECTED,
    COMPLETED,
    ON_HOLD
} OrderStatusTag
 
union order_status {
    struct accepted {OrderStatusTag tag};
    struct cancelled {OrderStatusTag tag; string reason, string cancelled_by};
    struct rejected {OrderStatusTag tag; string reason};
    struct completed {OrderStatusTag tag; GDateTime completed_at};
    struct on_hold {OrderStatusTag tag; string reason; GDatetime until};
} OrderStatus;
```

Fun things to support: GVariant _variant_ types (unboxing, serialization, etc.), GValue, JSON representations. 
