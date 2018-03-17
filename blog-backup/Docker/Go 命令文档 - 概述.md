Go 提供了一套程序来构建和处理 Go 源代码。

这个套件中的程序通常不是直接运行，而是由 `go` 程序调用，将这些程序作为 `go` 程序的子命令，例如 `go fmt`。这样运行时，该命令会在完整的 Go 源代码包上运行，go 程序使用适合于包级处理的参数调用基础二进制文件。

套件中的程序也可以通过使用 `go tool` 子命令（如 `go tool vet`）作为独立的二进制文件直接运行。这种调用方式允许检查单个源文件而不是整个包：例如 `go vet mypackage` 和 `go tool vet myprogram.go`。套件中的某些命令（如 `pprof`）只能通过 `go tool` 子命令访问。

因为经常被引用，`fmt` 和 `godoc` 命令被安装为常规二进制文件，称为 `gofmt` 和 `godoc`。

程序|	概要
-|-
[go](https://golang.org/cmd/go/)|	    	`go` 程序管理 Go 源代码并运行这里列出的其他命令。
[cgo](https://golang.org/cmd/cgo/)|	Cgo 支持创建调用 C 代码的 Go 包。
[cover](https://golang.org/cmd/cover/)|	Cover 用于创建和分析由“go test -coverprofile”生成的覆盖率（coverage profiles）。
[fix](https://golang.org/cmd/fix/)|	Fix 找出使用了语言和库中的旧特性的 Go 程序并改为使用对应的新特性。
[fmt](https://golang.org/cmd/go/)|	Fmt 格式化 Go 包，可以直接通过独立的 `gofmt` 命令配合选项使用。
[godoc](https://godoc.org/golang.org/x/tools/cmd/godoc/)|	Godoc 提取并生成 Go 包的文档。
[vet](https://golang.org/cmd/vet/)|	Vet 检查 Go 源代码并报告可疑结构，例如参数与格式字符串不匹配的 Printf 调用。

Go 语言中，完整的命令列表 [参考这里](https://golang.org/cmd/)。
