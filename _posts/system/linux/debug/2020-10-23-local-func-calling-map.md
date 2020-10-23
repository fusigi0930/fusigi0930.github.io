---
layout: post
title: Analyze linux program function calling status
description: debug method
tags: debug linux optimize reverse-design
---
# function calling status tracing

use linux utilities valgrind (it also can analyze memory leakage) and kcachegrind ui tool to help us analyze function usage status

## 1. install utilities
in ubuntu system, use the command to install these utilities:
```shell
install kcachegrind and valgrind tool
```

## 2. run the target process
use the valgrind to generate report for calling usage status
```shell
valgrind --tool=callgrind <process>
```

after the command, the report file likes "callgrind.out.<pid>" should be generated in your folder

## 3. UI tool
the report document has lots of information, we can use the utility kcachegrind to help us show these information:

```shell
kcachegrind callgrind.out.<pid>
```
