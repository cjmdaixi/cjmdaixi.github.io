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

## Compiling QtWasm from source

Before writing the codes, we need to setup the development toolset. There is no pre-built binaries available for QtWasm in the official releases. We have to compile it from the source. I've compiled successfully both on MacOS and Ubuntu for Windows Sub-system Linux(WSL for short). Let me brief the procedures on WSL:

1. Install WSL. There are lots of guide around, here is one example: [How to install Windows 10's Linux Subsystem on your PC](https://www.onmsft.com/news/how-to-install-windows-10s-linux-subsystem-on-your-pc). Please note that you should install newer version of WSL, e.g. Ubuntu 18.04 instead of earlier versions.
2. Install emscripten.  Refer to the official installing guide: [Emscripten download and install](https://kripken.github.io/emscripten-site/docs/getting_started/downloads.html). You can verify the installation by invoking the command `em++ --version`. If it complains with errors, something must go wrong, and you should try again.
3. Download & unzip the source of Qt 5.12.0. Please make sure to choose the correct version with linux newline ending. You can try is command: `wget https://download.qt.io/official_releases/qt/5.12/5.12.0/single/qt-everywhere-src-5.12.0.tar.xz`.
4. Compile the source according to the official post: [Getting Started With Qt for WebAssembly](http://blog.qt.io/blog/2018/11/19/getting-started-qt-webassembly/). It takes 30 minutes to 2 hours depending on your computer speed.

If things go well, you will have a compiled QtWasm ready to code.

## Common steps for coding QtWasm

Next we can start to write our awesome QtWasm apps. Here are my usual steps for QtWasm coding: 

1. Use the Qt Creator on your Windows to create, write and even debug your app. Please choose qmake as the build tool other than CMake or qbs.
2. Turn to WSL when it's ready. Compile the project using QtWasm's qmake. It will generate several related wasm, js and html files.
3. Setup a http server, e.g. Python http module, in the build directory. For Python2, the command is: `python -m SimpleHTTPServer 8080` and for Python 3 the command is: `python -m http.server 8080`.
4. Test your QtWasm app through browsers. If everything goes well, your little QtWasm app should run happily on your browsers now.

## Speed up the compilation

Excited about the shown up of your first QtWasm app? Soon or later you'll find it intolerant for the compilation when exploring other features of QtWasm. It almost takes 10 minutes to compile even only one line of code is modified. The problem exists both on my MacBook Pro and WSL on my desktop PC.

It has to be handled before we could do some really funny things with QtWasm. I am quite curious that haven't the Qt guys been facing the same problems? Why didn't they mention it?

After trials and errors, I came up with a solution. The core idea is to use `Loader`to dynamically load the QML files.

Here come the details. If we develop a Qt Quick app, the C++ part, or the so called loader part, is not changed so often as the UI part. We separate these two parts into two projects. The loader part needs compiling with QtWasm. It's still time-consuming, but as we said, it don't need recompilation that often. We add a `Loader` in this part, to load the QML files on the same server dynamically.  This way, whenever the QML files are changed, our QtWasm app will load the new UI after refresh the web page.

Honestly speaking, I haven't handle the compilation problem itself. Rather I separate the app into two parts. The loader part is essentially a qml engine or qmlscene on the QtWasm. Please note that due to the security restriction of the browsers, you can only put your QML files under the same domain as the QtWasm files.

## How to show pictures by `Image`?

By separating our QML into different part, we can rapidly test our new UI on the browsers. But you'll finally find that the `Image` won't work as usual. It cannot load pictures on the Internet. Why?

When loading Internet pictures, `Image` always work in an async way by starting separate threads. However, webassembly does not support multi-thread by now (due to security problems), `Image` just stops working.

One solution I came up is to compile all the pictures you need into the loader part via a qrc file, and then set the `source` of `Image` as the local urls starting with "qrc:/".

It's hard to prepare all the pictures in advance when developing complex app. But it is reasonable to do it several times.

## Where are my Chinese characters?

It's another annoying problems which bothers me quite a long time. I found the reason on the official [wiki](https://wiki.qt.io/Qt_for_WebAssembly) finally:

> Applications do not have access to system fonts. Font files must be distributed with the application, for example in Qt resources. Qt for WebAssembly itself embeds one such font.

Regarding with font embedding, it's OK for latin characters, but it's unacceptable for other languages, e.g. Chinese, whose font files can be easily larger than 40M.

Even though we could tolerate the massive size, the compilation won't succeed as I have tried many times.

Finally I came up with the idea that although the Chinese font is large, the actual characters used in one particular app is limited. How about extracting the characters we use and only embed this subset font into our app?

I found a little while power tool to do this: [Fontzip](https://github.com/forJrking/FontZip). I use it to extract the subset from a free Chinese font called [WenQuanYi Micro Hei Mono](https://www.freechinesefont.com/wenquanyi-micro-hei-download/). The font size is reduced to only 100KB from 40MB. 

We can then use the new font in our QML as:

```qml
FontLoader{
    id: chineseFont
    source: "qrc:/fonts/fontzipmin.otf"
}
```

And then set the font faimily as the `chineseFont.name` wherever we want to use this font:

```qml
Label{
    font{family: chineseFont.name}
    text: "你好"
}
```

Please note that if you want to show characters other than Latin on the UI part, you should use the Unicode codepoint instead of directly typing the characters, because the loader part will complains about strange encoding errors. For example, if you want to show "你好我们", you should code like this:

```qml
Label{
    id: label
    anchors.centerIn: parent
    text: "\u4f60\u597d\u6211\u4eec"
}
```

I recommend two plugins if you'd like VS code:

1. Unicode code point of current character. This plugin can convert the character under the cursor into Unicode code point.
2. unicodeToChinese. This plugin helps you convert the code point back to Chinese characters if you forget the original strings.

Of cause you can also try some online services like [unicode.scarfboy.com](http://unicode.scarfboy.com/).

## Examples

I've submit my demo to Github: [QtWasmLoader](https://github.com/cjmdaixi/QtWasmLoader). The source code of the loader part is in the src branch, and the compiled loader and the QML files are put into the master branch.

I also turned on the Github Pages, so you can see the result directly on your own browsers: [cjmdaixi.github.io/QtWasmLoader](https://cjmdaixi.github.io/QtWasmLoader/):

![img](../assets/images/qtwasmloader.jpg)

The QML content is my another repo: [DarkSwitch](https://github.com/cjmdaixi/DarkSwitch).

When cloning my repo and trying to test your own QtWasm app, please make sure that the URL has been set accordingly:

```qml
Loader{
    id: mainView
    anchors.fill: parent
    source: "Your own Github Pages URL/UI/MainView.qml"
    onLoaded: {
        loadingItem.visible = false;
    }
}
```

That's all. Thanks! Happy QtWasm coding!