# 自定义富文本编辑器

## 前言

在开发中，需实现一个可以弹出弹窗，在弹窗中添加或编辑链接的富文本编辑器，在一些vue-quill-editor等无法实现，且不考虑兼容性的情况下，仅满足项目需求，自定义了一个富文本编辑器，注意是实现插入自定义链接操作

## 环境准备

为了方便开发，基于vue框架开发的环境

安装依赖

```
npm install
```

运行

```
npm run serve
```

编译

```
npm run build
```

## 实现功能

主要是实现**新增**和**编辑**两个功能

新增：选中添加链接的文字	=>	点击插入链接按钮，弹出弹窗，添加链接，点击保存	=> 选中字体变蓝，添加链接成功

编辑：点击有链接的蓝色字段 =>	弹出弹窗，编辑链接，点击保存	=>	所点击的蓝色字段链接编辑成功

![image-20210417183039007](/Users/zhuwenyan/study/editor/img/image-20210417183039007.png)

![link](/Users/zhuwenyan/study/editor/img/link.png)

## 实现过程

### 富文本编辑功能

[contenteditable](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/contenteditable) , 是一个可枚举属性，表示元素是否可被用户编辑。 可以通过添加该属性，实现编辑功能。

我们需要让一个 `div` 成为可编辑状态，加入`contenteditable="true"` 属性即可。

```html
<div contenteditable="true" id="rich-editor"></div>
```

当	contenteditable

- `true` 或空字符串，表示元素是可编辑的；
- `false` 表示元素不是可编辑的。

### 添加链接

[document.execCommand](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/execCommand) , document暴露 execCommand 方法，该方法允许运行命令来操纵可编辑内容区域的元素。

因此，我们可以运用 `insertHTML` 的命令来实现插入a标签链接

```javascript
document.execCommand('insertHTML', false, `<a href="${this.formData.url}" target="_blank">${sText}</a>`);
```

### 光标操作

插入链接是一个很容易的操作，但是要获取是实现插入可自定义链接的难点。

浏览器提供了 [selection](https://developer.mozilla.org/zh-CN/docs/Web/API/Selection) 对象和 [range](https://developer.mozilla.org/zh-CN/docs/Web/API/Range) 对象来操作光标。

#### selection 对象

表示用户选择的文本范围或插入符号的当前位置。它代表页面中的**文本选区**，可能横跨多个元素。文本选区由用户拖拽鼠标经过文字而产生。

获取一个 selection 对象：

```javascript
const selection = window.getSelection()
```

该对象有几个重要属性：

```
focusNode：返回所选内容的结束位置部分所属的节点

anchorOffset：返回选区的锚节点（ Selection.anchorNode）起点偏移量的数字

focusOffset：返回选区终点（鼠标松开瞬间所记录的那个点）在焦点（Selection.focusNode）中的偏移量
```

该对象有几个重要方法：

```
addRange()：向选区（Selection）中添加一个区域（Range）

removeAllRanges():	会从当前selection对象中移除所有的range对象,取消所有的选择
```

**新增链接操作**注意：

在自定义的富文本编辑器中，新增链接操作时，选中文字后，弹出弹窗之前，需要保存用户选择的文本范围。因为弹出弹窗后，用户在弹窗层面操作，浏览器当前的选择的文本范围已经变成了弹窗部分，不再是要添加链接的选中的文本部分。

所以，弹出弹窗前，需保存用户选择的文本范围或插入符号的当前位置：

```
// 点击插入链接
addLink() {
  this.visible = true
  this.formData.url = ''
  const selection = window.getSelection()
  // 保存选中的文字范围
  this.selectElement = selection.focusNode
  this.position.anchorOffset = selection.anchorOffset
  this.position.focusOffset = selection.focusOffset
},
```

而在点击弹窗保存链接时，需重新获取之前用户选中文字部分。

```javascript
// 对选中对a标签添加链接
selectNewLink() {
  const selection = window.getSelection()
  const range = document.createRange()
  // 获取之前选中的文字范围
  const rangeDivDom = this.selectElement;
  const { anchorOffset, focusOffset } = this.position
  if (anchorOffset < focusOffset) {
    range.setStart(rangeDivDom, anchorOffset)
    range.setEnd(rangeDivDom, focusOffset)
  } else {
    range.setStart(rangeDivDom, focusOffset)
    range.setEnd(rangeDivDom, anchorOffset)
  }
  // 移除当前所有选区
  selection.removeAllRanges()
  selection.addRange(range)
},
```

**编辑链接操作**注意：

在自定义的富文本编辑器中，编辑链接操作时，点击该文字链接，弹出弹窗前，需要获取点击链接的dom节点，这样在弹窗保存时，能够重新获取该文字链接区域

```javascript
// 监听链接点击事件
onListenerLink() {
  const self = this
  self.editable = false
  const ahref = this.editorArea.getElementsByTagName("a");
  ahref.forEach((href) => {
    href.addEventListener("click", function () {
      // 记录点击a标签
      self.currentLink = this
      self.editable = true
      self.selectLink(this)

      self.visible = true
      self.formData.url = this.getAttribute('href')
      }, false);
    })
},
```

```javascript
// 选中点击a标签
selectLink(obj) {
  const selection = window.getSelection();
  selection.removeAllRanges();
  const range = document.createRange();
  range.selectNodeContents(obj); // 需要选中的dom节点
  selection.addRange(range)	// 为选中的dom节点添加区域
},
```

#### range 对象

表示一个包含节点与文本节点的一部分的文档片段，可以操作光标位置

该对象有几个重要属性：

```
selectNodeContents：用于设置 Range，使其选中dom节点

setStart：用于设置 Range的开始位置

setEnd：用于设置 Range的结束位置
```

`selectNodeContents` 可以对一个dom节点进行选中，然后可以利用selection.addRange添加文档片段

调用 `setStart()` 和 `setEnd()` 方法，来修改一个光标的位置或拖蓝范围

**新增链接操作**注意：

新增链接时，在选中文字时，有两种选择情况：

​	一、鼠标从前往后选中，选区范围为 [selection.anchorOffset, selection.focusOffset]

​	二、鼠标从后往前选中，选区范围为 [selection.focusOffset, selection.anchorOffset]

因此在给选中的文字添加链接时，需要判断这两种情况，比较anchorOffset和focusOffset的大小，否则插入链接会不生效

```
// 对选中对a标签添加链接
selectNewLink() {
  const selection = window.getSelection()
  const range = document.createRange()
  // 获取之前选中的文字范围
  const rangeDivDom = this.selectElement;
  const { anchorOffset, focusOffset } = this.position
  if (anchorOffset < focusOffset) {
    range.setStart(rangeDivDom, anchorOffset)
    range.setEnd(rangeDivDom, focusOffset)
  } else {
    range.setStart(rangeDivDom, focusOffset)
    range.setEnd(rangeDivDom, anchorOffset)
  }
  // 移除当前所有选区
  selection.removeAllRanges()
  selection.addRange(range)
},
```

### 其他功能

document.execCommand，这个API提供了很多命令，可以简单地实现一些常见的操作，如复制、剪切等功能

```vue
<el-button @click="handleCommand('copy')" size="mini">复制</el-button>
<el-button @click="handleCommand('cut')" size="mini">剪切</el-button>
<el-button @click="handleCommand('selectAll')" size="mini">全选</el-button>
<el-button @click="handleCommand('strikeThrough')" size="mini">删除线</el-button>
<el-button @click="handleCommand('justifyLeft')" size="mini">左</el-button>
<el-button @click="handleCommand('justifyCenter')" size="mini">居中</el-button>
<el-button @click="handleCommand('justifyRight')" size="mini">右</el-button>
<el-button @click="handleCommand('italic')" size="mini">斜体</el-button>
<el-button @click="handleCommand('underline')" size="mini">下划线</el-button>
<el-button @click="handleCommand('undo')" size="mini">撤销</el-button>
<el-button @click="handleCommand('redo')" size="mini">重做</el-button>
```

```javascript
// 复制
handleCommand(command) {
  const sText = window.getSelection()
  document.execCommand(command, false, sText);
}
```



## 最后

要实现一个自定义富文本编辑器并不复杂，在上述实现过程中，主要是要注意如何重新获取选区、操作光标。
