title: Golang1.6的cgo指针注意事项
categories: 小笔记
date: 2016-04-05 19:01:07
tags: [golang,1.6,cgo,指针]
description: golang1.6在使用cgo时对指针的一个运行时检测的坑
---

例如下面的代码
```go
package main

/*
#include <stdio.h>
void hello(char **happy) {
	puts(*happy);
}
*/
import "C"
import (
	"unsafe"
)

func main() {
	inbuf := []byte("hello kitty\x00")
	inptr := unsafe.Pointer(&inbuf[0])
	C.hello((**C.char)(unsafe.Pointer(&inptr)))
}
```
它在golang1.6以前的版本（比如1.5.3等）能正常运行。但在golang1.6时运行却会出一个这样的异常：
```shell
$ go run main.go
panic: runtime error: cgo argument has Go pointer to Go pointer

goroutine 1 [running]:
panic(0x469600, 0xc82000a1c0)
        /usr/local/go/src/runtime/panic.go:464 +0x3e6
main.main()
        /home/listarme/work/main.go:17 +0xbe
exit status 2
```

原因是golang1.6中修改了对指针的运行时检查，若要正常使用多重指针，则必须得借助uintptr，如下
```go
package main

/*
#include <stdio.h>
void hello(char **happy) {
	puts(*happy);
}
*/
import "C"
import (
	"unsafe"
)

func main() {
	inbuf := []byte("hello kitty\x00")
	inptr := uintptr(unsafe.Pointer(&inbuf[0]))
	C.hello((**C.char)(unsafe.Pointer(&inptr)))
}
```

> 作者原创，转载请注明出处
