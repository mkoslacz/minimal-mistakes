---
title: "The Android Studio tests problems fixing script"
date: "2018-10-16 08:38:09 +0100"
header:
  overlay_color: "#333"
  overlay_image: /assets/images/image3.jpg
  teaser: /assets/images/image3.jpg
  cta_label: "See the script"
  cta_url: "https://github.com/mkoslacz/Android-Studio-tests-fixing-script"
  overlay_filter: 0.5
  caption: "Photo credit: [**Luca Bravo @ Unsplash**](https://unsplash.com/@lucabravo)"
excerpt: "The remedy to get rid of annoying test issues."
tags:
  - Android
  - Android Studio
  - Tests
  - Test
  - TDD
---

**In our daily work we often experience problems with running tests in Android Studio. It happens pretty often when we switch branches (especially in projects that use code generating libraries), port a project between AS versions, use git modules, and in some other cases. To resolve that I created the script that fixes the test problems, and it's available [here](https://github.com/mkoslacz/Android-Studio-tests-fixing-script).**

{% include toc %}

# Introduction
Me and my team, we use TDD in our projects. It's a great methodology to create your code, and I believe that the great code **has** to be tested. But I won't talk about it today, as you can find plenty of articles that describes qualities of this approach. Today's article will be about resolving problems that disturb TDD.

In our environment we use Dagger2 that generates code and we have to switch between branches pretty often to do the code reviews, test recently added changes etc. In these circumstances we realized that Android Studio often has problems with executing tests.

Fighting with these problems turned out to be one of the most annoying distractions in our daily work, so I decided to take a closer look at them.

# Examples of problems
The problem is that we often get the infamous errors like:

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

Or:

```
Process finished with exit code 1
Class not found: "com.yourcodebase.Class"Empty test suite.
```

Or another:

```
No tests were found.
```

Or sometimes when you try to run a single test a whole test class is run. That makes debugging harder.

# When do these problems happen?
We noticed these test problems when:
- porting a project between AS versions,
- adding a git submodule root in AS and then changing a branch to the branch without submodule,
- switching between branches frequently (especially when having lots of generated code),
- and sometimes randomly.

# What were we doing to resolve them?
Anyway, these problems are a real pain in the neck, as they interrupt the TDD cycle. We didn't manage to find out what causes these problems (see the side note in ["The final solution" section](#the-final-solution)), but we have learnt how to deal with them temporarily. In the most cases we managed to fix them by:
- Removing old test configurations, (insert Images)
- Setting `MODULE_DIR` tests working directory (insert Images)
- Using `Rebuild project`*
- Using `Invalidate Caches / Restart`*

\* I'm sure that you, as an advanced Android Studio user, are aware that you can find all of these options using `cmd/ctrl + shift + A` shortcut and just typing them in :wink:
{: .notice--info}

Sometimes it is sufficient to perform one of these steps and it makes your tests run again, but we got used to performing all of these steps when the problem appears. We did that to avoid redoing the gradle test run when performing one step was not sufficient what led us to wait for another gradle test run.

# Was it helping?
In the most cases -- yes, it was.

But the world isn't perfect, and our simple procedure to refresh our test suite doesn't always work. There were still incidents in which someone was trying to run tests without success for many hours. The first main fallback was to simply restart the computer. But still - it sometimes doesn't help. Then we realized that removing a `.gradle` folder inside our project works in some cases.

After an another long hour of struggling with the problem I decided to focus on it until I resolve it ultimately. And now I present you the fruits of all this work.

# The final solution
Basically it's how [The Android Studio tests fixing script](https://github.com/mkoslacz/Android-Studio-tests-fixing-script) was created. It removes all of the Android Studio project data (from `.idea` project folder and internal Android Studio folders as well) and gradle data. This turned out to be sufficient to fix all of the test problems we have encountered so far.

**Side note** I am aware that it is rather a workaround than the real solution as this script just blindly wipes out all of the project config. The point is that it's not really known why these test problems happen - if it were known, they will probably be fixed. See [this](https://stackoverflow.com/questions/14381694/why-is-the-android-test-runner-reporting-empty-test-suite), [this](https://stackoverflow.com/questions/22582021/android-studio-no-tests-were-found), and [that](https://stackoverflow.com/questions/2422378/intellij-idea-with-junit-4-7-junit-version-3-8-or-later-expected) stackoverflow questions - each responder in these threads states that other solution worked.
{: .notice--warning}

# How does it work?
*Take note that the script is compatible with git projects that uses an Android Studio suitable `.gitignore` configuration, like [this one](https://gist.github.com/iainconnor/8605514).*

The script is really short and simple. It cleans all of the gradle and IDE caches inside the project by doing:
```
git stash
git clean -xdf
find ${HOME}${androidStudioPrefsLocation}${androidStudioVersion}/tasks | grep ${PWD##*/} | xargs rm
git stash pop
```
This command set basically stashes your current work, deletes all ignored files (`-x`) including directories (`-d`) ([see docs](https://git-scm.com/docs/git-clean)) what causes your project to be like it's freshly cloned from remote, removes project related files from `home/Library/Preferences/AndroidStudiox.x/tasks`, and then restores your stashed work.

Actually it's so simple that you can use these commands by hand instead of using my script. At first you can also try only the git commands, as it sometimes works.

# Downsides of the script
There is still a problem that the rebuilding process after the cleanup takes some time. In our project it usually lasts ~5min, but it's more than enough to make you distracted. Moreover, the script removes all of your project data like ignored warnings, test configurations, etc., so you have to restore it manually (reloading from backup is not recommended as it may restore some corrupted configuration files that cause test problems). I admit that it is a big deal but still, it's better than fighting with broken tests for long hours.

# The sum up
The script is not a painless solution for fixing tests, but it's the best solution I have found so far. It has the main upside - it has successfully fixed all known cases of tests failure in my case without putting too much effort. If you encounter test problems in your work I encourage you to give it a try.

**Another solution** It took me some time to publish this article, and in the meanwhile the another cleaning script got created: [deep-clean](https://github.com/rock3r/deep-clean). It requires some additional components to be on your `PATH`, but it allows you to perform deeper cleaning (system and IDE wise, as my script sticks only to the selected project) and decide how deep shall be the cleanup. I recommend you taking a look there as well, but in our cases my "lighter" cleaning was sufficient in 100% of situations.
{: .notice--info}
