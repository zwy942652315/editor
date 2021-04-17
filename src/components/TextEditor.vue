<template>
  <div id="text-container">
    <p>自定义富文本编辑器</p>
    <!-- 工具栏 -->
    <div id="control-area">
      <el-button @click="addLink" size="mini">插入链接</el-button>
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
    </div>
    <!-- 文本编辑器 -->
    <div id="text-area" contenteditable></div>
    <br />
    <!-- 插入链接 -->
    <el-dialog
      width="30%"
      title="插入链接"
      :visible.sync="visible"
      append-to-body>
      <el-form class="text-form" ref="formDef" :model="formData">
        <el-form-item label="链接" prop="url" required>
          <el-input placeholder="请输入链接" v-model="formData.url"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取 消</el-button>

        <el-button type="primary" @click="onSaveLink">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'TextEditor',
  data() {
    return {
      visible: false,
      editable: false,
      formData: {
        url: ''
      },
      content: '',
      selectElement: null,  // 选中的文字范围
      position: {
        start: 0,
        end: 0
      }
    }
  },
  mounted() {
    this.editorArea = document.getElementById('text-area')
    this.editorArea.innerHTML = this.content
  },
  methods: {
    //--------------------------------插入链接  start----------------------------------------
    addLink() {
      this.visible = true
      this.formData.url = ''
      const selection = window.getSelection()
      // 保存选中的文字范围
      this.selectElement = selection.focusNode
      this.position.anchorOffset = selection.anchorOffset
      this.position.focusOffset = selection.focusOffset
    },
    onSaveLink() {
      this.$refs.formDef.validate(valid => {
        if (valid) {
          const sText = window.getSelection()
          if (!sText) return
          // 下面判断主要是因为弹窗新增链接时，页面的光标已经移动到弹窗，当新增链接时，要将光标定位之前编辑器选中的内容
          // 直接点击编辑器的a标签进行编辑
          if (this.editable) {
            this.selectLink(this.currentLink)
          } else {
            // 新增a标签链接
            this.selectNewLink()
          }
          document.execCommand('insertHTML', false, `<a href="${this.formData.url}" target="_blank">${sText}</a>`);
          this.visible = false
          this.content = this.editorArea.innerHTML
          this.onListenerLink()
        }
      });
    },
    // 选中点击a标签
    selectLink(obj) {
      const selection = window.getSelection();
      selection.removeAllRanges();
      const range = document.createRange();
      range.selectNodeContents(obj); // 需要选中的dom节点
      selection.addRange(range) // 为选中的dom节点添加区域
    },
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
    //--------------------------------插入链接  end----------------------------------------
    // 复制
    handleCommand(command) {
      const sText = window.getSelection()
      document.execCommand(command, false, sText);
    }
  }
}
</script>

<style scoped lang="scss">
// 编辑器
#text-container {
  text-align: left;
  #control-area {
    width: 90%;
    margin-bottom: 10px;
    .link {
      height: 30px;
      line-height: 30px;
      display: inline-block;
      &:first-child {
        margin-right: 20px;
      }
      &:hover {
        color: #409eff;
      }
    }
  }
  #text-area{
    width: 90%;
    height: 300px;
    line-height: 1.5;
    border: 1px solid #DCDFE6;
    padding: 5px 15px;
    box-sizing: border-box;
    color: #606266;
    font-size: 14px;
    border-radius: 4px;
  }
}
</style>
