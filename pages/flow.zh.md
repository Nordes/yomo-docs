# yomo-flow

## yomo-flow 是什么？

`yomo-flow` 是一个 `Streaming Serverless` function，您只需在该 function 里面编写您的业务逻辑代码对 stream 数据进行计算处理即可。
例如：

```go
func Handler(rxstream rx.RxStream) rx.RxStream {
	stream := rxstream.
		Y3Decoder("0x10", float32(0)).
		AuditTime(100 * time.Millisecond).
		Map(printer).
		StdOut()

	return stream
}
```

## yomo-flow 能做什么？

YoMo 以 [Functional Reactive Programming](https://en.wikipedia.org/wiki/Functional_reactive_programming) 为编程范式，`yomo-flow` 的输入数据为 `RxStream`，您可以使用 [Rx](http://reactivex.io/) 的各种 [operators](http://reactivex.io/documentation/operators.html) 对 stream 进行操作。

![Rx](/flow/rx.png)

### 示例

#### 使用 [Map](http://reactivex.io/documentation/operators/map.html) 对数据进行转换

![Map](/flow/map.png)

#### 使用 [TakeLast](http://reactivex.io/documentation/operators/takelast.html) 获取最后 n 条数据

![TakeLast](/flow/takeLast.png)

## 如何编写 yomo-flow？

### 1. 安装 CLI

```bash
# 请使用 $GOPATH，因为 go 语言需要 plugin 和 main 的高度耦合
$ echo $GOPATH
```

如果未设置 `$GOPATH`，请先看这一节：[设置 $GOPATH 和 $GOBIN](#optional-set-gopath-and-gobin)。

```bash
$ GO111MODULE=off go get github.com/yomorun/yomo
$ cd $GOPATH/src/github.com/yomorun/yomo
$ make install
```

![YoMo 教程 1](/tutorial-1.png)

### 2. 创建一个 yomo-flow 应用

```bash
$ mkdir -p $GOPATH/src/github.com/{YOUR_GITHUB_USERNAME} && cd $_
$ yomo init yomo-app-demo
2020/12/29 13:03:57 Initializing the Serverless app...
2020/12/29 13:04:00 ✅ Congratulations! You have initialized the serverless app successfully.
2020/12/29 13:04:00 🎉 You can enjoy the YoMo Serverless via the command: yomo dev
$ cd yomo-app-demo
```

![YoMo 教程 2](/tutorial-2.png)

YoMo CLI 会自动创建带有以下内容的 `app.go` 文件：

```go
package main

import (
	"context"
	"fmt"
	"time"

	"github.com/yomorun/yomo/pkg/rx"
)

var printer = func(_ context.Context, i interface{}) (interface{}, error) {
	value := i.(float32)
	fmt.Println("serverless get value:", value)
	return value, nil
}

// Handler 将以 Rx 的方式处理数据
func Handler(rxstream rx.RxStream) rx.RxStream {
	stream := rxstream.
		Y3Decoder("0x10", float32(0)).
		AuditTime(100 * time.Millisecond).
		Map(printer).
		StdOut()

	return stream
}
```

### 3. 编译并运行

从 terminal 运行 `yomo dev`，可以看到：

![YoMo 教程 3](/tutorial-3.png)

恭喜你！你创建了你的第一个 `yomo-flow` 应用。

> **注意：** `yomo dev` 命令自动使用模拟噪声分贝值，`yomo run` 将使用 `yomo-zipper` 从 `yomo-source` 接收到的数据。

### 4. 修改代码为您的业务逻辑

1) YoMo 的数据传输使用高效的 [Y3 Codec](https://github.com/yomorun/y3-codec-golang) 进行编码，在 `yomo-flow` 的 `Handler` 方法里，第一步是使用 `Y3` 进行解码，您只需要修改 `Y3Decoder` 方法的第一个参数为您想监听的 `key`，第二参数为您要解码的值。

2) 使用相应的 [operators](http://reactivex.io/documentation/operators.html) 方法对 stream 进行操作。

### Optional: 设置 $GOPATH 和 $GOBIN

针对当前 session：

```bash
export GOPATH=~/.go
export PATH=$GOPATH/bin:$PATH
```

要永久设置这些变量，需要编辑 `.zshrc` 或 `.bashrc`：

`zsh` 用户：

```bash
echo "export GOPATH=~/.go" >> .zshrc
echo "path+=$GOPATH/bin" >> .zshrc
```

`bash` 用户：

```bash
echo 'export GOPATH=~/.go' >> .bashrc
echo 'export PATH="$GOPATH/bin:$PATH"' >> ~/.bashrc
```
