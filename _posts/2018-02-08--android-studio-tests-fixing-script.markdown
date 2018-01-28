---
title: "Moviper - the Android VIPER Library"
date: "2017-02-08 08:38:09 +0100"
header:
  overlay_color: "#333"
  overlay_image: /assets/images/image2.jpg
  cta_label: "See the Library"
  cta_url: "https://github.com/mkoslacz/moviper"
  teaser: /assets/images/image2.jpg
  overlay_filter: 0.5
  caption: "Photo credit: [**Aaron Burden @ Unsplash**](https://unsplash.com/@aaronburden)"
excerpt: "A walkthrough that will let you start using Moviper."
tags:
  - Android
  - Architecture
  - Moviper
  - VIPER
  - Kotlin
  - ReactiveX
  - RxJava
gallery:
  - url: /assets/images/LoginActivityLayout.png
    image_path: /assets/images/LoginActivityLayout.png
    alt: "Login screen screenshot from app created in this post."
    title: "Login screen"
  - url: /assets/images/ProfileActivityLayout.png
    image_path: /assets/images/ProfileActivityLayout.png
    alt: "Fake profile screen screenshot from app created in this post."
    title: "Fake profile screen"
  - url: /assets/images/HelpActivityLayout.png
    image_path: /assets/images/HelpActivityLayout.png
    alt: "Fake help screen screenshot from app created in this post."
    title: "Fake help screen"
---
Me and my team, we use TDD in our projects. It's a great methodology to create your code, and I believe that great code HAS to be tested. But I won't talk about it today, as you can find plenty of articles that describes qualities of this approach.

Our problem was that in our intntense-TDD environment using Dagger2 that generates, and with several developers commiting their code to various branches, we found that Android Studio often has problems with executing tests.

The infamous error codes like

```
!!! JUnit version 3.8 or later expected:

java.lang.RuntimeException: Stub!
	at junit.runner.BaseTestRunner.<init>(BaseTestRunner.java:5)
	at junit.textui.TestRunner.<init>(TestRunner.java:54)
	at junit.textui.TestRunner.<init>(TestRunner.java:48)
	at junit.textui.TestRunner.<init>(TestRunner.java:41)
	at com.intellij.rt.execution.junit.JUnitStarter.junitVersionChecks(JUnitStarter.java:233)
	at com.intellij.rt.execution.junit.JUnitStarter.canWorkWithJUnitVersion(JUnitStarter.java:216)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:75)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:147)

Process finished with exit code 253
```

Or

```
Process finished with exit code 1
Class not found: "com.yourcodebase.Class"Empty test suite.
```

Or another

```
No tests were found.
```

Or maybe when you try to run a single test, a whole test class is run, what makes debugging harder.

The mentioned problems appear often when changing branches, and less frequently randomly in the middle of the development. Anyway, it is a real pain in the neck, as it interrupts a TDD-cycle. We don't know what causes these problems, but we have learnt how to deal with them temporarily:

- Remove old test configurations, (insert Images)
- Set `MODULE_DIR` tests working directory (insert Images)
- Rebuid project
- Invalidate Caches / Restart

Actually, it is a generic way to fix your tests - sometimes only one of these steps works and makes your tests run again, but we got used to performing all of these steps when the problem appears to avoid waiting for another gradle test run which could fail.

The problem is that this whole process takes time. In our project it usually lasts ~5min, but it's more than enough to make you distracted. Unfortunatelly, we couldn't find a better solution for that (which idelly would be to completely avoid these problems instead of fighthing them).


But still, the world isn't perfect, and our simple procedure to refresh our test suite doesn't always work. There were still incidents in which someone was trying to run his tests without success for many hours. The first main fallback was to simply restart the computer. But still it sometimes doesn't help. We realized that removing a .gradle folder inside our project sometimes work.

We tried to migrate from Android Studio 2.3.3, to 3.0 and later 3.0.1, but in our Kotlin based project every test run caused the whole codebase to rebuild, as there is a bug that executes assemble- tasks instead of compile- tesks when tests are run. To not to waste so much time on every test we decide to stick to 2.3.3 version for now, and we're still waiting for AS 3.1 right now. So I don't know if mentioned problems are resolved in AS 3.0. It has its own downsides. (insert issues here)

After an another long hour of struggling with the problem I decided to focus on it until I resolve it ultimately. And now I present you the fruits of all this work.

The Android Studio tests fixing script. It removes all Android Studio project data (from .idea project folder and internal Android Studio folders as well) and gradle data. This turned out to be sufficient to fix all test problems we have encountered so far.

There is still a problem that the whole process takes time. Moreover it removes all of your project data like ignored warnings, test configurations, etc., so you have to restore it manually (reloading from backup is not recommended as it may restore some corrupted configuration files that cause test problems). So still, it's not a painless solution for fixing tests, but still, it's the best I have found so far. It's main upside - that it have succesfully fixed all test cases
