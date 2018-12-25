---
layout: post
title:  "Qt Quick on the Browser"
date:   2018-12-24 20:14:51 +0800
categories: qt qml webassembly
---

Do you want to run your lovely Qt Quick app on the browsers? Before Qt 5.12, the only way is to use the [Qt WebGL streaming plugin](https://blog.qt.io/blog/2018/11/23/qt-quick-webgl-release-512/). It is, however, just a mirror of the UI and suffers from some problems which will be mentioned later. Starting with Qt 5.12, a new module emerges, the Qt for WebAssembly(QtWasm for short). Briefly speaking, it's a trial to compile the Qt framework to webassembly, a new form of bytecode that can run on the browsers. This post will explan how to write your first QtWasm app in details, and try to provide solutions to some problems you will inevitably encounter.

## Benefits of QtWasm

Before diving into the details, let's talk about the goals QtWasm wants to achieve.

With the development of web technology, some kinds of web apps, like video games and graphics editors, quickly reach their bottleneck, the performance of JavaScript. Webassembly is a new standard of bytecode to solve this problem. It runs much faster than js, sometimes even close to native codes. It is now supported by the 4 major browsers: Chrome, Firefox, Edge and Safari on all the main operation systems.

Qt is a mature C++ framework. It contains IO operations, GUI, network, almost all the aspects that can facilitate us to build an app rapidly. By compiling Qt to webassembly, such a complex framework can now run smoothly on our browsers, changing the famous slogan of Qt from "Write once, compile & run everywhere" to "Compile once, run everywhere". Quite familous, right? That's the Java slogan! The client/server structure makes the distribution and upgrading be an trivial task.

## Current methods to run Qt on the browsers

There is no ways to to run Qt Widges on the browsers AFAIK. Two relative methods are found to run Qt Quick:

1. Qt WebGL Streaming. The official introduction is here: [Qt Quick WebGL release in Qt 5.12](http://blog.qt.io/blog/2018/11/23/qt-quick-webgl-release-512/). As mentioned previously, it's a Qt platform plugin that convert the OpenGL ES commands of Qt Quick to WebGL streaming, then execute on the customers' browsers. It likes a remote desktop, which means all the real execution is still conducted on the local device, interaction with the browsers is kind of impossible. And while being fluent for local network, it becomes laggy for non-local connection.
2. Compiling QML into JavaScript. A typical one is [qmlweb](https://github.com/qmlweb/qmlweb)。There are no offical releases of this kind. The QML is not really running on the browsers, therefore it lacks the power of Qt in other fields.

As a conclusion, QtWasm is currently the only method for Qt to running on the browsers.