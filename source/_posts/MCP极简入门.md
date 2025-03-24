---
title: MCP 极简入门
tags:
  - 编程
  - AI
permalink: 202503-3.html
date: 2025-03-24
categories:
---
## 何为MCP
_Model Context Protocol_ MCP 是一种开放协议，它规范了应用程序如何为大型语言模型提供上下文。可以把 MCP 想象成 AI 应用程序的 USB-C 接口。就像 USB-C 为您的设备连接各种外设和配件提供了标准化方式一样，MCP 也为 AI 模型连接不同的数据源和工具提供了标准化方式。

> 目前，MCP 被认为是构建更强大 AI Agent 的必经之路。


## 总体结构
![2025-03-23_19-12-25.png](https://s2.loli.net/2025/03/23/W2NxgyYzRr9uAOV.png)

这里面最重要的部分是 _MCP Protocol_，但真正需要我们开发（或从第三方仓库下载并安装）的则是 _MCP Server_：一种通过标准化模型上下文协议展示特定功能的轻量级程序。


## 开发Server

**_我们使用 python 语言开发一个本地服务，并将其集成到 VSCode 的 Cline 插件中_**

1. 首先安装 mcp 工具
```bash
# 官方推荐是用 uv 安装，不过我选择了自己习惯的工具，比如 conda

# 需要 python3.10 版本及以上
pip install mcp
```

2. 编写 server 代码
``` python
# mcp_hello.py
from mcp.server.fastmcp import FastMCP

# 创建一个 MCP 服务器
mcp = FastMCP("mcp_hello")

# 添加一个加法工具
@mcp.tool()
def add(a: int, b: int) -> int:
    """将两个数字相加"""
    return a + b

# 添加一个比较工具
@mcp.tool()
def compare(a: float, b: float) -> str:
    """比较两个数字的大小"""
    if a > b:
        return f"{a} > {b}"
    elif a < b:
        return f"{a} < {b}"
    else:
        return f"{a} = {b}"

## 设置启动方法，我一开始没有写这部分代码，导致无法启动
if __name__ == "__main__":
    mcp.run(transport='stdio')
```

3. 配置 mcp 文件
Cline 配置 MCP 服务如图所示：
![2025-03-24_22-38-08.png](https://s2.loli.net/2025/03/24/jVexocJF75pvHkW.png)
代码：
``` json
{
  "mcpServers": {
    "mcp_hello": {
      "command": "/Users/kyuu/miniconda3/envs/mcpenv/bin/python",
      "args": [
        "/Users/kyuu/development/mcp_demo/mcp_hello.py"
      ],
      "disabled": false,
      "autoApprove": [
        "add"
      ]
    }
  }
}
```
注意：由于我使用的是 Conda 环境，因此 `command` 指向了我的 Conda Python 路径。

## 运行mcp
回到 VSCode 中的 Cline 插件，勾选 **Auto-approve**，选择 **Act** 模式，问出哪个经典的问题：

> 比较 3.11 和 3.9 的大小？

回答结果是：
```
数值比较中，3.11 作为十进制数等于3 + 11/100 = 3.11，而3.9等于3 + 90/100 = 3.90。因此3.11 < 3.9
```

多次尝试无果后，重启 VSCode 并重新测试，最终成功调用 MCP 工具：

![2025-03-24_23-16-03.png](https://s2.loli.net/2025/03/24/8njJBTIfKtokWQp.png)

## 总结

MCP 很有可能成为扩展 AI 模型能力的标准协议，为 AI 提供了更强大的扩展性。然而，本次我的实验尚未完全成功，仍需进一步研究和调试。

## 参考资料
1.  [官方文档](https://modelcontextprotocol.io/introduction)

