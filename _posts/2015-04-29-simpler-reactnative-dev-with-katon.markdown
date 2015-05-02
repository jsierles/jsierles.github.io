---
layout: post
title:  "Simplify React Native development with katon"
date:   2015-04-29 10:59:37
categories: reactnative
---
**Setup your React Native projects to run their packagers without port conflicts, for the simulator and devices, without having to change code when your IP address changes.**

For Ruby web development, we have [pow][pow], for working on mulitple apps painlessly. In short, it employs fake local domains to route HTTP requests to your application. But it's Ruby-only.

Enter [katon][katon], inspired by Pow. By passing a random port to any command you specify, katon runs service on the local .ka domain, and supports loading from other devices on the network.

So, it's perfect for React Native development!

### Use katon for React Native projects

This is a simplest way to try this method. See below for a fully automated solution.

#### Install katon

```
$ npm install -g katon
```

#### In your project directory, add your project to katon

```
$ katon add 'node_modules/react-native/packager/packager.sh --port $PORT' myapp
```

#### Change the application URL in AppDelegate.m to the katon URL

{% highlight objective-c%}
jsCodeLocation = [NSURL URLWithString:@"http://myapp.ka/index.ios.bundle"];
{% endhighlight %}

#### Build your project and run your app in the iOS simulator

Katon will automatically start up your packager.

If you need to restart the packager, just kill it.

```
$ katon kill myapp
```

See the packager logs to track down problems.

```
$ katon tail myapp

```

### Access from a device

To access these domains from another device, katon supports [xip.io][xip.io] URLs using your local IP address and app name. For example, ```myapp.192.168.0.10.xip.io``` resolves to 192.168.0.10.

Let's automate generating this URL in our XCode project, both for devices and the simulator.

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
