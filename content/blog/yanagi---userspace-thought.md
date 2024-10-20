---
title: Yanagi - Userspace Thought
date: 2024-10-20T17:10:15.897Z
---

设想中的 API 较 Alconna v1 有较大的差别，主要是数个额外划分出的抽象层级，以及与 Sistana 的整合。

# 高层 API

用户不直接接触 `SubcommandPattern` 或是 `OptionPattern` 的构造，而是通过构造 Endpoint Model 或直接是 Endpoint Handler，从而能一次完成 1) Pattern 的构造 2) Endpoint 的 Assign。
## Endpoint Handler 方式

Endpoint Handler 直接从函数的签名提取声明，贴近 FastAPI 的依赖注入的写法。

```python
@command("hello")
def hello(name: str): ...
```

这是最简单的构造方法，类似 Typer 的写法。我们可以使用 `@command` 装饰器来声明一个 Endpoint Handler，这里的 `name` 会被构造为 Subcommand Track 上的 Fragment。

```python
@command("greet")
def greet(name: str, *, age: int | None = option("--age").param("age").default(None)): ...
```

这里 `age` 会被构造为 Option `--age` 上的 Fragment `age`。
使用 `*` 将之后的参数声明为仅关键字参数，这允许声明 Option。此外，Yanagi 将这里参数默认值上的 `option(...)` 和 `.param(...)` 都视作某种引用，在处理上类似于 `dict.setdefault`，如果不存在就创建，反之引用。

为了实现类似 `Default::default` 的特性，我们可以使用 Flywheel。比如说，如果 Flywheel Context 中实现了 `Default.generate_option`，那么 `(*, age: int)` 或许能直接借由这种方式生成出关键字为 `--age` 的 Option。当然，这可能对于其他情况来说不是我们所期望的行为，所以用 Flywheel 处理是好的选择。

Yanagi 被设计为尽可能的不处理类型注解，因为这带来了无数难以忍受的肮脏实现（如 `typing_extensions` 相关的一大堆烂摊子）。即使如此， Yanagi 依旧会提供魔女一脉相传的前沿级的类型支持。有可能用 Flywheel 提供这方面的扩展接口吗？也许会吧。

Sistana 严格区分 Mix, Track 与 Fragment，而按照这种逻辑，如果这里给的是 `option(...)`，那么将不会通过 —— 如果你想要获取 Track Header Fragment 的话，应该用 `option(...).header`。当然，实际上不会那么泾渭分明，如果将其声明为 `option.count` 或是 `option.flag`，这里也就能传递 `int` 或是 `bool` 了。

如果要声明 Subcommand，我们建议使用 `@subcommand`。

```python
@command("hello")
@subcommand("world")
def hello_world(name: str): ...
```

这里实际构造了 `hello world` 的 Command，由于 `command` 装饰器理论上会由应该中控对线的方法提供，这里的 `command(...)` 也自然 —— 是个引用或者代理之类的，而 `subcommand(...)` 则是标注。

如果要定义 `lp user ... permission` 这种，叠多个。

```python
@command("lp")
@subcommand("user")
@subcommand("permission")
```

而这里，显然我们希望 `lp user` 可以把他获取到的 `username` 参数暴露出去给他的子命令使用，前有 Endpoint Model。

当然，可能会允许 `@command("lp", "user", "permission")` 这种东西，不过这篇笔记只倾向于快速的将用户 API 大概是怎么样的给展示出来，所以这种问题还让我延后讨论。

## Endpoint Model 方式

Endpoint Model 通常被声明为类，并且还是个 `dataclass-like`。

```python
@command("lp")
@subcommand("user")
class LpUserCommand(Endpoint):
    name: str

    def handle(self, context: Context):
        print(f"Hello, {self.name}!")

@command("lp")
@subcommand("user")
@subcommand("permission")
class LpUserPermissionCommand(Endpoint):
    permission: str

	user: LpUserCommand = model(LpUserCommand)

    def handle(self, context: Context):
        print(f"Hello, {self.user.name}! You have {self.permission} permission.")
```

初步的思路如此，通过这种方法，我们可以将 `lp user` 的 `username` 传递给 `lp user permission` 的 handler，我倾向于使用描述符方式（`model(...)`）来声明对其他 Model 的引用。

> 如果 Python 版本在 3.10 及以上，可以用  `dataclasses.KW_ONLY` 声明仅关键字参数，由于我们可能会先绕个路，把 Model 本身先 `make_dataclass` 一下让他生成 `__init__`，这样我们可以复用上面对 Endpoint Handler 的处理，不过这样做或许太邪道了。

对于 `Option`，声明方法和 Endpoint Handler 类似，都是通过 `option(...)` 来声明。这里或许会将 `option` 返回的 Ref 对象实现描述符协议。

```python
@command("greet")
class GreetCommand(Endpoint):
    name: str
    age: int | None = option("--age").param("age").default(None)

    def handle(self, context: Context):
        print(f"Hello, {self.name}! You are {self.age} years old.")
```

差不多就是这样，对于 `option.assign / OptionAssign` 或是 `Track` 之类的魔法可能会在后续讨论，还有对于 `fuzzy` 可能也会用 Endpoint 的一些方法上实现（当然可能会用 Flywheel），`help` 的 Option 或 Subcommand 之类的可以通过重写一些 Endpoint 的方法实现，不过这样跨的步子或许太大，这里就不展开了。
