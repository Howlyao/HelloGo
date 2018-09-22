# HelloGo
## 1

要编译并运行简单的程序，首先要选择包路径（我们在这里使用 github.com/user/hello），并在你的工作空间内创建相应的包目录：
```
$ mkdir $GOPATH/src/github.com/user/hello
```

接着，在该目录中创建名为 hello.go 的文件，其内容为以下Go代码：
```
package main

import "fmt"

func main() {
	fmt.Printf("Hello, world.\n")
}
```

现在你可以用 go 工具构建并安装此程序了：
```
$ go install github.com/user/hello
```

注意，你可以在系统的任何地方运行此命令。go 工具会根据 GOPATH 指定的工作空间，在 github.com/user/hello 包内查找源码。



此命令会构建 hello 命令，产生一个可执行的二进制文件。 接着它会将该二进制文件作为 hello（在 Windows 下则为 hello.exe）安装到工作空间的 bin 目录中。 在我们的例子中为 $GOPATH/bin/hello，具体一点就是 $HOME/go/bin/hello。



go 工具只有在发生错误时才会打印输出，因此若这些命令没有产生输出， 就表明执行成功了。



现在，你可以在命令行下输入它的完整路径来运行它了：
```
$ $GOPATH/bin/hello
Hello, world.
```



若你使用源码控制系统，那现在就该初始化仓库，添加文件并提交你的第一次更改了。 再次强调，这一步是可选的：你无需使用源码控制来编写Go代码。
```
$ cd $GOPATH/src/github.com/user/hello
$ git init
Initialized empty Git repository in /home/user/work/src/github.com/user/hello/.git/
$ git add hello.go
$ git commit -m "initial commit"
[master (root-commit) 0b4507d] initial commit
 1 file changed, 1 insertion(+)
  create mode 100644 hello.go
```


将代码推送到远程仓库就留作读者的练习了。


## 2
你的第一个库



让我们编写一个库，并让 hello 程序来使用它。



同样，第一步还是选择包路径（我们将使用 github.com/user/stringutil） 并创建包目录：
```
$ mkdir $GOPATH/src/github.com/user/stringutil
```


接着，在该目录中创建名为 reverse.go 的文件，内容如下：
```
// Package stringutil contains utility functions for working with strings.
package stringutil

// Reverse returns its argument string reversed rune-wise left to right.
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

现在用 go build 命令来测试该包的编译：
```
$ go build github.com/user/stringutil
```


当然，若你在该包的源码目录中，只需执行：
```
$ go build
```


即可。这不会产生输出文件。想要输出的话，必须使用 go install 命令，它会将包的对象放到工作空间的 pkg 目录中。



确认 stringutil 包构建完毕后，修改原来的 hello.go 文件（它位于 $GOPATH/src/github.com/user/hello）去使用它：
```
package main

import (
	"fmt"

	"github.com/user/stringutil"
)

func main() {
	fmt.Printf(stringutil.Reverse("!oG ,olleH"))
}
```


无论是安装包还是二进制文件，go 工具都会安装它所依赖的任何东西。 因此当我们通过
```
$ go install github.com/user/hello

```

来安装 hello 程序时，stringutil 包也会被自动安装。



运行此程序的新版本，你应该能看到一条新的，反向的信息：
```
$ hello
Hello, Go!
```


做完上面这些步骤后，你的工作空间应该是这样的：
```
bin/
	hello                 # command executable
pkg/
	linux_amd64/          # this will reflect your OS and architecture
		github.com/user/
			stringutil.a  # package object
src/
	github.com/user/
		hello/
			hello.go      # command source
		stringutil/
			reverse.go    # package source
bin/
	hello                 # 可执行命令
pkg/
	linux_amd64/          # 这里会反映出你的操作系统和架构
		github.com/user/
			stringutil.a  # 包对象
src/
	github.com/user/
		hello/
			hello.go      # 命令源码
		stringutil/
			reverse.go       # 包源码
```


## 3

测试



Go拥有一个轻量级的测试框架，它由 go test 命令和 testing 包构成。



你可以通过创建一个名字以 _test.go 结尾的，包含名为 TestXXX 且签名为 func (t *testing.T) 函数的文件来编写测试。 测试框架会运行每一个这样的函数；若该函数调用了像 t.Error 或 t.Fail 这样表示失败的函数，此测试即表示失败。



我们可通过创建文件 $GOPATH/src/github.com/user/stringutil/reverse_test.go 来为 stringutil 添加测试，其内容如下：

```
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

接着使用 go test 运行该测试：
```
$ go test github.com/user/stringutil
ok  	github.com/user/stringutil 0.165s

```

​​​​​​​
