![预览图](prew.png)

# Vela_input_method
 三⽅输⼊法组件

## 手动安装

### 下载项目代码

```bash
git clone https://github.com/NEORUAA/Vela_input_method.git
```

### 拷贝代码
把项目中的 components 文件夹拷贝到要使用组件的项目 src 目录下。然后就可以按照自定义组件的使用方式来使用本组件了。

---

## 组件名称
input-method

## 概述
输⼊法功能

## ⼦组件
不⽀持

## 属性
| 名称 | 类型 | 默认值 | 必填 | 描述 |
| --------  | :----:  | :----:  | :----:  | :---- |
| hide | boolean | true | 是 | 是否显⽰键盘（开发者可以通过切换属性值隐藏或者唤醒键盘） |
| keyboardtype | string | "QWERTY" | 否 | 键盘布局，"QWERTY" 表⽰全键，"T9" 表⽰九键。默认为 "QWERTY"（当 screentype 为 "pill-shaped" 时，仅全键盘可用） |
| maxlength | number | 5 | 否 | 默认展⽰的拼⾳候选词数量， maxlength > 0 时有效；点击展开查看所有候选词 |
| vibratemode | string | "" | 否 | 振动模式，""表⽰输⼊时不振动，"long" 表⽰⻓振动，"short" 表⽰短振动。默认为 "" |
| screentype | string | "circle" | 否 | 设备屏幕类型，"rect" 表示方形屏布局（对应 designWidth ≥ 336），"circle" 表示圆形屏布局（对应 designWidth 为 480），"pill-shaped" 表示胶囊形屏布局（对应 designWidth ≥ 192）。方屏模式内置“切换”入口。 |
| traditional | boolean | false | 否 | 是否在方屏中文输入时使用繁体候选。默认 `false`，即简体；可通过方屏设置页即时修改。 |

## 事件
| 名称 | 参数 | 描述 |
| --------  | :-----  | :---- |
| complete | { detail: { content: string } } | 键盘输出字符时触发（当切换为中⽂输⼊法时候，当选中拼⾳对应⽂字时触发；当切换为英⽂输⼊法时，与 keyDown 触发条件⼀致）|
| delete | - | 键盘点击删除按钮触发 |
| keyDown | { detail: { content: string } } | 键盘按钮按下时触发 |
| visibilityChange | { detail: { visible: boolean } } | 键盘显示或隐藏时触发，visible 表⽰显示状态 |
| settingsChange | { detail: { keyboardtype: string, lang: string, vibratemode: string, maxlength: number, traditional: boolean } } | 方屏“切换”菜单中选择中/英/九或修改设置时触发；用于由宿主页面保存最新偏好。 |

## 方屏切换与设置

当 `screentype="rect"` 时，左侧控制按钮为“切换”。点击后会在键盘上方打开由圆形按钮组成的横向滚动菜单：**中、英、九、设置**。前三项分别切换中文全键、英文全键和 T9 九键；设置页提供按键振动（关/短/长）、候选词数量（3/5）和**繁体输入**（关/开）三项即时设置。繁体输入默认关闭；开启后，中文全键和 T9 的拼音候选均从完整繁体词典读取。横向滚动仅绑定在菜单中部容器，左右各保留 12px 边缘区域，以降低与手表系统右滑返回手势的冲突。

`settingsChange` 仅通知宿主页面保存最新偏好，方屏组件本身会立即应用所选值。若需要跨页面持久化，请在宿主页面接收事件后更新对应数据，再在下一次创建组件时通过 `keyboardtype`、`vibratemode`、`maxlength`、`traditional` 传入。

## ⽰例代码
```html
<import name="input-method" src="../../components/InputMethod/InputMethod.ux"></import>
<template>
  <div class="page" style="flex-direction: column;">
    <text class="text" @click="changeState">
      {{textValue}}_
    </text>
    <input-method
      hide="{{hide}}"
      keyboardtype="{{keyboardtype}}"
      maxlength="5"
      vibratemode="{{vibratemode}}"
      screentype="{{screentype}}"
      traditional="{{traditional}}"
      @visibility-change="onVisibilityChange"
      @key-down="onKeyDown"
      @delete="onDelete"
      @complete="onComplete"
      @settings-change="onSettingsChange"
    ></input-method>
  </div>
</template>

<script>
export default {
  private: {
    textValue: "",
    hide: false,
    keyboardtype: "QWERTY", //QWERTY, T9
    vibratemode: "short",
    screentype: "circle", //pill-shaped, rect, circle
    traditional: false, // 方屏繁体输入默认关闭
  },
  onVisibilityChange(evt) {
    console.log("显示状态变更:"+JSON.stringify(evt));
  },
  onKeyDown(evt) {
    // this.textValue += evt.detail.content;
    console.log("按下按键:"+JSON.stringify(evt));
  },
  onDelete() {
    this.textValue = this.textValue.slice(0, -1);
    console.log("删除字符");
  },
  changeState() {
    this.hide = !this.hide;
  },
  onComplete(evt) {
    this.textValue += evt.detail.content;
    console.log("返回字符:"+JSON.stringify(evt));
  },
  onSettingsChange(evt) {
    // 可将该值持久化，并在下次进入页面时回传给组件。
    this.keyboardtype = evt.detail.keyboardtype;
    this.vibratemode = evt.detail.vibratemode;
    this.traditional = evt.detail.traditional;
    console.log("方屏输入法设置:"+JSON.stringify(evt));
  },
};

</script>

<style>
.page{
  width:480px;
  height:480px;
}

.text{
  position:absolute;
  left:0;
  top:60px;
  width:100%;
  height:80px;
  text-align:center;
  color:white;
  background-color: red;
}
</style>
```
