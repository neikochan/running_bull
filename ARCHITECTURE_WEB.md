# 大角牛快跑 Web 版 - 产品架构文档

## 一、产品概述

### 1.1 项目定位
大角牛快跑 Web 版是一款基于 HTML5 Canvas + 原生 JavaScript 实现的跑酷游戏网页原型，支持横版和竖版两种游戏模式，配备 Python 后端服务和数据库系统。

### 1.2 技术栈概览
- **前端技术**: HTML5 Canvas、原生 JavaScript（无框架依赖）
- **后端技术**: Python 3.8+ 标准库（仅 MySQL 模式需要 pymysql）
- **数据库**: SQLite（默认）/ MySQL（生产环境）
- **部署方案**: Nginx + Systemd / PM2
- **音效系统**: WebAudio API

### 1.3 产品特性
- 横版跑酷（PC优化）+ 竖版三道跑酷（移动端优化）
- 游客账号系统（基于设备号）
- 在线排行榜、邀请拉新机制
- 商店系统（角色、地图、道具）
- 示意充值/广告复活（非真实支付）
- 防作弊机制（基于时间封顶）

---

## 二、系统架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────┐
│                      客户端层                            │
├──────────────────┬──────────────────────────────────────┤
│  横版前端        │  竖版前端 (mobile-web)                │
│  - index.html    │  - index.html                         │
│  - game.js       │  - game.js                            │
│  - ui.js         │  - sync.js                            │
│  - audio.js      │  - 竖屏手势控制                       │
│  - api.js        │  - 飞机大战小游戏                     │
└──────────────────┴──────────────────────────────────────┘
                           ↓ HTTP/HTTPS
┌─────────────────────────────────────────────────────────┐
│                      反向代理层                          │
│  Nginx (生产环境)                                        │
│  - 静态文件服务 (public/)                                │
│  - API 反向代理 (/api/ → 127.0.0.1:3000)                │
│  - HTTPS 证书管理                                        │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                     应用服务层                           │
│  Python HTTP Server (server.py)                         │
│  - ThreadingHTTPServer                                  │
│  - REST API 接口 (/api/*)                                │
│  - 静态文件服务（开发模式）                              │
│  - 业务逻辑处理                                          │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                     数据访问层                           │
│  db.py (数据库抽象层)                                    │
│  - SQLite 模式（默认，单文件）                           │
│  - MySQL 模式（生产环境，需配置）                        │
│  - 自动迁移机制                                          │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                      数据存储层                          │
│  SQLite (data/running_bull.db)  或  MySQL Server        │
└─────────────────────────────────────────────────────────┘
```

### 2.2 目录结构

```
running_bull_web/
├── server.py              # 后端主程序（HTTP 服务 + REST API）
├── db.py                  # 数据库抽象层（SQLite/MySQL 切换）
├── schema.mysql.sql       # MySQL 表结构定义
├── README.md              # 项目说明文档
│
├── public/                # 前端静态资源
│   ├── index.html         # 横版游戏主页
│   ├── game.js            # 游戏核心逻辑
│   ├── ui.js              # UI 交互逻辑
│   ├── api.js             # API 请求封装
│   ├── audio.js           # 音效系统
│   ├── style.css          # 样式表
│   ├── assets/            # 游戏素材（图片、音效）
│   └── mobile-web/        # 竖版移动端页面
│       ├── index.html     # 竖版主页
│       ├── game.js        # 竖版游戏逻辑
│       ├── sync.js        # 与后端同步
│       ├── about.html     # 关于页面
│       └── privacy.html   # 隐私政策
│
├── deploy/                # 部署配置
│   ├── nginx.conf         # Nginx 配置模板
│   ├── running-bull.service  # Systemd 服务配置
│   └── env.example        # 环境变量示例
│
├── tests/                 # 测试脚本
│   ├── smoke_test.py      # 接口冒烟测试
│   ├── account_xb_test.py # 账号系统测试
│   └── README_robustness.md  # 健壮性测试说明
│
├── docs/                  # 文档
│   └── TEST_CASES.md      # 测试用例文档
│
├── screenshots/           # 测试截图
└── tools/                 # 工具脚本
    └── make_assets.py     # 素材生成工具
```

---

## 三、前端架构

### 3.1 横版前端（PC 优化）

#### 3.1.1 核心模块

**game.js - 游戏引擎核心**
```javascript
主要职责：
- Canvas 渲染循环（requestAnimationFrame）
- 游戏状态机（标题 → 游戏中 → 结束）
- 物理系统（跳跃、滑铲、碰撞检测）
- 障碍物生成算法（低栏、高栏、草垛、木箱、石堆）
- 道具系统（金币、闪电冲刺）
- 速度递增机制
- 打击感效果（闪白、震屏、火花、扬尘）

关键技术：
- 透视投影绘制（栅栏前后两片营造立体感）
- 分层渲染（背景 → 装饰 → 障碍物 → 牛 → 特效 → UI）
- 碰撞盒优化（跳跃/滑铲状态下调整判定区域）
```

**ui.js - UI 交互层**
```javascript
主要职责：
- 商店界面（角色、地图、道具购买）
- 排行榜展示
- 邀请拉新面板
- 设置面板（音效开关）
- 游戏结算界面
- Toast 提示系统

设计模式：
- 模态对话框管理（弹出/关闭/切换）
- 金币动画效果（飘字 +10）
- 响应式布局适配
```

**api.js - 后端通信**
```javascript
主要职责：
- REST API 请求封装
- 设备号管理（localStorage）
- 错误处理与重试
- 请求队列管理

关键接口：
- POST /api/guest - 游客自动登录
- POST /api/runs - 开始一局
- POST /api/runs/{id}/finish - 结算
- GET /api/leaderboard - 排行榜
- POST /api/shop/buy - 购买物品
```

**audio.js - 音效系统**
```javascript
主要职责：
- WebAudio API 合成音效
- 音效池管理
- 音量控制
- 静音开关

音效类型：
- 跳跃/滑铲音效
- 金币收集音效
- 撞击音效
- 冲刺音效
- UI 点击音效
```

#### 3.1.2 操作控制

| 操作 | 键盘 | 功能 |
|-----|------|------|
| 换道 | ↑/↓ 或 W/S | 切换上下车道 |
| 跳跃 | J 或 空格 | 跳过低栏/石堆 |
| 滑铲 | K | 滑过高栏 |

### 3.2 竖版前端（移动端优化）

#### 3.2.1 特性差异

**布局设计**
- 竖屏三车道设计（左/中/右）
- 触屏手势控制（上下左右滑动）
- 首页导航（开始游戏、排行榜）
- 关于/隐私政策页面

**游戏玩法**
- 背后视角（障碍从远处迎面跑来）
- 距离计分（不同于横版的时间计分）
- 飞机大战小游戏
- 口号：「勇敢牛牛，不怕困难」

**技术实现**
```javascript
// mobile-web/game.js
- 三车道透视投影
- 触摸事件处理（touchstart/touchmove/touchend）
- 滑动方向识别
- 竖屏排行榜（生涯总积分/总金币/总距离）
```

#### 3.2.2 设备号自动登录

```javascript
登录流程：
1. 读取 localStorage.rb_device（8-64位字符）
2. 备份检查 Cookie.rb_device
3. 不存在则生成：d + 32位十六进制（crypto.getRandomValues）
4. 同时写入 localStorage 和 Cookie（Path=/, SameSite=Lax, 1年）
5. 自动 POST /api/guest（带 device_id 和 client_hint）

安全考虑：
- 设备号即登录凭据（不要外泄）
- client_hint 用于指纹识别（不参与查账号）
- 清除数据后会生成新账号（旧号不可恢复）
```

---

## 四、后端架构

### 4.1 服务端设计 (server.py)

#### 4.1.1 技术选型原因

```python
优势：
- 零依赖启动（仅用 Python 标准库）
- 快速原型验证
- 适合演示和小流量场景

局限：
- 单进程单连接（并发能力有限）
- 需手动实现 API 路由
- 生产环境建议换成 FastAPI/Flask + 连接池
```

#### 4.1.2 核心模块

**1. 商品目录管理**
```python
CATALOG = {
    'characters': [大角牛(免费), 蓝角牛(300), 金角牛(800), 黑旋风(1500)],
    'backgrounds': [青青草原(免费), 金秋沙原(200), 星空夜跑(500)],
    'consumables': [无敌护盾(60), 双倍金币(80)],
    'topup': [充值档位 - 仅示意],
}

规则：
- 服务端价格为准（防篡改）
- 角色/地图购买后自动装备
- 道具可购买多个
```

**2. 防作弊机制**
```python
# 按真实时长限制分数/金币上限
SCORE_PER_SEC = 220      # 横版：最快约170分/秒
COINS_PER_SEC = 12       # 平均金币 < 8个/秒
PORTRAIT_SCORE_PER_SEC = 900   # 竖版：220-400分/秒
PORTRAIT_DIST_PER_SEC = 800

def finish_run():
    elapsed_sec = (now() - run.started_at) / 1000
    max_score = elapsed_sec * SCORE_PER_SEC
    score = min(submitted_score, max_score)
    # 超限按上限记录
```

**3. 邀请拉新系统**
```python
规则：
- 每人生成 6位邀请码（去掉易混淆的 O/I）
- 新设备首次打开 /?invite=邀请码 触发
- 双方各得 200金币
- 防刷机制：
  * 只对新设备生效一次
  * 不能邀请自己
  * 同公网IP不发奖（本机127.0.0.1除外）
  * 每人最多奖励 50次
```

**4. 广告复活系统**
```python
流程：
1. POST /api/ads → 返回 {ad_id, seconds: 3, mock: true}
2. 前端播放3秒"示意广告"
3. POST /api/runs/{id}/revive (method: "ad", ad_id: ...)
4. 服务端检查：真的看满3秒 + 每局只能复活一次

复活方式：
- 看广告（免费，每局1次）
- 花100金币（每局1次）
```

### 4.2 REST API 设计

#### 4.2.1 认证方式

```
优先级（从高到低）：
1. X-Session 请求头（账号登录 token，30天有效）
2. X-Device-Id 请求头（设备号，8-64位）
3. Cookie: rb_session（微信网页授权，默认关闭）

设备号格式：[a-zA-Z0-9_-]{8,64}
```

#### 4.2.2 核心接口

**游客系统**
```
POST /api/guest
请求: {device_id, nickname?, invite_code?, client_hint?}
返回: {created: bool, invite: {...}, user: {...}}
说明: 新设备送100金币，自动处理邀请码
```

**游戏流程**
```
标准流程：
POST /api/runs {use_shield?, use_double?}
  → 返回 {run_id, shield, double, user}
  
→ 游戏进行中
  
→ (可选) POST /api/ads {run_id}
  → POST /api/runs/{run_id}/revive {method: "ad|coins", ad_id?}
  
→ POST /api/runs/{run_id}/finish
  {score, coins, board?, mode?, hero?, distance?, play_ms?}
  → 返回 {score, earned, capped, new_record, rank, user}
```

**商店系统**
```
GET /api/catalog - 获取商品列表（价格以此为准）
POST /api/shop/buy {item_id, qty?} - 购买物品
POST /api/me/equip {item_id} - 装备角色/地图
```

**账号系统（v5 新增）**
```
POST /api/account/register {nickname, pin}
  → 返回 token（30天有效）

POST /api/account/login {nickname, pin}
  → 昵称不区分大小写
  → PIN错误 403（带剩余次数）
  → 连错5次锁10分钟（429）

POST /api/account/set_pin {nickname, pin} + X-Device-Id
  → 游客升级为账号（可跨设备登录）

POST /api/account/delete {pin} + X-Session
  → 删除账号及全部数据
```

**竖版专用接口**
```
POST /api/runs?board=portrait
  → 竖版对局（分数不写入全服 best）

GET /api/portrait/leaderboard?metric=score|coins|distance
  → 竖版排行榜（生涯累计数据）

GET /api/history?board=portrait&limit=20
  → 当前账号的对局历史
```

**XBuilder 版专用接口**
```
POST /api/xb/* - 返回扁平 JSON（方便 spx 解析）

关键差异：
- 角色：粉粉猪(300)、咩咩羊(600) 仅 XB 版
- 装备记录：单独的 xb_prefs 表（不影响网页版）
- 返回格式：{ok, nick, wallet, chars(位掩码), ...}

GET /api/xb/top → {text: "前10名排版好的文字"}
```

#### 4.2.3 错误处理

```
HTTP 状态码：
200 - 成功
400 - 参数错误
401 - 未登录/认证失败
402 - 金币不足
403 - PIN 错误
404 - 资源不存在
409 - 状态冲突（如已购买）
429 - 请求过于频繁（限流）
503 - 服务未配置（如微信登录）

返回格式：
成功: {data, user?, ...}
失败: {"error": "中文说明"}
```

### 4.3 数据访问层 (db.py)

#### 4.3.1 数据库切换机制

```python
# SQLite 模式（默认）
SQLITE_PATH = os.environ.get('SQLITE_PATH', 'data/running_bull.db')

# MySQL 模式（配置任一组即切换）
# 方式一
DB_URL = 'mysql://bull:密码@127.0.0.1:3306/running_bull'

# 方式二
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=bull
DB_PASS=密码
DB_NAME=running_bull

安装依赖：
pip3 install pymysql  # 或 apt install python3-pymysql
```

#### 4.3.2 自动迁移机制

```python
def migrate():
    """启动时自动执行表结构迁移"""
    # 1. 执行 schema.mysql.sql 的 CREATE TABLE IF NOT EXISTS
    # 2. 给已有表补充新增列：
    #    - runs: board, mode, hero, distance, play_ms, wx_openid
    #    - users: client_hint
    # 3. 创建新表（不影响已有数据）
    
优点：
- 无需手工执行 ALTER TABLE
- 支持从旧版本平滑升级
- SQLite/MySQL 共用同一套迁移逻辑
```

---

## 五、数据库设计

### 5.1 核心表结构

#### users - 用户表
```sql
id            INT PRIMARY KEY AUTO_INCREMENT
nickname      VARCHAR(50)        -- 昵称
invite_code   VARCHAR(6) UNIQUE  -- 邀请码
coins         INT DEFAULT 100    -- 金币钱包
best          INT DEFAULT 0      -- 最高分（横版）
equipped_char VARCHAR(20)        -- 当前角色
equipped_bg   VARCHAR(20)        -- 当前地图
invited_count INT DEFAULT 0      -- 邀请人数
invite_earned INT DEFAULT 0      -- 邀请收益
device_ids    TEXT               -- 关联设备号列表
client_hint   VARCHAR(32)        -- 客户端指纹（不参与查账号）
created_at    BIGINT             -- 创建时间（毫秒时间戳）
```

#### runs - 对局记录
```sql
id          INT PRIMARY KEY AUTO_INCREMENT
user_id     INT                      -- 关联 users.id
score       INT                      -- 分数
coins       INT                      -- 本局金币
shield      INT                      -- 开局护盾数
double      TINYINT                  -- 是否双倍金币
revived     TINYINT                  -- 是否复活过
board       VARCHAR(10)              -- landscape/portrait
mode        VARCHAR(20)              -- 游戏模式
hero        VARCHAR(20)              -- 使用角色
distance    INT                      -- 跑过距离（竖版）
play_ms     INT                      -- 游戏时长（毫秒）
wx_openid   VARCHAR(64)              -- 微信 OpenID
started_at  BIGINT                   -- 开局时间
ended_at    BIGINT                   -- 结算时间

INDEX (user_id, board, ended_at)
INDEX (ended_at DESC)
```

#### inventory - 背包
```sql
user_id   INT           -- 关联 users.id
item_id   VARCHAR(20)   -- 物品ID（shield/double/bomb/...）
qty       INT           -- 数量

PRIMARY KEY (user_id, item_id)
```

#### coin_log - 金币流水
```sql
id         INT PRIMARY KEY AUTO_INCREMENT
user_id    INT                    -- 关联 users.id
delta      INT                    -- 金币变动（正负）
reason     VARCHAR(50)            -- 原因（buy/topup/run/invite）
created_at BIGINT                 -- 时间戳
```

#### invites - 邀请记录
```sql
inviter_id  INT       -- 邀请人
invitee_id  INT       -- 被邀请人
ip_hash     VARCHAR(64)  -- IP哈希（防刷）
created_at  BIGINT

PRIMARY KEY (inviter_id, invitee_id)
INDEX (invitee_id)
INDEX (ip_hash, created_at)
```

#### accounts - 账号系统（v5）
```sql
id          INT PRIMARY KEY AUTO_INCREMENT
nickname    VARCHAR(50) UNIQUE    -- 昵称（唯一）
pin_hash    VARCHAR(255)          -- PIN 哈希（PBKDF2-SHA256）
pin_salt    VARCHAR(32)           -- 盐值
user_id     INT UNIQUE            -- 关联 users.id
created_at  BIGINT
```

#### sessions - 会话管理
```sql
token       VARCHAR(64) PRIMARY KEY
account_id  INT                   -- 关联 accounts.id
expires_at  BIGINT                -- 30天有效期
```

#### xb_prefs - XBuilder 装备
```sql
account_id  INT PRIMARY KEY       -- 关联 accounts.id
char_id     INT                   -- 角色 0/1/2
map_id      INT                   -- 地图 0/1/2
```

#### wechat_users - 微信用户
```sql
openid      VARCHAR(64) PRIMARY KEY
user_id     INT UNIQUE            -- 关联 users.id
nickname    VARCHAR(100)
headimgurl  TEXT
created_at  BIGINT
```

#### oauth_states - 微信授权临时态
```sql
state       VARCHAR(64) PRIMARY KEY
used        TINYINT DEFAULT 0
created_at  BIGINT
```

### 5.2 竖版数据扩展

**portrait_totals - 竖版生涯统计（虚拟表）**
```sql
-- 从 runs 表实时计算，不持久化
SELECT 
  user_id,
  SUM(score) AS total_score,
  SUM(coins) AS total_coins,
  SUM(distance) AS total_distance
FROM runs
WHERE board = 'portrait' AND ended_at IS NOT NULL
GROUP BY user_id
```

**竖版排行榜查询**
```python
GET /api/portrait/leaderboard?metric=score|coins|distance

按生涯累计排序（非单局最高分）
```

---

## 六、部署方案

### 6.1 本地开发

```bash
# 最简启动（无需安装依赖）
cd running_bull_web
PUBLIC_URL=http://127.0.0.1:3000 python3 server.py

# 访问
http://127.0.0.1:3000          # 横版
http://127.0.0.1:3000/mobile-web/  # 竖版

# 手机测试（同一 Wi-Fi）
http://<电脑局域网IP>:3000
```

### 6.2 生产部署（Ubuntu + MySQL + Nginx）

#### 6.2.1 架构图

```
[用户] 
  ↓ HTTPS
[Nginx :443] 
  ├─ /api/* → [Python :3000]
  └─ /*     → /opt/running_bull_web/public/
```

#### 6.2.2 部署步骤

```bash
# 1. 安装依赖
sudo apt update
sudo apt install -y mysql-server python3 python3-pymysql nginx \
    certbot python3-certbot-nginx

# 2. 创建数据库
sudo mysql <<'SQL'
CREATE DATABASE running_bull DEFAULT CHARSET utf8mb4;
CREATE USER 'bull'@'localhost' IDENTIFIED BY '强密码';
GRANT ALL PRIVILEGES ON running_bull.* TO 'bull'@'localhost';
FLUSH PRIVILEGES;
SQL

# 3. 部署代码
cd /opt/running_bull_web
cp deploy/env.example deploy/env
nano deploy/env  # 配置数据库密码

sudo chown -R www-data:www-data /opt/running_bull_web
sudo chmod 600 deploy/env

# 4. Systemd 服务
sudo cp deploy/running-bull.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now running-bull
sudo systemctl status running-bull

# 5. Nginx + HTTPS
sudo certbot certonly --nginx -d bull.lingecho.com
sudo cp deploy/nginx.conf /etc/nginx/sites-available/bull.lingecho.com
sudo ln -s /etc/nginx/sites-available/bull.lingecho.com \
           /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

#### 6.2.3 环境变量

```bash
# deploy/env 示例
PORT=3000
HOST=127.0.0.1           # 只监听本地（Nginx 反代）
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=bull
DB_PASS=你的密码
DB_NAME=running_bull
PUBLIC_URL=https://bull.lingecho.com
TRUST_PROXY=1            # 识别 X-Forwarded-For
QUIET=1                  # 不打印访问日志

# 微信登录（可选，不配置则关闭）
WECHAT_APP_ID=wx...
WECHAT_APP_SECRET=...
WECHAT_REDIRECT_URI=https://bull.lingecho.com/api/wechat/callback
WECHAT_OAUTH_SCOPE=snsapi_userinfo
```

#### 6.2.4 日志管理

```bash
# 查看服务日志
journalctl -u running-bull -f

# 查看 Nginx 日志
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# 数据库日志
sudo tail -f /var/log/mysql/error.log
```

### 6.3 扩展方案

#### 6.3.1 水平扩展建议

```
当前瓶颈：
- 单进程单连接（ThreadingHTTPServer）
- 数据库连接未池化

改进方案：
1. 更换框架：FastAPI / Flask + Gunicorn
2. 数据库连接池：SQLAlchemy / asyncpg
3. 缓存层：Redis（排行榜、会话）
4. CDN：静态资源加速
5. 负载均衡：Nginx 反代多个后端实例
```

#### 6.3.2 监控与告警

```
监控指标：
- API 响应时间
- 数据库连接数
- 错误率 (4xx/5xx)
- 在线用户数
- 金币流水异常

工具建议：
- Prometheus + Grafana
- Sentry（错误追踪）
- ELK（日志分析）
```

---

## 七、安全与合规

### 7.1 防作弊措施

```python
1. 时间戳验证
   - 服务端记录开局时间
   - 结算时按真实时长封顶分数/金币
   
2. 邀请防刷
   - IP 哈希去重（同一公网 IP）
   - 每人最多奖励 50次
   - 不能邀请自己
   
3. 广告复活验证
   - 服务端记录广告开始时间
   - 必须看满3秒才能复活
   - 每局只能复活一次

4. PIN 限流
   - 同昵称连错 5次锁10分钟
   - 同 IP 10分钟失败30次限流
   
局限：
- 无接口签名（可伪造请求）
- 客户端可修改内存（需配合反作弊系统）
```

### 7.2 数据安全

```python
1. PIN 存储：PBKDF2-SHA256（6万次迭代 + 随机盐）
2. 设备号：本地存储（登录凭据，不要外泄）
3. client_hint：软指纹（不参与认证）
4. HTTPS：生产环境强制（protect token/session）
5. SQL 注入防护：参数化查询（所有 db.execute）
```

### 7.3 隐私合规

```
数据收集：
- 设备号（自动生成）
- 昵称、PIN（用户主动设置）
- IP 地址（哈希后存储，用于邀请防刷）
- client_hint（浏览器指纹，不可逆）
- 游戏数据（分数、金币、对局记录）

用户权利：
- POST /api/account/delete：删除账号及全部数据
- 清除本地存储：注销当前设备登录
- 隐私政策：/mobile-web/privacy.html

注意：
- 充值/广告仅示意（不接真实支付）
- 国内上线需版号+备案（当前未申请）
```

---

## 八、测试方案

### 8.1 接口测试

```bash
# 冒烟测试
python3 tests/smoke_test.py http://127.0.0.1:3000

# 账号系统测试
python3 tests/account_xb_test.py http://127.0.0.1:3000
```

### 8.2 手工测试

参考 [docs/TEST_CASES.md](docs/TEST_CASES.md)：

```
测试维度：
1. 商业化：充值、购买、广告复活、邀请拉新
2. 操作性：跳跃缓冲、空中滑铲、换道、冲刺
3. 趣味性：打击感、音效、新纪录提示
4. 健壮性：网络异常、参数篡改、并发请求
```

### 8.3 性能测试

```bash
# 并发压测（Apache Bench）
ab -n 1000 -c 50 http://127.0.0.1:3000/api/leaderboard

# 数据库性能
- 索引优化（runs.ended_at、invites.ip_hash）
- 慢查询日志分析
```

---

## 九、技术亮点

### 9.1 零依赖后端

```python
优势：
- 快速启动，无需配置复杂环境
- 适合教学和原型验证
- 代码可读性高（纯标准库）

实现细节：
- http.server.ThreadingHTTPServer
- JSON 序列化（json 模块）
- SQLite（sqlite3 模块）
- MySQL 驱动作为可选依赖
```

### 9.2 双模式前端

```
横版（PC优化）
- 侧视角跑酷
- 键盘操作友好
- 高分竞技导向

竖版（移动端优化）
- 背后视角三车道
- 触屏手势控制
- 距离/金币累积导向
- 首页导航设计

技术复用：
- 共用后端 API
- 共用账号体系
- 共用商店系统
- 独立计分机制（避免互相覆盖）
```

### 9.3 设备号自动登录

```javascript
优势：
- 无需手动注册/登录
- 打开页面即可开始游戏
- 数据自动同步到服务器

技术方案：
- localStorage + Cookie 双备份
- 自动生成唯一设备号（crypto API）
- 后台静默登录（POST /api/guest）
- 可升级为账号（设置昵称+PIN）
```

### 9.4 渐进式账号系统

```
三种账号模式：
1. 游客试玩（XBuilder 版，不联网）
2. 设备号账号（网页版，自动登录）
3. 昵称+PIN 账号（可跨设备）

升级路径：
游客 → 设备号 → 昵称账号

设计理念：
- 降低使用门槛
- 保护用户隐私
- 兼容多端登录
```

---

## 十、已知限制与改进方向

### 10.1 当前限制

```
1. 性能瓶颈
   - 单进程架构（无法水平扩展）
   - 数据库连接未池化
   
2. 安全问题
   - 无接口签名（可伪造请求）
   - 防作弊依赖时间戳（可绕过）
   
3. 功能缺失
   - 无背景音乐
   - 充值/广告为示意（不接真实平台）
   - 横版页面在竖屏设备上体验差
   
4. 合规问题
   - 无版号（国内上线需申请）
   - 无备案
```

### 10.2 改进建议

```
短期优化：
1. 替换框架（FastAPI + 连接池）
2. 添加接口签名验证
3. 引入 Redis 缓存
4. 完善错误监控

中期规划：
1. 接入真实支付平台
2. 接入广告 SDK
3. 添加背景音乐
4. 优化横版响应式布局

长期目标：
1. 微服务化拆分
2. 实时对战模式
3. 社交功能（好友、公会）
4. 赛季排行榜
```

---

## 十一、附录

### 11.1 环境变量清单

| 变量 | 默认值 | 说明 |
|-----|--------|------|
| PORT | 3000 | HTTP 监听端口 |
| HOST | 0.0.0.0 | 监听地址 |
| PUBLIC_URL | https://bull.lingecho.com | 邀请链接域名 |
| DB_URL | - | MySQL 连接字符串 |
| DB_HOST | 127.0.0.1 | MySQL 主机 |
| DB_PORT | 3306 | MySQL 端口 |
| DB_USER | bull | MySQL 用户 |
| DB_PASS | - | MySQL 密码 |
| DB_NAME | running_bull | 数据库名 |
| SQLITE_PATH | data/running_bull.db | SQLite 文件路径 |
| TRUST_PROXY | 0 | 是否信任 X-Forwarded-For |
| QUIET | 0 | 是否禁用访问日志 |
| WECHAT_APP_ID | - | 微信公众号 AppID |
| WECHAT_APP_SECRET | - | 微信 AppSecret |
| WECHAT_REDIRECT_URI | - | 微信授权回调 |
| WECHAT_OAUTH_SCOPE | snsapi_userinfo | 微信授权范围 |

### 11.2 常用命令

```bash
# 启动服务
python3 server.py

# 查看日志
journalctl -u running-bull -f

# 重启服务
sudo systemctl restart running-bull

# 重载 Nginx
sudo nginx -t && sudo systemctl reload nginx

# 数据库备份
mysqldump -u bull -p running_bull > backup.sql

# 清空数据（开发环境）
rm data/running_bull.db
```

### 11.3 相关文档

- [README.md](README.md) - 项目说明
- [docs/TEST_CASES.md](docs/TEST_CASES.md) - 测试用例
- [tests/README_robustness.md](tests/README_robustness.md) - 健壮性测试
- [schema.mysql.sql](schema.mysql.sql) - 数据库结构
- [deploy/nginx.conf](deploy/nginx.conf) - Nginx 配置
- [deploy/running-bull.service](deploy/running-bull.service) - Systemd 配置

---

**文档版本**: v1.0  
**最后更新**: 2026-10-26  
**维护者**: 开发团队
