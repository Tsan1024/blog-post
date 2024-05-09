---
title:       "golang 单元测试"
subtitle:    ""
description: ""
date:        2023-12-26T23:17:45+08:00
author:      ""
image:       ""
tags:        ["golang", "test"]
categories:  ["Tech" ]
---

# go Test 单元测试编写

Go 语言的单元测试默认采用官方自带的测试框架，通过引入 testing 包以及 执行 go test 命令来实现单元测试功能。

Go 单元测试的基本规范如下：
* 测试函数需导入 testing 包。测试函数的命名类似func TestName(t *testing.T)，入参是 *testing.T
* 测试函数的函数名须以大写的 Test 开头，后面紧跟的函数名，比如 func TestName(t *testing.T) 或者  func Test_name(t *testing.T)， 但是 func Testname(t *testing.T)不会被检测到
* 一般测试文件的命名，都是 {source_filename}_test.go，比如源代码文件是Add.go ，那么就会在 Add.go 的相同目录下，再建立一个 Add_test.go 的单元测试文件

go test 会遍历所有的 *_test.go 中符合上述命名规则的函数，然后生成一个临时的 main 包用于调用相应的测试函数，然后构建并运行、报告测试结果，最后清理测试中生成的临时文件。

## 基础示例
```
# 原函数
package test

func Add(args1, args2 int) int {
	result := args1 + args2
	return result
}

```

```
# 测试代码
package test

import (
	"fmt"
	"testing"
)

func TestAdd(t *testing.T) {
	value := Add(1, 2)
	fmt.Println(value)
}
```
```
# 运行所有test
go test

# 运行test并打印详情
go test -v 

# 运行具体某个方法的单元测试
# go test -v -run="xxx" 

# 执行单测并计算覆盖率
go test -v -cover

# 将覆盖率导入到文件
go test -cover -covermode=count -coverprofile=cover.out 
# covermode 代码分析模式（set：是否执行；count：执行次数；atomic：次数，并发执行）

# 查看测试文件
go tool cover -func=cover.out
```

## 子测试（Subtests）

```
func TestReduce(t *testing.T) {
	t.Run("add", func(t *testing.T) {
		if Reduce(2, 3) == 4 {
			t.Fatal("fail")
		}

	})
	t.Run("neg", func(t *testing.T) {
		if Reduce(2, 3) != -6 {
			t.Fatal("fail")
		}
	})
}

对于多个子测试的场景，推荐表格驱动测试(table-driven tests)的写法：

func TestReduce(t *testing.T) {
	cases := []struct {
		Name           string
		A, B, Expected int
	}{
		{"pos", 2, 3, 6},
		{"neg", 2, -3, -6},
		{"zero", 2, 0, 0},
	}

	for _, c := range cases {
		t.Run(c.Name, func(t *testing.T) {
			if ans := Add(c.A, c.B); ans != c.Expected {
				t.Fatalf("%d * %d expected %d, but %d got",
					c.A, c.B, c.Expected, ans)
			}
		})
	}
}

```

所有用例的数据组织在切片 cases 中，看起来就像一张表，借助循环创建子测试。这样写的优势如下：
* 新增用例非常简单，只需给 cases 新增一条测试数据即可。
* 测试代码可读性好，直观地能够看到每个子测试的参数和期待的返回值。
* 用例失败时，报错信息的格式比较统一，测试报告易于阅读。

如果数据量较大，或是一些二进制数据，推荐使用相对路径从文件中读取。

## 帮助函数(helpers)
对一些重复的逻辑，抽取出来作为公共的帮助函数(helpers)，可以增加测试代码的可读性和可维护性。 借助帮助函数，可以让测试用例的主逻辑看起来更清晰。

```
type addCase struct{ A, B, Expected int }

func createAddTestCase(t *testing.T, c *addCase) {
	t.Helper()
	if ans := Add(c.A, c.B); ans != c.Expected {
		t.Fatalf("%d * %d expected %d, but %d got",
			c.A, c.B, c.Expected, ans)
	}
}

func TestAdd(t *testing.T) {
	createAddTestCase(t, &addCase{4, 2, 6})   // success case
	createAddTestCase(t, &addCase{1, -3, -6}) // wrong case
}
```
为方便定位，调用 t.Helper() 可以让报错信息更准确，直接使用 t.Error 或 t.Fatal 即可，在用例主逻辑中不会因为太多的错误处理代码，影响可读性。

## setup 与 teardown
```
package codetest

import (
	"fmt"
	"reflect"
	"testing"
)

// setup 函数，在所有测试运行之前执行
func setup() {
	fmt.Println("Setup before tests")
	// 执行一些初始化操作
}

// teardown 函数，在所有测试运行之后执行
func teardown() {
	fmt.Println("Teardown after tests")
	// 执行一些清理操作
}

// TestMain 函数，用于自定义测试的行为
func TestMain(m *testing.M) {
	// 执行 setup 函数
	setup()

	// 运行所有测试
	exitCode := m.Run()

	// 执行 teardown 函数
	teardown()

	// 退出测试程序
	fmt.Println("Exiting with code", exitCode)
}

// 具体的测试函数
func TestSomething(t *testing.T) {
	// 在这里编写你的测试逻辑
	fmt.Println("Running TestSomething")
	// 断言或其他测试操作
}

func TestAnotherThing(t *testing.T) {
	// 在这里编写另一个测试逻辑
	fmt.Println("Running TestAnotherThing")
	// 断言或其他测试操作
}

func setupTest(tb testing.TB) func(tb testing.TB) {
	fmt.Printf("\033[1;34m%s\033[0m", ">> Setup Test\n")

	return func(tb testing.TB) {
		fmt.Printf("\033[1;34m%s\033[0m", ">> Teardown Test\n")
	}
}

func TestToString(t *testing.T) {
	type args struct {
		value interface{}
	}
	tests := []struct {
		name string
		args args
		want interface{}
	}{
		{
			name: "int",
			args: args{
				value: 101,
			},
			want: "100",
		},
		{
			name: "boolean",
			args: args{
				value: true,
			},
			want: "true",
		},
		{
			name: "float32",
			args: args{
				value: float32(100.01),
			},
			want: "100.01",
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			teardown := setupTest(t)
			defer teardown(t)
			if got := ToString(tt.args.value); !reflect.DeepEqual(got, tt.want) {
				t.Errorf("ToString() = %v, want %v", got, tt.want)
			}
		})
	}
}

// ToString convert any type to string
func ToString(value interface{}) string {
	if v, ok := value.(*string); ok {
		return *v
	}
	return fmt.Sprintf("%v", value)
}
```

## 测试网络

需要测试某个 API 接口的 handler 能够正常工作，例如 addHandler

```
package codetest

// test code
import (
	"bytes"
	"io"
	"net"
	"net/http"
	"testing"
)

func addHandler(w http.ResponseWriter, r *http.Request) {
	body, _ := io.ReadAll(r.Body)
	w.Write([]byte(body))
}

func handleError(t *testing.T, err error) {
	t.Helper()
	if err != nil {
		t.Fatal("failed", err)
	}
}

func TestConn(t *testing.T) {
	ln, err := net.Listen("tcp", "127.0.0.1:0")
	handleError(t, err)
	defer ln.Close()
	payload := []byte(`{"key": "value"}`)

	// 创建一个字节缓冲区，并将数据写入其中
	reqBody := bytes.NewBuffer(payload)
	http.HandleFunc("/add", addHandler)
	go http.Serve(ln, nil)

	resp, err := http.Post("http://"+ln.Addr().String()+"/add", "application/json", reqBody)
	handleError(t, err)

	defer resp.Body.Close()
	body, err := io.ReadAll(resp.Body)
	handleError(t, err)

	if string(body) != "add" {
		t.Fatal("expected add, but got", string(body))
	}
}
```

针对 http 开发的场景，更推荐使用标准库 net/http/httptest 进行测试。使用 httptest 模拟请求对象(req)和响应对象(w)，将TestConn替换成如下:
```
func TestConn(t *testing.T) {
	req := httptest.NewRequest("GET", "http://example.com/add", nil)
	w := httptest.NewRecorder()
	helloHandler(w, req)
	bytes, _ := ioutil.ReadAll(w.Result().Body)

	if string(bytes) != "add" {
		t.Fatal("expected add, but got", string(bytes))
	}
} 
```
## Benchmark 基准测试
基准测试用例的定义如下：

```
func BenchmarkName(b *testing.B){
    // ...
}
```
函数名必须以 Benchmark 开头，后面一般跟待测试的函数名
参数为 b *testing.B。
执行基准测试时，需要添加 -bench 参数。

执行如下：
```
go test -benchmem -bench .
```
以下参考极客兔兔，基准测试报告每一列值对应的含义如下：
```
type BenchmarkResult struct {
    N         int           // 迭代次数
    T         time.Duration // 基准测试花费的时间
    Bytes     int64         // 一次迭代处理的字节数
    MemAllocs uint64        // 总的分配内存的次数
    MemBytes  uint64        // 总的分配内存的字节数
}
```

如果在运行前基准测试需要一些耗时的配置，则可以使用 b.ResetTimer() 先重置定时器，例如：
```
func BenchmarkHello(b *testing.B) {
    ... // 耗时操作, 如 big := NewBig()
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        fmt.Sprintf("hello")
    }
}
```

使用 RunParallel 测试并发性能

```
func BenchmarkParallel(b *testing.B) {
	templ := template.Must(template.New("test").Parse("Hello, {{.}}!"))
	b.RunParallel(func(pb *testing.PB) {
		var buf bytes.Buffer
		for pb.Next() {
			// 所有 goroutine 一起，循环一共执行 b.N 次
			buf.Reset()
			templ.Execute(&buf, "World")
		}
	})
}
```

执行
```
go test -benchmem -bench .
```

# 参考连接：
1. https://juejin.cn/post/7172037988950474759
2. https://brantou.github.io/2017/05/24/go-cover-story/
3. https://ruichengm1987.github.io/docs/go/
4. https://pkg.go.dev/testing

