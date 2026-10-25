# 🐂 大角牛快跑 (Running Bull)

一款基于 HTML5 和 XBuilder 的跑酷游戏，支持横版（PC）和竖版（移动端）双模式。

> **「勇敢牛牛，不怕困难」**

---

## 🎮 在线体验

### XBuilder 版本（竖版跑酷）
**线上运行地址**: https://x.qiniu.com/project/qq-01938eb9bc96cba7/running_bull_v3

- 🎯 背后视角三车道跑酷
- 📱 支持键盘 + 触屏控制
- 🎨 3个角色（大角牛、粉粉猪、咩咩羊）
- 🌍 3张地图（草原、沙漠、夜晚）
- 💰 完整的商店和账号系统

### Web 版本
**竖版（移动端）**: https://bull.lingecho.com/mobile-web/

---

## 📦 项目结构

```
running_bull/
├── running_bull_web/          # Web版本（HTML5 + Python后端）
│   ├── public/                # 前端静态资源
│   │   ├── index.html         # 横版游戏主页
│   │   ├── game.js            # 游戏核心引擎
│   │   ├── ui.js              # UI交互层
│   │   ├── api.js             # API通信
│   │   ├── audio.js           # 音效系统
│   │   └── mobile-web/        # 竖版移动端
│   ├── server.py              # Python后端服务
│   ├── db.py                  # 数据库抽象层
│   ├── schema.mysql.sql       # 数据库表结构
│   ├── deploy/                # 部署配置
│   ├── tests/                 # 测试脚本
│   └── README.md              # Web版说明文档
│
├── running_bull_plugs/        # XBuilder版本（spx语言）
│   ├── running_bull_plugs.xbp # XBuilder工程包（可直接导入）
│   ├── project/               # 源码目录
│   │   ├── main.spx           # 舞台主脚本
│   │   ├── Bull.spx           # 主角控制
│   │   ├── Obstacle.spx       # 障碍物
│   │   ├── Shop.spx           # 商店系统
│   │   ├── Login.spx          # 登录系统
│   │   └── ... (共26个角色)
│   └── README.md              # XBuilder版说明文档
│
├── running_bull_xbuilder/     # XBuilder构建工具
│
├── 📄 文档文件
│   ├── README.md              # 项目总览（本文件）
│   ├── ARCHITECTURE_WEB.md    # Web版架构文档
│   ├── ARCHITECTURE_PLUGS.md  # Plugs版架构文档
│   └── TEAM_ROLES.md          # 团队分工文档
│
├── 🎬 演示文件
│   ├── 录屏2026-10-26 03.54.57.mov  # 游戏演示视频1
│   ├── 录屏2026-10-26 03.58.10.mov  # 游戏演示视频2
│   ├── running_ad.mp4         # 游戏宣传片
│   └── 推广物料.png           # 推广海报素材
```

---

## 📁 核心文件说明

### 📖 文档文件

#### [README.md](README.md)
**项目总览文档（当前文件）**
- 项目介绍与在线体验链接
- 快速开始指南
- 功能特性与操作说明
- 技术栈与架构概览
- 团队信息与开发进度

#### [ARCHITECTURE_WEB.md](ARCHITECTURE_WEB.md)
**Web版完整架构文档**（28KB，详细技术文档）
- 系统架构设计（前端/后端/数据库三层）
- 前端模块详解（Canvas引擎、UI系统、音效）
- REST API接口文档（40+接口）
- 数据库设计（12张表结构）
- 部署方案（Ubuntu + MySQL + Nginx）
- 防作弊机制、安全加固
- 技术亮点与优化方案

**适用人群**: 后端开发、运维工程师、架构师

#### [ARCHITECTURE_PLUGS.md](ARCHITECTURE_PLUGS.md)
**XBuilder版完整架构文档**（38KB，详细技术文档）
- XBuilder平台与spx语言介绍
- 游戏核心架构（26个角色模块）
- 透视投影系统（背后视角三车道）
- 输入处理（键盘+触屏）
- 网络通信（XBuilder专用API）
- 视觉效果与音效系统
- 性能优化与测试调试

**适用人群**: 游戏开发、XBuilder开发者

#### [TEAM_ROLES.md](TEAM_ROLES.md)
**团队分工与协作文档**（26KB，项目管理文档）
- 团队成员职责详解
  - 陈涛：XBuilder版开发、金币系统、云服务搭建
  - 梁超：Web版开发、用户增长、视频推广
  - 葛波：Web版优化、趣味性增强、覆盖测试
- 协作流程与沟通机制
- 开发里程碑（10周计划）
- KPI指标与质量标准
- 风险管理与应对措施

**适用人群**: 项目经理、产品经理、团队成员

---

### 🎬 演示与推广文件

#### 录屏2026-10-26 03.54.57.mov
**游戏实况演示视频 1**（177 MB）
- 内容：完整游戏流程录屏
- 时长：约 3 分钟
- 展示内容：
  - 开局→跳跃/滑铲/换道操作
  - 金币收集与冲刺道具
  - 商店购买角色/地图
  - 撞击复活流程
  - 结算与排行榜

**用途**: 
- 新用户操作教学
- 功能演示与汇报
- 推广视频素材

#### 录屏2026-10-26 03.58.10.mov
**游戏实况演示视频 2**（122 MB）
- 内容：进阶玩法与特性展示
- 时长：约 2 分钟
- 展示内容：
  - 高分技巧（连续跳跃、冲刺时机）
  - 账号登录与数据同步
  - 邀请拉新功能
  - 竖版移动端适配
  - 打击感与音效

**用途**:
- 进阶功能演示
- 竖版模式推广
- 用户增长功能展示

#### running_ad.mp4
**游戏宣传片**
- 内容：精心制作的 30秒游戏宣传短片
- 剪辑要点：
  - 开场：口号「勇敢牛牛，不怕困难」
  - 核心玩法快速展示（跳/滑/换道）
  - 打击感特效（闪白、震屏、火花）
  - 冲刺道具高光时刻
  - 排行榜竞技氛围
  - 结尾：二维码/链接引导下载

**发布平台**:
- 抖音、快手（短视频平台）
- B站、小红书（年轻用户社区）
- 微信朋友圈、微博（社交媒体）

**用途**:
- 社交媒体推广
- 广告投放素材
- 应用商店预览视频

#### 推广物料.png
**推广海报设计稿**
- 规格：高清PNG格式
- 内容元素：
  - 游戏Logo与标题
  - 核心角色形象（大角牛/粉粉猪/咩咩羊）
  - 关键卖点文案
  - 二维码/下载链接
  - 视觉风格：卡通风格、明亮配色

**使用场景**:
- 线上推广（朋友圈海报、公众号头图）
- 线下物料（易拉宝、宣传单）
- 应用商店配图
- 合作渠道推广素材

---

## 🎥 如何使用演示文件

### 查看演示视频
```bash
# macOS
open 录屏2026-10-26\ 03.54.57.mov

# 或直接在Finder中双击播放
```

### 导出视频截图
```bash
# 使用ffmpeg导出关键帧
ffmpeg -i 录屏2026-10-26\ 03.54.57.mov -vf "select='eq(n\,0)'" -frames:v 1 screenshot.png

# 或使用QuickTime Player手动截图（Command + Shift + 4）
```

### 编辑宣传片
推荐工具：
- **Final Cut Pro** (macOS)
- **Adobe Premiere Pro** (跨平台)
- **剪映** (简单易用)
- **DaVinci Resolve** (免费专业工具)

### 压缩视频文件
```bash
# 压缩为适合网络传播的大小
ffmpeg -i running_ad.mp4 -vcodec h264 -acodec aac -b:v 2M running_ad_compressed.mp4
```

---

## 🚀 快速开始

### Web版本 - 本地运行

**需要**: Python 3.8+（无需额外依赖）

```bash
# 进入Web版目录
cd running_bull_web

# 启动服务（默认端口3000）
PUBLIC_URL=http://127.0.0.1:3000 python3 server.py

# 浏览器访问
# 横版：http://127.0.0.1:3000
# 竖版：http://127.0.0.1:3000/mobile-web/
```

**手机测试**:
```bash
# 手机和电脑连同一个Wi-Fi
# 访问：http://<电脑局域网IP>:3000
```

详细说明请查看：[running_bull_web/README.md](running_bull_web/README.md)

### XBuilder版本 - 本地运行

**需要**: [XBuilder](https://xbuild.ai) 开发工具

```bash
1. 打开 XBuilder
2. 文件 → 导入工程
3. 选择 running_bull_plugs/running_bull_plugs.xbp
4. 点击"运行"按钮
```

详细说明请查看：[running_bull_plugs/README.md](running_bull_plugs/README.md)

---

## 🎯 核心功能

### 游戏玩法
- 🏃 跑酷核心：跳跃、滑铲、换道躲避障碍
- ⚡ 冲刺道具：5秒无敌加速、撞飞障碍、吸金币
- 💰 金币收集：购买角色、地图、道具
- 🏆 全服排行榜：实时更新最高分

### 商店系统
**角色**（Web版）:
- 大角牛（免费）
- 蓝角牛（300金币）
- 金角牛（800金币）
- 黑旋风（1500金币）

**角色**（XBuilder版）:
- 大角牛（免费）
- 粉粉猪（300金币）
- 咩咩羊（600金币）

**地图**:
- 青青草原（免费）
- 金秋沙原（200金币）
- 星空夜跑（500金币）

**道具**:
- 无敌护盾（60金币）：本局抵挡一次碰撞
- 双倍金币（80金币）：本局金币翻倍

### 账号系统
- 🎮 **游客模式**：快速开始，无需注册
- 📱 **设备号登录**（Web版）：自动登录，数据云端保存
- 🔐 **昵称+PIN**（XBuilder版）：跨设备同步

### 社交功能
- 🎁 **邀请拉新**：分享邀请码，双方各得200金币
- 📊 **排行榜**：全服Top 20实时更新
- 🏅 **成就徽章**：展示你的在线状态

### 复活机制
- 📺 看广告复活（示意，3秒倒计时）
- 💎 花100金币复活
- 每局只能复活1次

---

## 🎨 操作指南

### 横版（Web版）

| 操作 | 键盘       | 手机                |
| ---- | ---------- | ------------------- |
| 换道 | ↑/↓ 或 W/S | 上下滑动            |
| 跳跃 | J 或 空格  | 点右半屏/右下角"跳" |
| 滑铲 | K          | 点左半屏/左下角"滑" |

### 竖版（XBuilder版 + Web移动版）

| 操作 | 键盘       | 手机         |
| ---- | ---------- | ------------ |
| 换道 | ←/→ 或 A/D | 左右滑动     |
| 跳跃 | ↑/W/空格   | 上滑         |
| 滑铲 | ↓/S        | 下滑         |
| 商店 | B          | 点"商店"按钮 |
| 账号 | L          | 点"账号"按钮 |

**障碍物应对**:
- 🚧 **低栏/石堆** → 跳跃
- 🏗️ **高门** → 滑铲
- 📦 **草垛/木箱** → 换道

---

## 🛠️ 技术栈

### Web版本
- **前端**: HTML5 Canvas + 原生JavaScript（零依赖）
- **后端**: Python 3.8+（仅标准库，MySQL模式需pymysql）
- **数据库**: SQLite（开发）/ MySQL 8.0（生产）
- **部署**: Ubuntu + Nginx + systemd
- **音效**: WebAudio API

### XBuilder版本
- **平台**: XBuilder (https://xbuild.ai)
- **语言**: spx（Go-like语法的游戏脚本）
- **画面**: 480×360 固定尺寸（4:3比例）
- **网络**: HTTP REST API
- **音效**: XBuilder内置音频引擎

---

## 📚 文档导航

### 架构文档
- [Web版架构文档](ARCHITECTURE_WEB.md) - 完整技术架构、API设计、部署方案
- [Plugs版架构文档](ARCHITECTURE_PLUGS.md) - XBuilder实现细节、spx代码结构

### 团队协作
- [团队分工文档](TEAM_ROLES.md) - 成员职责、里程碑、KPI指标

### 子项目文档
- [Web版README](running_bull_web/README.md) - 本地运行、部署、接口文档
- [Plugs版README](running_bull_plugs/README.md) - XBuilder导入、操作说明
- [测试用例](running_bull_web/docs/TEST_CASES.md) - 竖版手工测试清单

---

## 🚀 部署指南

### 生产环境部署（Web版）

**环境要求**:
- Ubuntu 20.04 LTS
- MySQL 8.0
- Python 3.8+
- Nginx
- 域名 + SSL证书

**快速部署**:
```bash
# 1. 安装依赖
sudo apt update && sudo apt install -y mysql-server python3 \
  python3-pymysql nginx certbot python3-certbot-nginx

# 2. 创建数据库
sudo mysql <<'SQL'
CREATE DATABASE running_bull DEFAULT CHARSET utf8mb4;
CREATE USER 'bull'@'localhost' IDENTIFIED BY '你的密码';
GRANT ALL PRIVILEGES ON running_bull.* TO 'bull'@'localhost';
FLUSH PRIVILEGES;
SQL

# 3. 部署代码
cd /opt/running_bull_web
cp deploy/env.example deploy/env
nano deploy/env  # 配置数据库密码

# 4. 启动服务
sudo cp deploy/running-bull.service /etc/systemd/system/
sudo systemctl enable --now running-bull

# 5. 配置Nginx + HTTPS
sudo certbot certonly --nginx -d bull.lingecho.com
sudo cp deploy/nginx.conf /etc/nginx/sites-available/bull.lingecho.com
sudo ln -s /etc/nginx/sites-available/bull.lingecho.com \
           /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

详细部署文档：[ARCHITECTURE_WEB.md#六部署方案](ARCHITECTURE_WEB.md#六部署方案)

---

## 🧪 测试

### 接口测试
```bash
cd running_bull_web

# 冒烟测试
python3 tests/smoke_test.py http://127.0.0.1:3000

# 账号系统测试
python3 tests/account_xb_test.py http://127.0.0.1:3000
```

### 手工测试
参考：[docs/TEST_CASES.md](running_bull_web/docs/TEST_CASES.md)

测试维度：
- ✅ 商业化（充值、购买、广告、邀请）
- ✅ 操作性（跳跃缓冲、空中滑铲、换道）
- ✅ 趣味性（打击感、音效、新纪录）
- ✅ 健壮性（网络异常、参数篡改）

---

## 🎯 项目特色

### 技术亮点

#### 1. 零依赖后端
```python
# 仅用Python标准库实现完整后端
# 快速启动，无需配置复杂环境
python3 server.py
```

#### 2. 双模式前端
- **横版（PC优化）**：侧视角，键盘操作友好
- **竖版（移动端优化）**：背后视角三车道，触屏手势控制

#### 3. 设备号自动登录
```javascript
// 打开页面即自动登录，无需手动注册
// localStorage + Cookie 双备份
// 数据自动同步到服务器
```

#### 4. 渐进式账号系统
```
游客试玩 → 设备号账号 → 昵称+PIN账号
   ↓           ↓              ↓
 不联网    自动登录       跨设备同步
```

#### 5. 透视投影系统（XBuilder版）
```go
// 背后视角：障碍物从远处迎面而来
z := 障碍物距离
scale := 0.2 + z / 600      // 远小近大
y := GY - z * 0.4           // 远上近下
x := laneX * scale          // 透视收缩
```

#### 6. 跳跃缓冲机制
```go
// 空中按跳会"记住"，落地瞬间自动起跳
// 提升操作容错率，连续跳跃更流畅
if jumpH <= 0 {
    startJump()
} else {
    jumpBuf = 9  // 记住，落地时自动跳
}
```

---

## 📊 项目数据

### 代码规模
- **Web版前端**: ~2000行 JavaScript
- **Web版后端**: ~1500行 Python
- **XBuilder版**: ~5000行 spx代码（26个角色）
- **总素材**: 100+ 图片、10+ 音效

### 性能指标
- 游戏帧率: 稳定60 FPS
- 页面加载: < 2秒
- API响应: P95 < 200ms
- 内存占用: < 200MB

### 游戏数据（示例）
- 角色数: 4个（Web版）/ 3个（XBuilder版）
- 地图数: 3张
- 障碍物类型: 5种
- 道具类型: 4种
- 最高分: 服务器实时排行

---

## 👥 团队成员

| 成员     | 职责                        | 主要工作                                         |
| -------- | --------------------------- | ------------------------------------------------ |
| **陈涛** | XBuilder版负责人 & 基础设施 | XBuilder开发、金币系统设计、云服务搭建、证书配置 |
| **梁超** | Web版负责人 & 增长运营      | Web开发、邀请拉新、视频推广                      |
| **葛波** | Web版优化 & 质量保障        | 二创更新、趣味性增强、覆盖测试                   |

详细分工：[TEAM_ROLES.md](TEAM_ROLES.md)

---

## 🗓️ 开发里程碑

- ✅ **Week 1**: 需求评审、技术方案设计
- ✅ **Week 2-4**: XBuilder版 + Web版 Alpha
- ✅ **Week 3-4**: 云服务搭建、HTTPS配置
- ✅ **Week 5-6**: 商店/账号/邀请功能完善
- 🚧 **Week 6-7**: 二创开发 + 增长功能
- 📅 **Week 7-8**: 覆盖测试
- 📅 **Week 9**: Beta测试
- 📅 **Week 10**: 正式上线

---

## 📝 已知限制

### Web版
- 充值/广告为示意（不接真实支付/广告平台）
- 防作弊依赖时间戳（可被绕过）
- 单进程架构（并发能力有限）
- 横版页面在竖屏设备上体验欠佳

### XBuilder版
- 画面固定4:3（480×360像素）
- 无本地存储API（spx v3.3.0）
- 网络请求同步阻塞
- 同屏对象过多时帧率下降

---

## 🔮 未来规划

### 短期优化
- [ ] 替换后端框架（FastAPI + 连接池）
- [ ] 添加接口签名验证
- [ ] 引入Redis缓存（排行榜、会话）
- [ ] 完善错误监控（Sentry）

### 中期规划
- [ ] 接入真实支付平台
- [ ] 接入广告SDK
- [ ] 添加背景音乐
- [ ] 优化横版响应式布局

### 长期目标
- [ ] 微服务化拆分
- [ ] 实时对战模式
- [ ] 社交功能（好友、公会）
- [ ] 赛季排行榜

---

## 🤝 贡献指南

我们欢迎任何形式的贡献！

### 如何贡献
1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 提交 Pull Request

### 代码规范
- JavaScript: 遵循 [Airbnb Style Guide](https://github.com/airbnb/javascript)
- Python: 遵循 [PEP 8](https://pep8.org/)
- spx: 遵循 [spx Best Practices](https://github.com/goplus/spx)
- 提交前运行测试: `python3 tests/smoke_test.py`

---

## 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

---

## 🔗 相关链接

### 在线体验
- **XBuilder版**: https://x.qiniu.com/project/qq-01938eb9bc96cba7/running_bull_v3
- **Web横版**: https://bull.lingecho.com
- **Web竖版**: https://bull.lingecho.com/mobile-web/

### 📺 推广视频
- **抖音推广视频**: [勇敢牛牛 - 魔性上头游戏](https://v.douyin.com/CpxmeXitbdA/)
  - 作者：你南山哥哥
  - 标签：#魔性上头游戏 #搞笑沙雕视频
  - 平台：抖音极速版
  - 💡 点击链接在抖音观看精彩演示！

### 开发工具
- **XBuilder**: https://xbuild.ai
- **spx文档**: https://github.com/goplus/spx

### 技术支持
- **GitHub Issues**: https://github.com/your-org/running-bull/issues
- **开发文档**: [ARCHITECTURE_WEB.md](ARCHITECTURE_WEB.md)

---

## 📮 联系我们

- **邮箱**: support@example.com
- **GitHub**: https://github.com/your-org/running-bull
- **反馈**: 欢迎在 Issues 中提出建议和问题

---

## 🎉 致谢

感谢所有为本项目做出贡献的开发者和玩家！

特别感谢：
- XBuilder 团队提供优秀的游戏开发平台
- 社区成员的宝贵反馈和建议
- 所有测试玩家的支持

---

**最后更新**: 2026-10-26  
**项目版本**: v3.0  
**文档维护**: 陈涛

---

<div align="center">

**🐂 勇敢牛牛，不怕困难 🐂**

Made with ❤️ by Running Bull Team

[开始游戏](https://x.qiniu.com/project/qq-01938eb9bc96cba7/running_bull_v3) · [查看文档](ARCHITECTURE_WEB.md) · [报告问题](https://github.com/your-org/running-bull/issues)

</div>
