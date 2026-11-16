# CLAUDE.md

本文件为在本仓库中工作的 Claude Code 提供指引。

## 这个仓库是什么

`aiui-common` 是 AI UI 测试平台共享模块的**规范源码仓库**。仓库根目录就是
`aiui/common` 包的内容，包含：

- `node_protocol.py`：远程执行协议版本、信封字段与校验
- `limits.py`：AI 步数 / token / 重复动作上限
- `budget.py`：AI 预算设置、价格规则与成本估算
- `profiles.py`：环境 Profile（Playwright storage state）适配与敏感工具清单
- `screenshots.py`：每步截图 MCP 适配、文件持久化与安全读取
- `mcp_runtime.py`：stdio / SSE / HTTP MCP 客户端运行时
- `llm.py`：Anthropic / OpenAI 兼容 LLM 客户端与配置
- `paths.py`：项目根目录与 `runtime/` 路径约定

`aiui-platform` 和 `aiui-executor` 各自在 `aiui/common/` 下自带一份**逐字节相同**的
副本，以便独立安装、独立运行。不要把它作为独立 pip 包安装：`aiui` 是普通包，
装到 site-packages 会与消费方仓库内的 `aiui` 包冲突。

## 修改与同步

1. 共享模块的规范修改先落在本仓库。
2. 同步到 `aiui-platform/aiui/common/` 和 `aiui-executor/aiui/common/`，三份内容必须一致。
3. 提交前用 `diff -r` 或哈希校验确认三份副本一致；后续应由 CI 自动校验。
4. 不要只改消费方副本，否则下次同步会覆盖或产生漂移。
5. 修改公共接口或协议时更新 `VERSION`，并同时检查两个消费方仓库的调用点。

```bash
# 示例：三份副本一致性检查（在包含三个仓库的目录执行）
for f in aiui-common/*.py; do
  name=$(basename "$f")
  cmp "$f" "aiui-platform/aiui/common/$name"
  cmp "$f" "aiui-executor/aiui/common/$name"
done
```

## 关键约束

- **敏感数据**：LLM API key、Profile 原始 storage state、节点 token 不得写入日志、
  API 响应、运行记录或 Git。`profiles.py` 只允许把原始状态写入私有临时文件；
  `llm.py` 的公开配置必须掩码 key。
- **Profile**：不可变、版本化快照；当前只包含 cookies + localStorage；捕获只能来自
  成功的 `kind=setup` 用例；每个任务使用私有副本和全新浏览器上下文；运行时变更不写回。
- **截图**：截图是证据，不是测试判定；不要把 base64 写入运行记录；只读取本系统生成的
  文件引用，不信任外部 MCP 返回的路径；单张上限 8 MiB、采集上限 5 秒。
- **预算**：`budget.py` 只做提示性成本估算，不得影响 passed / failed / retry 判定；
  实际费用以供应商账单为准。
- **MCP 运行时**：`mcp_runtime.py` 负责会话、占位符替换和工具探测；stdio 子进程以项目根
  为 cwd；不要把敏感配置打印到 stdout/stderr。
- **路径**：`paths.py` 必须尊重 `AIUI_RUNTIME_DIR`。测试或子进程要用该变量隔离
  audit、Profile、截图、日志、默认数据库和 LLM 配置，而不是只改 `--db`。
- **协议**：`node_protocol.py` 的版本号、信封字段和校验规则是平台与执行机的共同契约；
  修改必须同时更新两侧并保持向后兼容。

## 测试

本仓库没有独立测试套件；相关测试在消费方仓库：

```bash
# 在 aiui-executor 或 aiui-platform 根目录
.venv/Scripts/python.exe -m unittest discover -s tests -v
```

修改共享模块后，至少运行两个消费方仓库的相关测试；涉及远程协议时还要运行
`aiui-platform` 的 `test_remote_*` 与 `aiui-executor` 的 `test_remote_worker.py`。

## 约定

- 用户可见文本、注释和文档使用中文。
- 只用标准库 + `mcp` / `playwright`；不要引入 Web 框架或前端构建链。
- 文件编码 UTF-8；不要提交 `runtime/`、`__pycache__/`、`.venv/`、日志、截图或 Profile。
