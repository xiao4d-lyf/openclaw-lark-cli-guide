---
name: lark-cli-guide
description: Lark/Feishu CLI tool usage guide and quick reference. Use when the user mentions飞书、Lark、lark-cli、飞书CLI、飞书命令行、飞书API、飞书日历、飞书文档、飞书任务、飞书多维表格、飞书通讯录、飞书消息等飞书相关功能。触发时先告知前置安装条件，再提供使用指导。
---

# lark-cli-guide

## 前置安装条件

在使用 lark-cli 之前，需要完成以下安装和配置：

```bash
# 1. 安装 lark-cli
npm install -g @larksuite/cli

# 2. 安装相关 skills
npx skills add https://github.com/larksuite/cli -y -g

# 3. 配置应用
lark-cli config init --new
```

## 概述

`lark-cli` 是飞书/ Lark 的命令行工具，用于调用飞书开放平台的各类 API。

## 基本语法

```bash
lark-cli <command> [subcommand] [method] [options]
lark-cli api <method> <path> [--params <json>] [--data <json>]
lark-cli schema <service.resource.method> [--format pretty]
```

## 主要命令

| 类别 | 命令 | 功能 |
|------|------|------|
| **通用** | `api` | 直接调用任意 Lark API |
| **认证** | `auth` | OAuth 凭证和授权管理 |
| **配置** | `config` / `profile` | 全局配置和多配置文件管理 |
| **日历** | `calendar` | 日历、日程、参会人管理 |
| **多维表格** | `base` | 表格、字段、记录、视图管理 |
| **文档** | `docs` | 云文档操作 |
| **通讯录** | `contact` | 联系人、用户搜索 |
| **消息** | `im` | 消息和群聊管理 |
| **任务** | `task` | 任务、清单、子任务 |
| **网盘** | `drive` | 文件、评论、权限、上传 |
| **审批** | `approval` | 审批实例和任务 |
| **会议** | `vc` | 视频会议和会议纪要 |
| **知识库** | `wiki` | 知识空间和节点 |
| **邮件** | `mail` | 邮件、草稿、文件夹 |
| **电子表格** | `sheets` | 表格操作 |
| **调试** | `doctor` | 配置、认证、连通性检查 |
| **Schema** | `schema` | 查看 API 参数和类型 |

## 常用示例

```bash
# 查看日程
lark-cli calendar +agenda

# 搜索用户
lark-cli contact +search-user --query "用户名"

# 通用 API 调用
lark-cli api GET /open-apis/calendar/v4/calendars

# 带参数调用
lark-cli calendar events instance_view --params '{"calendar_id":"primary"}'
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `--params <json>` | URL/查询参数 |
| `--data <json>` | 请求体（POST/PATCH/PUT/DELETE） |
| `--as <type>` | 身份类型：user / bot / auto |
| `--format <fmt>` | 输出格式：json / ndjson / table / csv / pretty |
| `--page-all` | 自动分页获取所有数据 |
| `--jq <expr>` | 用 jq 表达式过滤 JSON |
| `--dry-run` | 只打印请求，不执行 |
| `--profile <name>` | 使用指定配置文件 |

## 诊断命令

```bash
# 健康检查
lark-cli doctor
```

## 测试验证

安装完成后，可以通过以下命令验证是否正常工作：

```bash
# 查看版本
lark-cli --version

# 查看帮助
lark-cli --help

# 运行健康检查
lark-cli doctor
```

## 已知问题与解决方案

### 1. Windows 上 JSON 参数传递问题

**问题**：在 PowerShell 中直接传递 JSON 参数时，lark-cli 可能无法正确解析。

**解决**：通过管道传递 JSON 文件内容：

```bash
# 将 JSON 写入文件
echo '{"summary":"会议主题",...}' > event.json

# 通过管道传递
Get-Content event.json -Raw | lark-cli api POST /open-apis/calendar/v4/calendars/{calendar_id}/events --data -
```

### 2. jq 过滤时字符串 vs 数值比较

**问题**：时间戳在 JSON 中是字符串类型，直接用 `>=` 比较会按字典序而非数值比较。

**错误示例**：
```bash
# 字符串比较："1744272000" > "1775836800" 返回 true（因为 '7' > '4'）
-q '.items[] | select(.start_time.timestamp >= "1775836800")'
```

**正确做法**：使用 `tonumber` 转换为数值：
```bash
# 数值比较
-q '[.data.items[] | select((.start_time.timestamp | tonumber) >= 1775836800)]'
```

### 3. 查询特定日期范围的日程

**步骤**：
1. 计算正确的时间戳（注意时区）
2. 使用 API 查询
3. 用 jq 过滤出目标日期

```bash
# 计算时间戳（PowerShell）
[System.DateTimeOffset]::Parse("2026-04-11T00:00:00+08:00").ToUnixTimeSeconds()
# 输出: 1775836800

# 查询日程
lark-cli api GET "/open-apis/calendar/v4/calendars/{calendar_id}/events?start_time=1775836800&end_time=1775923199" -q '[.data.items[] | select((.start_time.timestamp | tonumber) >= 1775836800)]'
```

## 消息发送操作

### 1. 搜索用户

```bash
# 搜索用户获取用户 ID
lark-cli contact +search-user --query "用户名"
```

### 2. 发送消息

```bash
# 发送文本消息给用户
lark-cli im +messages-send --user-id "ou_xxxxxxxx" --text "消息内容"

# 发送消息到群聊
lark-cli im +messages-send --chat-id "oc_xxxxxxxx" --text "消息内容"
```

### 3. 发送其他类型消息

```bash
# 发送 Markdown 消息
lark-cli im +messages-send --user-id "ou_xxxxxxxx" --markdown "**加粗文本**"

# 发送富文本消息
lark-cli im +messages-send --user-id "ou_xxxxxxxx" --post '{"zh_cn":{"title":"标题","content":[[{"tag":"text","text":"内容"}]]}}'
```

### 4. 查看消息历史

```bash
# 查看聊天消息
lark-cli im +chat-messages-list --chat-id "oc_xxxxxxxx"

# 搜索消息
lark-cli im +messages-search --query "关键词"
```

## 更多帮助

```bash
# 查看特定命令帮助
lark-cli <command> --help

# 查看 API Schema
lark-cli schema <service.resource.method>
```
