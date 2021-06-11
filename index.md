# # Current progress of Setting up firebase performance monitor


First thing is to ensure that `@react-native-firebase/app` is installed and set up;

Next install firebase performance monitor with 

```sh
$ npm install @react-native-firebase/perf
```

Which made me get this error

```
npm ERR! code ERESOLVE
npm ERR! ERESOLVE unable to resolve dependency tree
npm ERR! 
npm ERR! While resolving: Shreddy@0.0.1
npm ERR! Found: @react-native-firebase/app@10.8.1
npm ERR! node_modules/@react-native-firebase/app
npm ERR!   @react-native-firebase/app@"^10.4.0" from the root project
npm ERR! 
npm ERR! Could not resolve dependency:
npm ERR! peer @react-native-firebase/app@"12.0.0" from @react-native-firebase/perf@12.0.0
npm ERR! node_modules/@react-native-firebase/perf
npm ERR!   @react-native-firebase/perf@"*" from the root project
npm ERR! 
npm ERR! Fix the upstream dependency conflict, or retry
npm ERR! this command with --force, or --legacy-peer-deps
npm ERR! to accept an incorrect (and potentially broken) dependency resolution.
npm ERR! 
npm ERR! See /Users/chidera/.npm/eresolve-report.txt for a full report.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/chidera/.npm/_logs/2021-06-11T14_36_48_736Z-debug.log
```

Technically, `@react-native-firebase/app` is at `v10.4.0` but `@react-native-firebase/perf` needs `@react-native-firebase/app` to be at `v12.0.0`. 

Because of posibility of breaking changes and to override the error, I used 
```sh
$ npm install @react-native-firebase/perf --legacy-peer-deps
```

to install after consulting with Tim.

The next step will be to run 
```sh
$ cd ios/ && pod install
```

which I did and got this: 
```sh
...
Installing RNFBPerf 12.0.0
Installing RNFBStorage 10.4.0
Installing nanopb 2.30906.0 (was 2.30908.0)
[!] The 'Pods-Shreddy' target has libraries with conflicting names: libgoogleutilities.a.

[!] NPM package '@react-native-firebase/perf' depends on '@react-native-firebase/app' v12.0.0 but found v10.4.0, this might cause build issues or runtime crashes.

[!] NPM package '@react-native-firebase/perf' depends on '@react-native-firebase/app' v12.0.0 but found v10.4.0, this might cause build issues or runtime crashes.

[!] NPM package '@react-native-firebase/perf' depends on '@react-native-firebase/app' v12.0.0 but found v10.4.0, this might cause build issues or runtime crashes.
```

After a lot of digging around, I reckoned that the simplest solution is the one looking at me right now, which is 
```sh
[!] NPM package '@react-native-firebase/perf' depends on '@react-native-firebase/app' v12.0.0 but found v10.4.0, this might cause build issues or runtime crashes.
```
In essence, upgrade `'@react-native-firebase/app'` to `v12.0.0`

So I went ahead and did it
```sh
$ npm install @react-native-firebase/app@12.0.0
```

After installing, I had to run `pod install`, obviously with a `--repo-update` because of this kind of error 
```sh
You have either:
 [!] CocoaPods could not find compatible versions for pod "Firebase/CoreOnly":
  In snapshot (Podfile.lock):
    Firebase/CoreOnly (= 7.3.0, ~> 7.3.0)

  In Podfile:
    RNFBApp (from `../node_modules/@react-native-firebase/app`) was resolved to 12.0.0, which depends on
      Firebase/CoreOnly (= 8.0.0)


You have either:
 * out-of-date source repos which you can update with `pod repo update` or with `pod install --repo-update`.
 * changed the constraints of dependency `Firebase/CoreOnly` inside your development pod `RNFBApp`.
   You should run `pod update Firebase/CoreOnly` to apply changes you've made.
```

What cleared the error was
```sh
$ pod install --repo-update
```

> *P.S:* I made the mistake of looking at the end of the error message only because there were actually two errors there. This will hunt me later. 

The pod install --repo-update threw another error 
```sh
[!] CocoaPods could not find compatible versions for pod "Firebase/CoreOnly":
  In snapshot (Podfile.lock):
    Firebase/CoreOnly (= 7.3.0, ~> 7.3.0)

  In Podfile:
    RNFBApp (from `../node_modules/@react-native-firebase/app`) was resolved to 12.0.0, which depends on
      Firebase/CoreOnly (= 8.0.0)


You have either:
 * changed the constraints of dependency `Firebase/CoreOnly` inside your development pod `RNFBApp`.
   You should run `pod update Firebase/CoreOnly` to apply changes you've made.
```

After some poking, I found out that `Firebase/CoreOnly` previously at `v7.3.0` needs to be upgraded to `v8.0.0` because the new `@react-native-firebase/app@12.0.0` depends on it.

> Funny how that this error was contained in the previous error but I didn't see it. Haunted!

So I need to fix it with 
```sh
$ pod update Firebase/CoreOnly
```

Finally got a success message 
```sh
Pod installation complete! There are 122 dependencies from the Podfile and 147 total pods installed.
```

But... if you have trust issues like I did, you can run chief ol
```
$ pod install
```

for confirmation, I got

```
Pod installation complete! There are 122 dependencies from the Podfile and 147 total pods installed.
```

To complete I ran 
```sh
$ npx react-native run-ios
```
This rebuilds the ios and everything seems fine. 10 - 15 mins later, I could see the ios Performance metric on the Shreddy firebase console.

But...... to set up android,

I ran 

```sh 
$ npx react-native run-android
```

and here's a new error
```sh
> Configure project :react-native-firebase_analytics
:react-native-firebase_analytics package.json found at /Users/chidera/Documents/activeCode/TestApp/node_modules/@react-native-firebase/analytics/package.json
:react-native-firebase_app package.json found at /Users/chidera/Documents/activeCode/TestApp/node_modules/@react-native-firebase/app/package.json
ReactNativeFirebase WARNING: NPM package '@react-native-firebase/analytics' depends on '@react-native-firebase/app' v10.4.0 but found v12.0.0, this might cause build issues or runtime crashes.

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.5/userguide/command_line_interface.html#sec:command_line_warnings

FAILURE: Build completed with 2 failures.

1: Task failed with an exception.
-----------
* Where:
Build file '/Users/chidera/Documents/activeCode/TestApp/node_modules/@react-native-firebase/analytics/android/build.gradle' line: 71

* What went wrong:
A problem occurred evaluating project ':react-native-firebase_analytics'.
> No signature of method: firebase_json_8dc0dzjhhantm4hghu09dnzn9$_run_closure1.doCall() is applicable for argument types: (String) values: [analytics_auto_collection_enabled]
  Possible solutions: doCall(java.lang.Object, java.lang.Object), findAll(), findAll(), isCase(java.lang.Object), isCase(java.lang.Object)

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.
==============================================================================

2: Task failed with an exception.
-----------
* What went wrong:
A problem occurred configuring project ':react-native-firebase_analytics'.
> compileSdkVersion is not specified. Please add it to build.gradle
```

*Key error points are at:*

> ReactNativeFirebase WARNING: NPM package '@react-native-firebase/analytics' depends on '@react-native-firebase/app' v10.4.0 but found v12.0.0, this might cause build issues or runtime crashes.

> * What went wrong:
A problem occurred evaluating project ':react-native-firebase_analytics'.
> No signature of method: firebase_json_8dc0dzjhhantm4hghu09dnzn9$_run_closure1.doCall() is applicable for argument types: (String) values: [analytics_auto_collection_enabled]
  Possible solutions: doCall(java.lang.Object, java.lang.Object), findAll(), findAll(), isCase(java.lang.Object), isCase(java.lang.Object)

> * What went wrong:
A problem occurred configuring project ':react-native-firebase_analytics'.
compileSdkVersion is not specified. Please add it to build.gradle;


I need to fix this before proceeding to set up performance monitor for android. Any one got an idea? I guess it's related to `firebase analytics`.

This is a hard one. So after a lot of looking around github issues and stackoverflow without success, I decided to: Delete the current `"@react-native-firebase/analytics": "^10.4.0"`, and reinstall latest firebase analytics with

```sh
$ npm i @react-native-firebase/analytics
```

This pushed the version to `12.0.0`.

I then ran 
```
cd android && ./gradlew clean
```

Then 
```
$ cd .. && yarn android
```

I got a successful android build.

I also ran `npx pod-install` and `yarn ios` and got a successful ios build.

To continue performance set up for android, go to `/android/build.gradle` and add 
```sh
buildscript {
    dependencies {
        // ...
        classpath 'com.google.firebase:perf-plugin:1.4.0'
    }
```

Also go to the top of `/android/app/build.gradle` file and add 
```sh
apply plugin: 'com.google.firebase.firebase-perf'
```

Finally rebuild android
```sh
$ yarn android
```

And everything works fine and in the firebase console, I could see the android performance metric.

> Obviousely, firebase is extremely sensitive when it comes to versions.

