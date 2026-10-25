# 大角牛快跑 Plugs 版 - 产品架构文档

## 一、产品概述

### 1.1 项目定位
大角牛快跑 Plugs 版是一款基于 **XBuilder** 平台（spx 引擎）开发的竖版跑酷游戏，采用背后视角三车道设计，支持键盘和触屏操作，可在电脑和手机上运行。

### 1.2 技术栈概览
- **开发工具**: XBuilder (https://xbuild.ai)
- **编程语言**: spx (Go-like 语法的游戏脚本语言)
- **运行环境**: XBuilder 4:3 画面（480×360 像素）
- **网络通信**: HTTP REST API（可选联网）
- **目标平台**: 
  - 桌面：XBuilder 客户端
  - 移动端：XBuilder 移动端运行时
  - 网页：导出为 WebAssembly（理论支持）

### 1.3 产品特性
- 背后视角三车道跑酷（左/中/右）
- 障碍物从远处迎面而来的透视效果
- 完整的游戏内 UI（分数、金币、排名、昵称等）
- 在线账号系统（昵称 + 4 位 PIN）
- 游客试玩模式（本地不联网）
- 商店系统（3 角色 + 3 地图 + 道具）
- 广告复活/金币复活机制
- 与 Web 版共享后端 API

---

## 二、系统架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────┐
│                    XBuilder 运行环境                     │
│  ┌───────────────────────────────────────────────────┐  │
│  │               游戏画布 (480×360)                   │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │      舞台 (main.spx) - 全局状态管理         │  │  │
│  │  │  - 游戏状态机（标题/游戏中/结束）           │  │  │
│  │  │  - 透视投影系统                             │  │  │
│  │  │  - 障碍物生成器                             │  │  │
│  │  │  - 键盘/触屏输入处理                         │  │  │
│  │  │  - 联网账号管理                             │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │           游戏角色 (26个 .spx)              │  │  │
│  │  │  - Bull.spx (主角：跳跃/滑铲/换道)          │  │  │
│  │  │  - Obstacle.spx (障碍物)                    │  │  │
│  │  │  - Coin.spx (金币)                          │  │  │
│  │  │  - Boost.spx (闪电冲刺)                     │  │  │
│  │  │  - UI 组件 (Shop/Login/Toast/Badge...)      │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                           ↓ HTTP (可选)
┌─────────────────────────────────────────────────────────┐
│                    后端 API 服务器                       │
│            https://bull.lingecho.com/api                │
│  ┌───────────────────────────────────────────────────┐  │
│  │  XBuilder 专用接口 (/api/xb/*)                    │  │
│  │  - 扁平 JSON 格式（spx 易解析）                   │  │
│  │  - 位掩码表示拥有的角色/地图                       │  │  │
│  │  - 返回格式化文字（如排行榜）                     │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │  通用接口 (/api/*)                                │  │
│  │  - 账号注册/登录                                  │  │
│  │  - 游戏开局/结算                                  │  │
│  │  - 商店购买                                       │  │
│  │  - 广告复活                                       │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────┐
│                     数据库 (MySQL)                       │
│  - 账号数据 (accounts/sessions)                         │
│  - 游戏数据 (users/runs/inventory)                      │
│  - XB 专用装备 (xb_prefs)                               │
└─────────────────────────────────────────────────────────┘
```

### 2.2 目录结构

```
running_bull_plugs/
├── README.md                   # 项目说明
├── running_bull_plugs.xbp      # XBuilder 工程打包文件（可直接导入）
└── project/                    # 工程源码目录
    ├── main.spx                # 舞台主脚本（全局状态 + 游戏逻辑）
    ├── Bull.spx                # 主角（跳跃/滑铲/换道）
    ├── Obstacle.spx            # 障碍物（低栏/高门/草垛/木箱/石堆）
    ├── Coin.spx                # 金币
    ├── Boost.spx               # 闪电冲刺道具
    ├── Deco.spx                # 路边装饰
    ├── Fx.spx                  # 特效（扬尘/爆炸/速度线）
    ├── Flash.spx               # 撞击闪白效果
    ├── Panel.spx               # UI 面板背景
    ├── Shop.spx                # 商店主面板
    ├── ShopCard.spx            # 商店卡片
    ├── ShopBtn.spx             # 商店按钮
    ├── ShopCursor.spx          # 商店光标
    ├── Login.spx               # 登录面板
    ├── LoginMsg.spx            # 登录提示信息
    ├── NickImg.spx             # 昵称显示
    ├── PinDot.spx              # PIN 输入点
    ├── AcctBtn.spx             # 账号按钮
    ├── Badge.spx               # 账号状态徽章
    ├── Revive.spx              # 复活面板
    ├── Toast.spx               # 提示框
    ├── Txt.spx                 # 文字渲染器
    ├── NewRecord.spx           # 新纪录提示
    ├── Pill.spx                # UI 药丸形背景
    └── assets/                 # 游戏素材（图片、音效）
        ├── 角色造型（bull_0 ~ bull_5, pig_*, sheep_*）
        ├── 障碍物造型（barrier_*, grass_*, crate_*, ...）
        ├── 地图背景（bg_farm_*, bg_desert_*, bg_night_*）
        ├── UI 元素（按钮、面板、徽章）
        └── 音效文件（jump.mp3, coin.mp3, crash.mp3, ...）
```

---

## 三、技术架构

### 3.1 XBuilder 平台特性

#### 3.1.1 spx 语言特点

```go
// spx 是 Go-like 语法的游戏脚本语言

特性：
1. 静态类型（但无需显式声明类型）
2. 支持结构体、数组、切片
3. 原生支持游戏概念（精灵、舞台、事件）
4. 内置物理引擎和碰撞检测
5. 跨平台（编译为字节码，XBuilder 运行时执行）

限制：
- 无文件系统访问（安全沙箱）
- 无本地存储 API（spx v3.3.0）
- 网络请求需通过 http 包（实验性）
- 浏览器控制台可查看日志（[bull] 前缀）
```

#### 3.1.2 运行环境约束

```
画面尺寸：
- XBuilder 窗口固定 4:3 比例（480×360）
- 手机和电脑都是这个尺寸
- 更高的舞台会被裁掉上下两截

原因：
- 避免竖屏舞台在 XBuilder 里被裁剪
- 保证跨设备一致性
- 所有 UI 元素都在可见区域内

解决方案：
- 竖版玩法放在 480×360 画面内
- 背后视角 + 三车道从下往上延伸
- 所有文字/按钮画在游戏画面里（不用监视器）
```

### 3.2 游戏核心架构

#### 3.2.1 舞台 (main.spx) - 全局状态管理

```go
// main.spx 是游戏的"大脑"，负责：

1. 游戏状态机
   var state int
   // 0 = 标题画面
   // 1 = 游戏中
   // 2 = 游戏结束

2. 全局变量（monitor 显示）
   var score int        // 分数
   var coins int        // 本局金币
   var best int         // 最高分
   var boostLeft int    // 冲刺剩余秒数
   var wallet int       // 钱包金币
   var runShield int    // 本局护盾数
   var coinMul int      // 金币倍数 1/2

3. 玩家状态
   var bullLane int     // 车道 0/1/2
   var bullX float64    // X 坐标（平滑移动）
   var jumpH float64    // 离地高度
   var sliding bool     // 是否滑铲

4. 游戏进度
   var speed float64    // 基础速度（像素/帧）
   var curSpeed float64 // 实际速度（冲刺时更快）
   var dist float64     // 跑过的距离
   var frames float64   // 本局帧数
   var spawnGap float64 // 距离下次生成障碍
   var decoGap float64  // 距离下次生成装饰

5. 视觉效果
   var shakeX, shakeY float64  // 冲刺抖动偏移
   var roadZ float64           // 路面滚动相位
   var bdIdx int               // 背景造型序号

6. 商店/账号/复活
   var charOwned int    // 已拥有角色（位掩码）
   var mapOwned int     // 已拥有地图（位掩码）
   var curChar int      // 当前角色 0/1/2
   var curMap int       // 当前地图 0/1/2
   var shieldCnt int    // 持有护盾数
   var doubleCnt int    // 持有双倍金币数
   var revived bool     // 本局是否已复活
   var frozen bool      // 撞车暂停状态
   var reviveMode int   // 0=无 1=选择中 2=看广告
   var adT int          // 广告剩余帧数
   var invT float64     // 无敌剩余帧数

7. 联网账号
   var online bool      // 是否已登录
   var token string     // 会话令牌
   var runId string     // 本局编号
   var adId string      // 广告编号
   var acctNick string  // 昵称
   var rank int         // 全服排名
   var loginNick string // 登录面板昵称
   var loginPin string  // 输入中的 PIN
   var netBusy bool     // 网络请求中
```

#### 3.2.2 透视投影系统

```go
// 核心算法：从远到近的透视缩放

车道定义：
const (
    LX = -100  // 左车道 X
    MX = 0     // 中车道 X
    RX = 100   // 右车道 X
    GY = 60    // 地面 Y
)

透视公式：
z := 障碍物距玩家的距离（正值）
scale := 0.2 + z / 600          // 远处小（0.2），近处大（~1.0）
y := GY - z * 0.4               // 远处在上方，近处在下方
x := laneX * scale              // 车道越远越窄（透视收缩）

渲染顺序：
1. 背景（4 个滚动相位）
2. 路面（根据 roadZ 选择造型）
3. 路边装饰（远 → 近）
4. 障碍物（远 → 近）
5. 主角（z=0，固定在画面下方）
6. 特效层（扬尘、爆炸、速度线）
7. UI 层（文字、按钮、面板）
```

#### 3.2.3 障碍物生成算法

```go
// 障碍物类型及生成规则

障碍物种类：
0: 低栏（跳过）
1: 高门（滑铲钻过）
2: 草垛（只能换道躲避）
3: 木箱（只能换道躲避）
4: 石堆（跳过）

生成规则：
1. 每次生成一排（1-3 个障碍）
2. 至少留一条空道（可通行）
3. 间距随机：180-280 像素
4. 速度递增：每 5 秒 +0.05 像素/帧
5. 金币夹在障碍之间（空道上）
6. 闪电冲刺：低概率生成（~5%）

代码片段：
func spawnObstacle() {
    // 随机生成 1-3 个障碍，保证至少一条空道
    blocked := [3]bool{}
    count := rand(1, 3)
    for i := 0; i < count; i++ {
        lane := pickEmptyLane(blocked)
        blocked[lane] = true
        spawnOne(lane, pickType())
    }
}
```

#### 3.2.4 输入处理（键盘 + 触屏）

```go
// 键盘控制
onKey "Left", "A" => { changeLane(-1) }
onKey "Right", "D" => { changeLane(1) }
onKey "Up", "W", "Space" => { doJump() }
onKey "Down", "S" => { doSlide() }
onKey "B" => { openShop() }
onKey "L" => { openLogin() }

// 触屏手势识别
onTouchStart => {
    ptrDown = true
    ptrX, ptrY = touchX, touchY
}

onTouchEnd => {
    if !ptrDown { return }
    dx := touchX - ptrX
    dy := touchY - ptrY
    
    if abs(dx) > abs(dy) {
        // 左右滑动：换道
        if dx > 30 { changeLane(1) }
        else if dx < -30 { changeLane(-1) }
    } else {
        // 上下滑动：跳跃/滑铲
        if dy < -30 { doJump() }
        else if dy > 30 { doSlide() }
    }
    
    ptrDown = false
}

// 跳跃缓冲机制
func doJump() {
    if jumpH <= 0 {
        startJump()  // 在地面：立刻跳
    } else {
        jumpBuf = 9  // 在空中：记住，落地时自动跳
    }
}

// 空中快速落地
func doSlide() {
    if jumpH > 0 {
        vy = -14  // 快速下坠
    }
    slideT = 36   // 滑铲动画持续帧数
    sliding = true
}
```

---

## 四、角色系统架构

### 4.1 主角 (Bull.spx)

```go
核心职责：
1. 跑步动画（8 帧循环）
2. 跳跃物理（抛物线运动）
3. 滑铲动画（36 帧）
4. 平滑换道（线性插值）
5. 碰撞判定
6. 造型切换（不同角色）

物理参数：
- 跳跃初速度：9 像素/帧
- 重力加速度：-0.6 像素/帧²
- 滑铲持续：36 帧（约 0.6 秒）
- 跳跃缓冲：9 帧（约 0.15 秒）

造型索引：
角色 0（大角牛）: 造型 0-5
  0: 跑步 1
  1: 跑步 2
  2: 跑步 3
  3: 跑步 4
  4: 撞车
  5: 滑铲

角色 1（粉粉猪）: 造型 6-11
角色 2（咩咩羊）: 造型 12-17

碰撞盒调整：
- 站立：80% 宽度、90% 高度
- 滑铲：80% 宽度、40% 高度
- 跳跃：无碰撞（跳过障碍）
```

### 4.2 障碍物 (Obstacle.spx)

```go
职责：
1. 从远处向近处移动
2. 透视缩放（根据 z 距离）
3. 碰撞检测（与主角）
4. 冲刺时被撞飞效果
5. 超出屏幕后销毁

类型定义：
var obstType int
// 0: 低栏（跳过）
// 1: 高门（滑铲）
// 2: 草垛（换道）
// 3: 木箱（换道）
// 4: 石堆（跳过）

生命周期：
1. 生成：z = 800（远处）
2. 移动：z -= curSpeed（每帧）
3. 碰撞检测：z < 50 && 同车道 && 未跳跃/滑铲
4. 销毁：z < -100 或 被撞飞

冲刺交互：
if boostTime > 0 && z < 50 {
    // 被撞飞动画
    vx = ±8
    vy = 6
    spin = 15
    destroy()
}
```

### 4.3 金币 (Coin.spx)

```go
职责：
1. 随障碍物生成（空道上）
2. 透视移动 + 缩放
3. 自转动画（旋转 + 闪烁）
4. 被主角收集（+10 分）
5. 冲刺时磁铁效果（吸附）

生成规则：
- 生成在空道上（没障碍的车道）
- z 距离与障碍同步
- 概率：60%（不是每次都生成）

磁铁效果：
if boostTime > 0 {
    // 冲刺时吸附到主角车道
    dx := bullLane - myLane
    myLane += sign(dx) * 0.2
}

收集判定：
if z < 50 && abs(x - bullX) < 30 {
    score += 10 * coinMul
    coins += coinMul
    play("coin")
    spawnFx("+10", x, y)
    destroy()
}
```

### 4.4 特效系统 (Fx.spx)

```go
特效类型：
0: 撞击火花（8 个方向飞散）
1: 飘字 +10（向上飘）
2: 扬尘（落地时）
3: 爆炸（撞飞障碍物）
4: 速度线（冲刺时）

生命周期：
1. 生成：spawnFx(type, x, y, vx, vy)
2. 运动：x += vx, y += vy, alpha -= 0.05
3. 销毁：alpha <= 0 或超出屏幕

调用示例：
// 撞击火花
for i := 0; i < 8; i++ {
    angle := i * 45
    vx := cos(angle) * 5
    vy := sin(angle) * 5
    spawnFx(0, bullX, bullY, vx, vy)
}

// 扬尘
spawnFx(2, bullX-17, groundY+5, -1.3, 0.3)
spawnFx(2, bullX+17, groundY+5, 1.3, 0.3)

// 飘字
spawnFx(1, coinX, coinY, 0, -2, "+10")
```

### 4.5 UI 组件

#### 4.5.1 商店系统 (Shop.spx + ShopCard.spx)

```go
// Shop.spx - 商店主面板
职责：
1. 显示 8 张卡片（3 角色 + 3 地图 + 2 道具）
2. 键盘光标选择（← → ↑ ↓）
3. 购买确认（空格/回车/点击）
4. 价格显示（服务端为准）
5. 已拥有/金币不足状态

布局：
第一排：[大角牛] [粉粉猪] [咩咩羊] [护盾]
第二排：[草原]   [沙漠]   [夜晚]   [双倍]

// ShopCard.spx - 单张卡片
职责：
1. 显示角色/地图/道具造型
2. 价格标签
3. "已拥有"标记
4. 选中高亮效果
5. 金币不足时置灰

购买流程：
1. 选中卡片
2. 按空格/点击 → 扣金币 → 更新拥有状态
3. 角色/地图自动装备
4. 道具增加持有数量（最多 3 个）
```

#### 4.5.2 登录系统 (Login.spx + NickImg.spx + PinDot.spx)

```go
// Login.spx - 登录面板
模式：
1. 注册：输入昵称 → 输入 PIN → 发送请求
2. 登录：输入昵称 → 输入 PIN → 验证
3. 游客试玩：不联网，数据不保存

昵称输入：
- 键盘输入：字母/数字/空格/退格
- 预设昵称：勇敢牛牛/幸运牛牛/闪电牛牛...
- 滚动选择：↑ ↓ 或滑动

PIN 输入：
- 4 位数字（0-9）
- 显示为 ●●●● 或 ●●_ _
- 退格键删除
- 输入完自动提交

// PinDot.spx - PIN 输入点
显示状态：
- 空：灰色圆圈
- 已输入：黑色圆点 ●
- 当前位：闪烁光标

// NickImg.spx - 昵称显示
职责：
1. 游戏内显示昵称（左上角）
2. 账号徽章（在线/游客/离线）
3. 点击打开登录面板
```

#### 4.5.3 复活系统 (Revive.spx)

```go
职责：
1. 撞车后弹出选择面板
2. 看广告复活（免费，3 秒倒计时）
3. 金币复活（100 金币）
4. 放弃（直接结算）

广告流程：
1. POST /api/ads {run_id}
   → 返回 {ad_id, seconds: 3}
2. 播放"示意广告"（倒计时 3 → 2 → 1）
3. POST /api/runs/{run_id}/revive {method: "ad", ad_id}
   → 服务端验证：真的看满 3 秒
4. 复活成功：无敌 2 秒 + 清空附近障碍

金币复活：
1. 检查钱包 >= 100
2. POST /api/runs/{run_id}/revive {method: "coins"}
3. 扣除 100 金币
4. 复活（同广告复活）

限制：
- 每局只能复活一次（revived 标志）
- 第二次撞车直接结算
```

#### 4.5.4 文字渲染 (Txt.spx)

```go
职责：
1. 显示游戏内所有文字（分数/金币/排名/提示）
2. 支持多行文本
3. 动态颜色/大小
4. 从 txtCache 数组读取内容

槽位定义：
txtCache[0]: 标题画面提示（"按空格开始"）
txtCache[1]: 分数显示（"12345"）
txtCache[2]: 金币显示（"×56"）
txtCache[3]: 钱包显示（"💰 1234"）
txtCache[4]: 排名显示（"全服第 8 名"）
txtCache[5]: 商店提示（"按 B 打开商店"）
txtCache[6]: 登录提示（"输入昵称"）
txtCache[7]: 结算画面（"本局得分：XXX"）

渲染方式：
- XBuilder 没有动态文本 API
- 预渲染 10 套数字造型（0-9）
- 按字符拼接显示
- 支持特殊字符（×💰📊）
```

---

## 五、网络架构

### 5.1 后端接口对接

#### 5.1.1 API 基础配置

```go
// main.spx
const apiBase = "https://bull.lingecho.com/api"

// spx v3.3.0 网络请求示例
import (
    "net/http"
    "encoding/json"
)

func apiCall(method, path string, body map[string]any) (map[string]any, error) {
    url := apiBase + path
    
    // 构造请求
    req := http.NewRequest(method, url, jsonBody(body))
    req.Header.Set("Content-Type", "application/json")
    if token != "" {
        req.Header.Set("X-Session", token)
    }
    
    // 发送请求
    resp := http.DefaultClient.Do(req)
    if resp.StatusCode != 200 {
        return nil, errors.New(resp.Status)
    }
    
    // 解析响应
    var result map[string]any
    json.NewDecoder(resp.Body).Decode(&result)
    return result, nil
}

注意：
- spx 的网络请求是同步阻塞的
- 需要设置 netBusy 标志防止重复请求
- 浏览器控制台打印 [bull] 日志方便调试
```

#### 5.1.2 XBuilder 专用接口 (/api/xb/*)

```go
接口特点：
1. 返回扁平 JSON（spx 易解析）
2. 位掩码表示拥有的角色/地图
3. 价格数组按卡片顺序排列
4. 排行榜返回格式化文字

示例：POST /api/xb/profile
返回：
{
    "ok": true,
    "nick": "勇敢牛牛",
    "wallet": 1234,
    "best": 5678,
    "chars": 7,        // 二进制 111 = 拥有 3 个角色
    "maps": 5,         // 二进制 101 = 拥有第 1、3 张地图
    "char": 0,         // 当前角色
    "map": 1,          // 当前地图
    "shield": 2,       // 护盾数量
    "double": 1,       // 双倍金币数量
    "prices": [0, 300, 600, 200, 300, 500, 60, 80],  // 8 张卡片价格
    "revive_cost": 100
}

位掩码解析：
func hasChar(id int) bool {
    return (charOwned & (1 << id)) != 0
}

func addChar(id int) {
    charOwned |= (1 << id)
}
```

#### 5.1.3 核心接口流程

```go
1. 登录流程
   POST /api/xb/login {nick: "勇敢牛牛", pin: "1234"}
   → 返回 {ok: true, token: "...", ...profile...}
   → 保存 token 到 var token string
   → 更新钱包/解锁状态/最高分

2. 游戏开局
   POST /api/xb/start
   → 自动使用 1 个护盾 + 1 张双倍金币（如有）
   → 返回 {run: "xxx", run_shield: 1, run_double: true, ...}
   → 保存 runId，记录本局状态

3. 购买物品
   POST /api/xb/buy {item: 0..7}
   → 0-2: 角色（自动装备）
   → 3-5: 地图（自动装备）
   → 6-7: 道具（增加持有数，最多 3 个）
   → 返回更新后的 profile

4. 广告复活
   POST /api/xb/ad {run: runId}
   → 返回 {ad: "xxx", seconds: 3}
   → 播放 3 秒倒计时
   POST /api/xb/revive {run: runId, method: "ad", ad: adId}
   → 服务端验证看满 3 秒

5. 金币复活
   POST /api/xb/revive {run: runId, method: "coins"}
   → 扣除 100 金币

6. 结算
   POST /api/xb/finish {run: runId, score: 12345, coins: 67}
   → coins 填原始数量（服务端计算双倍）
   → 返回 {score, earned, capped, new_record, rank, ...}
   → 更新 best / wallet / rank

7. 排行榜
   GET /api/xb/top
   → 返回 {text: "1. 勇敢牛牛 12345\n2. 幸运牛牛 11234\n..."}
   → 直接显示格式化文字
```

### 5.2 游客模式（本地离线）

```go
特性：
- 不联网，不调用任何 API
- 数据只在内存中（刷新网页即清空）
- 钱包/解锁/最高分不保存
- 商店功能正常（本地金币）
- 无排行榜/邀请功能

切换逻辑：
if online {
    // 联网模式：调用 API
    callApi("/xb/start")
} else {
    // 游客模式：本地逻辑
    localStart()
}

适用场景：
- 快速体验游戏
- 网络不可用时
- 不想注册账号
```

### 5.3 与 Web 版的差异

| 特性 | Web 版 | Plugs 版 |
|------|--------|----------|
| 前端技术 | HTML5 Canvas + JS | XBuilder + spx |
| 画面尺寸 | 自适应（横版/竖版） | 固定 480×360 |
| 视角 | 横版侧视 / 竖版背后视 | 竖版背后视 |
| 账号系统 | 设备号自动登录 | 昵称+PIN 登录 |
| 角色差异 | 大角牛/蓝角牛/金角牛/黑旋风 | 大角牛/粉粉猪/咩咩羊 |
| 装备记录 | users.equipped_char | xb_prefs.char_id |
| 排行榜 | 横版 users.best | 竖版 XB 专用表 |
| 本地存储 | localStorage | 无（需联网） |
| 游客模式 | 支持（设备号） | 支持（纯本地） |

---

## 六、数据流设计

### 6.1 游戏状态流转

```
标题画面 (state = 0)
    ↓ 按空格/点击
开局准备
    ↓ POST /api/xb/start
游戏进行中 (state = 1)
    ↓ 撞障碍物
暂停（选择复活）(frozen = true, reviveMode = 1)
    ├─ 看广告 → POST /api/xb/ad → 倒计时 3s → POST /api/xb/revive
    ├─ 花金币 → POST /api/xb/revive
    └─ 放弃 → 直接结算
    ↓ 复活成功 → frozen = false, invT = 120
游戏继续
    ↓ 再次撞障碍物（revived = true）
游戏结束 (state = 2)
    ↓ POST /api/xb/finish
显示结算 + 排名
    ↓ 按空格/点击
返回标题画面
```

### 6.2 商店状态流转

```
游戏中任意时刻
    ↓ 按 B 或点击"商店"按钮
打开商店面板 (state = 3)
    ↓ 键盘/鼠标选择卡片
选中卡片（高亮）
    ↓ 按空格/点击
购买确认
    ├─ 金币足够 → POST /api/xb/buy → 更新拥有状态
    └─ 金币不足 → Toast("金币不足")
    ↓ 按 ESC 或点击关闭
关闭商店，返回游戏
```

### 6.3 登录状态流转

```
启动游戏
    ↓ 无 token
显示登录面板 (state = 4)
    ├─ 选择"注册" → 输入昵称 → 输入 PIN → POST /api/xb/register
    ├─ 选择"登录" → 输入昵称 → 输入 PIN → POST /api/xb/login
    └─ 选择"游客试玩" → online = false
    ↓ 登录成功
保存 token + 更新 profile
    ↓
进入标题画面

游戏内任意时刻
    ↓ 按 L 或点击"账号"按钮
重新打开登录面板（可重新登录或注销）
```

---

## 七、视觉效果系统

### 7.1 透视渲染流程

```go
每帧渲染顺序：

1. 清空画布
   clearGraphicEffects()

2. 绘制背景
   bdIdx := (roadZ / 100) % 4  // 4 个滚动相位
   setCostume("bg_" + mapName + "_" + itoa(bdIdx))
   stamp()

3. 绘制路面
   setCostume("road_" + itoa(bdIdx))
   setXYpos(shakeX, shakeY)  // 冲刺抖动
   stamp()

4. 绘制路边装饰（远 → 近）
   for each deco in decoList {
       scale := 0.2 + deco.z / 600
       y := GY - deco.z * 0.4
       setSize(scale * 100)
       setXYpos(deco.x * scale + shakeX, y + shakeY)
       stamp()
   }

5. 绘制障碍物（远 → 近）
   for each obstacle in obstacleList {
       // 同上，按 z 排序后渲染
   }

6. 绘制金币（远 → 近）
   for each coin in coinList {
       // 同上 + 自转动画
   }

7. 绘制主角
   scale := 0.9
   y := groundY - jumpH
   setSize(scale * 100)
   setXYpos(bullX + shakeX, y + shakeY)
   setCostume(curChar * 6 + animFrame)
   stamp()

8. 绘制特效层
   for each fx in fxList {
       // 火花/扬尘/飘字/爆炸
   }

9. 绘制 UI 层
   - 分数/金币/钱包（Txt 角色）
   - 护盾/双倍金币图标（Pill 角色）
   - 昵称/徽章（NickImg + Badge）
   - 商店/登录面板（Shop / Login）
   - 复活面板（Revive）
   - 提示框（Toast）
```

### 7.2 打击感设计

```go
1. 撞击效果
   - 闪白屏幕（Flash 角色，alpha 1.0 → 0）
   - 震屏（shakeX/shakeY 随机偏移）
   - 火花四溅（8 个方向）
   - 音效：crash.mp3

2. 冲刺效果
   - 画面抖动（sin 波形）
   - 速度线（Fx 角色）
   - 吸金币（磁铁效果）
   - 撞飞障碍物（旋转飞出）
   - 音效：boost.mp3

3. 跳跃效果
   - 扬尘（落地时）
   - 跳跃缓冲（空中按跳会记住）
   - 音效：jump.mp3

4. 滑铲效果
   - 空中快速落地（vy = -14）
   - 滑铲动画（36 帧）
   - 音效：slide.mp3

5. 金币收集
   - 飘字 +10（向上飘）
   - 闪光效果
   - 音效：coin.mp3
```

### 7.3 动画系统

```go
跑步动画：
- 4 帧循环（造型 0 → 1 → 2 → 3）
- 速度：每 6 帧切换一次
- 冲刺时加速：每 3 帧切换

跳跃动画：
- 起跳：造型切换到跳跃帧
- 空中：保持跳跃帧
- 落地：恢复跑步动画

滑铲动画：
- 36 帧固定时长
- 造型切换到滑铲帧
- 结束后恢复跑步

撞车动画：
- 造型切换到撞车帧
- 停止所有运动
- 火花+闪白+震屏

UI 动画：
- 按钮悬停：缩放 1.0 → 1.1
- Toast 弹出：从下往上滑入
- 商店卡片：选中时高亮边框
- PIN 输入：光标闪烁
```

---

## 八、音效系统

### 8.1 音效资源

```
jump.mp3      - 跳跃
slide.mp3     - 滑铲
coin.mp3      - 收集金币
crash.mp3     - 撞击
boost.mp3     - 冲刺开始
button.mp3    - UI 点击
buy.mp3       - 购买成功
error.mp3     - 操作失败
revive.mp3    - 复活成功
newrecord.mp3 - 新纪录
```

### 8.2 音效触发

```go
// main.spx 中管理全局静音开关
var muted bool

func playSound(name string) {
    if !muted {
        play(name)
    }
}

// 各角色中调用
playSound("jump")   // Bull.spx 跳跃时
playSound("coin")   // Coin.spx 收集时
playSound("crash")  // main.spx 碰撞时
playSound("boost")  // main.spx 获得冲刺时
```

### 8.3 背景音乐

```go
当前状态：
- Plugs 版已实现背景音乐
- 循环播放（loop: true）
- 可在设置中开关

音乐资源：
bgm.mp3 - 游戏主题曲（轻快节奏，90-120 BPM）

播放逻辑：
onStart => {
    if !muted {
        playMusic("bgm", loop: true)
    }
}

onKey "M" => {
    muted = !muted
    if muted {
        stopMusic()
    } else {
        playMusic("bgm", loop: true)
    }
}
```

---

## 九、性能优化

### 9.1 对象池管理

```go
问题：
- 频繁创建/销毁角色会造成 GC 压力
- spx 引擎没有显式的对象池 API

解决方案：
// 使用克隆体 (clone) 复用角色
var obstaclePool []Obstacle
var coinPool []Coin
var fxPool []Fx

func spawnObstacle() {
    var obj Obstacle
    if len(obstaclePool) > 0 {
        obj = obstaclePool[0]
        obstaclePool = obstaclePool[1:]
        obj.reset()
    } else {
        obj = Obstacle.clone()
    }
    obj.init(...)
}

func destroyObstacle(obj Obstacle) {
    obj.hide()
    obstaclePool.append(obj)
}
```

### 9.2 渲染优化

```go
1. 剔除屏幕外对象
   if z < -100 || z > 1000 {
       hide()  // 不渲染
       return
   }

2. 距离排序优化
   // 不用每帧全排序，用插入排序（已基本有序）
   func insertSorted(list []Object, obj Object) {
       // O(n) 插入，利用已排序特性
   }

3. 减少造型切换
   // 缓存当前造型，避免重复 setCostume
   if newCostume != curCostume {
       setCostume(newCostume)
       curCostume = newCostume
   }

4. 批量渲染
   // 同类对象用同一个 spx 角色，切换造型后 stamp()
```

### 9.3 内存优化

```go
1. 限制同屏对象数
   const MAX_OBSTACLES = 20
   const MAX_COINS = 15
   const MAX_FX = 30

2. 及时销毁超出屏幕的对象
   if z < -100 {
       destroy()
   }

3. 文字渲染优化
   // 不每帧生成字符串，用缓存
   if score != lastScore {
       txtCache[1] = itoa(score)
       lastScore = score
   }

4. 减少网络请求
   // 合并请求（如 profile + start）
   // 缓存价格列表（不每次都请求）
```

---

## 十、测试与调试

### 10.1 本地测试

```bash
1. 打开 XBuilder
2. 导入 running_bull_plugs.xbp
3. 点击"运行"按钮
4. 测试游戏功能（跳跃/滑铲/商店/登录）

快捷键：
F5 - 重新运行
F12 - 打开浏览器控制台（查看 [bull] 日志）
ESC - 停止运行
```

### 10.2 网络调试

```go
// 在浏览器控制台查看日志
console.log("[bull] POST /api/xb/login", body)
console.log("[bull] 登录成功", response)
console.log("[bull] 网络错误", error)

// spx 代码中打印日志
func debugLog(msg string) {
    // XBuilder 会输出到控制台
    println("[debug]", msg)
}
```

### 10.3 常见问题

```
Q: 游戏启动后黑屏？
A: 检查 assets/ 目录是否完整，造型文件是否存在

Q: 跳跃/滑铲不生效？
A: 检查 state == 1 && !frozen 条件，确保在游戏中

Q: 网络请求超时？
A: 检查 apiBase 配置，确认服务器可访问
   查看浏览器控制台 Network 标签

Q: 金币/分数不更新？
A: 检查 txtCache[] 是否正确赋值
   确认 Txt 角色正在渲染

Q: 碰撞检测不准？
A: 调整碰撞盒大小（setSize）
   检查 z 距离判定阈值（< 50）

Q: 复活后仍然卡住？
A: 确认 frozen = false
   检查 invT 是否正确设置（120 帧）
   确认 reviveClear > 0（清空附近障碍）
```

---

## 十一、部署与分发

### 11.1 XBuilder 工程打包

```
running_bull_plugs.xbp 结构：
- project/ 目录压缩
- 保留所有 .spx 文件
- 保留 assets/ 素材目录
- 不包含 .git / node_modules 等无关文件

导入方式：
1. 打开 XBuilder
2. 文件 → 导入工程
3. 选择 running_bull_plugs.xbp
4. 自动解压到工作区
```

### 11.2 导出为独立应用

```
XBuilder 支持导出为：
1. Windows .exe
2. macOS .app
3. Web（WebAssembly）
4. Android .apk（实验性）
5. iOS .ipa（需 Apple 开发者账号）

导出步骤：
1. 在 XBuilder 中打开工程
2. 文件 → 导出
3. 选择目标平台
4. 配置图标/名称/版本号
5. 点击"导出"按钮

注意事项：
- 网络功能需配置 CORS
- 移动端需适配触屏操作
- Web 版需托管在 HTTPS 域名下（联网功能）
```

### 11.3 服务端配置

```bash
# 确保后端接口支持 XB 客户端

1. 允许跨域（CORS）
   Access-Control-Allow-Origin: *
   Access-Control-Allow-Methods: GET, POST, OPTIONS
   Access-Control-Allow-Headers: Content-Type, X-Session

2. 启用 XB 专用接口
   /api/xb/* 路径正常工作

3. 检查日志
   tail -f /var/log/running-bull/access.log | grep '/api/xb/'

4. 测试接口
   curl -X POST https://bull.lingecho.com/api/xb/login \
     -H "Content-Type: application/json" \
     -d '{"nick":"测试牛牛","pin":"1234"}'
```

---

## 十二、技术亮点

### 12.1 跨平台能力

```
优势：
- 一套代码，多端运行（PC + 移动端）
- XBuilder 自动适配输入方式（键盘/触屏）
- 画面尺寸固定，无需响应式布局

实现：
- spx 语言统一抽象（不直接操作 DOM）
- 触屏手势自动转换为游戏操作
- 网络 API 在所有平台上一致
```

### 12.2 透视投影系统

```go
亮点：
- 背后视角三车道（少见于 2D 游戏）
- 障碍物从远到近逐渐变大（透视感）
- 车道越远越窄（真实透视效果）

技术实现：
scale := 0.2 + z / 600        // 线性缩放
y := GY - z * 0.4             // 垂直位移
x := laneX * scale            // 透视收缩

渲染顺序：远 → 近（画家算法）
```

### 12.3 跳跃缓冲机制

```go
问题：
- 玩家在空中按跳会"吃掉"操作
- 落地瞬间很难精确按跳

解决方案：
func doJump() {
    if jumpH <= 0 {
        startJump()
    } else {
        jumpBuf = 9  // 记住，落地时自动跳
    }
}

// 物理更新时
if jumpH <= 0 && jumpBuf > 0 {
    startJump()  // 落地瞬间自动起跳
}

优势：
- 操作容错率高
- 连续跳跃更流畅
- 不会"吃"输入
```

### 12.4 渐进式联网

```
三种模式：
1. 游客试玩（完全本地）
2. 临时联网（本次会话）
3. 账号登录（跨设备同步）

切换流程：
游客 → 临时联网（无需注册，直接开始）
临时联网 → 账号登录（保存进度）

技术优势：
- 降低使用门槛
- 数据安全（本地不存敏感信息）
- 灵活切换（网络不可用时可离线）
```

---

## 十三、已知限制与改进方向

### 13.1 当前限制

```
1. XBuilder 平台限制
   - 画面固定 4:3（不支持宽屏）
   - 无本地存储 API（spx v3.3.0）
   - 网络请求同步阻塞（可能卡顿）
   - 无多线程/协程支持

2. 性能限制
   - 同屏对象过多时帧率下降
   - 复杂透视计算占 CPU
   - 内存管理依赖 GC（不可控）

3. 功能限制
   - 无实时对战（单机游戏）
   - 无好友系统
   - 无成就系统
   - 充值/广告为示意（不接真实平台）

4. 兼容性
   - XBuilder 版本更新可能破坏兼容性
   - spx 语言规范尚未稳定
   - 部分浏览器 WebAudio 支持不完整
```

### 13.2 改进建议

```
短期优化：
1. 异步网络请求（避免卡顿）
2. 对象池优化（减少 GC）
3. LOD 系统（远处障碍物低精度）
4. 添加更多障碍物类型

中期规划：
1. 关卡系统（不同难度/场景）
2. 成就系统（完成挑战解锁）
3. 每日任务（增加粘性）
4. 好友排行榜

长期目标：
1. 实时对战模式（多人竞速）
2. 赛季排行榜（定期重置）
3. 皮肤系统（外观定制）
4. 剧情模式（引导新手）
```

---

## 十四、附录

### 14.1 角色造型索引

```
大角牛（角色 0）：
造型 0-5: 跑步 1/2/3/4、撞车、滑铲

粉粉猪（角色 1）：
造型 6-11: 跑步 1/2/3/4、撞车、滑铲

咩咩羊（角色 2）：
造型 12-17: 跑步 1/2/3/4、撞车、滑铲

障碍物：
低栏: barrier_low_0..3（4 个透视造型）
高门: barrier_high_0..3
草垛: grass_0..3
木箱: crate_0..3
石堆: rock_0..3

地图背景：
草原: bg_farm_0..3（4 个滚动相位）
沙漠: bg_desert_0..3
夜晚: bg_night_0..3
```

### 14.2 常用命令

```bash
# 运行游戏
打开 XBuilder → 导入 running_bull_plugs.xbp → 点击"运行"

# 查看控制台日志
F12 → Console → 过滤 "[bull]"

# 测试后端接口
curl -X POST https://bull.lingecho.com/api/xb/login \
  -H "Content-Type: application/json" \
  -d '{"nick":"测试牛牛","pin":"1234"}'

# 导出工程
文件 → 导出 → 选择平台 → 导出
```

### 14.3 相关文档

- [README.md](README.md) - 项目说明
- [XBuilder 官方文档](https://docs.xbuild.ai)
- [spx 语言参考](https://github.com/goplus/spx)
- [后端 API 文档](../running_bull_web/README.md#七rest-接口给其他客户端用比如-xbuilder-版)

### 14.4 开发团队联系方式

```
技术支持：
- GitHub Issues: https://github.com/your-repo/running-bull/issues
- 邮箱: support@example.com

XBuilder 社区：
- 官方论坛: https://forum.xbuild.ai
- Discord: https://discord.gg/xbuilder
```

---

**文档版本**: v1.0  
**最后更新**: 2026-10-26  
**维护者**: 开发团队  
**XBuilder 版本**: v3.3.0+  
**spx 语言版本**: v1.x
