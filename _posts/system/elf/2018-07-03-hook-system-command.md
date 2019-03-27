---
layout: post
title: use LD_PRELOAD to hooking api
description: hooking api
tags: linux hook C system
---

# Hook System Process

當需要記錄某一個被執行或是某一群被執行的system process，在這裡介紹一種技巧

## LD_PRELOAD

android system的libc裡依舊有保留LD_PRELOAD這樣的環境變數可以供我們利用，這個環境變數是要求libc的linker在執行process時，一併將LD_PRELOAD裡的library載入到process的virtual memory以供process使用，使用方式為

```shell
export LD_PRELOAD=/var/lib/xxx.so
...
./process

```
這樣 process在執行時，就會把xxx.so也載入

## run function before main function
可以做到這個功能是基於elf (executable and link format) 所定義的一些section, 詳細的就不在這裡描述，這裡只用到一個section -- init_array, 使用方式如下：

```c++
int fakemain(int argc, char **argv, char **env) {
	...
}

__attribute__((section(".init_array"))) typeof(fakemain) *__fakemain = fakemain;
```

這樣在執行時，fakemain就會在main之前被執行並且可以得到要帶入到main function的arguments

## replace standard C function

在linux裡hooking api比windows系統簡單很多，只要直接寫一模一樣的prototype 跟function name就可以取代原function，然後再使用dlopen, dlsym, ...， 去呼叫原來的function，就可以得到輸入到原function跟原function要return的值，使用方式如下：

```c++
int printf(const char *fmt, ...) {
	va_list arguments;
	char message[1024] = {0};
	int nRet = 0;

	int (*ori_vprintf)(const char *, va_list);
	int (*ori_vsprintf)(char *, const char *, va_list);

	void *libc_so = reinterpret_cast<void *>(dlopen("/system/lib64/libc.so", RTLD_NOW));
	if (libc_so) {
		ori_vprintf = reinterpret_cast<int (*)(const char *, va_list)>(
						dlsym(libc_so, "vprintf"));

		ori_vsprintf = reinterpret_cast<int (*)(char *, const char *, va_list)>(
						dlsym(libc_so, "vsprintf"));

		va_start(arguments, fmt);

		if (ori_vprintf) nRet = ori_vprintf(fmt, arguments);
		if (ori_vsprintf) ori_vsprintf(message, fmt, arguments);
		dlclose(libc_so);
	}

	va_end(arguments);

	processLog(message, TYPE_WITHOUT_TIME);
	return nRet;
}
```

