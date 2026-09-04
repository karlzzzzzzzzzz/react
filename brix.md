**Brix**（Bricks 的谐音，砖块）是一淘 UX 做的**应用层组件框架**：把页面拆成一块块可复用「砖」，再拼成完整页面。口号就是 *Build Your City with Brix*。

它管的是 **UI 组件**（日历、下拉、表格、业务区块），不是路由和整页生命周期。你们笔记里「东风（udradar）用 magix-brix 做 ADC 节点配置」，指的就是 **Magix 管页面 + Brix 管组件** 这一套。

---

## 和 Magix 怎么分工

2012 年一淘内部的讲法很直白：**Magix for OPOA，Brix for Components**。

| | Magix | Brix |
|---|---|---|
| 管什么 | 单页路由、View/VFrame、区块生命周期 | 组件渲染、模板、数据绑定、组件库 |
| 类比 | React Router + 页面骨架 | Ant Design / Element 那一层 |
| 钩子 | `mx-view` | `bx-name`、`bx-path`、`bx-config` |
| 底层 | 可接 KISSY / SeaJS + jQuery | PC 端主要基于 **KISSY**，移动端 Zepto + SeaJS |

Magix 源码里还提到过：弹层这类「有层级」的组件，Brix 2.0 会把内部 VFrame 挂到 `body` 上，避免嵌在页面树里互相干扰。说明后期 Brix 是挂在 Magix 区块模型上的。

后期阿里妈妈很多 Magix 3 项目改用 **magix-gallery**（你们 `magix-project` 里的 `app/gallery/mx-dropdown` 就是这类），不再叫 Brix，但角色一样：框架下面的组件层。

---

## 核心模型：Brick + Pagelet

Brix 不把「一个按钮」和「一整页」当成同一种东西，而是两层：

1. **Brick**：单个组件基类。下拉、日历、业务卡片都继承它。  
2. **Pagelet**：一页上的组件管理器。负责把模板渲出来、按钩子 extra 出 Brick、管生命周期。

典型用法（KISSY 时代）：

```js
KISSY.ready(function () {
  Brix.ready(function () {
    Brix.pagelet.ready(function () {
      var brick = Brix.pagelet.getBrick('#id');
    });
  });
});
```

或显式 `new Pagelet({ container, tmpl, data, autoRender: true })`，等 `ready` 后再 `getBrick`。

内部还有：

- **Chunk**：Brick 和 Pagelet 的父类  
- **Tmpler**：用 KISSY XTemplate 解析模板  
- **Dataset**：数据变了通知模板做局部刷新  

这和 Vue 的「组件 + 响应式数据」有点像，但是 **Pagelet 集中管一页组件**，不是 React 那种组件树自己递归。

---

## 页面上怎么声明组件

不用 `new Dropdown()` 满地写，而是在 HTML 上打钩子，Pagelet 扫描后自动构建：

| 钩子 | 作用 |
|---|---|
| `bx-name` | 组件名 |
| `bx-path` | 组件路径（核心组件可省略） |
| `bx-config` | 初始化配置 |
| `bx-tmpl` + `bx-datakey` | 指定模板和数据 key，数据变了只重渲这一块 |

组件来源约定成三类，用法相同：

- `brix/gallery/xxx`：官方基础控件（按钮、输入、日历等）  
- `components/xxx`：本项目业务组件  
- `imports/命名空间/xxx`：从别的业务库引进来的成熟组件  

另外还有 **Brix Style**（视觉规范/样式）和 **Brix Core**（框架本身）。

---

## 组件怎么共享：BPM

业务页面上「砖」可能有几百块，不能全塞进一个巨大组件库。Brix 搞了一套 **BPM（Brix Package Manager）**，思路接近 npm：

- 基础控件库：统一视觉、统一 API  
- 各业务子库：活在各自仓库里，成熟后再推进共享中心 **Brix-lib**  
- 组件可带 **Extension**，同一块砖换皮肤/换行为，Extension 也能分享  

目标是：区块能跨页面、跨业务复用，又不会把所有业务耦在一个仓库里。

---

## magix-brix 在实际项目里长什么样

常见组合是：

- Magix：路由、`mx-view` 拆页面、View 生命周期  
- Brix：View 内部的下拉、日期、表格、配置节点等控件  
- 有的方案里渲染还接 Pat 等模板做局部刷新（例如 Magix 2 + Brix 4 + Pat）

对应你们梳理里的 **东风 / udradar**：页面骨架是 Magix，ADC 节点配置那些可复用控件走 Brix（或 magix-brix 封装）。  
对照 **op / mmcrm-op**：Magix 2/3 + gallery，组件层换成 magix-gallery，不再强调 Brix 这个名字。

---

## 和 React / Vue 组件库对比

| | Brix | Ant Design / Element |
|---|---|---|
| 运行方式 | Pagelet 扫描 DOM 钩子，再 new Brick | 组件写在 JSX / template 树里 |
| 数据更新 | Dataset + 模板局部刷新 | 响应式 / setState |
| 包管理 | BPM + 业务子库 | npm |
| 设计体系 | Brix Style（一淘视觉） | Ant / Element 设计语言 |
| 现状 | 历史栈，维护老后台 | 现在的主流 |

可以记成：

- **Magix** ≈ 怎么拆页面、怎么走路由  
- **Brix** ≈ 拆完之后，每一块 UI 用哪套砖、怎么渲、怎么共享  

读东风这类 magix-brix 代码时，先找 `bx-name` / `bx-path`（或项目里对 Brix 的封装），再找 Magix 的 `mx-view`，两套钩子叠在一起就是整页。



东风仓库是 `ud_radar`。这里的 Brix **不是** 一淘那套 KISSY 老 Brix，而是 **`magix-ports` 里的配置化表单引擎**：用 ADC 节点树把表单渲出来。在东风里，它主要服务**自定义预警配置页**，不是整站所有监控大盘。

---

## 在东风里干什么

东风本身是 Magix 单页：路由、VFrame、监控大盘、诊断页，大部分还是手写 View。

Brix 补的是这一层：

**后台配一份 ADC 组件树 → 前端按树生成表单 → 用户改值写回 store → 提交时 `brixIns.getValue()` 拿出整份数据。**

对照你们笔记「magix-brix 开发，adc 节点配置页面及组件」，落在代码里就是：

| 角色 | 代码位置 | 作用 |
|---|---|---|
| 引擎 | CDN 上的 `magix-ports/brix` | 扫 ADC、渲子页、管数据和依赖 |
| 项目适配 | `viewBrix.ts` | `useBrix`、拉 ADC、监听、销毁 |
| 业务入口 | `views/pages/warning/handle.ts` | 预警新建/编辑表单 |
| 业务组件 | `views/pages/comps/*` | 下拉、帮助、通知列表等砖块 |
| 数据处理 | `views/pages/handlers/*` | 每个砖块的 `process` / `update` |
| 配置源 | `/api/component/findComponentList.json` | 用 `componentCode` 拉节点树 |

预警页模板几乎不写字段，只挂引擎子页：

```1:2:/Users/sk/kz/ud_radar/src/ud_radar/views/pages/warning/handle.html
<!-- 加载组件 -->
<mx-vframe src="magix-ports/brix/comps/subPage" ... *adc-config="{{@adcConfig}}" *brix-space-name="{{@brixSpaceName}}" />
```

字段长什么样，由 ADC 里的 `m_warning_config` 决定，不写死在 HTML 里。

---

## 一次表单怎么跑起来

以预警配置为例，链路是固定的：

1. **建实例**：`createBrixIns("m_warning_config")` → `useBrix(空间名, this)`，一块表单一个命名空间，避免和别的页抢数据。
2. **拉 ADC**：`initBrixNew("m_warning_config")` 把字符串当 `componentCode`，走 `getViewConfig` → `api_component_findComponentList_get`，拿到 `rootItem`（含 `subComponentList`）。
3. **绑数据和配置**：`observeAdcConfigAndData` 把 ADC 和 store 同步起来；改节点或改值会触发刷新。
4. **初始化引擎**：`brixIns.init(rootItem, { projectName, data, fn: brixTmplFn })`，再 `handleUpdate()`。
5. **提交**：`check()` 里 `brixIns.getValue()` → 校验 → `formatHandle` 把 `ruleConfigMap` 收成 `ruleConf.ruleConfigItems` → 调保存接口。
6. **销毁**：View `destroy` 时清 observe、拆实例。

`brixTmplFn.ts` 用来补 ADC 动态 bean（例如带日期的默认名），引擎渲模板时会调这些纯函数。

全局还有 `constant.ts` 里的 `brixConfig.ignoreKeys`（如 `campaignId`），用来跳过某些字段的依赖检查。

---

## 东风自己写的「砖」：comps + handlers

引擎只认 ADC 节点类型。东风为预警场景补了一批业务砖，**视图和数据处理拆开**：

**comps（怎么画）** 都继承 `viewBrix`，读 `adcConfig.properties`，改值走 `handleUpdate`：

- `dropdown_server`：服务端搜索下拉  
- `lg_dropdown_server` / `lg_list_select`：规则配置里的下拉、多选  
- `notifyConfList` / `targetConfList`：通知人、监控对象列表  
- `text` / `help` / `linkBtn`：文案、帮助气泡、跳转按钮  

**handlers（怎么算）** 导出 `process` / `update`：

- `process`：第一次把 ADC + 已有数据收成控件要的 props（选项、默认值、文案）  
- `update`：用户操作后写回 store；简单字段直接用 `magix-ports/brix` 的 `commonUpdate`  

例如下拉变更会按节点 `code` 写回：

```12:20:/Users/sk/kz/ud_radar/src/ud_radar/views/pages/comps/dropdown_server.ts
  'change<change>'(e) {
    let {adcConfig} = this.updater.get()
    this.handleUpdate({
        item: e.item,
        data: {
          [ adcConfig.code ]: e.selected,
        },
        actionType: "change"
    })
  },
```

ADC 改字段名、校验、选项来源，往往不用改页面骨架，只改配置或对应 handler。

---

## 和东风其它页面的关系

Brix **没有**铺到所有东风页面。

- **走 Brix 的**：`warning/handle`（自定义预警新建/编辑）。这是典型「ADC 配出来的复杂表单」。
- **同业务、手写的旧版**：`brand1bp_warning_config/handle.ts` 继承普通 `view`，字段、校验、接口全写在 View 里，几百行。新预警页用 Brix 就是为了不再这样堆。
- **监控大盘 / 诊断**：`monitor_*`、`diagnosis_*` 等仍是 Magix View + gallery。ADC 接口（如 `dashboard_online`）也会用来找菜单/节点，但那是 `getViewConfig` / `findADCCompOf`，**没有**走 `useBrix`。

所以：ADC 在东风里更宽（菜单、大盘节点也可能配），**Brix 引擎只吃「要动态生成表单」的那棵树**。

---

## 和 Magix、gallery 怎么分层

```
Magix          路由、VFrame、生命周期
magix-ports/brix   ADC → 表单、store、依赖、subPage
viewBrix       东风适配：拉配置、路由参数、销毁
zs-gallery     基础控件（mx-btn、mx-popover、mx-loading）
ADC 后端       节点 code / properties / 子节点
```

Brix 砖块内部仍然用 gallery 画 UI；Brix 管的是 **按配置拼哪些砖、数据怎么流**。`magix-ports` 在 `package.json` 的 `crossConfigs` 里以外链工程接入（`g.alicdn.com/mm/magix-ports/...`），引擎不在东风仓库里。

---

## 一句话

在东风里，Brix 是 **预警配置表单的配置化渲染层**：运营/开发在 ADC 配 `m_warning_config` 节点树，东风用 `viewBrix` 拉树、用 comps/handlers 实现业务控件，提交时从 Brix 实例取值。监控大盘仍是传统 Magix 页面；Brix 解决的是「表单字段多、常改、不适合每次改 HTML」这一块。