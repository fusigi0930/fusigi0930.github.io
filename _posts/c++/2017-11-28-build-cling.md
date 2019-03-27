---
layout: post
title: Build cling for windows
description: build cling for windows
tags: c++ interpreter windows
---
# cling for windows
it records the build procedure for how to build cling for windows.
in the normal case, it should have 2 method to build cling for windows.
* build cling in windows with visual studio (or visual studio express)
* build cling in linux with mingw environment cross compiler (gcc based)

I think the first method should be the better one, because when I trace the
cling source code, it contains some directive and definition like:
* _MSC_VER <== visual studio version
* upper/lower case header files, likes: Windows.h, Shlwapi.h, ...
  it makes linux+mingw build failed, because the ext4 partition and linux system is case sensitive for file name
* pragma comment(lib, ...),  it is not supportted by gcc
* __uuidof, it should be defined in the _mingw.h file, it depends on USE___UUIDOF definition, and the USE___UUIDOF macro is defined to 1 while _MSC_VER is defined

but, I will try to build cling by using the second method (linux+mingw) first.

