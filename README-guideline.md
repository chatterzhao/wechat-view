# 微信聊天数据分析系统

完整的微信聊天记录分析解决方案：数据解密 + API服务 + 可视化分析 + 报告生成

## 项目组成

```
my-wechat/
├── chatlog/          # 核心后端 (Go) - 解密数据、提供API
├── chatlog-web/      # Web界面 (Vue.js) - 数据可视化
└── wechat-view/      # 报告生成 (Go) - 生成HTML日报
```

## 使用步骤

### 1. 获取代码

```bash
mkdir my-wechat && cd my-wechat

git clone https://github.com/sjzar/chatlog.git           # 核心后端
git clone https://github.com/sinyu1012/chatlog-web.git   # Web界面（可选）
git clone https://github.com/myysophia/wechat-view.git   # 报告生成器（可选）
```

### 2. 编译 chatlog

```bash
cd chatlog/

# 国内用户：设置镜像加速，避免依赖下载超时
go env -w GOPROXY=https://goproxy.cn,direct  # ✅ 已验证可用

# 编译（需要C编译器）
make build                                   # 或: CGO_ENABLED=1 go build -o bin/chatlog main.go

# 编译成功后，可执行文件在 bin/chatlog (macOS/Linux) 或 bin/chatlog.exe (Windows)
```

### 3. 获取密钥（重要！不同系统步骤不同）

#### Windows 用户

```powershell
# 确保微信正在运行
bin\chatlog.exe key                          # 获取密钥，自动保存到配置文件

# 输出示例：
# Data Key: [c0163e***ac3dc6]
# Image Key: [38636***653361]
# 配置已保存: %USERPROFILE%\.chatlog\chatlog.json

# 注意：微信版本需 < 4.0.3.36
# 界面乱码？使用 Windows Terminal
```

#### macOS 用户（需要临时关闭SIP）

```bash
# === 第一步：关闭SIP ===
# Intel Mac: 重启时按住 Command+R 进入恢复模式
# Apple Silicon: 关机后长按电源键，选择"选项"进入恢复模式
# 在恢复模式的终端中执行：
csrutil disable                              # 关闭系统完整性保护
# 然后重启电脑

# === 第二步：安装依赖（首次需要）===
xcode-select --install                       # 安装Xcode命令行工具

# === 第三步：获取密钥（确保微信正在运行）===
./bin/chatlog key                            # 获取密钥，自动保存到 ~/.chatlog/chatlog.json
# 输出示例：
# Data Key: [c0163e***ac3dc6]
# Image Key: [38636***653361]

# === 第四步：立即重新启用SIP（重要！） ===
# 再次进入恢复模式，在终端执行：
csrutil enable                               # 重新启用系统完整性保护
# 然后重启电脑

# ⚠️ SIP关闭期间只做获取密钥这一件事，完成后立即启用！
# ⚠️ 后续所有操作（解密、启动服务等）都在SIP启用状态下进行
# 注意：微信版本需 < 4.0.3.80
```

### 4. 解密数据（SIP已启用状态）

```bash
# Windows
bin\chatlog.exe decrypt                      # 自动读取配置中的密钥，解密所有数据库

# macOS
./bin/chatlog decrypt                        # 自动读取配置中的密钥，解密所有数据库

# 解密后的数据保存在工作目录（默认 ~/.chatlog/decrypted/）
```

### 5. 启动API服务

```bash
# Windows
bin\chatlog.exe server                       # 启动HTTP服务，默认 :5030

# macOS
./bin/chatlog server                         # 启动HTTP服务，默认 :5030

# 测试服务
curl http://127.0.0.1:5030/api/v1/session   # 获取会话列表
```

### 6. 启动Web界面（可选）

```bash
cd ../chatlog-web/

npm install                                  # 首次安装依赖
npm run serve                                # 启动开发服务器，默认 :8080

# 浏览器访问 http://localhost:8080
# 功能：数据图表、聊天记录搜索、联系人管理
```

### 7. 生成HTML报告（可选）

```bash
cd ../wechat-view/

# 编辑配置文件
vi report.config.json
# 修改 talker 为你的群聊ID（可从 http://127.0.0.1:5030/api/v1/session 获取）

# 生成报告
go run ./cmd/report --date 2025-01-15 -v    # 指定日期
go run ./cmd/report -v                       # 默认昨天

# 查看报告
open site/index.html                         # 或浏览器打开
```

## 使用技巧

### 使用Terminal UI（推荐新手）

```bash
./bin/chatlog                                # 启动图形界面
# ↑↓ 选择菜单 | Enter 确认 | Esc 返回 | Ctrl+C 退出
```

### API接口示例

```bash
# 聊天记录
curl "http://127.0.0.1:5030/api/v1/chatlog?time=2025-01-01&talker=wxid_xxx"

# 联系人、群聊、会话
curl http://127.0.0.1:5030/api/v1/contact   # 联系人列表
curl http://127.0.0.1:5030/api/v1/chatroom  # 群聊列表
curl http://127.0.0.1:5030/api/v1/session   # 会话列表

# 多媒体文件
# http://127.0.0.1:5030/image/<id>          # 图片
# http://127.0.0.1:5030/voice/<id>          # 语音（自动转MP3）
# http://127.0.0.1:5030/video/<id>          # 视频
```

### 多账号管理

```bash
# 配置文件自动保存多个账号信息： ~/.chatlog/chatlog.json
# Terminal UI 中可切换账号
# 命令行使用 last_account 指定的账号
```

### 定时生成报告

```bash
# macOS/Linux crontab
0 1 * * * cd /path/to/wechat-view && go run ./cmd/report -v

# Windows 任务计划程序（创建 daily.ps1）
$date = (Get-Date).AddDays(-1).ToString('yyyy-MM-dd')
cd C:\path\to\wechat-view
go run .\cmd\report --date $date -v
```

## 常见问题

```bash
# Q1: 编译时依赖下载慢？
go env -w GOPROXY=https://goproxy.cn,direct

# Q2: macOS获取密钥失败？
csrutil status                               # 检查SIP状态，应显示 disabled
xcode-select --install                       # 安装依赖
# 检查微信版本 < 4.0.3.80

# Q3: Windows获取密钥失败？
# 确认微信正在运行
# 以管理员身份运行 chatlog.exe key
# 检查微信版本 < 4.0.3.36

# Q4: 如何获取群聊ID？
curl http://127.0.0.1:5030/api/v1/session | jq  # API查询
# 或访问 http://localhost:8080 -> 会话列表

# Q5: 电脑聊天记录不全？
# 手机微信：我 - 设置 - 通用 - 聊天记录迁移与备份 - 迁移到电脑
```

## 进阶功能

### Docker 部署

```bash
# 本地获取密钥后，通过环境变量传入
docker pull sjzar/chatlog:latest
docker run -d \
  --name chatlog \
  -p 5030:5030 \
  -v /path/to/wechat/data:/app/data \
  -e DATA_KEY=你的密钥 \
  -e IMG_KEY=你的图片密钥 \
  sjzar/chatlog:latest

# 详见 chatlog/docs/docker.md
```

### AI集成 (MCP)

```bash
# 启动服务后访问 http://127.0.0.1:5030/mcp
# 支持：ChatWise、Cherry Studio、Claude Desktop（需mcp-proxy）
# 详见 chatlog/docs/mcp.md
```

## 参考文档

- [chatlog 完整文档](chatlog/README.md) - 核心功能、API、配置
- [chatlog-web 文档](chatlog-web/README.md) - 前端界面、图表功能
- [wechat-view 文档](wechat-view/README.md) - 报告生成、部署
- [Docker 部署指南](chatlog/docs/docker.md)
- [MCP 集成指南](chatlog/docs/mcp.md)
- [Prompt 示例](chatlog/docs/prompt.md)
- [免责声明](chatlog/DISCLAIMER.md) - ⚠️ 使用前必读

## 安全提示

- ⚠️ 仅处理您自己的合法数据
- ⚠️ macOS关闭SIP有安全风险，获取密钥后立即重新启用
- ⚠️ 妥善保管解密后的数据，避免隐私泄露
- ⚠️ 所有数据处理均在本地完成

## 许可证

- chatlog: Apache-2.0
- chatlog-web: Apache-2.0
- wechat-view: 查看各自许可证

---

**致谢**: [sjzar/chatlog](https://github.com/sjzar/chatlog) | [sinyu1012/chatlog-web](https://github.com/sinyu1012/chatlog-web) | 所有开源贡献者
