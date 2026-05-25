# 🛠️ 技术文档 | DarkPattern Shield (套路终结者)

根据本项目作为 **“HCI 课程高保真交互演示原型（基于单文件 HTML/CSS/JS 仿真）”** 的技术定位，本技术文档专注于 **前端状态模型、模拟插件-页面通信、演示逻辑架构以及轻量化交互决策**。

---

## 1. 状态模型 (State Model)

由于本项目采用单文件高保真仿真架构（无物理数据库），数据模型主要表现为**前端运行时状态管理器（Frontend Runtime State Manager）**。该状态决定了虚拟手机屏幕的呈现内容、悬浮盾的唤醒时机以及页面的重绘状态。

### 1.1 全局仿真状态模型 (Global Simulation State)

```javascript
const simulationState = {
  currentStage: "AD_STAGE", // 当前所处的演示场景: "AD_STAGE" | "VIP_STAGE" | "UNSUB_STAGE"
  shieldState: "HIDDEN",     // 悬浮盾状态: "HIDDEN" | "ALERTING" | "PURIFIED"
  scanLogActive: false,      // AI 扫码一秒打字机动效是否正在执行
  isVipPurchased: false,     // 用户是否已选择自动续费VIP
  isAutoRenewCancelled: false // 自动续费是否成功取消
};
```

### 1.2 模拟页面套路元素属性表 (Mock Threat DOM Attributes)

在仿真靶场中，被标识的黑暗模式元素挂载了自定义的 `data-` 属性，用于触发前端 JS 的样式重绘：

*   **开屏广告假关闭按钮：**
    `div#decoy-close-btn` -> 绑定 `data-pattern="decoy-button"`，净化后不透明度降至 `20%`，样式增加置灰与 `ⓘ 诱饵` 标签。
*   **高压倒计时文本：**
    `span#fake-timer` -> 绑定 `data-pattern="fake-timer"`，净化后不透明度降至 `50%`，增加置灰，并在右上角渲染产品徽章。
*   **隐藏式退订按钮：**
    `button#real-cancel-btn` -> 绑定 `data-pattern="hidden-escape"`，引导后增加 `halo-glow-blue`（蓝色呼吸光晕）动画并提高对比度。

---

## 2. 模拟接口设计 (Simulated API & Event Design)

虽然系统运行在本地单文件内，但为了模拟**浏览器插件（Extension Popup）与网页内容（Content Script）之间的通信契约**，系统设计了内部事件派发机制（Custom Event Broker）来替代真实的 API 请求：

### 2.1 模拟插件内部通信事件 (Extension Message Passing)

#### ① 发送：请求分析当前 DOM (Request DOM Analysis)
*   **事件名称：** `REQUEST_DOM_ANALYSIS`
*   **触发时机：** 用户点击悬浮盾展开抽屉时，本地 JS 触发模拟分析。
*   **请求荷载 (Request Payload)：**
    ```json
    {
      "stage": "VIP_STAGE",
      "action": "trigger_scan"
    }
    ```

#### ② 接收：返回重绘指令集 (Receive Purify Directives)
*   **事件名称：** `RECEIVE_PURIFY_DIRECTIVES`
*   **触发时机：** 1.0秒扫描动效（打字机）结束后。
*   **响应荷载 (Response Payload)：**
    ```json
    {
      "status": "success",
      "directives": [
        { "selector": "#fake-timer", "action": "desaturate", "opacity": 0.5 },
        { "selector": "#auto-renew-checkbox", "action": "highlight", "glowColor": "#1e90ff" }
      ]
    }
    ```

---

## 3. 架构与演示流转图 (Architecture & Flow)

### 3.1 系统原型概念架构 (Conceptual Architecture)

```
       ┌─────────────────────────────────────────────────────────┐
       │                 主演示窗口 (Browser View)                 │
       │                                                         │
       │  ┌───────────────────┐           ┌───────────────────┐  │
       │  │  仿真 iPhone 容器   │           │ 悬浮盾控制抽屉    │  │
       │  │                   │           │ (Bottom Sheet)    │  │
       │  │ [1. 广告/支付/退订]│◄─────────►│                   │  │
       │  │                   │  样式重绘 │ 1. 扫描日志动效   │  │
       │  │  (靶场DOM节点群)  │           │ 2. [一键净化]按钮 │  │
       │  └───────────────────┘           └───────────────────┘  │
       └─────────────────────────────────────────────────────────┘
```

### 3.2 1分钟沉浸式演示状态迁移流 (1-Min Demo State Transitions)

```
   [开屏广告] ──(悬浮球亮起)──► [点击悬浮球] ──► [抽屉滑出(1s打字机)] ──(一键净化)──► [假按钮变灰/安全跳过]
                                                                                     │
                                                                                 (进入首页)
                                                                                     ▼
   [订阅退订] ──(点击退订)──► [挽留弹窗阻障] ──(悬浮球亮起) ──► [开始引导] ──► [挽留模糊/退订按钮光晕锐化]
```

---

## 4. 技术决策记录 (Technical Decision Records - TDR)

### 4.1 技术栈选择：单文件原生 HTML/CSS/JS (Claude Code 生成)
*   **决策理由：** 
    1.  **演示安全性：** 在 8 分钟的高压答辩中，任何由于网络延迟、云端服务器宕机或跨域安全政策限制（CORS）引起的加载失败都会导致演示中断。本地单文件可确保 100% 运行安全。
    2.  **极速迭代：** 采用单文件架构，大语言模型（Claude Code）可以在秒级完成整套交互逻辑的修改，极利于原型快速验证。
*   **避开的方案：** 物理打包的 Chrome 扩展程序（由于本阶段属于概念验证，开发真实插件会分散精力到环境配置上，偏离了 HCI 课程注重“界面设计与用户体验”的本源）。

### 4.2 容器设计决策：使用“拟物 iPhone 容器”承载
*   **决策理由：** 
    本产品定位为移动端交互。直接将手机比例的页面平铺在电脑大屏上会导致视觉失真与排版空旷。手绘一个带圆角和前置镜头的 iPhone 模拟容器，能让评委在电脑端直观感受到真实的“移动端手势操作与交互语境”。

### 4.3 视觉重绘决策：使用“淡蓝色呼吸光晕”与“低饱和度置灰”
*   **决策理由：**
    根据 HCI 的**“情感化设计（Emotional Design）”**原则，过度的红色警示和霓虹配色会大幅增加用户的上网压力，与产品“消除用户焦虑”的核心宗旨冲突。采用优雅的淡蓝色（#1e90ff）呼吸光晕与不透明度减半（30%~50%）的柔和置灰，能在不污染用户视觉的同时，提供温和、有效的“支架式引导”。