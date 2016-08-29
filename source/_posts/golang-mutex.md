title: golang锁竞争性能
categories: 小笔记
date: 2016-08-29 17:54:21
tags: [Go,锁]
description: 测试golang锁的性能
---

闲来没事，便试着测试了下golang的锁的性能。代码如下：
```golang
package main

import (
	"fmt"
	"os"
	"strconv"
	"sync"
	"time"
)

var lock sync.Mutex
var count = 1

func work() {
	for {
		lock.Lock()
		count++
		lock.Unlock()
	}
}

func main() {
	nthread := 1
	if len(os.Args) > 1 {
		nthread, _ = strconv.Atoi(os.Args[1])
	}

	for i := 0; i < nthread; i++ {
		go work()
	}
	old := 0
	last := time.Now()
	for {
		time.Sleep(time.Second)
		lock.Lock()
		k := count
		lock.Unlock()
		now := time.Now()
		fmt.Println(int(float64(k-old) / (float64(now.UnixNano()-last.UnixNano()) / 1e9)))
		old = k
		last = now
	}
}

```

测试结果：

竞争的协程数 | 每秒总的lock次数 
-------------|------------------
1            | 5千万            
2            | 2.8千万          
4            | 9百万            
10           | 6百万            
100          | 5百万            
1000         | 5百万            
10000        | 5百万            
100000       | 5百万            
1000000      | 5百万            
