# Vue-uploader

基于 Vue 3 + ElementPlus 的文件上传组件，支持单文件 & 多文件上传、多文件悬浮列表单独控制等特性。

## 🚀 特性
- ✨ 适用于出于某些原因不能直接使用 `el-upload` 组件的情况（如不想使用浏览器自带文件选择，需要获得绝对路径等）
- ✅ 轻松选择单文件 & 多文件上传
- ✅ 多文件悬浮列表，可单独删除文件
- ✅ 自定义上传校验（大小、类型、数量）
- ✅ 集成 ElementPlus 原生 UI 风格
- 💻 基于 JavaScript

## 📦 运行

1. 需要安装 [Node.js](https://nodejs.org/)

2. 以 Vite 和 npm 为例创建一个 Vue 3 项目：
```bash
npm create vite@latest
// 后依次输入项目名，选择 Vue 和 JavaScript

cd your-project-name
npm install
```

3. 安装依赖包 ElementPlus 及 Axios：
```bash
npm install element-plus
npm install axios
```

4. 根据仓库内容修改 `src/main.js` 及 `src/App.vue`，新增 `src/components/FileUploader.vue`

5. 运行
```bash
npm run dev
```

## 🪄 效果与功能

![运行效果示例](/public/demo.png)

组合组件的上传功能和自定义功能实现详见 `src/components/FileUploader.vue`。

组合组件的调用详见 `src/App.vue`。

详细功能使用：

- 点击文本框右侧的灰色上传形状按钮进行文件选择；点击文本框之后的按钮完成文件上传。
- 单个文件上传时，可以通过清除按钮取消选择（`clearable` 为 `true` 时，下同）或再次选择文件进行覆盖。
- 多个文件上传时，可以通过清除按钮删除所有选择文件，或通过悬浮框删除单个文件，不能通过再次选择进行覆盖。
- 当文件的数量、大小、类型未通过设定的校验时，浏览器会弹出 ElMessage 的 Error 提示。
- 上传成功后，浏览器会弹出 ElMessage 的 Success 提示，否则弹出 Error。

## 🔧 API

### Props
| 名称           | 描述                | 类型      | 默认值   |
|----------------|---------------------|-----------|----------|
| **action**  *(required)* | 请求URL     | `string`  | #        |
| **max-files**  | 允许上传文件的最大数量（设置为 1 时即为单文件上传） | `number`  | 3        |
| **max-size**  | 允许上传文件的最大大小 (MB) | `number`  | 5        |
| **allowed-types**  | 允许上传的文件类型 | `array`  | ['image/png', 'application/pdf'] |
| **headers**    | 设置上传的请求头部  | `object`  | {'Content-Type': 'multipart/form-data'} |
| **method**     | 设置上传请求方法    | `string`  | post     |
| **input-width** | 文本框宽度    | `string`  | 350px     |

### 原生 Attributes 和 Events
所有 `el-input` 的原生 Attributes 和 Events 均透传到了组合组件中的 `el-input` 元素上，可在父组件中直接使用。

以下为默认设定：
| 名称        | 描述               | 类型      | 默认值   |
|-------------|--------------------|-----------|----------|
| **placeholder** | 输入框占位文本     | `string`  | 未选择文件 |
| **clearable**   | 是否显示清除按钮   | `boolean` | true     |

❗特别的是，`class` 和 `style` 属性将被透传到组合组件最外层即根节点上，以便于控制封装后的组件的整体样式。

### Slots 和 Exposes
由于组件功能为文件上传，所以为了限制不必要的扩展性，没有向父组件暴露 `el-input` 原生的 Slots 和 Exposes。

上传文件功能的 `el-button` 按钮设置为了新定义的组件插槽，可对其样式和文字进行自定义。