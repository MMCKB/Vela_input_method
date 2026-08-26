# Vela_input_method

适用于 Xiaomi Vela / 快应用的输入法组件，包含圆屏、方屏与胶囊屏布局。方屏支持中文全键、英文全键、手机式 T9、简繁候选、词组候选、数字/符号页，以及候选栏上方的回车与光标控制栏。

> **配置边界：** 键盘组件不提供设置页、不保存设置，也不会在返回、发送、删除或候选选择时读写配置。快应用的设置页负责保存用户偏好；承载输入面板的页面将偏好作为组件属性单向传入。

## 安装

```bash
git clone https://github.com/MMCKB/Vela_input_method.git
```

将仓库中的 `components` 目录复制到快应用项目的 `src` 目录，再按自定义组件方式导入 `InputMethod.ux`。

```html
<import name="input-method" src="../../components/InputMethod/InputMethod.ux"></import>
```

## 快应用设置接入

### 推荐数据流

```text
快应用设置页
  └─ 保存键盘偏好（唯一写入点）
       └─ 承载输入面板的页面读取偏好
            └─ 通过属性传入 input-method
                 └─ 输入法仅应用配置，不反向保存
```

推荐在快应用设置页保存以下七项，并在创建输入面板前将当前值绑定到 `input-method`：**默认输入布局、默认简体/繁体、按键振动长度、候选词数量、方屏主题、英文双击 Shift 锁定大写、英文自动首字母大写**。

| 设置页项目 | 输入法属性 | 可传值 | 组件默认值 | 方屏行为 |
|---|---|---|---|---|
| 默认输入布局 | `keyboardtype` | `QWERTY`、`T9` | `QWERTY` | `QWERTY` 为中文全键；`T9` 为中文手机式九键。 |
| 默认简体/繁体 | `traditional` | `true`、`false` | `false` | `false` 为简体候选；`true` 为繁体候选。只影响方屏中文输入。 |
| 按键振动长度 | `vibratemode` | `""`、`short`、`long` | `""` | 空字符串关闭振动；其余值传递给 Vela 系统振动接口。 |
| 候选词数量 | `maxlength` | 推荐 `3`、`5` | `5` | 控制默认候选栏每行展示数量；展开后仍可查看更多候选。 |
| 方屏主题 | `keyboardtheme` | `dark`、`white`、`blue-gray` | `dark` | `white` 为纯白主题；`blue-gray` 为蓝灰主题；其他值安全回退为深色主题。只影响方屏。 |
| 双击 Shift 锁定大写 | `doubletapshiftlock` | `true`、`false` | `false` | 仅方屏英文全键生效。两次快速点按上箭头锁定大写；锁定后单击解除。关闭时保持原有单击大小写切换。 |
| 英文自动首字母大写 | `autocapitalize` | `true`、`false` | `false` | 仅方屏英文全键生效。切换至英文或点按顶部回车后，下一次英文输入自动大写一次。 |

> 建议把 `keyboardtype`、`traditional`、`vibratemode`、`maxlength`、`keyboardtheme`、`doubletapshiftlock`、`autocapitalize` 作为快应用的持久化偏好。组件支持这些属性在运行时更新，但设置页建议采用“**保存后下次打开键盘生效**”，避免在正在输入时切换布局、候选数量、主题或大写辅助行为。

### 最小接入示例

以下示例中的七个变量由快应用设置页或应用级配置模块维护。输入法不关心这些值来自本地存储、网络同步还是固定默认值。

```html
<import name="input-method" src="../../components/InputMethod/InputMethod.ux"></import>

<template>
  <div class="page">
    <input-method
      hide="{{hide}}"
      screentype="rect"
      keyboardtype="{{keyboardtype}}"
      traditional="{{traditional}}"
      vibratemode="{{vibratemode}}"
      maxlength="{{maxlength}}"
      keyboardtheme="{{keyboardtheme}}"
      doubletapshiftlock="{{doubletapshiftlock}}"
      autocapitalize="{{autocapitalize}}"
      @visibility-change="onVisibilityChange"
      @key-down="onKeyDown"
      @delete="onDelete"
      @complete="onComplete"
      @cursor="onCursor"
    ></input-method>
  </div>
</template>

<script>
export default {
  private: {
    hide: true,

    // 这些值应由快应用设置页保存并在打开输入面板前恢复。
    keyboardtype: "QWERTY", // "QWERTY" 或 "T9"
    traditional: false,      // false 为简体，true 为繁体
    vibratemode: "",        // ""、"short" 或 "long"
    maxlength: 5,            // 推荐 3 或 5
    keyboardtheme: "blue-gray", // "dark"、"white" 或 "blue-gray"
    doubletapshiftlock: false, // false 关闭；true 开启英文双击 Shift 锁定大写
    autocapitalize: false, // false 关闭；true 开启英文首字母自动大写
  },

  // 设置页保存后，由上层页面更新这七个状态；
  // 不要要求 input-method 自己保存，也不要在返回/发送回调中保存。
  applyKeyboardPreferences(preferences) {
    this.keyboardtype = preferences.keyboardtype === "T9" ? "T9" : "QWERTY";
    this.traditional = preferences.traditional === true;
    this.vibratemode = preferences.vibratemode || "";
    this.maxlength = preferences.maxlength === 3 ? 3 : 5;
    this.keyboardtheme = preferences.keyboardtheme === "white" || preferences.keyboardtheme === "blue-gray"
      ? preferences.keyboardtheme
      : "dark";
    this.doubletapshiftlock = preferences.doubletapshiftlock === true;
    this.autocapitalize = preferences.autocapitalize === true;
  },

  onVisibilityChange(evt) {
    console.log("键盘显示状态：" + JSON.stringify(evt));
  },

  onKeyDown(evt) {
    console.log("按键：" + JSON.stringify(evt));
  },

  onDelete() {
    // 由宿主在自己的文本状态中删除光标前的字符。
  },

  onComplete(evt) {
    // 由宿主在自己的文本状态中插入 evt.detail.content。
  },

  onCursor(evt) {
    // action 为 enter、up、down、left、right。
    // 由宿主维护文本和光标位置。
    console.log("光标操作：" + JSON.stringify(evt));
  },
};
</script>
```

### 设置页保存边界

设置页可使用快应用自身已经验证的配置服务或存储模块保存七项偏好，但应在设置页或应用级配置层完成，不应放入输入法组件或输入面板生命周期。主题和英文大写辅助均仅通过属性单向传入，输入法不提供设置按钮，也不保存这些偏好。

| 允许的位置 | 不允许的位置 |
|---|---|
| 快应用设置页的“保存/应用”操作 | `InputMethod` 的按键、候选、删除、回车和光标事件。 |
| 应用初始化或打开输入面板前的配置读取 | `FullScreenInput` 的返回和发送函数。 |
| 独立的偏好规范化函数，例如把候选数限制为 3 或 5 | 键盘显示/隐藏、右滑返回、候选展开等高频路径。 |

> `ExitInput()` 应继续只负责退出事件，`InputFinish()` 应继续只负责发送文本。不要为设置联动在这些路径增加本地存储、定时器、设置回写或跨组件广播。

## 组件属性

| 名称 | 类型 | 默认值 | 必填 | 描述 |
|---|:---:|:---:|:---:|---|
| `hide` | boolean | `true` | 是 | 是否显示键盘。 |
| `keyboardtype` | string | `QWERTY` | 否 | 默认布局。`T9` 启用方屏中文九键；其他值按全键处理。胶囊屏仅支持全键。 |
| `traditional` | boolean | `false` | 否 | 方屏中文候选的默认字形。`false` 简体，`true` 繁体。 |
| `vibratemode` | string | `""` | 否 | 按键振动模式。`""` 为关闭，`short`、`long` 为系统振动模式。 |
| `maxlength` | number | `5` | 否 | 默认候选展示数量。推荐只由设置页传 `3` 或 `5`。 |
| `keyboardtheme` | string | `dark` | 否 | 方屏主题。`white` 为纯白，`blue-gray` 为蓝灰，`dark` 为深色；未知值回退为 `dark`。 |
| `doubletapshiftlock` | boolean | `false` | 否 | 仅方屏英文全键有效。开启后，两次快速点按 Shift 上箭头锁定大写；锁定后单击解除。默认关闭。 |
| `autocapitalize` | boolean | `false` | 否 | 仅方屏英文全键有效。开启后，切换至英文或点按顶部回车后，下一次英文输入自动大写一次。默认关闭。 |
| `screentype` | string | `circle` | 否 | `rect` 为方屏，`circle` 为圆屏，`pill-shaped` 为胶囊屏。 |

## 组件事件

| 名称 | 参数 | 描述 |
|---|---|---|
| `complete` | `{ detail: { content: string } }` | 确认中文候选、输入英文/数字/符号或空格时触发。 |
| `delete` | `-` | 删除键在没有待确认拼音时触发。宿主负责删除光标前的文本。 |
| `keyDown` | `{ detail: { content: string } }` | 普通键按下时触发。 |
| `visibilityChange` | `{ detail: { visible: boolean } }` | 键盘显示或隐藏时触发。 |
| `cursor` | `{ detail: { action: string } }` | 方屏顶部栏触发，`action` 为 `enter`、`up`、`down`、`left` 或 `right`；宿主负责文本光标操作。 |

## 方屏行为

方屏左侧控制按钮为“切换”。切换页只有**中、英、九、数**四项：中文全键、英文全键、中文 T9、数字/符号页。键盘内不包含设置入口，也不保存默认偏好。

方屏中文全键与 T9 模式显示“简/繁”快捷按钮，它只影响**当前已打开键盘**的候选字形。若需要让下一次打开键盘默认使用繁体，应由快应用设置页保存 `traditional=true`，再通过属性传入。

候选词典按需惰性加载。设置候选数量不会建立全量词组索引，也不会改变简繁词典或 T9 分段回退逻辑。`keyboardtheme` 仅改变方屏背景、候选栏、顶部五键栏、字母/T9 键位、候选展开面板和切换页的配色，不改变键盘布局或输入逻辑。`doubletapshiftlock` 与 `autocapitalize` 均默认关闭，只调整方屏英文全键的大小写状态，不改变键盘几何、候选逻辑或宿主返回/发送路径。

## 兼容与升级说明

旧版本 README 曾描述方屏“设置”菜单及 `settingsChange` 事件。该说明已过期：当前组件不提供该菜单，也不发出该事件。请改为由快应用设置页保存偏好，并使用本 README 中的七个属性传入组件。

当前用户指令优先于本 README。主题接口当前只支持 `dark`、`white`、`blue-gray`；若需要增加更多主题、顶部五键栏开关或实时预览等设置，请先在宿主侧设计新的只读属性，再单独进行方屏真机稳定性验证。
