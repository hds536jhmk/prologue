# Extend Context

`Prologue` provides flexible way to extend `Context` object. User-defined `Context` should inherit from the `Context` object.

## A simple example

The following example shows how to make a `UserContext` object which contains
a new data member of `int` type. You should use `extend` method to initialize
the new attributes. Finally use `app.run(UserContext)` to register the type.

```nim
import prologue

type
  UserContext = ref object of Context
    data: int

# initialize data
method extend(ctx: UserContext) =
  ctx.data = 999

proc hello*(ctx: UserContext) {.async.} =
  doAssert ctx.data == 999
  resp "<h1>Hello, Prologue!</h1>"

var app = newApp()
app.get("/", hello)
app.run(UserContext)
```

**Note**: If you want to use `UserContext` directly as the parameter of the function, make sure your proc is 
`nimcall` instead of `closure`. Otherwise you have to use

```nim
proc hello*(ctx: Context) {.async.} =
  doAssert ctx.UserContext.data == 999
  resp "<h1>Hello, Prologue!</h1>"
```

## Make a middleware for personal use

```nim
import prologue


type
  UserContext = ref object of Context
    data: int

proc init(ctx: UserContext) =
  ctx.data = 12

proc experimentMiddleware(): HandlerAsync =
  result = proc(ctx: UserContext) {.async.} =
    doAssert ctx.data == 12
    inc ctx.data
    await switch(ctx)

method extend(ctx: UserContext) =
  init(ctx)

proc hello*(ctx: UserContext) {.async.} =
  assert ctx.data == 13
  echo ctx.data
  resp "<h1>Hello, Prologue!</h1>"

var app = newApp()
app.use(experimentMiddleware())
app.get("/", hello)
app.run(UserContext)
```

## Make a general purpose middleware

**Notes**: use prefix or suffix denoting data member to avoid conflicts with other middlewares.

```nim
import prologue

# middleware for general purpose
type
  ExperimentContext = concept ctx
    ctx is Context
    ctx.data is int

proc init[T: ExperimentContext](ctx: T) =
  ctx.data = 12

proc experimentMiddleware[T: ExperimentContext](ctxType: typedesc[T]): HandlerAsync =
  result = proc(ctx: Context) {.async.} =
    doAssert ctx.ctxType.data == 12
    inc ctx.ctxType.data
    await switch(ctx)


type
  UserContext = ref object of Context
    data: int

method extend(ctx: UserContext) =
  init(ctx)

proc hello*(ctx: UserContext) {.async.} =
  assert ctx.data == 13
  echo ctx.data
  resp "<h1>Hello, Prologue!</h1>"

var app = newApp()
app.use(experimentMiddleware(UserContext))
app.get("/", hello)
app.run(UserContext)
```
