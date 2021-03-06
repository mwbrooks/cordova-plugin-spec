# Cordova Plugin Specification

Last edited June 8, 2012

## About this document

This document defines a format for distributing and packaging plugins for use
with Apache Cordova, nee PhoneGap. It is based on the work done as part of the
PhoneGap Build (https://build.phonegap.com) web service, where many of the tools
and techniques have been implemented.

This is not an official document of the Apache Cordova project. Issues and pull
requests are welcome, and can be submitted through GitHub.

## Sample Tooling

* __pluginstall__: a node.js based tool to install compliant plugins
  * https://github.com/alunny/pluginstall

## Sample Plugins

* __ChildBrowser__
  * https://github.com/alunny/ChildBrowser
* __PG-SQLite Plugin__
  * https://github.com/ApplicationCraft/PGSQLitePlugin

## Design Goals

The goals of this format are:

* facilitate programmatic installation and manipulation of plugins
* detail the dependencies and components of individual plugins
* allow code reuse between different target platforms

## Structure of a plugin

A plugin is typically a combination of some web/www code, and some native code.
However, plugins may have only one of these things - a plugin could be a single
JavaScript, or some native code with no corresponding JavaScript.

Assuming both web code and native code are present, the plugins should be
structured like so:

    foo-plugin # top-level directory
    -- plugin.xml # xml-based manifest, described below
    -- src # native source-code, to be compiled for a single platform
      -- android
        -- Foo.java
      -- ios
        -- CDVFoo.h
        -- CDVFoo.m
    -- www # assets to be copied into the www directory of the Cordova project
      -- foo.js
      -- foo.png

## Plugin Manifest Format

The `plugin.xml` file is an XML document in the plugins namespace -
`http://www.phonegap.com/ns/plugins/1.0`. It contains a top-level `plugin`
element defining the plugin, and children that define the structure of the
plugin.

A sample plugin element:

    <plugin xmlns="http://www.phonegap.com/ns/plugins/1.0"
        xmlns:android="http://schemas.android.com/apk/res/android"
        id="com.alunny.foo"
        version="1.0.2">

### &lt;plugin&gt; element

The `plugin` element is the top-level element of the plugin manifest. It has the
following attributes:

#### xmlns (required)

The plugin namespace - `http://www.phonegap.com/ns/plugins/1.0`. If the document
contains XML from other namespaces - for example, tags to be added ot the
AndroidManifest.xml file - those namespaces should also be included in the
top-level element.

#### id (required)

A reverse-domain style identifier for the plugin - for example, `com.alunny.foo`

#### version (required)

A version number for the plugin, that matches the following major-minor-patch
style regular expression:

    ^\d+[.]\d+[.]\d+$

### Child elements

All child elements of the &lt;plugin&gt; element are optional.

### &lt;name&gt; element

A human-readable name for the plugin. The text content of the element contains
the name of the plugin. An example:

    <name>Foo</name>

At this point in time, the tools prototyped for this format do not make use of
this element. If this document progresses, localization will also need to be
accounted for.

### &lt;asset&gt; element

One or more elements listing the files or directories to be copied into a
Cordova app's `www` directory. A couple of samples:

    <!-- a single file, to be copied in the root directory -->
    <asset src="www/foo.js" target="foo.js" />
    <!-- a directory, also to be copied in the root directory -->
    <asset src="www/foo" target="foo" />

All assets tags require both a `src` attribute and a `target` attribute.

#### src (required)

Where the file or directory is located in the plugin package, relative to the
`plugin.xml` document.

#### target (required)

Where the file or directory should be located in the Cordova app, relative to
the `www` directory.

Assets can be targeted to subdirectories - for instance:

    <asset src="www/new-foo.js" target="js/experimental/foo.js" />

would create the `js/experimental` directory in the `www` directory, if not
present, and then copy the file `new-foo.js` as `foo.js` into that directory.

If a file exists at the target location, tools based on this specification
should stop the installation process and notify the user of the conflict.

### &lt;platform&gt;

Platform tags identify platforms that have associated native code. Tools using
this specification can identify supported platforms and install the code into
Cordova projects.

A sample platform tag:

    <platform name="android">
    <!-- android specific elements -->
    </platform>
    <platform name="ios">
    <!-- ios specific elements -->
    </platform>

#### name (required)

The `name` attribute identifies a platform as supported - it also associates the
element's children with that platform.

Platform names should be all-lowercase. Platform names, as arbitrarily chosen,
are listed:

* android
* ios

Tools may accept alternate names for platforms. Whatever.

### &lt;source-file&gt;

`source-file` elements identify executable source code that should be installed
into a project. A couple of examples:

    <!-- android -->
    <source-file src="src/android/Foo.java"
                    target-dir="src/com/alunny/foo" />
    <!-- ios -->
    <source-file src="CDVFoo.m" />

As with assets, if a `source-file` would overwrite an existing file, tools
should notify the user and stop, like, right away.

#### src (required)

Where the file is located, relative to the `plugin.xml` file.

#### target-dir

A directory where the files should be copied into, relative to the root of the
Cordova project.

In practice, this is most important for Java-based platforms, where a file in
the package `com.alunny.foo` has be located under the directory
`com/alunny/foo`. For platforms where the source directory is not important,
plugin authors should omit this attribute.

### &lt;config-file&gt;

Identifies an XML-based configuration file to be modified, where in that
document the modification should take place, and what should be modified.

At this stage in the spec, the `config-file` element only allows for appending
new children into an XML document. The children are XML literals that are the
to be inserted in the target document.

Example:

    <config-file target="AndroidManifest.xml" parent="/manifest/application">
        <activity android:name="com.foo.Foo"
                  android:label="@string/app_name">
                  <intent-filter>
                  </intent-filter>
        </activity>
    </config-file>

#### target

The file to be modified, and the path relative to the root of the Cordova
project.

If this file does not exist, tools should self-destruct over a period of ten
minutes. Within the first five minutes, the self-destruct procedure MAY be
cancelled with the appropriate security clearance. After five minutes have
passed, the self-destruct procedure MUST complete.

Or they could just stop.

#### parent

An absolute XPath selector pointing to the parent of the elements to be added to
the config file.

### &lt;plugins-plist&gt;

Specifies a key and value to append to the correct `AppInfo.plist` file in an
iOS Cordova project. Example:

    <plugins-plist key="Foo"
                   string="CDVFoo" />

This may be an implementation detail leaking through, and could be merged with
the `config-file` element at some later point.

### &lt;resource-file&gt; and &lt;header-file&gt;

Like source files, but specifically for platforms that distinguish between
source files, headers, and resources (iOS)

Examples:

    <resource-file src="CDVFoo.bundle" />
    <resource-file src="CDVFooViewController.xib" />
    <header-file src="CDVFoo.h" />

This is probably an implementation detail leaking through, and future versions
of this document will likely merge these elements with `source-file`.

### &lt;framework&gt;

Identifies a framework (usually part of the OS/platform) that the plugin depends
on. Example:

    <framework src="libsqlite3.dylib" />

Tools should identify the framework through the `src` attribute and attempt to
add the framework to the Cordova project, in the correct fashion for a given
platform.

## Authors

Andrew Lunny

## License

MIT

Copyright 2012 Andrew Lunny, Adobe Systems
