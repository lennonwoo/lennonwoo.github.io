---
title: Support creating Windows packages for ROS2 in Bloom – Google Summer of Code (GSoC) 2019 summary
date: 2019-08-20 21:36:24
tags:
---

# Support creating Windows packages for ROS2 in Bloom – Google Summer of Code (GSoC) 2019 summary

I am very fortunate to have the chance to take part in GSoC 2019. The last three months, I was working on the [Bloom](https://github.com/ros-infrastructure/bloom) project in [OSRF](https://www.openrobotics.org/), with my mentors: [Steven! Ragnarök](https://github.com/nuclearsandwich) and [Scott K Logan](https://github.com/cottsay). I gained new knowledge every day and used it while contributing to the project. Whenever I am confused about the project, the mentors and the community are always so friendly to help me solve the problem, thanks a lot. The following post is my summary of this summer.

## Goal

Bloom is the release creation tool for the ROS ecosystem. From a standard package manifest format, bloom can generate Debian or RPM metadata for building binary packages on some Linux distributions. In order to bring support for modular binary packages to Windows and other possible future target platforms(mac OS or Arch Linux) the generators should share more code to reduce the work required to add new ones. The project should be carried out in three phases:

1. Refactor Bloom internals to re-use distribution functionality between all distribution generators
2. Create a proof of concept generating Vcpkg ports or Chocolatey packages for ROS 2.
3. Integrate the proof of concept work into the refactored mainline.

## Result

For Bloom's generator refactor part, I go through the code of Debian and RPM's generator to find out the common part and platform specific part need pay attention when refactor the code. Then I send a pull request: [Refactor generator](https://github.com/ros-infrastructure/bloom/pull/539#).



For distribute ROS2's modular binary packages in Windows. I research the Vcpkg and Chocolatey both, come out a solution that use Chocolatey to solve the ROS2 packages' dependencies, then use Vcpkg to build the binary files of every ROS2 package, pack it to Chocolatey for distributing in Windows finally. Then I send a pull request:  [Vcpkg support for Bloom](https://github.com/ros-infrastructure/bloom/pull/541). As you can see in the pull request, I pack 240 ROS2 package to Chocolatey package format, and you can try it out from my [release repository](https://github.com/lennonwoo/vcpkg/releases).

## Project Thoughts

There are some of my thoughts about this project, it may not contains too much coding details, since you can view the pull request as mentioned in the `Result` part. I wanna to discuss here about how I learn Bloom source code started from scratch, my opinion about the open source project of `Vcpkg` and `Chocolatey` used in this project, and the package packing flow.

### The Bloom

I am not familiar with Bloom when I know my proposal was accepted in the early May day, but the Bloom is a project that have more than 7 years development. How to dig into it and understand the core logic of Bloom is a challenge for me. From my three months travel with Bloom can tell how to understand it more quickly:

1. Understand the source code before you know how to use it, start at [Bloom's tutorials](http://wiki.ros.org/bloom/Tutorials/FirstTimeRelease) is a good idea.
2. You know how to use Bloom, so you can dig into the [scripts](https://github.com/ros-infrastructure/bloom/tree/master/scripts) the Bloom provided now.
3. The key of Bloom is releasing package, this release [test](https://github.com/ros-infrastructure/bloom/blob/master/test/system_tests/test_catkin_release.py) may be helpful to understand the releasing operations
4. You know the releasing operations well, then go through [generators](https://github.com/ros-infrastructure/bloom/tree/master/bloom/generators) you are interested.
5. Find the [issues](https://github.com/ros-infrastructure/bloom/issues) you seem familiar, try to solve it to enhance you understanding of Bloom.

On the other side, be familiar with Bloom's dependencies is important in my experience. You'd better get the source of [rosdep](https://github.com/ros-infrastructure/rosdep) and [rosdistro](https://github.com/ros-infrastructure/rosdep) and install these locally if you want to add platform support for Bloom.

### Why Vcpkg

Why we choose [Vcpkg](https://github.com/microsoft/vcpkg)? Why not the [colcon](https://github.com/colcon/colcon-core) build tools provided from ROS?

First of all, we need to be aware what part Vcpkg played in the project. We use `Vcpkg` here to:

1. Download the package source code from released repository archive.
2. Patch the source code if needed.
3. Configure the package and built it (the configure interface should be simple)
4. Install the binary files to certain directory for future packing.

If you are Debian user, you know that [debhelper](https://github.com/Debian/debhelper) could execute these job. But in Windows platform, there still not exist project `winhelper`. So we utility Vcpkg to do these job for following the Debian distribute method.

For `colcon`, it is a project for building local repository (like `catkin` if you are ROS1 user), it couldn't play the steps 1 & 2 as mentioned.

### Why Chocolatey

Why use [Chocolatey](https://chocolatey.org/)? There are many package manager in Windows, like [NuGet](https://www.nuget.org/), [Scoop](https://scoop.sh/), why we don't choose them but use `Chocolatey`?

As far as I know, the `NuGet` is a package manager for distributing software library, it's semantic version is powerful, but it lacks ability to install via `.exe` or `.msi` setup operation. 

For `Scoop`, it could install `.exe` or `.msi` format, also support semantic version. But compare to Chocolatey, it lack one killer feature that Chocolatey contains, which is that Chocolatey could [choose source](https://chocolatey.org/docs/commandsinstall#alternative-sources) when install package. Since our ROS2 package contains many python dependencies, it is very useful for port Bloom to Windows.

## Things need to improve

The dependencies like `openssl`, `python-pep8` that are not purely ROS2 need to improve. We can use source option in Chocolatey for install python related package dependencies, and pack other dependencies to Chocolatey package like we deal with `asio`.

We may utility the cross-platform feature of Vcpkg and Chocolatey. Now we have the building and packing workflow in Windows, we could use the same logic to distribute ROS2 to other platforms like mac OS or Arch Linux in the future.
