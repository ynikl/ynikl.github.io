---
title: "Go  errgroup 的基本用法"
date: 2022-09-19T09:19:31+08:00
publishDate: 2022-09-19T09:19:31+08:00
draft: false
tags:
- golang
---

## 实现并发控制

在 golang 代码中如果要对一段代码进行并发限制. 通常的做法都是在写一个 `channel`
进行传入和传出.

``` go
func main() {

	concurrencyNum := 10
	limitCh := make(chan bool, concurrencyNum)
	wg := new(sync.WaitGroup)

	for i := 0; i < 100; i++ {

		limitCh <- true
		wg.Add(1)
		go func() {
			defer func() {
				<-limitCh
				wg.Done()
			}()

			time.Sleep(1 * time.Second)
			fmt.Println("do some things...")
		}()
	}

	wg.Wait()
	fmt.Println("ok")
}
```

如果如果中间运行代码有可能存在错误, 捕获错误. 有两种方法:

- 声明一个 err channel 用于承接错误
- 声明一个外部 err 变量, 并通过互斥锁进行保护

```
func main() {

	concurrencyNum := 10
	limitCh := make(chan bool, concurrencyNum)
	errCh := make(chan error, concurrencyNum)

	var externalErr error
	wg := new(sync.WaitGroup)
	func() {
		for i := 0; i < 100; i++ {

			select {
			case err := <-errCh:
				externalErr = err
				return
			default:
			}

			wg.Add(1)
			limitCh <- true
			go func() {
				defer func() {
					<-limitCh
					wg.Done()
				}()

				time.Sleep(1 * time.Second)
				fmt.Println("do some things...")
				if rand.Intn(5) == 1 {
					err := errors.New("this is a error")
					errCh <- err
				}
			}()
		}
	}()

	wg.Wait()
	fmt.Println("ok")
	fmt.Println(externalErr)

}
```

每个地放都写这么多代码, 就有了重复的感觉. 本质上就两点: 

- 通过 channel 控制并发数
- 通过 waitgroup 保证所有的协程都执行完毕
- 通过另一个 errchannel 接受中间执行的错误

## `errgroup`

可以通过使用, 官方的拓展包 `errgroup` 更快实现

声明 errgroup

- 普通声明 `new(errgroup.Group)`
- 使用 context `errgroup.WithContext`

限制开启的协程数据 

`eg.SetLimit(goroutineNum)`

开启协程

- `eg.Go` 
- `eg.TryGo`

整体代码

```
	eg := new(errgroup.Group)
	eg.SetLimit(10)

	for i := 0; i < 100; i++ {

		eg.Go(func() error {
			time.Sleep(1 * time.Second)
			fmt.Println("hello go")
			return nil
		})
	}

	err := eg.Wait()
	fmt.Println("done", err)
```

目前有个使用场景没办法满足: 

就是没办法在开启协程之前, 知道原先已经执行的协程是否有发生错误.
如果有发生错误的. 就停止再继续开启协程.

可以通过添加一个 外部的 errChannel , 覆盖到上面的需求.
