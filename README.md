gox - Code generator for the Go language
========

[![Build Status](https://github.com/goplus/gox/actions/workflows/go.yml/badge.svg)](https://github.com/goplus/gox/actions/workflows/go.yml)
[![Go Report Card](https://goreportcard.com/badge/github.com/goplus/gox)](https://goreportcard.com/report/github.com/goplus/gox)
[![GitHub release](https://img.shields.io/github/v/tag/goplus/gox.svg?label=release)](https://github.com/goplus/gox/releases)
[![Coverage Status](https://codecov.io/gh/goplus/gox/branch/main/graph/badge.svg)](https://codecov.io/gh/goplus/gox)
[![GoDoc](https://pkg.go.dev/badge/github.com/goplus/gox.svg)](https://pkg.go.dev/mod/github.com/goplus/gox)

## Quick start

Create a file named `hellogen.go`, like below:

```go
package main

import (
	"go/token"
	"go/types"
	"os"

	"github.com/goplus/gox"
	"github.com/goplus/gox/packages"
)

func ctxRef(pkg *gox.Package, name string) gox.Ref {
	return pkg.CB().Scope().Lookup(name)
}

func main() {
	imp, _, _ := packages.NewImporter(nil, "fmt", "strings", "strconv")
	pkg := gox.NewPackage("", "main", &gox.Config{Importer: imp})
	fmt := pkg.Import("fmt")
	v := pkg.NewParam(token.NoPos, "v", types.Typ[types.String]) // v string

	pkg.NewFunc(nil, "main", nil, nil, false).BodyStart(pkg).
		DefineVarStart(token.NoPos, "a", "b").Val("Hi").Val(3).EndInit(2). // a, b := "Hi", 3
		NewVarStart(nil, "c").Val(ctxRef(pkg, "b")).EndInit(1).            // var c = b
		NewVar(gox.TyEmptyInterface, "x", "y").                            // var x, y interface{}
		Val(fmt.Ref("Println")).
		/**/ Val(ctxRef(pkg, "a")).Val(ctxRef(pkg, "b")).Val(ctxRef(pkg, "c")). // fmt.Println(a, b, c)
		/**/ Call(3).EndStmt().
		NewClosure(gox.NewTuple(v), nil, false).BodyStart(pkg).
		/**/ Val(fmt.Ref("Println")).Val(v).Call(1).EndStmt(). // fmt.Println(v)
		/**/ End().
		Val("Hello").Call(1).EndStmt(). // func(v string) { ... } ("Hello")
		End()

	gox.WriteTo(os.Stdout, pkg, false)
}
```

Try it like:

```shell
go mod init hello
go mod tidy
go run hellogen.go
```

This will dump Go source code to `stdout`. The following is the output content:

```go
package main

import fmt "fmt"

func main() {
	a, b := "Hi", 3
	var c = b
	var x, y interface {
	}
	fmt.Println(a, b, c)
	func(v string) {
		fmt.Println(v)
	}("Hello")
}
```
