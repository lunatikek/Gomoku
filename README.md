# 五子棋 · Gomoku

![License](https://img.shields.io/badge/license-MIT-blue)
![Language](https://img.shields.io/badge/language-JavaScript-f7df1e)
![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)
![Platform](https://img.shields.io/badge/platform-Web%20%7C%20Mobile-lightgrey)

一款基于原生 HTML / CSS / JavaScript 的人机五子棋游戏，搭载 **Minimax + Alpha-Beta 剪枝** AI，支持桌面与移动端，零依赖，开箱即用。

---

## 目录

- [快速开始](#快速开始)
- [功能特性](#功能特性)
- [AI 算法](#ai-算法)
- [代码架构](#代码架构)
- [Bug 修复记录](#bug-修复记录)
- [未来方向](#未来方向)
- [技术栈](#技术栈)

---

---

## 快速开始

无需任何依赖或构建工具。

```bash
# 方法一：直接双击
open index.html

# 方法二：本地服务器（推荐）
npx serve .
# 或
python3 -m http.server 8080
```

访问 `http://localhost:8080` 即可开始游戏。

---

## 功能特性

### 游戏核心

- 15 × 15 标准棋盘，含天元与四星位标记
- 玩家执黑，AI 执白，黑棋先行
- 五子连珠判胜，支持横、竖、正斜、反斜四方向
- 棋盘落满自动判平局
- 获胜五子以蓝色连线高亮标出

### 操作

| 功能 | 说明 |
|------|------|
| 落子预览 | 鼠标悬停时显示半透明棋子，提示落点 |
| 悔棋 | 撤回上一手，支持多步（历史栈） |
| 取消悔棋 | 恢复刚刚悔掉的一手 |
| 重新开始 | 清空棋盘，重置所有状态 |

### 界面与体验

- **Apple 设计语言**：系统字体、毛玻璃卡片、圆角、柔和阴影
- **渐变背景**：蓝紫 → 暖橙三段渐变 + 三枚动态光晕气泡
- **高清渲染**：Canvas 乘以 `devicePixelRatio`（最高 3×），Retina 无模糊
- **响应式**：`ResizeObserver` 动态缩放，280 px ～ 520 px 自适应
- **移动端**：`touchstart` 支持，禁用默认滚动，手机可直接点击落子
- **AI 提示**：320 ms 延迟 + 状态文字「AI 思考中…」

### 名言模块

- 收录 **90 条**名言诗句（棋道谚语、先秦诸子、唐诗、宋词、近现代）
- 每 25 秒自动随机切换，不重复上一条
- 「换一句」手动切换，带 CSS keyframe 旋转动效
- 双帧 `requestAnimationFrame` 保证淡入过渡可靠触发

---

## AI 算法

### 整体流程

```
玩家落子
  → 胜负检测
  → 生成候选点（距已有棋子 ≤ 2 格的空位）
  → 启发式排序，取前 12 个
  → Minimax 深度 4 搜索
  → 取评分最高的点落子
```

### Minimax + Alpha-Beta 剪枝

```
minimax(depth, isMax, α, β)

  终止条件：
    depth == 0          → 返回静态评估值
    评估值 ≥  90000     → AI 必胜，立即返回
    评估值 ≤ -90000     → 玩家必胜，立即返回

  isMax（AI 走）：
    遍历候选点，取最大值
    best > β → β 剪枝，跳出

  isMin（玩家走）：
    遍历候选点，取最小值
    best < α → α 剪枝，跳出
```

Alpha-Beta 剪枝在最优排序下将搜索节点数从 `O(b^d)` 压缩至 `O(b^(d/2))`。

### 静态评估函数

扫描所有行、列、正斜、反斜线，按**连子数 × 开放端数**评分：

| 连子数 | 双端开放（活） | 单端开放（眠） |
|:------:|:--------------:|:--------------:|
| 5 | 100,000（必胜）| 100,000 |
| 4 | 50,000（活四）| 10,000（冲四）|
| 3 | 5,000（活三）| 1,000（眠三）|
| 2 | 500（活二）| 100（眠二）|
| 1 | 50 | 10 |

AI 棋子得正分，玩家棋子得负分，返回两者之差。

---

## 代码架构

### 项目文件

```
gomoku/
├── index.html    页面结构与按钮
├── Gomoku.css    样式（变量、布局、动画、响应式）
├── Gomoku.js     Gomoku 类 + 名言模块
└── README.md     本文档
```

### `Gomoku` 类方法速查

```
构造 & 初始化
  constructor(canvas)       初始化状态，启动 ResizeObserver
  init()                    重置游戏

响应式
  _resize()                 计算 RATE/PADDING，设置 HiDPI canvas
  _onResize()               ResizeObserver 回调

绘制
  _redraw()                 全量重绘
  _drawBoard()              棋盘底色、网格（1 物理像素）、星位
  _drawPiece(i,j,c,ghost)   径向渐变棋子 + 投影
  _drawWinLine(idx)         获胜连线

游戏
  playerRound(x,y)          处理落子 → 触发 AI
  _aiRound(pi,pj)           AI 落子，压入历史栈
  _place(i,j,who)           落子并更新赢法计数
  _checkWin(i,j,mw)         五连珠检测
  _isBoardFull()            平局检测
  _over(msg,cls,idx)        游戏结束

AI
  _aiChoose()               Minimax 主入口
  _minimax(d,isMax,α,β)     Alpha-Beta 递归
  _evalBoard()              静态评估
  _patternScore(len,opens)  连子评分表
  _getCandidates()          邻域候选点
  _rankCandidates(cs)       启发式预排序

悔棋
  back()                    弹出历史栈
  cancel()                  重新压入历史栈
```

### 关键数据结构

```js
chessBoard[i][j]   // 0=空  1=玩家  2=AI
wins[i][j]         // 该格参与的赢法下标列表（稀疏）
winPatterns[k]     // 第 k 种赢法的五个坐标 [{i,j}×5]
playerWin[k]       // 玩家在该赢法上占的格数（6=被封死）
AIWin[k]           // AI 在该赢法上占的格数
history[]          // 悔棋栈 {pi,pj,ai,aj,pSnap,aSnap}
```

---

## Bug 修复记录

| # | 问题 | 修复 |
|---|------|------|
| 1 | `toFixed()` 返回字符串，下标类型错误 | 改用 `Math.round()` |
| 2 | `canBack=true` 在无效点击时也触发 | 移至棋子落下后赋值 |
| 3 | 坐标计算不考虑页面滚动 | 改用 `getBoundingClientRect()` |
| 4 | `init()` 未重置悔棋状态 | 补充 `canBack/canCancel` 重置 |
| 5 | `cancel()` 中 `player` 翻转时机错误，双棋同色 | 重构恢复逻辑 |
| 6 | `winCollection` 硬编码 `15` | 改为 `this.LINES` |
| 7 | 高分屏 Canvas 模糊 | 物理像素 × `devicePixelRatio` |
| 8 | 网格线亚像素模糊 | `lineWidth=1/DPR` + `0.5/DPR` 对齐偏移 |
| 9 | 无平局检测 | 落子后检测 `chessBoard` 是否全非零 |
| 10 | 悔棋只能悔一步 | 改为历史栈 |
| 11 | 名言不显示（opacity 时序） | 双帧 `requestAnimationFrame` |
| 12 | 换一句旋转只有首次生效 | CSS `@keyframes` + class 移除/添加 |

---

## 未来方向

- **难度分级** — 简单 / 普通 / 困难（贪心 / 深度 2 / 深度 4）
- **双人对战** — 本地双人轮流模式
- **计时模式** — 每步限时，超时自动落子
- **战绩统计** — `localStorage` 持久化胜负与胜率
- **对局回放** — 游戏结束后逐步复盘
- **最后落子标记** — 高亮上一步位置
- **落子音效** — 木质落子声
- **棋盘主题** — 深色石板 / 白色宣纸 / 竹简

---

## 技术栈

| 项目 | 说明 |
|------|------|
| 语言 | 原生 HTML5 / CSS3 / ES6+ |
| 渲染 | Canvas 2D API（HiDPI 适配）|
| 布局 | Flexbox + CSS 自定义属性 |
| 字体 | `-apple-system` / SF Pro / PingFang SC |
| 视觉 | `backdrop-filter` 毛玻璃、径向渐变、CSS keyframes |
| 响应式 | `ResizeObserver` + `getBoundingClientRect` |
| 依赖 | **零依赖** |

---

## License

[MIT](LICENSE)
