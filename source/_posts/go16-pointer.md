title: Golang1.6的cgo指针注意事项
categories: 小笔记
date: 2016-04-05 19:01:07
tags: [golang,1.6,cgo,指针]
description: golang1.6在使用cgo时对指针的一个运行时检测的坑
---

```go
package main
/*
#include <stdio.h>

void hello(char **happy) {
	puts(*happy);
}
*/
//import "C"
import (
	"fmt"
	"unsafe"
)

func main() {
	inbuf := []byte("hello kitty\x00")
	inptr := uintptr(unsafe.Pointer(&inbuf[0]))
	C.hello((**C.char)(unsafe.Pointer(&inptr)))
	fmt.Println(one.Tag)
}
```

> 作者原创，转载请注明出处
