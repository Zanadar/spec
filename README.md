# spec

[![GoDoc](https://godoc.org/github.com/sclevine/spec?status.svg)](https://godoc.org/github.com/sclevine/spec)

Spec is a simple BDD test organizer for Go. It minimally extends the standard
library `testing` package by facilitating easy organization of Go 1.7+
[subtests](https://blog.golang.org/subtests).

Spec differs from other BDD libraries for Go in that it:
- Does not reimplement or replace any functionality of the `testing` package
- Does not provide assertions
- Does not encourage the use of dot-imports
- Does not reuse any closures between test runs (to avoid test pollution)
- Does not use global state, excessive interface types, or reflection

Spec is intended for gophers who want to write BDD tests in idiomatic Go using
the standard library `testing` package. Spec aims to do "one thing right,"
and does not provide a wide DSL or any functionality outside of test
organization.

### Features

- Clean, simple, straightforward syntax
- Supports focusing and pending tests
- Supports sequential, random, reverse, and parallel test order
- Provides granular control over test order and subtest nesting
- Provides a generic, asynchronous reporting interface
- Provides multiple reporter implementations

### Notes

- Use `go test -v` to see individual subtests.

### Todo

- Test coverage

### Examples

[Most functionality is demonstrated here.](spec_test.go)

Quick example:

```go
func TestObject(t *testing.T) {
    spec.Run(t, "object", func(t *testing.T, when spec.G, it spec.S) {
        var (
            someObject *myapp.Object
            someMock   *mocks.Mock
        )

        it.Before(func() {
            someObject = myapp.NewObject()
            someMock = mocks.NewMock()
        })

        it.After(func() {
            someObject.Close()
        })

        it("should have some default", func() {
            if someObject.Default != "value" {
                t.Error("bad default")
            }
        })

        when("something happens", func() {
            it.Before(func() {
                someMock.EXPECT().Setup("data").Return("stuff")
            })

            it("should do one thing", func() {
                someMock.EXPECT().Finish().Return("other stuff")
                if err := someObject.DoThing(); err != nil {
                    t.Error(err)
                }
            })

            it("should do another thing", func() {
                if result := someObject.DoOtherThing(); result != "good result" {
                    t.Error("bad result")
                }
            })
        }, spec.Random())

        when("some slow things happen", func() {
            it("should do one thing in parallel", func() {
                if result := someObject.DoSlowThing(); result != "good result" {
                    t.Error("bad result")
                }
            })

            it("should do another thing in parallel", func() {
                if result := someObject.DoOtherSlowThing(); result != "good result" {
                    t.Error("bad result")
                }
            })
        }, spec.Parallel(), spec.Report(report.Terminal{}))
    })
}
```

With less nesting:

```go
func TestObject(t *testing.T) {
    spec.Run(t, testObject)
}

func testObject(t *testing.T, "object", when spec.G, it spec.S) {
    var someObject myapp.Object
    ...
}
```
