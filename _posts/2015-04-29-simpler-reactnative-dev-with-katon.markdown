---
layout: post
title:  "Simplify React Native development with katon"
date:   2015-04-29 10:59:37
categories: reactnative
---
**In this post, we'll learn how to use [katon][katon] to automate the web packager for multiple React Native applcations.**

For Ruby web development I use the excellent [pow][pow] for launching my Rails apps painlessly. In short, it uses fake local domains to route HTTP requests to your application. But it's Ruby-only.

Enter [katon][katon], inspired by Pow. By passing a random port to any command you specify, katon runs service on the local .ka domain.

So this is perfect for React Native development!

### Use katon for React Native projects

First, install katon.

```
$ npm install -g katon
```

In your project directory, add your project to katon.

```
$ katon add 'node_modules/react-native/packager/packager.sh --port $PORT' myapp
```

Finally, change the application URL in AppDelegate.m to the katon URL.

{% highlight objective-c%}
jsCodeLocation = [NSURL URLWithString:@"http://myapp.ka/index.ios.bundle"];
{% endhighlight %}

Now build your project and run your app in the iOS simulator. Katon will automatically start up your packager.

If you need to restart the packager, just kill it.

```
$ katon kill myapp
```

Tail the packager logs to track down problems.

```
$ katon tail myapp

```

### Access from a device

To access these domains from another device, katon supports [xip.io][xip.io] URLs using your local IP address and app name.

So let's automate this in our XCode project by:

* Adding the katon command using our project name
* Discovering our IP address
* Modifying our source code to send devices (and the simulator) to the correct domain

To do this, follow these steps, outlined in the video:

#### Remove the default React packager script

In your XCode project, click on *Libraries -> React*. In the main window, under *Build Phases*, remove the entry named *Run Script*.

#### Add our new build script

In our main project, under *Build Phases*, add a new entry of *Run Script* type.

Use *bin/bash -l* as the shell, in case we're using nvm to manage nodejs, then paste this code:

```bash
APP_NAME=`echo $PROJECT_NAME | tr '[:upper:]' '[:lower:]'`
katon add 'node_modules/react-native/packager/packager.sh --port $PORT' $APP_NAME
JS_URL="http://${APP_NAME}.`ipconfig getifaddr en0`.xip.io"
echo "#define JSURL @\"$JS_URL/index.ios.bundle\"" > ${SRCROOT}/iOS/JSUrl.h
```

This creates a source file named JSUrl.h, containing a macro with our IP-based URL. If you don't see your IP, check if you can retrieve it using this command: `ipconfig getifaddr en1`.

Finally, update your AppDelegate.m to use the newly defined macro.

{% highlight objective-c%}
jsCodeLocation = [NSURL URLWithString:JSURL];
{% endhighlight %}

Build and run your project - you should be up and running!

### Possible issues

If you already use Pow, xip.io URLs may conflict with it. Since you can use katon for Ruby apps as well, I'd recommend replacing pow with katon.

Happy coding!

[xip.io]: http:/xip.io
[pow]: http://pow.cx
[katon]: https://github.com/typicode/katon