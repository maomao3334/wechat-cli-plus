<div align="center">

# WeChat CLI Plus

**WeChat CLI 增强版 - 支持微信 4.1+ 版本**

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)](https://github.com/maomao3334/wechat-cli-plus)
[![WeChat](https://img.shields.io/badge/WeChat-4.1.13+-green.svg)](https://github.com/maomao3334/wechat-cli-plus)

命令行查询本地微信数据 · 支持最新版微信 · 增强密钥提取

[English](README_EN.md) | 中文文档

</div>

---

## 🎯 关于本项目

这是 [wechat-cli](https://github.com/huohuoer/wechat-cli) 的维护分支，专门适配**微信 4.1+ 版本**。

### 🆕 相比原版的改进

- ✅ **支持微信 4.1.13+** - 使用 Config.Cipher 运行时扫描技术
- ⚡ **提取速度更快** - 8-10 秒完成所有密钥提取（原版需数分钟）
- 🎯 **准确率更高** - 基于 [wcdb-key-tool](https://github.com/ppwwyyxx/wechat-dump) 的成熟实现
- 🔒 **保持兼容** - 同时支持新旧版本微信

### 📦 版本支持

| 微信版本 | 提取方法 | 支持状态 |
|---------|---------|---------|
| 3.9.x 及更早 | 传统内存扫描 | ✅ 支持 |
| 4.0.x - 4.1.x | Config.Cipher 扫描 | ✅ 支持 |
| 4.1.13+ | Config.Cipher 扫描 | ✅ 已测试 |

---

## ✨ 核心特性

- **🚀 零配置安装** — `pip install` 即用，无需额外依赖
- **📦 11 个命令** — sessions、history、search、contacts、members、stats、export、favorites、unread、new-messages、init
- **🤖 AI 优先** — 默认 JSON 输出，为 LLM agent 工具调用设计
- **🔒 完全本地** — 实时 SQLCipher 解密，数据不离开本机
- **📊 丰富分析** — 消息统计、活跃时段、成员排名
- **📝 灵活导出** — Markdown 或纯文本，支持时间范围过滤

---

## 📥 安装

### pip（推荐）

```bash
pip install git+https://github.com/maomao3334/wechat-cli-plus.git
```

需要 Python >= 3.10。

### 从源码安装

```bash
git clone https://github.com/maomao3334/wechat-cli-plus.git
cd wechat-cli-plus
pip install -e .
```

---

## 🚀 快速开始

### 第 1 步 — 初始化

确保微信正在运行，然后：

```bash
# Windows: 以管理员权限运行
wechat-cli init

# macOS/Linux: 需要 sudo 权限扫描内存
sudo wechat-cli init
```

自动检测微信数据目录、提取加密密钥，配置保存到 `~/.wechat-cli/`。

**测试结果示例（Windows 微信 4.1.13）：**
```
找到 24 个数据库, 24 个不同的 salt
[+] Weixin.exe PID=27100 (890MB)
[*] Config.Cipher 扫描: 找到 1 个名称匹配
[+] Config.Cipher 扫描匹配 24/24 salts (候选=31)
扫描完成: 8.6s
```

#### Windows 注意事项

- 需要管理员权限扫描 Weixin.exe 进程内存
- 确保微信正在运行
- 如遇权限问题，右键"以管理员身份运行"命令行

#### macOS 注意事项

详见原版文档中的"macOS: Grant Full Disk Access to Terminal"和"macOS: task_for_pid failed Error"部分。

### 第 2 步 — 使用

```bash
wechat-cli sessions                        # 最近会话
wechat-cli history "Alice" --limit 20      # 聊天记录
wechat-cli search "deadline" --chat "Team" # 搜索消息
```

---

## 🤖 与 AI Agent 集成

WeChat CLI Plus 是为 AI agent 工具调用设计的，所有命令默认输出结构化 JSON。

### Claude Code

在项目的 `CLAUDE.md` 中添加：

```markdown
## WeChat CLI

你可以使用 `wechat-cli` 查询我的本地微信数据。

常用命令：
- `wechat-cli sessions --limit 10` — 最近会话
- `wechat-cli history "名字" --limit 20 --format text` — 聊天记录
- `wechat-cli search "关键词" --chat "聊天名"` — 搜索消息
- `wechat-cli contacts --query "名字"` — 搜索联系人
- `wechat-cli unread` — 未读消息
- `wechat-cli new-messages` — 自上次检查后的新消息
```

然后你可以问 Claude：
- "查看我的微信未读消息"
- "在团队群里搜索关于项目截止日期的消息"
- "这周谁在 AI 群里发消息最多？"

---

## 📖 命令参考

完整命令参考请见原版 README 的"Command Reference"部分。

主要命令：
- `sessions` — 最近会话
- `history` — 聊天记录
- `search` — 搜索消息
- `contacts` — 联系人查询
- `members` — 群成员列表
- `stats` — 聊天统计
- `export` — 导出聊天记录
- `favorites` — 微信收藏
- `unread` — 未读会话
- `new-messages` — 增量新消息

---

## 🔧 工作原理

微信将聊天数据存储在本地的 SQLCipher 加密 SQLite 数据库中。WeChat CLI Plus：

1. **提取密钥** — 扫描微信进程内存提取加密密钥（`init`）
   - **Windows 4.1+**: 使用 Config.Cipher 运行时扫描 + XOR 解码
   - **旧版本**: 回退到传统十六进制模式匹配
2. **实时解密** — 透明的页级 AES-256-CBC 解密，带缓存
3. **本地查询** — 所有数据留在本机，无网络访问

### 技术细节（Windows 4.1+）

微信 4.1+ 使用 `com.Tencent.WCDB.Config.Cipher` 结构在运行时存储密钥，并通过 XOR 混淆保护。本工具：

1. 在进程内存中搜索 Config.Cipher 字符串
2. 定位指向该结构的引用
3. 读取并 XOR 解码密钥 blob
4. 使用 HMAC-SHA512 验证提取的密钥

---

## 💻 系统要求

| 平台 | 状态 | 备注 |
|------|------|------|
| Windows | ✅ 已测试 | 微信 4.1.13.12 @ Windows 11 |
| macOS | ✅ 应支持 | 未在最新版测试 |
| Linux | ✅ 应支持 | 需要 root 权限 |

---

## 📄 许可证

[Apache License 2.0](LICENSE)

继承自原项目 [wechat-cli](https://github.com/huohuoer/wechat-cli)。

---

## ⚖️ 免责声明

本项目仅为个人数据查询工具，供个人使用。请注意：

- **只读** — 仅读取本地存储的数据，不发送、修改或删除任何消息
- **无云端传输** — 所有数据留在本地，不上传到任何服务器
- **不干扰微信生态** — 不干扰微信正常运行，不自动化任何操作，不违反微信服务条款
- **风险自负** — 本项目仅供个人学习研究，使用者需确保遵守当地法律法规

---

## 🙏 致谢

- 原版项目：[wechat-cli](https://github.com/huohuoer/wechat-cli) by huohuoer
- 核心解密：[wechat-decrypt](https://github.com/ylytdeng/wechat-decrypt)
- 密钥提取：[wcdb-key-tool](https://github.com/ppwwyyxx/wechat-dump) 的 Windows 4.1+ 实现

---

## 🔗 相关项目

- [wechat-cli](https://github.com/huohuoer/wechat-cli) - 原版项目（已提交 [PR #22](https://github.com/huohuoer/wechat-cli/pull/22)）
- [wcdb-key-tool](https://github.com/ppwwyyxx/wechat-dump) - 微信密钥提取工具

---

## 📮 维护者

[@maomao3334](https://github.com/maomao3334)

如有问题或建议，欢迎提 issue。
