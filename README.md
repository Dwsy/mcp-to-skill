# mcp-to-skill

将任何 MCP 服务器封装为 Claude Skill，支持 stdio/SSE/HTTP 传输协议，使用 uv 管理依赖，实现一键转换、验证和测试。

## Features

- ✅ **多传输协议支持**: stdio/SSE/HTTP
- ✅ **uv 依赖管理**: 比 pip 快 10-100 倍
- ✅ **自动 introspect**: 自动获取工具列表
- ✅ **一键转换**: 从配置到可用技能
- ✅ **验证和测试**: 内置 validate 和 test 命令
- ✅ **上下文节省**: 96%+ 上下文节省

## Supported Transports

### stdio (默认)

标准输入输出传输，大多数 MCP 服务器使用此协议。

```json
{
  "name": "github",
  "transport": "stdio",
  "command": "npx",
  "args": ["@modelcontextprotocol/server-github"],
  "env": {"GITHUB_TOKEN": "your-token"}
}
```

### SSE

通过 HTTP SSE 连接 MCP 服务器，适用于远程 MCP 服务。

```json
{
  "name": "deepwiki",
  "transport": "sse",
  "endpoint": "https://mcp.deepwiki.com/sse"
}
```

### HTTP

HTTP 轮询传输协议（实验性）。

```json
{
  "name": "http-mcp",
  "transport": "http",
  "endpoint": "https://api.example.com/mcp"
}
```

## Installation

```bash
# 确保 uv 已安装
curl -LsSf https://astral.sh/uv/install.sh | sh

# 或使用 brew
brew install uv
```

## Usage

### Convert MCP to Skill

```bash
# 基本用法
bun ~/.pi/agent/skills/mcp-to-skill/lib.ts convert my-mcp.json

# 指定输出目录
bun ~/.pi/agent/skills/mcp-to-skill/lib.ts convert my-mcp.json --output=/custom/path

# 仅生成不安装
bun ~/.pi/agent/skills/mcp-to-skill/lib.ts convert my-mcp.json --no-install
```

### Validate Skill

```bash
bun ~/.pi/agent/skills/mcp-to-skill/lib.ts validate ~/.claude/skills/my-mcp
```

### Test Skill

```bash
# 列出工具
bun ~/.pi/agent/skills/mcp-to-skill/lib.ts test ~/.claude/skills/my-mcp --list

# 查看工具详情
bun ~/.pi/agent/skills/mcp-to-skill/lib.ts test ~/.claude/skills/my-mcp --describe tool_name
```

## MCP Config Format

```json
{
  "name": "my-mcp",
  "transport": "stdio|sse|http",
  "command": "npx",  // stdio only
  "args": ["@example/mcp-server"],  // stdio only
  "endpoint": "https://...",  // sse/http only
  "env": {"API_KEY": "your-key"},
  "keep_alive": {
    "enabled": true,
    "timeout": 3600,
    "check_interval": 60
  }
}
```

## Examples

### Example 1: GitHub MCP

```bash
cat > github-mcp.json << 'EOF'
{
  "name": "github",
  "transport": "stdio",
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": {"GITHUB_TOKEN": "ghp_your_token"}
}
EOF

bun ~/.pi/agent/skills/mcp-to-skill/lib.ts convert github-mcp.json
```

### Example 2: DeepWiki (SSE)

```bash
cat > deepwiki.json << 'EOF'
{
  "name": "deepwiki",
  "transport": "sse",
  "endpoint": "https://mcp.deepwiki.com/sse"
}
EOF

bun ~/.pi/agent/skills/mcp-to-skill/lib.ts convert deepwiki.json
```

### Example 3: Custom MCP

```bash
cat > local-mcp.json << 'EOF'
{
  "name": "local-tools",
  "transport": "stdio",
  "command": "node",
  "args": ["./my-mcp-server.js"],
  "env": {},
  "keep_alive": {
    "enabled": true,
    "timeout": 1800
  }
}
EOF

bun ~/.pi/agent/skills/mcp-to-skill/lib.ts convert local-mcp.json
```

## Generated Skill Structure

转换后的技能包含以下文件：

```
~/.claude/skills/{name}/
├── SKILL.md              # 技能文档
├── executor.py           # Python 执行器
├── process_manager.py    # 进程管理器（可选）
├── pyproject.toml        # uv 项目配置
├── mcp-config.json       # MCP 服务器配置
├── package.json          # 元数据
├── .mcp.pid              # 进程 PID（运行时）
└── .mcp.last_active      # 最后活跃时间（运行时）
```

## Using Generated Skills

```bash
cd ~/.claude/skills/my-mcp

# 列出工具
uv run executor.py --list

# 查看工具详情
uv run executor.py --describe tool_name

# 调用工具
uv run executor.py --call '{"tool": "tool_name", "arguments": {...}}'

# 查看状态
uv run executor.py --status

# 查看统计
uv run executor.py --stats

# 查看日志
uv run executor.py --logs 100

# 过滤日志
uv run executor.py --logs 100 --tool tool_name

# 重置统计
uv run executor.py --reset-stats
```

## Statistics and Logging

### Status

显示技能的整体状态：

```bash
uv run executor.py --status
```

输出包括：
- 总调用次数
- 成功/失败次数
- 运行时间
- 日志文件大小

### Statistics

显示详细的工具调用统计：

```bash
uv run executor.py --stats
```

输出包括：
- 每个工具的调用次数
- 成功率
- 平均响应时间
- 首次/最后调用时间

### Logs

查看最近的调用日志：

```bash
# 查看最近 100 条日志
uv run executor.py --logs

# 查看最近 50 条日志
uv run executor.py --logs 50

# 过滤特定工具的日志
uv run executor.py --logs 100 --tool tool_name
```

### Reset

重置所有统计数据：

```bash
uv run executor.py --reset-stats
```

## Performance

### Context Savings

| Tools | MCP | Skill | Saved |
|-------|-----|-------|-------|
| 8 | 4000 tokens | 150 tokens | 96% |
| 20 | 10000 tokens | 150 tokens | 98.5% |

### Dependency Management

| Metric | pip | uv | Improvement |
|--------|-----|-----|-------------|
| Install time | 10s+ | <1s | 10x+ |
| Virtual env | Manual | Auto | ✅ |
| Resolution | Slow | Fast | 5x+ |

### Process Reuse

| Scenario | Without Reuse | With Reuse | Improvement |
|----------|---------------|------------|-------------|
| First call | 5s | 5s | - |
| Subsequent calls | 5s | <0.5s | 10x |
| 10 calls | 50s | 5s | 10x |

## Keep Alive (Process Reuse)

启用进程复用以提升性能：

```json
{
  "keep_alive": {
    "enabled": true,
    "timeout": 3600,  // 1 hour
    "check_interval": 60  // 1 minute
  }
}
```

## Requirements

- Python 3.10+
- uv (https://astral.sh/uv)
- Bun runtime

## Troubleshooting

### "uv not found"

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### "Unsupported transport"

检查 `transport` 字段是否为 `stdio`/`sse`/`http`。

### "endpoint required"

SSE/HTTP 传输需要 `endpoint` 字段。

### "uv sync failed"

```bash
cd ~/.claude/skills/my-mcp
uv sync --verbose
```

## CLI Commands

```bash
# 显示帮助
bun ~/.pi/agent/skills/mcp-to-skill/lib.ts

# Convert
bun ~/.pi/agent/skills/mcp-to-skill/lib.ts convert <config> [--output=/path] [--no-install]

# Validate
bun ~/.pi/agent/skills/mcp-to-skill/lib.ts validate <path>

# Test
bun ~/.pi/agent/skills/mcp-to-skill/lib.ts test <path> [--list | --describe <tool>]
```

## Executor Commands

```bash
cd ~/.claude/skills/my-mcp

# List tools
uv run executor.py --list

# Describe tool
uv run executor.py --describe tool_name

# Call tool
uv run executor.py --call '{"tool": "...", "arguments": {...}}'

# Show status
uv run executor.py --status

# Show statistics
uv run executor.py --stats

# Show logs
uv run executor.py --logs [limit]

# Filter logs by tool
uv run executor.py --logs 100 --tool tool_name

# Reset statistics
uv run executor.py --reset-stats
```

## Documentation

- [MCP 规范](https://modelcontextprotocol.io)
- [uv 文档](https://astral.sh/uv)
- [mcp-to-skill-converter](https://github.com/GBSOSS/-mcp-to-skill-converter)

## License

MIT

## Credits

Based on [mcp-to-skill-converter](https://github.com/GBSOSS/-mcp-to-skill-converter) by GBSOSS