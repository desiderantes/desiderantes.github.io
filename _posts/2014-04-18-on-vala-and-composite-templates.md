---
title: On Vala and Composite Templates
date:   2014-04-18 00:19:27
published: true
comments: true
categories:
- Vala
tags:
- composite templates
- glade
- gresources
author: desiderantes
---
If you want to use the GTK+ composite templates in Vala, please remember: you *have* to use a [GResource](https://developer.gnome.org/gio/2.32/gio-GResource.html "GResources on GNOME"). The syntax is pretty simple, here's an example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
  <gresources>
    <gresource prefix="/org/gtk/Example">
      <file compressed="true">window.ui</file>
    </gresource>
  </gresources>
```

Let's say you called that file project.gresource.xml, to use it in your project you need to pass a `gresource` at compile time:

    valac --target-glib=2.38 --gresources project.gresource.xml --pkg your_package1 --pkg you_package2 ...

Of course, it can't be that simple, so first you have to _compile_
that xml file (crazy, isn't it?), doing `glib-compile-resources project.gresource.xml` and then include the generated C file. As this won't scale, and you probably aren't compiling without a build system, the [documentation](https://developer.gnome.org/gio/2.32/glib-compile-resources.html) suggest, in the case of make, to include a rule, which looks like this:

    project-gresource.c: project.gresource.xml $(shell $(GLIB_COMPILE_RESOURCES) --generate-dependencies project.gresource.xml)

and we can include the generated C file in our valac call:

    valac --target-glib=2.38 --gresources project.gresource.xml --pkg your_package1 --pkg you_package2 file1.vala file2.vala project-gresource.c ...

Please don't forget to read the announcement [here](http://blogs.gnome.org/tvb/2013/05/29/composite-templates-lands-in-vala/) for more info.
