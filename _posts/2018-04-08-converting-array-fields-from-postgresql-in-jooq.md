---
title: Converting Array fields from PostgreSQL in jOOQ
date: 2018-04-08 04:49:03
published: true
comments: true
categories:
- Programming
tags:
- java
- jOOQ
- PostgreSQL
author: desiderantes
---
There's currently a "bug" (not really a bug, but it works for me in MySQL) that prevents jOOQ from reusing a forcedType when it appears as an array instead of a single value. I have a workaround for this: a forcedType for the array field, and binding it by field name. To facilitate the creation of an Array Converter, just subclass this and pass the right converter in the `super()` call:

```java

package com.github.desiderantes.jooq;


import org.jooq.Converter;

import java.lang.reflect.Array;
import java.util.ArrayList;
import java.util.List;

public class ArrayConverter<T,U> implements Converter<T[], U[]> {

    private Converter<T,U> converter;

    public ArrayConverter(Converter<T, U> converter) {
        this.converter = converter;
    }

    @Override
    @SuppressWarnings("unchecked")
    public U[] from(T[] databaseObject) {
        List<U> list = new ArrayList<>();
        for (T type : databaseObject) {
            U from = converter.from(type);
            list.add(from);
        }
        return (U[]) list.toArray();
    }

    @Override
    @SuppressWarnings("unchecked")
    public T[] to(U[] userObject) {
        List<T> list = new ArrayList<>();
        for (U type : userObject) {
            T to = converter.to(type);
            list.add(to);
        }
        return (T[]) list.toArray();
    }

    @Override
    @SuppressWarnings("unchecked")
    public Class<T[]> fromType() {
        return (Class<T[]>) Array.newInstance(converter.fromType(), 0).getClass();
    }

    @Override
    @SuppressWarnings("unchecked")
    public Class<U[]> toType() {
        return (Class<U[]>) Array.newInstance(converter.toType(), 0).getClass();
    }
}

```


