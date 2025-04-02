---
title: 如何在写库的时候正确的使用 loguru
date: 2025-04-02T08:47:40.467Z
---

[loguru](https://loguru.readthedocs.io/en/stable/overview.html) 已经是 Python 中仅有的算是比较符合人体工程学的日志库了。当然这里不吐槽些什么，仅是用于记录在写第三方库的时候如何使用 loguru。

如果你不希望第三方库擅自引用 loguru……那我没什么办法，虽然自然会有办法处理，如每一行日志都加上 `if LOG_ENABLED: ...`。不过这不是我们今天要谈到的东西。

loguru 提供了 `logger.disable`，可以屏蔽掉某个模块的日志输出，当然，也有 `logger.enable` 用于重新启用日志输出。  
在这里我们将选择权交给用户实际应用的 bootstrap 部分，即默认禁用掉用到日志的模块的 logger。

```python
from loguru import logger

logger.disable(__name__)

# take your loguru.logger as usual
logger.info(...)
```

在默认情况下，用户不会看到该模块的日志输出。需要注意的是，你需要为每个引入并使用了 `loguru.logger` 的模块都套用一个 `logger.disable(__name__)`。

如果某人需要查看你提供的第三方库的日志输出，你只需要引导他们启用对应模块的 logger，这里建议使你模块的划分尽量围绕实际功能，且保持适当的细粒度，因为 `loguru.{enable, disable}` 也会对模块及其子模块一并作用。

比如说，如果你希望 `firework.__main__` 作为用户接口和 CLI 工具的入口[^1]能正常打印日志，但不希望 `firework.networking` 的日志默认打印出来，你就不能在 `firework` 的 `__init__.py` 中写 `logger.disable(__name__)`，因为这样会把 `firework.__main__` 的日志禁用掉。

这之后，当出于调试等目的，用户希望输出日志时，他们只需要添加这一行即可。

```python
logger.enable("firework.networking")
```

[^1]: 在 `project.scripts` 中填写相关条目，对应的模块会被创建 shell 入口。
