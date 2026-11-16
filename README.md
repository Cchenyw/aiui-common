# aiui-common

`aiui/common` 的规范源码仓库。平台仓库和执行机仓库都以 `aiui/common/` 的形式
自带一份副本，保证各自可以独立安装、独立运行。

## 文件

- `node_protocol.py`：远程执行协议常量与信封校验
- `limits.py`：AI 步数/token/重复动作上限
- `budget.py`：AI 预算与计费规则
- `profiles.py`：环境 Profile 适配
- `screenshots.py`：截图 MCP 适配与持久化
- `mcp_runtime.py`：MCP stdio/SSE/HTTP 客户端
- `llm.py`：Anthropic / OpenAI 兼容客户端
- `paths.py`：运行目录路径

## 使用方式

本仓库根目录就是 `aiui/common` 的内容。消费方应把本仓库复制/同步到自己的
`aiui/common/` 目录，而不是作为独立 pip 包安装（`aiui` 是普通包，安装到
site-packages 会与仓库内的 `aiui` 包冲突）。

同步时保持三个仓库的 `aiui/common` 内容一致；后续可改为 CI 校验或 submodule。
